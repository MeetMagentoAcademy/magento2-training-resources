# Fun with Dependency Injection

Magento 2 uses constructor based dependency injection.
I also comes with it's own implementation of dependency injection container.
In `etc/di.xml` file we can not only specify preferences for specific interfaces:
* we can define plugin methods that can intercept public method executions (altering parameters, return values or even completely skip method execution)
* alter parameters injected to given types, more over define new types that can be injected using existing types with custom attribute configuration 


