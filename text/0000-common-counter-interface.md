- Feature Name: Common Interface for reading Clocks and other counters
- Start Date: 2024-05-22
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

The embedded-hal traits `InputPin`, `OutputPin`, `I2c`, etc. key elements on the journey towards rust becoming
a first-class-citizen in the embedded devolopment space. Of notable absence is such a trait for reading an incremental counter,
and coupling that read to a unit being counted. This RFC calls for the development of such a trait. 

**Example 1: Clock**
The trait would be implemented such that it reads from a clocks tick-count register, and assosciated type(s)/const(s) would allow
this to be expressed as a measure of time. This implementation can be used wherever one needs a clock to get a measure of time, and
`embedded_hal::counters::Clock` trait is in scope.

**Example 2: Usage Tracking**

A embedded dev wishes to track total movement of a machine. The couple an AB encoder to a ++ only pulse-count peripheral. 
They implement counter such that a `read` will read from the pulse-count peripheral. They define their own trait extension internally
named `LifetimeMoveCount: embedded_hal::count::Counter`, and constrain it such that Counter must specify a distance for each count.
Their `LifetimeMoveCount` defers to `Counter` to get a read of movement, but its own impls defines long-term caching of data (e.g. 
non volatile storage, or send reports to a server, etc). 

** Core Principal**

> the goal of embedded-hal is to make it possible to write drivers that are portable across HALs.



# Motivation
[motivation]: #motivation

`esp-hal` has one implementation of representing time. `embassy-time` is another. If you look through any platform specific hal, 
you are sure to find two things:

- Comprehensive implementation of the embedded-hal common interface.
- Their own implementation of getting time.
- Where applicable, their own interface for reading and interpreting counters, such as pulse counters.

One can put together a library that is platform agnostic by means of using common interfaces. Much like the way
`where LedP: SetDutyCycle` clause in a method makes it usable across the board, such as with this e.g....:

```rust
fn set_brightness<LedP: SetDutyCycle>(pin: LedP, brightness_pcnt: u8) -> Encoder {
    pin.set_duty_cycle_percent(brightness_pcnt)
}
```

...embedded developers ought to have access to a common counter interface

```rust
// rotate brightness between 50 and 100% every 15 seconds using a clock-specific extension of a counting interface.
fn time_rotated_brightness<LedP: SetDutyCycle, ClkSrc: ReadClock>(led_pin: LedP, clk: ClkSrc) {
    let age: MiliSecs<_> = clk.read_count().time_scaled();
    let age: u16 = age.into();
    let modulod = age.integer() % 15_000;
    let numer = // math to map and scale the modulod to a triange pattern between 7_500 and 15_000
    led_pin.set_duty_cycle_fraction(numer, 15_000)
}
```

This example is quite contrived, but for every situation in which someone needs to read an oscilator count, there is no core
standard for this, the way there is for things like gpio pins, i2c, etc.

# Detailed design
[design]: #detailed-design

The core influence comes from two sources:

- `embedded-hal`: Just as this crate makes no impositions on specific implementation, but rather, make it easy to write highly portable embedded-rust
- `embedded-time`: An independant endeavour that does a great job of already encapsulating this core idea.

This is the code defining the `embedded_time::Clock` trait. I have changed the comments for the context of this RFC:

<details>
<summary> Expand to view annotated summary `embedded_time::Clock` trait </summary>
```rust
// I would rename this. A consistent oscilator is not necisarily a clock. It could be a PWM at 50% duty cycle,
// or an ab encoder tracking total distance moved, etc. The only assumption is that it is an incremental counter, and that
// each increment has a specific meaning (be it a meanure of time, distance, etc.).
pub trait Clock: Sized {

    // This `T` assosciated type is constrained to u32 or u64. I agree with constraining to an unsigned integer, but I am of the
    // opinion that it can be _any_ unsigned integer; It shall be the decision of the implementor to choose the type.
    // 
    // In addition, maybe integers-only as too limiting, as exotic types like `Wrapper(u32)` should be available. The guiding
    // principal here is "You are reading a thing that counts the number of occurances of an event." Unsigned integers make 
    // the most sense on what to reach for, but that should be a decision for the implementor.
    type T: TimeInt + Hash;

    // The original code runs its own fraction implementation. Strictly with respect to time, I feel using `fugit` is more
    // prudent.
    //
    // More broadly, I feel this should be an encapsulation of an interval. An interval could be a measure of time, a distance,
    // angle, and so on, depending on what is being counted.
    // 
    // I feel this should remain a compile-time defined statement. 
    const SCALING_FACTOR: Fraction;

    // With respect to assosciated types for errors, the question that defines the design choices made should be "What is best
    // for creating a standardised interface?"

    // I would rename this to `try_read`. I would also like to raise the idea of mirroring the standard `Read` pattern, where
    // it's not a return value that's used, but rather a passid in `&mut`. At its core, this interface should encapsulate
    // the reading of a count that increments, tightly coupled to an encapsulation of what is being counted. This implies:
    //  - it can read time: An implementation of this trait would define a duration-per-count
    //  - it can read absolute movement: each increment would correspond to a distance.
    //  - it is a _reader_: but is it different enough to `Read` implementors that it merits different patterns?
    //  - it reads _generic_ units: the notion of `now` and `time` are implementation dependant
    fn try_now(&self) -> Result<Instant<Self>, Error>;

    // Not exactly sure about how to read into this, or what should be done. Will update this part of RFC in response to
    // questions, feedback, and any further insight I gather. My thinking is anything that respembles construction or 
    // initialisation is out, much the same as any other trait in embedded-hal.
    fn new_timer<Dur: Duration>(
        &self,
        duration: Dur,
    ) -> Timer<param::OneShot, param::Armed, Self, Dur>
    where
        Dur: FixedPoint,
    {
        Timer::<param::None, param::None, Self, Dur>::new(&self, duration)
    }
}
```
</details>

Below is an initial draft design.

<details>
<summary>Expand to view pseudo-rust illustrative concept. </summary>
```rust
/// Base encapsulation for reading a clock perihpereal. Intended typical use-case is 
/// a HAL implementation adds this trait to a HAL type, and uses this interface to 
/// read a clock/timer register. Through implementation of this trait, this object 
/// will carry with it the means to read a clocks tick counter, and communicate the 
/// total duration this represents.
pub trait TimeCount: Sized {
    /// Typically u32 or u64 (or held within a type-wrapper) that corresponds with the 
    /// data type held within the peripheral register being read. 
    type RawData;

    /// The length of time that each clock-tick measures. Setting up a clock with a 80MHz, 
    /// a HAL might opt to set this as `fugit::TimerDuration<Self::RawData, 1, 80_000_000>`.
    ///
    /// Another option might something that expresses the measure as a frequency, in which
    /// case, something that encapsulates "eighty million" (as opposed to "one eighty 
    /// millionth").
    ///
    /// META: This might merit being a const instead.
    /// META: Should it also be documented about the Mul constraint?
    type TickMeasure: Duration + Mul<Self::RawData, Output = Self::TimeMeasure> + Default;

    /// A combinasion of raw count and measure. 80k ticks with 80MHz => 1 second.
    type TimeMeasure: Duration;


    /// Intended as an interface to a raw-register read.
    fn try_now_raw(&self) -> Result<Self::RawData, crate::clock::Error>;

    /// Interprates the tick-count of `try_now_raw`, and scales based on `TickMeasure` 
    fn try_now(&self) -> Result<Self::TimeMeasure, crate::clock::Error> {
        Self::TickMeasure::default() * self.try_now_raw()?
    }
}
```

</details>

And here is an initial thought as to how to handle the need to address wrapping.

<details>
<summary>`/// There are many sc...</summary>

```rust
/// There are many scenarios where a time-based tick-counter needs to handle the case
/// where the counter overflows. This trait is intended to provide a HAL with the means
/// of providing a `TimeCount` implementor with specialised interupt-based overflow
/// tracking. 
pub trait TimeWrap: TimeCount {
    /// A HAL implementation might register the provided method to handle an overflow
    /// interupt and increment a field that tracks overflow.
    ///
    /// META: I honestly don't know the best way to do this. This might prove to be
    ///       a key technical challenge
    fn register_wrap_interupt(self, ???);
}
```
</details>

This RFC initially proposed a base `Counter` trait, where a `Clock` trait would extend
it in a manner to handle time. This has changed. For background context, the initial
illustration of this concept is included here.

```rust
pub trait Clock: Counter
where <Self as Counter>::Output: Into<Seconds<...>>
{
    fn read_time(&self) -> Seconds<...> {
        let ticks = <Self as Counter>::read_ticks(self);
        Seconds::from(ticks)
    }
}
```

### Upstreamed feedback

James Munn shared a valuable summary of their thoughts following a lengthy discussion in DMs:

> you need to provide an interface that is reasonable for all parties, within reasonable use case [expectations] of all of them

<details>
<summary>It is expected that...</summary>
- ..a timer/counter will need to be shared - there are relatively few of them on many chips
- ..users will want high precision in some cases, 1k-1M are very reasonable number
- ..some timers have a limited range - 16/24/32 bit are common, but many chips only have 16/24
- ..programs will be expected for months to years at a time
- ..some users want to use the lowest possible power design they can
- ..not all chips have atomics, and may need to use critical sections
- ..users may have other interrupts, some of which may have higher priority than your timer/counter interrupt
- ..users may want to use a global timer inside of other interrupts
</details>


# How We Teach This
[how-we-teach-this]: #how-we-teach-this

Personally, I intend to make PRs into embedded-hal reverse dependencies, and update their encapsulation of time-tracking
to include implementations that are officially supported by the embedded working group. I also see the need to add examples
of both implementation, and use (such as the time-rotating e.g. I hypothesised above).

# Drawbacks
[drawbacks]: #drawbacks

I'm mostly basing my thoughts on the models that I see throughout the embedded-hal crate. The drawbacks assosciated with the
various modules that define a common interface for things like gpio pins, I2c, etc would ideally apply in the same manner here.

# Alternatives
[alternatives]: #alternatives

embassy-time has an encapsulation of time, however that lives in an async-first context.
many MCU hals have their own way of encapsulating time, and other incremental counts.
fugit is an embedded-first encapsulation of mapping time-oscilator counts to a measurement of time in SI units.


# Unresolved questions
[unresolved]: #unresolved-questions

How to move forward? I propose an `embedded-count` added to the rust-embedded repository, initially just a placeholder,
I'll fork it, and begin initial work. At an appropriate time, it will be upstreamed, and v0.0.2 will be released. If all goes
well, its form will be representative of this RFC.

Are we comfortable with using `typenum` instead of const generics. This will remove the limitations of const generics
entirely, including the need for nightly features, at the risk of interface ergonomics, and compromising user familiarity.
I wish to explore the use of this crate, though I feel it's a core requirement that the change in UX to be trivial. I.e. other
than theneed to use `typenum::U5` where `5u32/64/size/8` would be used to set a generic const.

How should we approach assosciated error types?

What is the wish-list of industry, such as the various teams writing the hal crates for specific MCUs/platforms?

What constraits make sense? I feel that `Ord` ought to be required for the raw data being read. In my mind, a type that can
be used to count something, would necissarily have an ordering between any two possible values. I also think some thought should
be put into providing the maths where compatable. in pseudo-rust: `impl<A: Counter, B: Counter> Add<A::Output> for for B::Output`
with where-clauses that constrain that the output types can do the Add.


