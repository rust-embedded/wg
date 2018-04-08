# Interoperability with existing codebases

> **This section should cover:**
>
> * Reminder on how Rust has no runtime, etc, and plays nice with other systems languages
> * Practical examples of integrating Rust and other languages in both directions, including techniques for integration
> * WHY you would want to replace certain parts with Rust
>     * Parsers, critical components
> * WHY you would want to NOT replace certain parts with Rust
>     * Large, complex, and stable components, like drivers, network stacks, internal libraries
> * Tooling necessary for integration