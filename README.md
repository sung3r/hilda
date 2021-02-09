# Hilda

## Description

Hilda is a debugger which combines both the power of LLDB and iPython for easier work. Currently, the project is
intended for iOS debugging, but in the future we will possibly add support for the following platforms as well:

* OSX
* Linux
* Android

Since LLDB allows abstraction for both platform and architecture, it should be possible to make the necessary changes
without too many modifications.

Pull requests are more than welcome 😊.

## Example

![](example.gif)

## Installation

Requirements:

* `iproxy` in PATH (`brew install libimobiledevice`)
* `debugserver` in device's PATH

In order to install please run:

```shell
xcrun python3 -m pip install --user -U -e .
```

*⚠️ Please note that Hilda is installed on top of XCode's python so LLDB will
be able to use its features.*

## How to use

### Commands

In order to view the list of available commands with their documentation, please run the following from within Hilda
shell:

```python
show_commands()
```

### Symbol objects

In Hilda, almost everything is wrapped using the `Symbol` Object. Symbol is just a nicer way for referring to addresses
encapsulated with an object allowing to deref the memory inside, or use these addresses as functions.

In order to create a symbol from a given address, please use:

```python
s = symbol(0x12345678)

# peek(/read) 20 bytes of memory
s.peek(20)

# write into this memory
s.poke('abc')

# jump to `s` as a function, passing (1, "string") as its args 
s(1, "string")

# monitor each time a symbol is called into console and print its backtrace (`bt` option)
s.monitor(bt=True)

s.item_size = 1  # change the size of each item_size inside `s` for derefs
s[0] = 1  # will store the value `1` into *s
s[1] = 1  # will store the value `1` into *(s+1)
```

### Globalized symbols

Usually you would want/need to use the symbols already mapped into the currently running process. To do so, you can
either access them using `symbols.<symbol-name>`. For example:

```python
x = symbols.malloc(20)
```

or just write their name as if they were in the global scope. Hilda will check if no name collision exists, and if so,
will perform the following lazily for you:

```python
x = malloc(20)

# is equivalent to:
malloc = symbols.malloc
x = malloc(20)
```

### Objective-C Classes

The same as symbols applies to Objective-C classes name resolution. You can either:

```python
d = NSDictionary.new()

# which is equivalent to:
NSDictionary = objc_get_class('NSDictionary')
d = NSDictionary.new()
```

As you can see, you can directly access all the class' methods. In order to monitor each time a single one is called,
you can:

```python
objc_class.get_method('methodName:').address.monitor()
```

Viewing the the class layout can be achieved using:

```python
objc_class.show()
```

You can also monitor all method of a given class to see which is called when using:

```python
objc_class.monitor()
```

### Objective-C Objects

In order to work with ObjC objects, each symbol contains a property called
`objc_symbol`. After calling, you can work better with each object:

```python
d = NSDictionary.new().objc_symbol
d.show()  # print object layout
```


