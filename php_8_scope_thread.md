### light reading

read about [adapt](https://github.com/dharkflower/syntax/blob/main/php_5_adapt.md) to brainstorm potential ways to `adapt` scopes

### heredocs suck

`scope` syntax is similar to heredocs accidentally but heredocs are a PHP visual s***thorn... thumb twiddle this is awkward and better

### `scope` and `produce`

`scope` sections out blocks you want entered and ran, threaded, TTL'd, and/or imported elsewhere

`produce` is just `return` but for `scope`, `produce` does close the current scope and "returns" a value from it but does not return from the parent function: `sendMessage`

`scope` supports "produce types" like return types, C-level helpful for sure

```php
<?php

function sendMessage () : string {

    scope SEND : bool {

        produce TRUE; // closes SEND

    }

    $result = SEND; // TRUE

    if ($result) {

        return 'ok'; // closes sendMessage

    }

    return 'not ok'; // closes sendMessage

}

$status = sendMessage(); // 'ok'
```

### `enter`

`enter` before `scope` flags the scope to be both entered/ran **and** defined/reused:

```php
<?php

function sendMessage () : void {

    // SEND scope is entered and ran
    enter scope SEND {

        // sends something...

    }

    $sendAgain = TRUE;
    if ($sendAgain) {

        SEND; // SEND scope is reusable

    }
}

sendMessage();
```

### `thread`

`thread` is an array of `scope` blocks to fork defined at the beginning of a function. It could be associative... hmm. I have some ideas on how this could work using IPC to avoid the latency I'm trying to decrease but this is the PHP side:

```php
class IndexController extends Controller {

    #[Route('/', name: 'index')]
    public function index (

        Incrementer $incrementer

    ) : Response {

        thread {

            // flagged to fork when `index` runs
            INCREMENT

        }

        // `enter` token
        // enters like an if (TRUE)
        enter scope INCREMENT : bool {

            // injected dependency available
            // even if being imported elsewhere
            // good luck, autoloader man
            $incrementer->increment();

            // produces produce type "bool"
            // ignored if not captured
            produce TRUE;
        }

        // no `enter` token
        // defines but does not enter
        // like an if (FALSE)
        scope GET_NUMBERS : array {

            produce [4, 5];

        }

        // GET_NUMBERS is defined and can produce
        return $this->render('index', [
            'numbers' => GET_NUMBERS,
        ]);
    }

    #[Route('/coin', name: 'coin')]
    public function coin (

        User $user

    ) : Response {

        // gets cached for two hours
        scope COIN_FLIP => 7200 : bool {

             produce (bool) rand(0, 1);

        }

        return $this->render('coin');
    }
}
```

It's meta to have another granularity but.

I will admit this is similar to utilization of threading classes if you admit that even Mr. Clean himself... Mr. Clean himself would drop his stupid, disgusting sponge in shock at the sight of something so fresh; I'd bet on it, all in, every time on that

```php
<?php

namespace App\Controller;

// singular qualifier (index) format
scope App\Controller\IndexController\index ::: {

    INCREMENT
    GET_NUMBERS

};

// set a custom TTL using a var
$ttl = 11000;
scope App\Controller\IndexController\coin ::: {

    COIN_FLIP => $ttl

}

// multiple qualifier (index, coin) format
scope App\Controller\IndexController {

    index ::: {

        INCREMENT
        GET_NUMBERS

    }

    coin ::: {
        
        COIN_FLIP => $ttl

    }

};

// maybe be able to pull in qualifiers
scope AppController\IndexController {

    index
    coin

}

// and use them later like this
scope coin ::: COIN_FLIP => $ttl;

// or expanded out
scope coin ::: {

    COIN_FLIP => $ttl

}

class AnalyticsController extends AbstractController {

    #[Route('/analytics', name: 'analytics')]
    public function analytics (

        Request $request

    ) : Response {

        // in IndexController
        // INCREMENT didn't need to block

        // in AnalyticsController
        // INCREMENT needs to block GET_NUMBERS

        // no thread {} block is flagging it to fork
        // so the scope runs synchronously
        INCREMENT;

        // now that the INCREMENT scope ran
        // you can GET_NUMBERS and Twig inject
        return $this->render('analytics', [
            'numbers' => GET_NUMBERS,
        ]);
    }
}
```

### brainium radicale
here's some more radical concepts that I think are actually pretty clean. I extremely like the thread block being above the function contents, it could normalize and mainstream super low-level ZTS thread-safe approaches

```php
<?php

namespace App\Controller;

class InfoController extends AbstractController {

    #[Route('/info', name: 'info')]
    public function info (

        Request $request,
        User $user

    ) : Response {

        // additional optional curly brace block
        // that's just the thread block but unnamed
        // that accomplishes the same thing smoothly

        DISPATCH_MESSAGE
    
    } {

        // DISPATCH_MESSAGE is flagged to enter, thread
        // so when it gets here it forks
        enter scope DISPATCH_MESSAGE : int {

            $this->logger('message', 'sending message');
            produce 200;

        }

        // programmatically generate a TTL based on need
        $ttl = 3600;

        // every time this runs uncached, it "tries"
        try scope HOURLY_NUMBER => $ttl : int {

            // random number each hour
            produce rand(1, 10);

        } catch (\Exception $e) {

            // handle error...

        }

        return $this->render('info', [

            'info' => []

        ]);
    }
}
```

### nesting

```php
<?php

scope MAIN : string | bool {

    scope GET : string {

        $message = 'hello';
        produce $message;

    }

    scope CHECK : bool {

        // you would need to pass in a parameter
        // to get access to $message
        // still thinking about parameters, hmm...
        if ($message !== 'hello') {
            
            // check fails
            // wants to exit MASTER so produce 2 up
            produce 2 FALSE;
        }

        // check passes
        // wants to exit CHECK, so produce inferred 1 up
        produce TRUE;
    }

    $result = GET;

    if (CHECK) {

        produce 'passed check';

    }

    else {

        produce 'did not pass check';

    }
}
```

### Node.js snippet that inspired this

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