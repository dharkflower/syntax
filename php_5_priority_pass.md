This would need to be written in C, in php-src. Potentially an extension would work too. Pretend some of this is C code. This example has `pcntl` as a dependency.

Let's imagine some kind of simple web server like this. It's untested and it's not perfect but you'll see the point, it spawns a bunch of PHP threads using `pcntl` that all run on different ports at different process priorities:

```php
<?php

$multiplier = 10;
$max = 100;

foreach ($i = 1; $i * $multiplier <= $max; $i++) {

  $pid = pcntl_fork();

  if ($pid == -1) {
  
    die('could not fork');
  
  } else if ($pid) { // we are the parent thread

    pcntl_wait($status);

  } else { // we are a child thread

    // set the port
    $port = 3000 + $i;

    // set the priority for this thread/port
    pcntl_setpriority($i * $multiplier);

    while (TRUE) {

      listen($port, function(Request $request) {

        require_once 'autoload.php';

      });

    }
  }
}

```

An example of what this does could be explained with a setup using Apache2 - some kind of reverse proxy that would divert requests to specific ports based on desired priority. It's a little strange. But does that make sense?

This could potentially be handled natively by PHP. What if there was a new syntax token called `migrate` that worked something like this?:

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

  ) {

    // migrate this thread to the thread listener running at priority 50
    migrate 50;

    // this line runs at priority 50 on port 3005
    $content = $this->repository->findAll();

    // migrate this thread to the thread listener running at priority 10
    migrate 10;

    // this line runs at priority 10 on port 3001
    $json = json_encode([
      'statusCode' => 200
    ]);

    // migrate this thread to the thread listener running at default priority
    migrate default;

    // this line runs at the default priority on port 3000
    return $this->render('index', [
      'json' => $json,
      'content' => $content
    ]);
  }
}
```

It would essentially allow you to make tiny little micro-suggestions of where to run more granular parts of PHP, if you could figure out how to have a single request be able to access the same request headers and information from different threads.

It could potentially be used as a property as well like this:

```php
<?php

class HomeController extends Controller {

  migrate 50;

  public function __construct (

    private ContentRepository $repository
  
  ) {}

  #[Route('/', name: 'index')]
  public function index (

    Request $request,
    User $user

  ) {

    // this all runs at 50...

    $content = $this->repository->findAll();

    $json = json_encode([
      'statusCode' => 200
    ]);

    return $this->render('index', [
      'json' => $json,
      'content' => $content
    ]);

    //...

  }
}
```

It could potentially use UNIX sockets instead of TCP sockets as well. It would have to track where that process's state is in RAM but if it could pass that identifier from thread to thread it might work.
