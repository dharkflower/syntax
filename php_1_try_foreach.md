```php

try foreach ($items as $item) {

  // if $items is falsey, something's wrong but it's not a big component so log it and move past it
  // it should throw naturally but not break the whole application
  // but it allows the syntax to make a little more sense

} catch (\Exception $e) {
  
}

###

foreach ($items as $item) try {

} catch() {

}

```

