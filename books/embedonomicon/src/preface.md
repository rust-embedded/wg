# Preface

The Embedonomicon documents the low level magic that binds together no-std binaries. This
information is particularly useful to people to want to the initial work of getting Rust working on
a new architecture / target.

For the author's convenience the Embededonomicon has been written as a bottom up guide that uses
the ARM Cortex-M as an example but many of the concepts here can be ported to other architectures /
targets.

The code in this book was tested using the following compiler:

``` console
$ rustc -V
rustc 1.26.0-nightly (adf2135ad 2018-03-17)
```
