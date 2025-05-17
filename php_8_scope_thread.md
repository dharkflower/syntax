### light reading

check out [trace](https://github.com/dharkflower/syntax/blob/main/php_7_trace.md) if you want to read about compiling potential

read about [adapt](https://github.com/dharkflower/syntax/blob/main/php_5_adapt.md) if you want to brainstorm some potential ways to adapt TTL's and stuff

### heredoc

scope syntax is the same as heredoc. I don't want to change it because I simply don't like heredoc, period

code UX tells me they suck

i'm not going to change it because heredoc should be deprecated and this should take its place

lots to think about

### tokens

what if at the start of a PHP function you could **prepare** it for what it's about to execute?

new tokens: **`thread`, `scope`, `produce`, `dead`**

### `thread` is an array

`thread` is an array of scopes that has to be defined at the start of a function

if `thread` was upgraded to an associative array it could become insanely helpful to kind of take control of some low level stuff with minimal syntax

oh and uh if it allowed multiple tokens in it like TTL's, something with `yield`, functions, statics, constants, variables, use class imports, environment variables, hell shell commands... PHP might be able to more efficiently analyze and coordinate using some pretty dope C-level code

### `produce` is just `return` but for `scope`

to produce a `scope` into a variable you call the scope kind of like a function, like this:

```php
<?php

function sendMessage () : bool {

    scope SEND {
        produce TRUE;
    }

    $sent = SEND;
    return $sent;

}

$messageSent = sendMessage();
```

the only point of `produce` is to have a clear syntax distinction between `returning from a function` and `producing from a scope` because they would hypothetically be used in pretty tight unison

`produce` is ignored if it doesn't stdout into something

it most definitely does not exit out of the next generation up function, just the scope it's in

produce is a little limited here because of nesting, I'll circle back on this but this is my best idea for nesting so far:

```php
<?php

scope MAIN : string | bool {

    scope GET : int {

        $i = 1 + 1;
        produce $i;

    }

    scope CHECK : bool {

        // you would need to pass in a parameter to get $i
        // still thinking about parameters, hmm...
        if ($i !== 2) {
            
            // check fails
            // wants to exit MASTER so produce 2
            produce 2 FALSE;
        }

        // check passes
        // wants to exit CHECK, so produce inferred 1
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

MAIN;
```

### `scope` sections out code

the point of scopes is to section out a block you want either threaded, TTL'd, or imported elsewhere

`scope` has optional "produce types" that follow the same syntax as return types that could be helpful, as well as a default TTL syntax that's double-arrowed

### `dead` skips the scope for now

because it will have been dead before it ever even started, to be alive - the block of code

```php
class IndexController extends Controller {

    #[Route('/', name: 'index')]
    public function index (

        User $user

    ) : Response {

        thread {

            // neither of these need to block UX
            TRACK_GENERIC_VIEW
            TRACK_INDEX_VIEW

        }

        // enters here like it was an if (TRUE)
        scope TRACK_GENERIC_VIEW : bool {

            // $user injected dependency available
            // even if being imported elsewhere
            $user->trackGenericView();
            $user->save();

            // produces bool
            produce TRUE;

            // continues...
        } //// continues past curly brace...

        // enters here like it was an if (TRUE)
        scope TRACK_INDEX_VIEW : void {

            // ...
            $user->trackIndexView();
            $user->save();

            // does not produce, hence void
            // ...

            // continues...
        } //// continues past curly brace...

        // dead but importable/executable scope elsewhere
        // does not enter here, like it was an if (FALSE)
        dead scope GET_GENERIC_VIEWS : array {

            produce [4, 5];

            // continues...
        } //// continues past curly brace...

        // GET_GENERIC_VIEWS can still produce no worries
        return $this->render('views', [
            'views' => GET_GENERIC_VIEWS,
        ]);
    }

    #[Route('/other', name: 'other')]
    public function other (

        User $user

    ) : Response {

        dead scope COIN_FLIP => 7200 : bool {

             // flips coin every two hours
             produce (bool) rand(0, 1);
        }

        return $this->render('other');
    }
}
```

It's meta to have another granularity but these tokens enable smart, dynamic, low-level threading and code reusability, TTL caching, all kinds of weird stuff; they do have a point

I will admit some of this is close to just being a type of function or utilization of a threading class if you admit that even Mr. Clean himself... Mr. Clean himself would drop his stupid, disgusting sponge in shock at the sight of something so fresh; I'd bet on it, all in, every time on that

Full snippet.

```php
<?php

namespace App\Controller;

// singular format idea, my vote
scope App\Controller\IndexController\index ::: {

    TRACK_GENERIC_VIEW
    GET_GENERIC_VIEWS

};

// one for each method, pretty clean var but functional
// set a custom TTL for coin flipping
$ttl = 11000;
scope App\Controller\IndexController\other ::: {

    COIN_FLIP => $ttl

}

// multiple format idea, my vote also, stronger
scope App\Controller\IndexController ::: {

    index ::: {

        TRACK_GENERIC_VIEW
        GET_GENERIC_VIEWS

    }

    other ::: {
        
        COIN_FLIP => $ttl

    }

};

// so maybe be able to pull in multiple qualifiers
scope AppController\IndexController ::: {

    index
    other

}

// and use them later like this
scope other ::: COIN_FLIP => $ttl;

// or expanded, by qualifier, like this
scope other ::: {

    COIN_FLIP => $ttl

}

class AnalyticsController extends AbstractController {

    #[Route('/analytics', name: 'analytics')]
    public function analytics (

        Request $request

    ) : Response {

        // an example of synchronous and asynchronous
        // in the IndexController,
        // TRACK_GENERIC_VIEW didn't need to block
        // in the AnalyticsController,
        // TRACK_GENERIC_VIEW needs to block
        // two different usages of code in two places
        // drop-in use a scope synchronously
        // because no thread {} block is defined
        TRACK_GENERIC_VIEW;

        // now that the TRACK_GENERIC_VIEW scope ran
        // you can get the generic views
        // inject the full views query res into Twig
        return $this->render('analytics', [
            'views' => GET_GENERIC_VIEWS,
        ]);
    }
}
```

### brainium radicale
here's some more radical concepts that I think are actually pretty clean, honestly; they're pretty huge 

```php
<?php

namespace App\Controller;

class PhpInfoController extends AbstractController {

    #[Route('/phpinfo', name: 'request')]
    public function phpinfo (

        Request $request,
        User $user

    ) : Response {

        // additional curly brace block
        // that accomplishes the same thing as `thread`
        TRACK_GENERIC_VIEW
        DISPATCH_MESSAGE
    
    } {

        // TRACK_GENERIC_VIEW is marked to thread
        // so when it gets here it forks
        scope TRACK_GENERIC_VIEW : bool {

            $user->trackGenericView();
            $user->save();

            produce TRUE;

        } //// continues post-fork past curly brace...
        ////// forgets about it

        // DISPATCH_MESSAGE is marked to thread
        // so when it gets here it forks
        scope DISPATCH_MESSAGE : StatusCode {

            $this->logger('message', 'sending message');
            produce 200;

        } //// continues post-fork past curly brace...
        ////// forgets about it

        // every time this runs, it tries
        try dead scope HOURLY_NUMBER => $ttl : int {

            // random number each hour
            produce rand(1, 10);

        } catch (\Exception $e) {

        } finally {

        }

        return $this->render('phpinfo', [

            'info' => phpinfo()

        ]);
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