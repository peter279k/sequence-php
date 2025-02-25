# PHP Iterator-based sequences

![Packagist PHP Version Support](https://img.shields.io/packagist/php-v/smoren/sequence)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/Smoren/sequence-php/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/Smoren/sequence-php/?branch=master)
[![Coverage Status](https://coveralls.io/repos/github/Smoren/sequence-php/badge.svg?branch=master)](https://coveralls.io/github/Smoren/sequence-php?branch=master)
![Build and test](https://github.com/Smoren/sequence-php/actions/workflows/test_master.yml/badge.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

![MathPHP Logo](docs/images/sequence-php-logo.png)

Python-like sequences with iterators for PHP.

## How to install to your project
```
composer require smoren/sequence
```

## Quick Reference

### Loops
| Functionality                         | Description                  | Code Snippet                                      |
|---------------------------------------|------------------------------|---------------------------------------------------|
| [`Range-based for`](#Range-based-for) | Python-like range-based loop | `foreach(xrange($start, $size, $step) as $value)` |

### Sequences
| Class                                  | Description                     | Code Snippet                                                                |
|----------------------------------------|---------------------------------|-----------------------------------------------------------------------------|
| [`Range`](#Range)                      | Iterable arithmetic progression | `new Range($start, $size, $step)`                                           |
| [`Exponential`](#Exponential)          | Iterable geometric progression  | `new Exponential($start, $size, $step)`                                     |
| [`DynamicSequence`](#Dynamic-Sequence) | Callback-configurable sequence  | `new DynamicSequence($start, $size, $nextValueGetter, $indexedValueGetter)` |

### Data containers
| Class                            | Description              | Code Snippet               |
|----------------------------------|--------------------------|----------------------------|
| [`IndexedArray`](#Indexed-Array) | Python-like indexed list | `new IndexedArray($array)` |

### Functions
| Function            | Description                                                              | Code Snippet                                   |
|---------------------|--------------------------------------------------------------------------|------------------------------------------------|
| [`xrange`](#xrange) | Creates iterable range                                                   | `xrange($start, $size, $step)`                 |
| [`map`](#map)       | Maps iterable collections and returns IndexedArray of mapped values      | `map($mapper, ...$collections)`                |
| [`filter`](#filter) | Filters iterable collection and returning IndexedArray of filtered items | `filter($collection, $filter)`                 |
| [`reduce`](#reduce) | Reduces an iterable collection                                           | `reduce($collection, $reducer, $initialValue)` |

## Usage

### Loops

#### Range-based for

Unlike the PHP built-in function `range()`, `xrange()` does not create an array, but a `Traversable` object 
that takes up a small amount of memory, regardless of the number of elements in the sequence.

```php
use function Smoren\Sequence\Functions\xrange;

foreach(xrange(5) as $i) { // start: 0; count: 5; step: 1
    echo "{$i} ";
}
// 0 1 2 3 4

foreach(xrange(1, 5) as $i) { // start: 1; count: 5; step: 1
    echo "{$i} ";
}
// 1 2 3 4 5

foreach(xrange(1, 5, 2) as $i) { // start: 1; count: 5; step: 2
    echo "{$i} ";
}
// 1 3 5 7 9
```

### Sequences

#### Range

Iterable arithmetic progression.

```new Range(int|float $start, ?int $size, int|float $step)```

For infinite sequence use `$size = null`.

```php
use Smoren\Sequence\Structs\Range;
use Smoren\Sequence\Exceptions\OutOfRangeException;

/* Simple int range */
$range = new Range(1, 3, 2); // (from, size, step)
var_dump($range->isInfinite()); // false

foreach($range as $value) {
    echo "{$value} ";
}
// 1 3 5

var_dump($range[0]); // 1
var_dump($range[1]); // 3
var_dump($range[2]); // 5

try {
    $range[3];
} catch(OutOfRangeException $e) {
    echo "cannot get value from index out of range\n";
}

var_dump($range[-1]); // 5
var_dump($range[-2]); // 3
var_dump($range[-3]); // 1

try {
    $range[-4];
} catch(OutOfRangeException $e) {
    echo "cannot get value from index out of range\n";
}

/* Infinite int range */
$range = new Range(1, null, 2);
var_dump($range->isInfinite()); // true

foreach($range as $i => $value) {
    echo "{$value} ";
    if($i > 100) break;
}
// 1 3 5 7 9 11 13...

/* Float range */
$range = new Range(1.1, 3, 2.1);
var_dump($range->isInfinite()); // false

foreach($range as $value) {
    echo "{$value} ";
}
// 1.1 3.2 5.3
```

#### Exponential

Iterable geometric progression.

```new Exponential(int|float $start, ?int $size, int|float $step)```

For infinite sequence use `$size = null`.

```php
use Smoren\Sequence\Structs\Exponential;
use Smoren\Sequence\Exceptions\OutOfRangeException;

/* Simple int exponential sequence */
$sequence = new Exponential(1, 4, 2); // (from, size, step)
var_dump($sequence->isInfinite()); // false

foreach($sequence as $value) {
    echo "{$value} ";
}
// 1 2 4 8

var_dump($sequence[0]); // 1
var_dump($sequence[1]); // 2
var_dump($sequence[2]); // 4
var_dump($sequence[3]); // 8

try {
    $sequence[4];
} catch(OutOfRangeException $e) {
    echo "cannot get value from index out of range\n";
}

var_dump($sequence[-1]); // 8
var_dump($sequence[-2]); // 4
var_dump($sequence[-3]); // 2
var_dump($sequence[-4]); // 1

try {
    $sequence[-5];
} catch(OutOfRangeException $e) {
    echo "cannot get value from index out of range\n";
}

/* Infinite int exponential sequence */
$sequence = new Exponential(1, null, 2);
var_dump($sequence->isInfinite()); // true

foreach($sequence as $i => $value) {
    echo "{$value} ";
    if($i > 100) break;
}
// 1 2 4 8 16 32 64...

/* Infinite float exponential sequence */
$sequence = new Exponential(0.5, null, 2);
var_dump($sequence->isInfinite()); // true

foreach($sequence as $value) {
    echo "{$value} ";
}
// 0.5 0.25 0.125...
```

#### Dynamic Sequence

Implementation of sequence configured with callables.

```new DynamicSequence(mixed $start, ?int $size, callable $nextValueGetter, ?callable $indexedValueGetter = null)```

For infinite sequence use `$size = null`.

```php
use Smoren\Sequence\Structs\DynamicSequence;

// (from, size, nextValueGetter, indexValueGetter)
$sequence = new DynamicSequence(1, 5, static function($previousValue) {
    return $previousValue + 1;
}, static function($index, $startValue) {
    return $startValue + $index;
});

var_dump(iterator_to_array($sequence));
// [1, 2, 3, 4, 5]
```

### Data containers

#### Indexed Array

Python-like indexed list.

Its keys are always an unbroken sequence of natural numbers starting from zero.

It is also allowed to access array elements from the end with negative indices.

OutOfRangeException will be thrown when trying to access a non-existent index.

```php
use Smoren\Sequence\Structs\IndexedArray;
use Smoren\Sequence\Exceptions\OutOfRangeException;

$array = new IndexedArray([10, 20, 30]);

$array[0] = 11;
$array[-1] = 33;
$array[1] = 22;
var_dump(count($array)); // 3
print_r($array->toArray()); // [11, 22, 33]

unset($array[1]);
print_r($array->toArray()); // [11, 33]

$array[] = 111;
print_r($array->toArray()); // [11, 33, 111]

try {
    $array[3];
} catch(OutOfRangeException $e) {
    echo "cannot get value from index out of range\n";
}

try {
    $array[3] = 1;
} catch(OutOfRangeException $e) {
    echo "cannot set value from index out of range\n";
}

try {
    unset($array[3]);
} catch(OutOfRangeException $e) {
    echo "cannot unset value from index out of range\n";
}
```

### Functions

#### xrange

Creates iterable range.

Works like `xrange()` function in python2 or `range()` function in python3.

```xrange(int $start, ?int $size = null, int $step = 1): Range```

```php
use function Smoren\Sequence\Functions\xrange;

$range = xrange(5);
print_r(iterator_to_array($range));
// [0, 1, 2, 3, 4]

$range = xrange(1, 5);
print_r(iterator_to_array($range));
// [1, 2, 3, 4, 5]

$range = xrange(1, 5, 2);
print_r(iterator_to_array($range));
// [1, 3, 5, 7, 9]
```

#### map

Maps iterable collection and creating IndexedArray of mapped values as a result.

```map(callable $mapper, iterable ...$collections): IndexedArray```

```php
use function Smoren\Sequence\Functions\map;

$ids = [1, 2, 3];
$names = ['Mary', 'Jane', 'Alice'];
$result = map(static function(int $id, string $name) {
    return "{$id}. {$name}";
}, $ids, $names);
print_r($result->toArray());
// ['1. Mary', '2. Jane', '3. Alice']
```

#### filter

Filters iterable collection and returning IndexedArray of filtered items.

```filter(iterable $collection, callable $filter): IndexedArray```

```php
use function Smoren\Sequence\Functions\filter;

$input = [1, 2, 3, 4, 5];
$result = filter($input, static function($item) {
    return $item > 2;
});
print_r($result->toArray());
// [3, 4, 5]
```

#### reduce

Reduces an iterable collection.

```reduce(iterable $collection, callable $reducer, mixed $initialValue = null): mixed```

```php
use function Smoren\Sequence\Functions\reduce;

$input = [1, 2, 3, 4, 5];
$result = reduce($input, static function($carry, $item) {
    return $carry + $item;
}, 0);
var_dump($result);
// 15
```

## Unit testing
```
composer install
composer test-init
composer test
```

## Standards

PHP Sequence conforms to the following standards:

* PSR-1 — [Basic coding standard](https://www.php-fig.org/psr/psr-1/)
* PSR-4 — [Autoloader](https://www.php-fig.org/psr/psr-4/)
* PSR-12 — [Extended coding style guide](https://www.php-fig.org/psr/psr-12/)


## License

PHP Sequence is licensed under the MIT License.
