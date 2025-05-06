Check out [trace](https://github.com/dharkflower/syntax/blob/main/php_7_trace.md) if you want to read about some compiling potential here; if `thread` and `scope` were to be implemented, it would be a lot more likely to be possible to C-level trace a Symfony route and Frankenstein it back together in the name of speed. These new code block types would help bring some definition and clarity to what is supposed to be threaded and how.

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

This little Node.js snippet inspired this syntax idea. At the start of the execution of a PHP function you could **prepare** the function for what it's about to execute by defining some threading logic with new block types `thread` and `scope`

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

            $user->trackGenericView();
            $user->save();

        }

        scope TRACK_SPECIFIC_VIEW {

            $user->trackSpecificView('index');
            $user->save();

        }

        // the GET_GENERIC_VIEWS scope gets defined but never ran and lies dormant
        // still allowed to be imported, it just doesn't default run in this scope
        #[Dormant]
        scope GET_GENERIC_VIEWS {

            // pretend get_generic_views is a real function
            return get_generic_views();
        }

        // the FETCH_DATA scope runs synchronously
        // if defined as dormant, it can still be scoped into Twig later
        #[Dormant]
        scope FETCH_DATA {

            // this does not make the `index` func return, it makes FETCH_DATA return
            return fetch_data();
        }

        // scope yields variable to inject Twig with
        return $this->render('index', [
            'data' => FETCH_DATA,
            'data' => scope FETCH_DATA, // or ?
        ]);
    }
}
```

Maybe some coffee and bring into existence some kind of import syntax like `scope IndexController ::: index => TRACK_GENERIC_VIEW;` that allows you to import (and thread, if you want to) scopes of code. `scope` is a `code block` that gets followed into and interpreted like an if statement.

It's a little meta to have another depth of stuff that you can import, fair.

But the point of scopes is smart, dynamic, low-level threading. `scope` begs questions.

I will admit it's close to just being a type of function but even Mr. Clean himself would drop his stupid, disgusting sponge at the sight of something so fresh; I'd bet on it. Check out the ES6 import formatting:

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

The reason I include this is to show off how well Node.js has done with keeping up to date with edge UX trends; if I were to mimick one thing from Node.js in a PHP syntax idea it would be based on the above. Huge fan.

Full snippet, here's a fully namespaced PHP class with some scope and class imports:

```php
<?php

// correct namespace
namespace App\Controller;

// you probably need to import the class that contains scope code
use App\Controller\IndexController;

// core imports
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

// proper
scope App\Controller\IndexController ::: index => TRACK_GENERIC_VIEW; // explicit, neat
scope App\Controller\IndexController ::: index => GET_GENERIC_VIEWS; // same

// list style
scope App\Controller\IndexController ::: index {

    TRACK_GENERIC_VIEW,
    GET_GENERIC_VIEWS,

}

// list style other
scope App\Controller\IndexController {

    index ::: {

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

        // if you must, prefix with `scope` to not interfere with existing syntax
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
I would openly invite anyone from the following communities to jump in on this, just as a starting point:

- PHP
- Linux
- Symfony
- Drupal
- Node.js
- Python
- Django
- Ruby
- Rails
