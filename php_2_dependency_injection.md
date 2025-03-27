This is how dependency injection should look in a PHP Class:

```php
<?php

class MyClass {

  public function __construct (
  ) {
  }

  private function inject ( // break to next line

      // indent like you would a Python script
      private User $user,
      private Session $session

      /*
      ^
      | psychological line
      */

      // return type is visually in line with the indention of the dependencies

      /*
      ^
      | psychological line
      */

      // lines up with return type "string"
  ) : string { // space on both sides of the :
    
  }

}
```
