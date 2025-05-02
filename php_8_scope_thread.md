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
This simple async pattern inspired this syntax idea. At the start of the execution of a PHP function you could **prepare** the function for what it's about to execute by outlining some details about the components the function will be using. By defining asynchronous elements in advance it might simplify under-the-hood threading. What if there was a new token that would facilitate some low-level performance improvements via some hinting that does stuff like asynchronous curl calls that *autostart* or something using a new token `thread` like this?

```php
<?php

class HomeController extends Controller {

    public function __construct (

        private ContentRepository $repository

    ) {}

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
        User $user

    ) : Response {

        thread {

            // some properties
            $this->repository

            // some function calls
            curl_init,
            curl_setopt,
            curl_exec,
            curl_close,

            // some constants
            CURLOPT_URL,
            CURLOPT_HEADER,
            CURLOPT_RETURNTRANSFER,
            CURLOPT_POST,
            CURLOPT_POSTFIELDS,

        }

        // at this point of the method it already knows a lot about what is *about* to be executed
        $content = $this->repository->loadAll();

        // at this point, it could have already ran some curl logic
        $ch = curl_init();

        curl_setopt($ch, CURLOPT_URL, "http://www.example.com/");
        curl_setopt($ch, CURLOPT_HEADER, 0);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_POST, 1);

        $data = array(
            'username' => 'foo',
            'password' => 'bar'
        );

        curl_setopt($ch, CURLOPT_POSTFIELDS, $data);

        $contents = curl_exec($ch);

        curl_close($ch);

        return $this->render('index', $content);
    }
}
```

Using `thread` could stick to a more strict approach or a more flexible one, allowing multiple types of tokens like function calls, constants, and objects. At a lower level, at the start of a function call when it hits `thread`, it could have certain per-method performance improvements that are made possible by the ability to more effectively manage what's happening. You could even have the `thread` block be required to have each declaration be in order, so that the declarations are accessed sequentially. Sort of like a `thread` array that is a forecast of what order each function is going to be executed in. Here's another example:

```php
<?php

class HomeController extends Controller {

    public function __construct (

        private ContentRepository $repository

    ) {}

    #[Route('/', name: 'index')]
    public function index (

        Request $request,
        User $user

    ) : Response {

        thread {

            // first function call
            str_replace,

            // second function call
            rand,

            // third function call
            json_encode

        }

        $header = str_replace('', '', $request->header1);
        $number = rand(1, 10);
        $json = json_encode([
            'statusCode' => 200
        ]);

        return $this->render('index', $json);
    }
}
```

Okay quick sidequest for a second. Let's talk about an additional token that could be created that would allow some strange yet potentially efficient import patterns: `scope`

If `scope` was used to section out certain parts of the code of a function in a way that would allow the code to be imported elsewhere, it might provide a solid framework that could be combined with `thread`. Let's take a look at this code example:

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

    		TRACK_USER => QUEUE,
    		HIT_API => SYNC

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
}
```

And then some kind of import syntax like `HomeController ::: index => TRACK_USER;`.

`scope` is kind of a weird idea because if you're importing pieces of a function into other functions it begs the question of why certain blocks of code couldn't be optimized on a more granular level - with threading. Smart threading.
