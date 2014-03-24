# Symfony AutoInject

[![Build Status](https://travis-ci.org/heyday/symfony-autoinject.svg?branch=master)](https://travis-ci.org/heyday/symfony-autoinject)

Symfony AutoInject provides opt-in service auto-injection for Symfony Dependency Injection.

## Installation (with composer)

	$ composer require heyday/symfony-autoinject

## Usage

Opting into auto-injection is achieved via two service tags "autoinject" and "autoinject.provides"

The tags are used as follows:

### "autoinject"

Example: `{ name: autoinject }`

Example: `{ name: autoinject, arguments: true, setter: true, adder: true }`

When "all" is set to true or when no extra arguments are provided, the below settings all apply

When "arguments" is set to true the compiler pass will attempt to find services for the arguments, and provides if
they are found and error if not. Parameters that can't be auto-injected need to be provided manually.

Both "setter", "arguments" and "adder" are optional

When "setter" is set to true, the compiler pass will attempt to find provided services that match
the argument of setters found on the class

When "adder" is set to true the compiler pass will attempt to find services that match the
argument of adder methods found of the class, and it will add a method call to the adder
for each service found


### "autoinject.provides"

Example: `{ name: autoinject.provides }`

Example: `{ name: autoinject.provides, interfaces: true, classes: true }`

When "all" is set to true or when no extra arguments are provided, the below settings all apply

Both "interfaces" and "class" are optional

When "interfaces" is set to true, the compiler pass will register the service as providing an instance of
all interfaces that the class implemented, so when interfaces are encountered in arguments, setters and adders;
the provided service will be supplied

When "classes" is set to true, the compiler pass will register the service as providing an instance of
that class and parent classes, so when a class of the same type is encountered in a arguments, setter or add; the provided
service will be supplied

## Putting it all together

### Constructor injection

The classes

```php
class Service
{
}

class Service2
{
	protected $service;
	public function __construct(Service $s)
	{
		$this->service = $s;
	}
}
```

The configuration

```yml
services:
	my_service:
		class: Service
		tags:
			- { name: "autoinject.provides" }
			
	my_service2:
		class: Service2
		tags:
			- { name: "autoinject" }
```

In this example, `my_service` provides itself as a service instance of the class `Service` and `my_service2`
requests auto-injection.

These tags result in the compiler pass seeing that `my_service2` is requesting auto-injection, that it has a constructor
that expects an instance of `Service` and that there is a instance of `Service` provided by `my_service`. The auto-injection
is resolvable and the container is configured to provide `my_service` to `my_service2` via constructor injection.

## Unit testing

    $ composer install --dev
    $ vendor/bin/phpunit