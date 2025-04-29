Let's say a web request comes in and here's the code of `index.php` line by line:

- $kernel = new Kernel()
- $output = $kernel->run()
- echo trim($output)

Let's break it down, as an example, and say that a request to /user traces like this:

- Kernel -> __construct
- Kernel -> run // runs 1, 2, 3, 2
- Kernel -> runNested1
- Kernel -> runNested2
- Kernel -> runNested3
- Kernel -> runNested2
- echo
- trim

### Hypothesis
In the same way a Symfony route can function trace at the PHP level, a Symfony route can function trace at the C level. If we can route-level know what sequence the PHP functions run in and then run Reflection classes on those functions, we could route-level know what sequence the C functions run in. We could programatically populate package.json methodness a Frankenstein sequence of the C source code of each PHP function that's used and then compile it all into a micro-binary. Then, do some Apache2 config to use micro-binaries instead of the full PHP binary when they're available. Potentially quiker than nes.

### Syntax
I think that there are some potential decorator/docblock type things that would allow for the informing of this hypothetical build process of helpful things via the usage of new syntax. I'm going to circle back to this, but just to get the wheels turning for now I'd check out [hint](https://github.com/dharkflower/syntax/blob/main/php_4_hint.md)
