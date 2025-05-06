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

class HomeController extends Controller {

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
        User $user

    ) : Response {

        thread {

            TRACK_GENERIC_VIEW,
            TRACK_SPECIFIC_VIEW,

        }

        scope TRACK_GENERIC_VIEW {

            $user->trackGenericView();
            $user->save();

        }

        scope TRACK_SPECIFIC_VIEW {

            $user->trackSpecificView('index');
            $user->save();

        }

        scope HIT_API {

            $data = hit_api();

        }

        return $this->render('index', ['data' => $data]);
    }
}
```

And then some kind of import syntax like `HomeController ::: index => HIT_API;` that allows you to import (and thread, if you want to) scopes of code. It's a little meta. Fair.

But `scope` begs questions. Smart, dynamic threading. It doesn't prevent the code from getting executed, all of it still gets ran; it's a `code block type` that gets followed into and interpreted. Interesting.
