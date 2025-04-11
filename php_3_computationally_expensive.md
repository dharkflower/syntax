### computationally expensive syntax

I'm going to start messing around with some prototypes of how this would work and what the purpose would be. The test I'm going to run will be if I can run multiple PHP threads that listen in a `while(TRUE)` block or something with FPM or CGI at different processor priorities, respectfully. For example, if there was a mechanism that would allow the syntax to work like this, with a new `guess` token:

```php

<?php

$loop1 = 3;
$loop2 = 298342;

guess 3 {
  foreach ($i = 0; $i < $loop1; $i++) {
    $unitOfWork = $i * 2;
  }
}

guess 298342 {
  foreach ($i = 0; $i < $loop2; $i++) {
    $unitOfWork = $i * 2;
  }
}

```

In this example, the code block after the `guess 3` token flags the PHP process to run the block of code at a lower process priority than the code block after `guess 298342`. If there was a way to pass a PHP thread to a different, already instantiated PHP thread running at a different priority it might allow for better utilization of resources.

In the above example it's using static numbers, which ruins the point of this functionality. So here's a more complex example:

```php

<?php

$loop1 = rand(1, 10);
$loop2 = rand(100, 1000);

guess 5 {
  foreach ($i = 0; $i < $loop1; $i++) {
    $unitOfWork = $i * 2;
  }
}

guess 500 {
  foreach ($i = 0; $i < $loop2; $i++) {
    $unitOfWork = $i * 2;
  }
}

```

As the script grows in complexity the ability for a PHP script to determine what priority to run the process at gets blurry; `$loop1` and `$loop2` are random numbers now. How computationally expensive it will be will be different every time this script is run. However, if you use `guess` to say "this will be roughly 5 computationally expensive" and "this will be roughly 500 computationally expensive" then the script might be able to better know how to handle each block.
