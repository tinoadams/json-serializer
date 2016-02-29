# Json Serializer for PHP

[![Build Status](https://travis-ci.org/zumba/json-serializer.png)](https://travis-ci.org/zumba/json-serializer)
[![Code Coverage](https://scrutinizer-ci.com/g/zumba/json-serializer/badges/coverage.png?s=56e61922c00f25b9afae3e97af853f3eb68d9c1a)](https://scrutinizer-ci.com/g/zumba/json-serializer/)
[![Scrutinizer Quality Score](https://scrutinizer-ci.com/g/zumba/json-serializer/badges/quality-score.png?s=16511f820e0c53cdcbbbc62b5de07d493ded1181)](https://scrutinizer-ci.com/g/zumba/json-serializer/)

This is a library to serialize PHP variables in JSON format. It is similar of the `serialize()` function in PHP,
but the output is a string JSON encoded. You can also unserialize the JSON generated by this tool and have you
PHP content back.

Supported features:
- Encode/Decode of scalar, null, array
- Encode/Decode of objects
- Support nested serialization
- Support not declared properties on the original class definition (ie, properties in `stdClass`)
- Support object recursion
- Closures (via 3rd party plugin. See details below)

Unsupported serialization content:
- Resource (ie, `fopen()` response)
- Binary String or malformed UTF8 strings (ie, resulsts from `SELECT AES_ENCRYPT(:content, :key) as encrypted`)
	- These strings will need to be properly handled by converting to hex using `bin2hex` or `utf8_encode` in the `__sleep()` method

This project should not be confused with `JsonSerializable` interface from PHP 5.4. This interface is used on
`json_encode` to encode the objects. There is no unserialization with this interface, differently from this project.

*Json Serializer requires PHP >= 5.4*

## Example

```php

class MyCustomClass {
	public $isItAwesome = true;
	protected $nice = 'very!';
}

$instance = new MyCustomClass();

$serializer = new Zumba\Util\JsonSerializer();
$json = $serializer->serialize($instance);
// $json will contain the content {"@type":"MyCustomClass","isItAwesome":true,"nice":"very!"}

$restoredInstance = $serializer->unserialize($json);
// $restoredInstance will be an instance of MyCustomClass
```

## How to Install

If you are using composer, install the package [`zumba/json-serializer`](https://packagist.org/packages/zumba/json-serializer).

```
$ composer require zumba/json-serializer
```

Or add the `zumba/json-serializer` directly in your `composer.json` file.

If you are not using composer, you can just copy the files from `src` folder in your project.

## Serializing Closures

For serializing PHP closures you have to pass an object implementing `SuperClosure\SerializerInterface`.
This interface is provided by the project [SuperClosure](https://github.com/jeremeamia/super_closure). This
project also provides a closure serializer that implements this interface.

Closure serialization has some limitations. Please check the SuperClosure project to check if it fits your
needs.

To use the SuperClosure with JsonSerializer, just pass the SuperClosure object as the first parameter
on JsonSerializer constructor. Example:

```php
$toBeSerialized = array(
	'data' => array(1, 2, 3),
	'worker' => function ($data) {
		$double = array();
		foreach ($data as $i => $number) {
			$double[$i] = $number * 2;
		}
		return $double;
	}
);

$superClosure = new SuperClosure\Serializer();
$jsonSerializer = new Zumba\Util\JsonSerializer($superClosure);
$serialized = $jsonSerializer->serialize($toBeSerialized);
```

PS: JsonSerializer does not have a hard dependency of SuperClosure. If you want to use both projects
make sure you add both on your composer requirements.
