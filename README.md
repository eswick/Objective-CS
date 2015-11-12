# The Objective-CS Language Specification Version 1 Revision 1

This document provides the specification for the Objective-CS language.

Objective-CS was inspired by the [Logos](https://github.com/DHowett/theos/blob/master/bin/logos.pl) preprocessor originally by [Dustin Howett](http://github.com/DHowett). Objective-CS improves upon Logos and dramatically increases the simplicity and readability of tweaks and other programs.

Objective-CS is a direct superset of Objective-C and Objective-C++, therefore all code valid in Objective-C and Objective-C++ is valid in Objective-CS.

Objective-CS adds the following new features to Objective-C:

- Method Hooking
- Adding Methods at Runtime
- Adding Properties at Runtime

Other language features:
- Direct Instance Variable Access
- Hook Groups

## Method Hooking

Objective-CS provides functionality for hooking methods. Methods are hooked by defining a method with the same selector as the method to be hooked inside a hook implementation. A hook implementation is defined by the `@hook` directive, followed by the class name. A hook implementation must end with the `@end` directive. For example:

```
@hook NSObject

- (NSString*)description {
  return @"Hooked!";
}

@end
```

To define a hook implementation, a corresponding interface declaration must first be defined.

### `@orig`

The original implementation of a method can be called by using the `@orig` directive. `@orig` is followed by an argument list enclosed in parenthesis. For example:

```
@hook NSObject

- (NSString*)description {
  return [@orig() stringByAppendingString:@"Hooked!"];
}

@end
```

### `super`
`super` can be used inside a method just like it is used in a normal class implementation to call the superclass' implementation of a method.

```
@interface ExampleClass : NSObject
@end

@implementation ExampleClass : NSObject

- (NSString*)description {
  return @"New Description!";
}

@end

@hook ExampleClass

- (NSString*)description {
  return [super description]; // Returns "<ExampleClass 0x123456>"
}

@end

```


## Adding Methods at Runtime

Methods can be added to classes at runtime by defining them in a class extension below an `@new` directive. Their implementations can then be defined in a hook implementation. For example:

```
@interface NSObject ()
@new
- (void)newMethod;
@end

@hook NSObject

- (void)newMethod {
  NSLog(@"newMethod called!");
}

@end

```

## Adding Properties at Runtime

Properties can be added to classes at runtime by defining them in a class extension, and then using the `@synthesize` directive inside a hook implementation. For example:

```
@interface NSObject ()
@property (nonatomic, retain) NSString *newProperty;
@end

@hook NSObject
@synthesize newProperty;

- (id)init {
  self = @orig();
  if (self) {
    self.newProperty = @"Hello, world!";
    NSLog(@"%@", [self newProperty]);
  }
  return self;
}

@end
```

When synthesizing a property inside a hook implementation, instance variable names can *not* be specified, as Objective-CS does not define an instance variable for new properties, only getters and setters. The following code is *invalid*:

```
@interface NSObject ()
@property (nonatomic, retain) NSString *newProperty;
@end

@hook NSObject
@synthesize newProperty = _newProperty; // Invalid!
@hook
```

## Direct Instance Variable Access

Methods inside hook implementations have direct access to instance variables. The offset of instance variables are looked up at runtime, which allows access to instance variables even if their definition in the interface is out-of-order.

```
@interface ExampleClass : NSObject {
  NSString *_exampleIvar;
}
- (void)testMethod;
@end

@hook ExampleClass

- (void)testMethod {
  _exampleIvar = @"Example!";
  @orig();
}

@end
```

## Hook Groups

Hook implementations can be defined inside groups. A group is defined by the `@group` directive, followed by the group name. A group definition must end with the `@end` directive.

By default, hook implementations inside groups are not initialized automatically at runtime. They must be manually initialized by using the `@init` directive. This is commonly done in the constructor.

```
@hook UIScreen

- (CGRect)bounds { // Initialized automatically
  return CGRectMake(0, 0, 100, 100);
}

@end

@group SpringBoard
@hook UIApplication

- (void)applicationDidFinishLaunching:(id)application { // Initialized in the constructor
  @orig(application);
  NSLog(@"SpringBoard did finish launching!");
}

@end
@end

__attribute__((constructor))
static void ctor() {
  if ([[[NSBundle mainBundle] bundleIdentifier] isEqualToString:@"com.apple.springboard"]) {
    @init(SpringBoard); // Initialize the SpringBoard group
  }
}
```

# Final Notes

As you can see, Objective-CS is a *huge* improvement over the Logos preprocessor. By extending the compiler directly instead of using a preprocessor, we gain some extremely powerful features. The language will continue to evolve and more features will be added over time. A compiler for Objective-CS, titled `clang-objcs` is in the process of being developed and will be available soon!
