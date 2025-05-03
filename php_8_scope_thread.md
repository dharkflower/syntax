### Asynchronous Pattern

```javascript
// source some libs
import { createServer } from 'express'
import React from 'react'

// do something asynchronous
process.nextTick(() => {
    const app = createServer(() => {})
    app.listen(3000)
})
```
This Node.js snippet inspired this syntax idea. At the start of the execution of a PHP function you could **prepare** the function for what it's about to execute. By defining threaded scopes in advance it might allow some pretty dynamic threading control at a per-function level. What if it did stuff like a curl post that *autostarts* and then waits or something using a new block type `thread`?

Allowing multiple types of tokens in it like function calls, constants, and objects might facilitate some pretty dope C-level code coordination, but you could take a simpler approach like this:

```php
<?php

class HomeController extends Controller {

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
        User $user

    ) : Response {

        thread {

            str_replace,
            rand,

        }

        $header = str_replace('', '', $request->header1);
        $number = rand(1, 10);
        $json = json_encode([
            'data' => 'ok'
        ]);

        return json_encode($json);
    }
}
```
### Scope
Let's talk about an additional code block type, though, that could be created that would allow some curiously potentially efficient import patterns and potentially provide a needed architecture for more complex threading patterns: `scope`

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
