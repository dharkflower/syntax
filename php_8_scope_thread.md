### light reading

read about [adapt](https://github.com/dharkflower/syntax/blob/main/php_5_adapt.md) to brainstorm potential ways to `adapt` scopes

### heredocs suck

`scope` syntax is similar to heredocs accidentally but heredocs are a PHP visual s***-thorn, thumb twiddle this is awkward and better

### `scope` and `produce`

`scope` sections out blocks you want entered, ran, threaded, TTL'd, and/or imported elsewhere

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

$status = sendMessage();
echo $status; // 'ok'
```

### `enter`

`enter` before `scope` flags the scope to be entered, executed, and exited, not just defined/reused

```php
<?php

function sendMessage () : void {

    // SEND scope is entered and ran
    enter scope SEND {

        // sends something...

    }

    $sendAgain = TRUE;
    if ($sendAgain) {

        SEND;

    }
}

sendMessage();
```

### `thread`

`thread` is an array of `scope` blocks to fork when they run that's defined at the beginning of a function. IPC might help to avoid latency, I'm trying to decrease forking overhead

```php
class IndexController extends Controller {

    #[Route('/index', name: 'index')]
    public function index (

        Incrementer $incrementer

    ) : Response {

        thread {

            // flagged to fork
            INCREMENT

        }

        // enters like an if (TRUE), forked
        enter scope INCREMENT : bool {

            // injected dependency available
            // even if being imported elsewhere
            // good luck, autoloader man
            $incrementer->increment();

            // produces produce type "bool"
            // ignored if not captured
            // like echo vs. print
            produce TRUE;
        }

        // does not enter, like an if (FALSE)
        scope GET_NUMBER : int {

            produce $incrementer->getNumber();

        }

        // but GET_NUMBER is defined and can produce
        return $this->render('index', [

            'number' => GET_NUMBER

        ]);
    }

    #[Route('/coin', name: 'coin')]
    public function coin (

        User $user

    ) : Response {

        // gets cached for two hours
        scope LAST_COIN => 7200 : bool {

             produce (bool) rand(0, 1);

        }

        return $this->render('coin');
    }
}
```

it's meta to have another granularity, fair

I will admit there's other ways to do this in PHP if you admit that even Mr. Clean himself... Mr. Clean himself would drop his stupid, disgusting sponge in shock at the sight of something so fresh; I'd bet on it, all in, every time on that

```php
<?php

namespace App\Controller;

// singular qualifier (index) format
scope App\Controller\IndexController\index ::: {

    INCREMENT
    GET_NUMBER

};

// set a custom TTL using a var
$ttl = 11000;
scope App\Controller\IndexController\coin ::: {

    LAST_COIN => $ttl

}

// multiple qualifier (index, coin) format
scope App\Controller\IndexController {

    index ::: {

        INCREMENT
        GET_NUMBER

    }

    coin ::: {
        
        LAST_COIN => $ttl

    }

};

// maybe be able to pull in qualifiers
scope AppController\IndexController {

    index
    coin

}

// and use them later like this
scope coin ::: LAST_COIN => $ttl;

// or expanded out
scope coin ::: {

    LAST_COIN => $ttl

}

class AnalyticsController extends AbstractController {

    #[Route('/analytics', name: 'analytics')]
    public function analytics (

        Request $request

    ) : Response {

        // in IndexController
        // INCREMENT didn't need to block

        // in AnalyticsController
        // INCREMENT needs to block GET_NUMBER

        // no thread {} block is flagging INCREMENT
        // so INCREMENT runs blocked
        INCREMENT;

        // now that the INCREMENT scope ran
        // you can GET_NUMBER and Twig inject
        return $this->render('analytics', [

            'number' => GET_NUMBER

        ]);
    }
}
```

### brainium radicale
here's some concepts that I think are pretty clean. I extremely like the curly brace thread block being above the function contents but below the injected dependencies

```php
<?php

namespace App\Controller;

class InfoController extends AbstractController {

    #[Route('/info', name: 'info')]
    public function info (

        Request $request

    ) : Response {

        // additional optional curly brace block
        // that's just the thread block but unnamed
        // that accomplishes the same thing, smoothly

        DISPATCH_MESSAGE
    
    } {

        // DISPATCH_MESSAGE flagged to enter and thread
        // so it enters here, forks, moves on
        enter scope DISPATCH_MESSAGE : int {

            $this->logger('message', 'sending message');
            produce 200;

        }

        // programmatically generate a TTL
        $ttl = 3600;

        // every time this runs uncached, it try/catches
        try scope HOURLY_NUMBER => $ttl : int {

            produce rand(1, 10);

        } catch (\Exception $e) {

            // handle error...

        }

        // facts
        $info = [
            'grass is green',
            'sky is blue',
        ];

        // with $str scope param
        scope BOLD (string $str) : string {
            
            produce "<b>$str</b>";

        }

        // or formatted like I want params to be
        scope ITALIC (

            string $str

        ) : string {
            
            produce "<i>$str</i>";

        }

        // strictly just for indexed arrays
        // $info is always a reference
        // array_map on roids
        scope BOLD ($info);

        return $this->render('info', [

            'info' => $info,

        ]);
    }
}
```

### nesting

```php
<?php

scope MAIN : string | bool {

    $message = NULL;

    enter scope MESSAGE : string {

        $message = 'hello';

    }

    scope CHECK : bool {

        // you would need...
        // access to $message
        // lots to think about
        if ($message !== 'hello') {
            
            // check fails
            // wants to exit MAIN so produce 2 up
            produce 2 FALSE;
        }

        // check passes
        // wants to exit CHECK, so produce inferred 1 up
        produce TRUE;
    }

    if (CHECK) {

        produce 'passed check';

    }

    else {

        produce 'did not pass check';

    }
}

$result = MAIN;
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