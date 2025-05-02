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
This pattern inspired this syntax idea. At the start of the execution of a PHP function you could **prepare** the function for what it's about to execute. By defining threaded elements in advance it might simplify under-the-hood threading and allow some dynamic threading control at a per-function level. What if there was some hinting that does stuff like asynchronous curl calls that *autostart* or something using a new token `thread` like this?

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
            $this->repository,

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

        return $this->render('index');
    }
}
```

Allowing multiple types of tokens like function calls, constants, and objects might facilitate some pretty dope C-level code coordination, but you could take a simpler approach and stick to sequential.

```php
<?php

class HomeController extends Controller {

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

        return $this->render('index');
    }
}
```
### Scope
Let's talk about an additional token that could be created that would allow some curiously potentially efficient import patterns: `scope`

If `scope` was used to section out certain parts of the code of a function in a way that would allow the code block to be imported elsewhere, it might provide a solid framework that could be combined with `thread`. Let's take a look at this code example:

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

`scope` begs the question of why certain blocks of code couldn't be either threaded or ran async depending on how they are configured. Smart, dynamic threading.
