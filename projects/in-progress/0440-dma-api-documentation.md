# Metadata

* Task Shepherd: @korken89
* Contributors: @thalesfragoso, @ra-kete
* Relevant repository: https://github.com/ra-kete/dma-poc

# Tracking Issues

- [Traits Discussion](https://github.com/ra-kete/dma-poc/issues/1)

# Background

Recently it was discovered that the example of a DMA API provided in the [embedonomicon](https://github.com/rust-embedded/embedonomicon) is unsound in relation to the guarantees it claims (see [embedonomicon#64](https://github.com/rust-embedded/embedonomicon/issues/64)).

A safe and flexible DMA API has shown to be a complex task to achieve. There are several complications that must be addressed:
- As the DMA engine might be considered an outside agent, care must be taken to ensure rust's guarantees and avoid problems with misoptimizations.
- In current days, we rely on pointer capture/escape to avoid some misoptimizations, this is an implementation specific behavior (LLVM).
- There is need for a flexible API capable of interoperation with others components of the ecosystem, as well as working by itself without deeper dependencies.

Given the above points and others, DMA has shown to be a complex system to get right in the current ecosystem. This project aims to experiment and document different implementations to find a suitable option to recommend to the community.

# Suggested Task

* The final goal of this project is to provide a sound and flexible API and update the documentation found in the Embedonomicon.

# Unresolved Questions

- We need a trait that is capable of marking a buffer as suitable for DMA transfers, [StableDeref](https://crates.io/crates/stable_deref_trait) is a good candidate, but its guarantees only hold for moves and not mutation of the type, the guarantees hold for `deref_mut` but not for other methods that take `&mut`. This seems to be enough for DMA, since we just need `&mut` for exclusive access, and usually all that is needed is a pointer to the buffer and a size to configure the DMA peripheral, but it is unsure if this holds true for all implementations. Should we use `StableDeref` or create another trait, or even go with a solution that does not require marker traits but that might be less flexible ?
