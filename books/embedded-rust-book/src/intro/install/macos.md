> **⚠️: This section contains exports from [Japaric's Discovery] book.**
>
> Contents should be reviewed for consistency in the context
> of this book before "publishing"

[Japaric's Discovery]: https://japaric.github.io/discovery/

> **⚠️: This section has not been checked as of 2018-05-26**
>
> Contents should be checked to still be working with current `nightly`
> or `stable` Rust

# MacOS

All the tools can be install using [Homebrew]:

[Homebrew]: http://brew.sh/

``` console
$ brew cask install gcc-arm-embedded
$ brew install minicom openocd
```

If the `brew cask` command doesn't work (`Error: Unknown command: cask`), then run `brew tap
Caskroom/tap` first and try again.

That's all! Go to the [next section].

[next section]: 03-setup/verify.html
