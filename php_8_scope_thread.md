Check out [trace](https://github.com/dharkflower/syntax/blob/main/php_7_trace.md) if you want to read about some compiling potential here.

Lots to talk about. Let us proceed.

### Asynchronous Pattern

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

This little Node.js snippet inspired this syntax idea. At the start of the execution of a PHP function you could **prepare** the function for what it's about to execute by defining threadable elements via new code block types `thread` and `scope`

Allowing multiple types of tokens in `thread` like `scope` references, function calls, constants, variables, imported classes, and objects might facilitate some pretty dope C-level code coordination.

```php
class IndexController extends Controller {

    #[Route('/', name: 'index')]
    public function index (

        User $user

    ) : Response {

        thread {

            TRACK_GENERIC_VIEW, // doesn't need to block user interaction
            TRACK_SPECIFIC_VIEW, // doesn't need to block user interaction

        }

        scope TRACK_GENERIC_VIEW {

            // has access to $user variable..............
            // dependency injection nightmare importing it but oh so smooth
            // seamless flow into and out of the scope
            $user->trackGenericView();
            $user->save();

        }

        scope TRACK_SPECIFIC_VIEW {

            $user->trackSpecificView('index');
            $user->save();

        }

        // the GET_GENERIC_VIEWS scope is dormant
        // it just doesn't run **right now** even though it's synchronous
        // you can still call the scope later
        // even in the parent scope of GET_GENERIC_VIEWS
        #[Dormant]
        scope GET_GENERIC_VIEWS {

            // pretend get_generic_views is a real function
            return get_generic_views();
        }

        #[Dormant]
        scope FETCH_DATA {

            // this does not make the `index` func return
            // it makes FETCH_DATA return
            return fetch_data();
        }

        // FETCH_DATA scope yields data
        return $this->render('index', [
            'data' => FETCH_DATA,
        ]);
    }
}
```

It's a little meta to have another depth of stuff that you can import, fair. But the main point of scopes are smart, dynamic, low-level threading. You might be able to even do some kind of caching of them or some kind of static scope that can be accessed from different threads. Weird.

I will admit it's close to just being a type of function but even Mr. Clean himself would drop his stupid, disgusting sponge at the sight of something so fresh; I'd bet on it. Check out some ES6 import formatting:

```javascript
// proper
import { createElement, createClass } from 'react'

// list style of yuck
import {
    createElement,
    createClass, // trailing commas not allowed but maybe they should be allowed
} from 'react' // this line looks out of place

// somehow I wish this was how it worked
from 'react' import {
    createElement,
    createClass,
}
```

The reason I include this is to show off how well Node.js has done with keeping up to date with edge code UX trends; if I were to mimick one thing from Node.js in a PHP syntax idea it would be based on the latter. Huge fan.

Full snippet, here's a fully namespaced PHP class with some scope and class imports:

```php
<?php

// correct namespace
namespace App\Controller;

// core imports
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

// you probably need to import the class that contains scope code
scope App\Controller\IndexController;

// proper
scope IndexController\index => TRACK_GENERIC_VIEW; // explicit, neat
scope IndexController\index => GET_GENERIC_VIEWS; // same

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
        // the reason why, in a business intelligence setting, is because it needs that last view to track
        // it needs to be synchronous here because it's the analytics page and it needs to be accurate, now
        // two different usages of code in two places in two different ways
        // to drop-in use a scope synchronously, just trail it with a semicolon
        // important: synchronous here
        TRACK_GENERIC_VIEW;

        // or, if you must, prefix with `scope` to not interfere with existing syntax
        // important: synchronous here
        scope TRACK_GENERIC_VIEW;

        // now that the synchronous TRACK_GENERIC_VIEW scope has ran, you can pull a full report
        // import GET_GENERIC_VIEWS even though it's dormant where it's defined
        $views = scope GET_GENERIC_VIEWS;

        // inject the full report into Twig
        return $this->render('analytics', [
            'views' => $views
        ]);
    }
}
```
