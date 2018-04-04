# `.cargo/config`

Our `cargo` invocations are looking a bit unwieldy because we pass some many arguments to them. We
can simplify the invocations using a Cargo configuration file. Create a `.cargo/config` file in the
root of the Cargo project with the following contents.

``` console
$ cat .cargo/config
```

``` toml
[target.thumbv7m-none-eabi]
rustflags = [
  "-C", "linker=arm-none-eabi-ld",
  "-Z", "linker-flavor=ld",
]

[build]
target = "thumbv7m-none-eabi"
```

Now `xargo build` does pretty much the same thing as the long `xargo rustc` invocation did. I have
left out the `-e` linker flag because we'll be using something different in the next chapter. If you
test this out you should get the empty output binary.

``` console
$ xargo build

$ arm-none-eabi-size target/thumbv7m-none-eabi/debug/app
   text    data     bss     dec     hex filename
      0       0       0       0       0 target/thumbv7m-none-eabi/debug/app
```
