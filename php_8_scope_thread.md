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
<?php

namespace App\Controller;

class IndexController extends Controller {

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
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

        // the GET_GENERIC_VIEWS scope runs synchronously
        $views = scope GET_GENERIC_VIEWS {

            // pretend get_generic_views is a real function
            return get_generic_views();
        }

        // the FETCH_DATA scope runs synchronously
        $data = scope FETCH_DATA {

            // this does not make `index` return, it makes FETCH_DATA return
            return fetch_data();
        }

        // the GET_GENERIC_VIEWS scope yields the $views variable
        // the FETCH_DATA scope yields the $data variable, which is now available in this scope
        // $data = FETCH_DATA;` in other places will fetch the result into the variable

        return $this->render('index', [
            'data' => $data,
            'views' => $views,
        ]);
    }
}
```

And then some kind of import syntax like `scope IndexController ::: index => TRACK_GENERIC_VIEW;` that allows you to import (and thread, if you want to) scopes of code. It's a little meta. Fair. But `scope` is versatile like `use`

`scope` begs questions. Smart, dynamic threading. It doesn't prevent the code from getting executed, all of it still gets ran; it's a `code block type` that gets followed into and interpreted like an if statement.

Interesting. Here's an example of a thread definition of a scope import:

```php
<?php

namespace App\Controller;

scope App\IndexController ::: index => TRACK_GENERIC_VIEW;
scope App\IndexController ::: index => GET_GENERIC_VIEWS;

class AnalyticsController extends Controller {

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
        TRACK_GENERIC_VIEW;

        // if you must, prefix with `scope` to not interfere with existing syntax
        scope TRACK_GENERIC_VIEW;
        $views = scope GET_GENERIC_VIEWS;

        return $this->render('analytics', [
            'views' => $views
        ]);
    }
}
```
