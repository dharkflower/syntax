### adapt
I don't think this is a threat to a more standard architecture like something built on Symfony Messenger; I think it might even compliment it. It's just some code golf that could have some under-the-hood speed improvements if harnessed correctly and provide some simplifications for people coding dynamic math-heavy applications - like a load balancer, for example. Here's a simple example without `adapt`:

```php
<?php

class HomeController {

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
        User $user

    ) : Response {

        // hardcode to 4
        $load = getload();
        
        if ($load < 5) {
            $iterations = 100;
        }

        else {
            $iterations = 75;
        }

        // runs 100 times
        for ($i = 0; $i < $iterations; $i++) {

        }

        return $this->render('index');
    }
}

```
What if we abstracted out some of the mathematical type code we write with a new type/token called `adapt` that is meant to dynamically accomplish the same thing as above? It could select a value from a set of numbers based on criteria like load. I imagine it working something like this - at least, without defining the criteria as load:
```php
<?php

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
        User $user

    ) : Response {

        // new `adapt` type
        $iterations = (adapt) 100 75 50 25;

        // runs either 100, 75, 50, or 25 times
        for ($i = 0; $i < $iterations; $i++) {

        }

        return $this->render('index');
    }
```
### What's the point?
It's just potentially a concept that actually exists in the AI realm; for every "forgivable, bendable, stretchable" variable that determines something like number of compute-intensive iterations there is an opportunity to make micro-decisions on not just what to think but when *not to* and throw in the towel - in the name of performance. It makes way for other computations by arming each thread with logical deductive reasoning that smoothly knows when to limit itself, if you think about it; killing off zombie threads is a form of thinking backwards.

It's almost like a switch case or a CSS3 keyframes transition. It could end up being something like this:
```php
<?php

adapt ($iterations) {

    load > 4:
        // run only 75 times
        break;

    default:
        // run full 100 times
        break;

}
```
The coolest thing about this is that `adapt` doesn't only have to determine iterations, and wouldn't actually need to select based on load; that's just an example of adapting an important iteration variable. Basically `adapt` could abstract out a pretty smooth API to mathematically react to things based on other things - generally.

I think there is a syntax to be had that allows a sort of definition of what criteria determines selection of a value but I have been having a hard time starting on it for a few reasons:

- that's some complex logic to be encapsulated in such a short line and syntax
- other stuff
- trying to wrap my head around how to make it super duper flexible

Here's what I have so far.

- I want the syntax to be able to define a shell command to run to determine adaption criteria
- I also want the syntax to be able to define a class to automatically run as an adaption criteria definition determiner

This has been leading to some syntax ideas that go something like this, I'm going to just sketch them all out and do your best to understand what kind of logic it would execute. Crude, not well-thought-out examples. Will update soon.

```php
<?php

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
        User $user

    ) : Response {

        // new: be able to programatically define adapt values into an arr
        $values = [
            100,
            75,
            50,
            25,
        ];

        // changes this
        $iterations = (adapt) 100 75 50 25;

        // to this
        $iterations = (adapt) $values;

        // but we need to be able to define adaptation criteria one of two ways:
        $criteria = 'top -b -n 1 | grep "load average" | awk "{print $10, $11, $12}"; // a shell command, example command to determine system load
        $iterations = (adapt) $values $criteria;

        // or through a reference to an adaptation criteria determiner class
        $iterations = (adapt) $values SystemLoadAdaptation;

        // runs either 100, 75, 50, or 25 times
        for ($i = 0; $i < $iterations; $i++) {
        }

        // awesome example, abstracts granular control over the number of iterations to run to an external Node.js script
        $criteria = 'node /var/www/criteria/index.js';
        $iterations = (adapt) $values $criteria;

        // runs either 100, 75, 50, or 25 times, depending upon the result of the Node.js script
        for ($i = 0; $i < $iterations; $i++) {
        }

        return $this->render('index');
    }
```
