This is how dependency injection should look in a PHP Class. With a large number of dependencies being injected it makes sense to break them to their own line instead of horizontal.

```php
<?php

class MyClass {

  public function __construct (
  ) {
  }

  private function goodExample ( // break to next line

      // indent like you would a Python script
      private User $user,
      private Session $session,
      private EntityManager $entityManager,
      private EntityRepository $entityRepository

      /*
      ^
      | psychological line
      */

      // return type is visually in line with the indention of the dependencies

      /*
      ^
      | psychological line
      */

      // lines up with the left side of the return type - "string"
  ) : string { // space on both sides of the :

      /*
      ^
      | psychological line
      */

      return 'ok';

  }

  // instead of this
  private function badExample(User $user, Session $session, EntityManager $entityManager, EntityRepository $entityRepository): string {
      return 'ok';
  }

}
```
