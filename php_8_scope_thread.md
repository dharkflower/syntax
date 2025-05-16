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

### `thread` is an array

`thread` is just an array of scopes to thread.

If `thread` was upgraded to an associative array syntax, allowing multiple tokens in it like TTL integers, scope references, something with `produce` or `yield`, functions, statics, constants, variables, imported classes, environment variables, it might especially facilitate some pretty dope C-level code coordination.

### `produce` is just `return` but for `scope`

To return/yield/execute a `scope` into a variable like `$view = TRACK_GENERIC_VIEW;` you use `produce`

The only point of `produce` is to have a clear syntax distinction between `returning from a function` and `producing from a scope`, `produce` is ignored if it doesn't stdout into something.

### `scope` sections out code

The point of scopes is to section out either a block you want threaded, a block you want TTL'd, or a block you want imported elsewhere.

`scope` has optional produce types that follow the same syntax as return types that could be helpful in code coordination, as well as a default TTL syntax that's double-arrowed.

```php
class IndexController extends Controller {

    #[Route('/', name: 'index')]
    public function index (

        User $user

    ) : Response {

        thread {

            // neither of these need to block UX
            TRACK_GENERIC_VIEW
            TRACK_INDEX_VIEW

        }

        // enters here as if it was an if (TRUE)
        scope TRACK_GENERIC_VIEW : bool {

            // $user injected dependency available
            // even if being imported elsewhere
            $user->trackGenericView();
            $user->save();

            // produces something
            // call like this: `$view = TRACK_GENERIC_VIEW;
            produce TRUE;

            // continues past produce...
        } //// and continues past curly brace...

        // enters here as if it was an if (TRUE)
        scope TRACK_INDEX_VIEW {

            // same here
            $user->trackIndexView();
            $user->save();

            // does not produce anything
            // call like this: `TRACK_INDEX_VIEW;`

            // continues...
        } //// and continues past curly brace...

        // produces random number that lives for one hour
        // dormant but importable elsewhere
        dormant scope GET_RANDOM_NUMBER => 3600 : int {

            produce rand(1, 10);
        }

        dormant scope GET_GENERIC_VIEWS : array {

            produce [4, 5];
        }

        dormant scope FETCH_DATA {

            produce [3];
        }

        // FETCH_DATA dormant scope still yields data
        return $this->render('index', [
            'data' => FETCH_DATA,
        ]);
    }
}
```

It's meta to have another granularity but these tokens enable smart, dynamic, low-level threading and code reusability, caching, all kinds of weird stuff. They do have a point.

I will admit some of this is close to just being a type of function or utilization of a threading class if you admit that even Mr. Clean himself would drop his stupid, disgusting sponge in shock at the sight of something so fresh; I'd bet on it.

Full snippet.

```php
<?php

namespace App\Controller;

// singular format idea, my vote
scope App\Controller\IndexController\index ::: {

    TRACK_GENERIC_VIEW
    GET_GENERIC_VIEWS

};

// multiple format idea, my vote also
scope App\Controller\IndexController ::: {

    index ::: {

        TRACK_GENERIC_VIEW
        GET_GENERIC_VIEWS

    }

    other ::: {
        
       // ...

    }

};

// or pull in the scope's short qualifier
scope App\Controller\IndexController\index;

// and use it later
scope index ::: TRACK_GENERIC_VIEW;

// singular format with short qualifier
scope index ::: {
    
    TRACK_GENERIC_VIEW

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
        // you can get the generic views
        // inject the full views count into Twig
        return $this->render('analytics', [
            'views' => GET_GENERIC_VIEWS,
        ]);
    }
}
```
