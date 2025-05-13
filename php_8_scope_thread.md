Check out [trace](https://github.com/dharkflower/syntax/blob/main/php_7_trace.md) if you want to read about compiling potential.

Node.js snippet that inspired this syntax idea:

```javascript
// do some stuff
import { createServer } from 'express'
const port = 3000

// do some stuff later with that stuff
process.nextTick(() => {
    const app = createServer(() => {})
    app.listen(port)
})
```

What if at the start of a PHP function you could **prepare** it for what it's about to execute?

New tokens: **`thread`, `scope`, and `produce`**

### `thread` is currently an array.

It could end up an associative array type syntax, allowing multiple types of tokens in it like scope references, something with `produce`, functions, constants, variables, imported classes, environment variables. It might facilitate some pretty dope C-level code coordination.

### `produce` is just `return` but for `scope`

To return/yield/execute a `scope` into a variable like `$view = TRACK_GENERIC_VIEW;` you use `produce`

The only point of `produce` is to have a clear syntax distinction between `returning from a function` and `producing from a scope`

### `scope` sections out code

It has optional produce types that follow the same syntax as return types, and a TTL that's double-arrow defined

```php
class IndexController extends Controller {

    #[Route('/', name: 'index')]
    public function index (

        User $user

    ) : Response {

        thread {

            // neither of these need to block UX
            TRACK_GENERIC_VIEW,
            TRACK_INDEX_VIEW,

        }

        // enters here as if it was an if (TRUE)
        scope TRACK_GENERIC_VIEW : bool {

            // $user injected dependency
            // even if being imported elsewhere
            $user->trackGenericView();
            $user->save();

            // produces something
            // like this: `$view = TRACK_GENERIC_VIEW;

            // producing doesn't return or break
            produce TRUE;

            // continues past curly brace...
        } //// and continues on here...

        // enters here as if it was an if (TRUE)
        scope TRACK_INDEX_VIEW {

            // same here
            $user->trackIndexView();
            $user->save();

            // does not produce anything
            // like this: `TRACK_INDEX_VIEW;`

            // continues past curly brace...
        } //// and continues on here...

        // the next scopes are static
        // they get defined but not entered
        // can be imported and used elsewhere
        // produces hour int with a 3600 TTL
        static scope GET_CURRENT_HOUR => 3600 : int {

            // once per hour
            produce date('h');
        }

        static scope GET_GENERIC_VIEWS : array {

            produce [4, 5];
        }

        static scope FETCH_DATA {

            produce [3];
        }

        // FETCH_DATA scope produces data
        return $this->render('index', [
            'data' => FETCH_DATA,
        ]);
    }
}
```

It's meta to have another granularity: fair. But the point of these new tokens are performance. They enable smart, dynamic, low-level threading, code reusability, caching, all kinds of weird stuff here.

I'm going to think more about how we could implement some ways to check a scope's available variables and if it's using variables as parameters be able to hash them and look up if certain functions don't have to be ran. Beef up the TTL.

I will admit some of this is close to just being a type of function or utilization of a threading class if you admit that even Mr. Clean himself would drop his stupid, disgusting sponge in shock at the sight of something so fresh; I'd bet on it.

Full snippet.

```php
<?php

namespace App\Controller;

// pull in the parent scope, enables short qualifier
scope ::: App\Controller\IndexController\index;

// uses short qualifier
scope index ::: TRACK_GENERIC_VIEW;
scope index ::: GET_GENERIC_VIEWS;

// list format, kind of cool
scope index ::: {
    
    TRACK_GENERIC_VIEW,
    GET_GENERIC_VIEWS,

};

class AnalyticsController extends AbstractController {

    #[Route('/analytics', name: 'analytics')]
    public function analytics (

        Request $request

    ) : Response {

        // an example of synchronous and asynchronous
        // in the IndexController,
        // TRACK_GENERIC_VIEW didn't need to block
        // in the AnalyticsController,
        // TRACK_GENERIC_VIEW needs to block
        // two different usages of code in two places
        // drop-in use a scope synchronously
        // because no thread {} block is defined
        TRACK_GENERIC_VIEW;

        // now that the TRACK_GENERIC_VIEW scope ran
        // you can pull a full report
        // inject the full report into Twig
        return $this->render('analytics', [
            'views' => GET_GENERIC_VIEWS,
        ]);
    }
}
```
