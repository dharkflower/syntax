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

      // lines up with the left side of the return statement - "return"
      return 'ok';

  }

  // instead of this
  private function badExample(User $user, Session $session, EntityManager $entityManager, EntityRepository $entityRepository): string {
      return 'ok';
  }

}
```

This concept can also be applied to HTML:

```html
<html>

  <head>
  </head>

  <body>

    <a // break to next line

      href=""
      id="navigation"
      data-index="0"
      data-element="navigation

      /*
      ^
      | psychological line
      */
      
    ><!-- --> // visually end the tag a little bit more to the right with an empty comment

      /*
      ^
      | psychological line
      */

      <span

        /*
        ^
        | psychological line
        */

        class="list-item"
        data-index="1"
        data-color="blue"

        /*
        ^
        | psychological line
        */

      >

      /*
      ^
      | psychological line
      */

      </span>
    </a>

  </body>

</html>
```
