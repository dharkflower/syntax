Let's say that the trail code looked like this:

- index.php
- $kernel = new Kernel()
- $output = $kernel->run()
- echo trim($output)

Let's break it down, as an example, and say that a request to /user traces like this:

- index.php
- Kernel -> __construct
- Kernel -> run // runs 1, 2, 3, 2
- Kernel -> runNested1
- Kernel -> runNested2
- Kernel -> runNested3
- Kernel -> runNested2
- echo
- trim

I don't know how to explain what I think is possible here yet - not fully. It involves reflections at the PHP and C level. But couldn't PHP source code be programatically sequenced out to produce a Frankenstein'd version of PHP source code? It would only contain the necessary C code for the PHP functions that are used for that individual request. You could then compile the sequenced C code and cache the resulting micro-binary.

Key point: if we already know what sequence the C functions will run in, for a specific request that we code out, can't we programatically create a C sequence beforehand and then compile it?

I know that entails alot of other stuff.

You could run a build function that kicks out micro-binaries for all of the routes in advance, and then route to them programatically instead of hoisting up the *entire* PHP binary at execution time. It almost seems like an Apache2 plugin could be utilized here somehow.

### Syntax
I think that there are some potential decorator/docblock type things that would allow for the informing of this hypothetical build process of helpful things via the usage of new syntax. I'm going to circle back to this, but just to get the wheels turning for now I'd check out [hint](https://github.com/dharkflower/syntax/blob/main/php_4_hint.md)
