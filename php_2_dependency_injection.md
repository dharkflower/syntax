This is how dependency injection should look in a PHP Class. With a large number of dependencies being injected it makes sense to break them to their own line instead of horizontal.

```php
<?php

class MyClass {

  private function goodExample ( // break to next line

      // indent like you would a Python script
      User $user,
      Session $session,
      EntityManager $entityManager,
      EntityRepository $entityRepository

      /*
      ^
      | psychological line
      */

      // return type is visually in line with the indention of the dependencies
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

  private function otherExample (

      User $user,
      Post $post

  ) : Post {

      // bad
      $uid = $user->getUid();
      $post->setUid($uid, TRUE, FALSE, 3);
  
      // good
      $post->setUid( // break to new line
  
          // I vote to normalize breaking arguments to a new line, just like the __construct function
          $user->getUid(),
  
          /*
          ^
          | psychological line
          */
  
          TRUE,
  
          FALSE,
  
          3
  
      );

      return $post;

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
      class="
          bg-light
          container-fluid
          p-3
      "
      data-index="0"
      data-element="navigation"

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

      ><!-- -->

      /*
      ^
      | psychological line
      */

      </span>
    </a>

  </body>

</html>
```
