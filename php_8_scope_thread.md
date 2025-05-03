### Asynchronous Pattern

```javascript
// source some libs
import { createServer } from 'express'
const port = 3000

// do something asynchronous
process.nextTick(() => {
    const app = createServer(() => {})
    app.listen(port)
})
```
This little Node.js snippet inspired this syntax idea. At the start of the execution of a PHP function you could **prepare** the function for what it's about to execute. It's some pretty dynamic threading control at a per-function level. What if there was a new block type `thread`?

Allowing multiple types of tokens in it like function calls, constants, and objects might facilitate some pretty dope C-level code coordination.

A new block type `scope` could be created with this that would allow some curiously potentially efficient import patterns as well as provide a needed architecture for more complex threaded code.

Let's take a look at this code example:

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

    		TRACK_USER,

    	}

    	scope TRACK_USER {

    		$user->trackPageView();
    		$user->save();

    	}

    	scope HIT_API {

    		$data = hit_api();

    	}

     return $this->render('index', ['data' => $data]);
}
```

And then some kind of import syntax like `HomeController ::: index => HIT_API;`

`scope` begs questions. Smart, dynamic threading.
