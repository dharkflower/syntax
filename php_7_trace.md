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
- trim
- echo

### Hypothesis
In the same way a Symfony route can function trace at the PHP level, a Symfony route could function trace at the C level to assemble a sequence of the C functions into a micro-binary. You might have to compile a custom PHP binary meant for tracing, or maybe use an extension. I'm not sure.

If we route-level know what sequence the PHP functions run in and then run Reflections classes against all of those functions to get a per-PHP-function sequence of C code (by using PHP source code as a reference) we might be able to also use PHP source code as a reference to reconstruct the source code on a per-route basis.

The goal would be to programmatically Frankenstein sequence the C source code of each PHP function that's used and then compile it all into a route-specific PHP micro-binary. Then store it with an ID somewhere and do some Apache2 config to use micro-binaries if they're available instead of the full PHP binary. Potentially quiker than nes.

### Syntax
I think that there are some potential decorator/docblock type things that would allow for the informing of this hypothetical build process of helpful things via the usage of new syntax. I'm going to circle back to this.
