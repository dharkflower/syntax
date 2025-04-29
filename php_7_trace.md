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
In the same way a Symfony route can function trace at the PHP level, a Symfony route could function trace at the C level I think. You might have to compile a custom PHP binary meant for tracing, or maybe use an extension but I'm not sure.

We could route-level know what sequence the PHP functions run in and then either run Reflection classes, run a script that imports the source code, or hit a database to get a per-PHP-function dependency map of C functions it's using. We might be able to then use PHP source code as a reference and reconstruct the source code per-route in a `/tmp` folder or something.

The goal would be to programatically populate package.json methodness a Frankenstein sequence of the C source code of each PHP function that's used and then compile it all into a PHP micro-binary. Store it with an ID somewhere and then do some Apache2 config to use micro-binaries instead of the full PHP binary when they're available. Potentially quiker than nes.

### Syntax
I think that there are some potential decorator/docblock type things that would allow for the informing of this hypothetical build process of helpful things via the usage of new syntax. I'm going to circle back to this, but just to get the wheels turning for now I'd check out [hint](https://github.com/dharkflower/syntax/blob/main/php_4_hint.md)
