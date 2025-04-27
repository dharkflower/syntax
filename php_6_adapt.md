### adapt
I don't think this is a threat to a more standard architecture like Symfony Messenger kind of stuff, I think it might even compliment it. It's just some code golf that could have some under-the-hood speed improvements if harnessed correctly and some simplifications for people coding dynamic math-heavy applications - like a load balancer, for example.

```php
<?php

class HomeController {

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
        User $user

    ) : Response {

        // okay, so how you *could* currently do load calculation is like this:
        $iterations = 0;

        // you'd have to run a shell command like top or something
        $load = getload();
        $load = 4; // hardcode this for now
        
        if ($load < 5) {
            $iterations = 100;
        }

        else {
            $iterations = 75;
        }

        for ($i = 0; $i < $iterations; $i++) {

            // this block runs either 100 times if the computer is pretty freed up or else 75 times if it's not

        }

        return $this->render('index');
    }
}

```
What if we abstracted some of the mathematical type code we write with a new type/token called `adapt` that is meant to be a sort of dynamic type? It could select a value from a set of numbers based on load. I imagine it working something like this:
```php
<?php

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
        User $user

    ) : Response {

        // new `adapt` type
        $iterations = (adapt) 100 75 50 25;

        // now, $iterations is 75 if the load is high
        for ($i = 0; $i < $iterations; $i++) {

        }

        return $this->render('index');
    }
```
### What's the point?
It's just potentially a concept that actually exists in the AI realm; for every "forgivable, bendable, stretchable" variable that determines things like compute-intensive iterations there is an opportunity to make decisions on not just what to think but when *not to* and throw in the towel in the name of performance. It makes way for other computations. It's logical deductive reasoning, if you think about it; killing off zombie threads is a form of thinking backwards.

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
The coolest thing about this is that `adapt` doesn't only have to determine iterations, and wouldn't actually need to select based on load; that's just an example of adapting an important iteration variable. Basically `adapt` could provide a pretty smooth API to mathematically react to things based on other things - like load.

I think there is a syntax to be had that allows a sort of definition of what criteria determines selection.