Check out [trace](https://github.com/dharkflower/syntax/blob/main/php_7_trace.md) if you want to read about some compiling potential here.

Lots to talk about. Let us proceed. Node.js snippet that inspired this syntax idea:

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

What if at the start of the execution of a PHP function you could **prepare** the function for what it's about to execute by defining threadable elements via new tokens `thread` and `scope`

Allowing multiple types of tokens in `thread` like scope references, function calls, constants, variables, imported classes, and instantiated objects might facilitate some pretty dope C-level code coordination.

```php
class IndexController extends Controller {

    #[Route('/', name: 'index')]
    public function index (

        User $user

    ) : Response {

        thread {

            // neither of these need to block UX
            TRACK_GENERIC_VIEW,
            TRACK_SPECIFIC_VIEW,

        }

        // vanilla enters here as if it was an if (TRUE)
        scope TRACK_GENERIC_VIEW {

            // has access to $user variable somehow.......
            // dependency injection hell but oh so smooth
            $user->trackGenericView();
            $user->save();

            // either returns var from scope
            // or does nothing and moves on
        }

        scope TRACK_SPECIFIC_VIEW {

            $user->trackSpecificView('index');
            $user->save();

        }

        // this gets cached, gets the hour with an hour TTL
        static scope GET_CURRENT_HOUR : 3600 {

            // returns the hour once per hour
            // weird
            return date('h');
        }

        // the GET_GENERIC_VIEWS scope is dormant
        // so it doesn't run
        //
        // even though not defined as threaded
        // you can still call the scope later
        // even in the parent scope
        dormant scope GET_GENERIC_VIEWS {

            // returning GET_GENERIC_VIEWS yields var
            return [4, 5];
        }

        dormant scope FETCH_DATA {

            // returning FETCH_DATA yields a var
            return [3];
        }

        // FETCH_DATA scope yields data
        return $this->render('index', [
            'data' => FETCH_DATA,
        ]);
    }
}
```

It's a little meta to have another granularity of stuff that you can import, fair. But the main point of scopes (I think) is smart, dynamic, low-level, configurable with syntax threading. You might be able to even do some kind of scope caching. If the scope is decorated or defined as cacheable with a TTL then I guess you don't need to run it? Weird. PHP 2.0. :) Example above.

I will admit it's close to just being a type of function or class if you admit that even Mr. Clean himself would drop his stupid, disgusting sponge at the sight of something so fresh; I'd bet on it.

Full snippet, here's a fully namespaced PHP class with some scope and class imports:

```php
<?php

namespace App\Controller;

// core imports
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

// pull in the class that has the methods with scopes
// different than `use` so a few different options here
// some of it pulling some concepts from Symfony routing
scope App\Controller\IndexController;

// short qualifier
// with the extra granularity qualifier `index`
scope IndexController\index => TRACK_GENERIC_VIEW;
scope IndexController\index => GET_GENERIC_VIEWS;

// or list style
scope IndexController ::: index {

    TRACK_GENERIC_VIEW,
    GET_GENERIC_VIEWS,

}

// or list style other
scope IndexController ::: {

    index {

        TRACK_GENERIC_VIEW,
        GET_GENERIC_VIEWS,

    }

}

class AnalyticsController extends AbstractController {

    #[Route('/analytics', name: 'analytics')]
    public function analytics (

        Request $request

    ) : Response {

        // an example of wanting to use a scope synchronously and asynchronously in different places
        // in the IndexController, TRACK_GENERIC_VIEW didn't need to block for the UX
        // in the AnalyticsController, TRACK_GENERIC_VIEW needs to block
        // two different usages of code in two places in two different ways
        // to drop-in use a scope synchronously
        // I forget what those >>> blocks are called
        // they bother me so cross functionality syntax
        // important: synchronous here
        TRACK_GENERIC_VIEW;

        // now that the TRACK_GENERIC_VIEW scope has ran synchronously, you can pull a full report
        // inject the full report into Twig
        return $this->render('analytics', [
            'views' => GET_GENERIC_VIEWS
        ]);
    }
}
```
