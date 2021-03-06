# Ts-Enums

A TypeScript library for enums, inspired by Java enums and forked from [Enumify](https://github.com/rauschma/enumify).

## Motivation

Dr. Axel Rauschmeyer created [Enumify](https://github.com/rauschma/enumify) to bring an analogue of Java's Enums to JavaScript. In a [blog post](http://2ality.com/2016/01/enumify.html), he explained why a naive implementation of enums in JavaScript lacks the power of Java Enums.

[TypeScript Enums](https://www.typescriptlang.org/docs/handbook/enums.html) provide a start to a Java-style Enum implementation, but they lack the killer feature of Java's Enum: classes. [Java's Enums](https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html) are full classes, enabling properties and methods (including abstract methods and overriding within an enum). In contrast, TypeScript enums are limited to being namespaced integers or strings.

[Enumify](https://github.com/rauschma/enumify) is a strong introduction to class-style enums in JavaScript, and this project ports the idea to TypeScript. This port loses the feature from Enumify of creating an enumeration from an array of Strings, but string-based TypeScript enums were added in 2.4.1, so that is not a large loss.


## The basics

Install:

```text
"dependencies": {
    "ts-enums": "git+https://github.com/rawlinxx/ts-enums.git#master"
}
```

Basic:

```typescript
import { Enum, EnumValue, Payload, Desc } from 'ts-enums';

class ColorEnum extends Enum<Desc> {

  red = new Desc('it is RED');
  green = new Desc('it is GREEN');
  blue = new Desc('it is BLUE');

  constructor() {
    super();
    this.initEnum();
  }
}

export const colorEnum = new ColorEnum();

// examples of use
console.log(colorEnum.red.propName); // 'red'
console.log(colorEnum.red.ordinal); // 0
console.log(colorEnum.red.description); // 'it is RED'
console.log(colorEnum.red.toString()); // 'red'

colorEnum.values

```

With payload:

```typescript

class ColorEnum extends Enum<Payload<{ hex: string }>> {

  red = new Payload('it is RED', { hex: '0xffffff' });
  green = new Payload('it is GREEN', { hex: '0xcccccc' });
  blue = new Payload('it is BLUE', { hex: '0xbbbbbb' });

  constructor() {
    super();
    this.initEnum();
  }
}

```


## Properties of Enum classes

Enum exposes the methods `byPropName` and `byDescription`, to extract the EnumValue instance by either the property name of the object in the Enum or the description string passed into the EnumValue's constructor, respectively:

```typescript
console.log(colorEnum.byPropName('red') === colorEnum.red); // true
console.log(colorEnum.byDescription('it is RED') === colorEnum.red); // true
true
```


## EnumValue Customization

The EnumValues are full TypeScript classes, enabling you to add properties and methods (see the [tests](test) for more examples).


Unfortunately, this is not as terse as Enumify's syntax. Here are the steps: 
1. We define the implementation of EnumValue that defines each of the instances.
2. We implement each instance of the EnumValue as a property on the Enum. Within the Enum, we call `initEnum()` with a unique name to set up all of the Enum-specific behavior.
3. We export an instance of the enum so that other modules can use it.

```typescript
import {Enum, EnumValue} from 'ts-enums';
import {Color, ColorEnum} from 'color';
 
class BridgeSuit extends EnumValue {
  constructor(name: string, private _isMajor: boolean = false) {
    super(name);
  }
 
  // can use simple properties (defensively-copied 
  // to maintain immutability)
  get isMajor(): boolean{
    return this._isMajor;
  }
 
  // can also use methods, though this isn't an ideal implementation. 
  // (I would probably used another property instead of an if-else)
  get color(): Color {
    if (this === BridgeSuiteEnum.SPADES 
        || this === BridgeSuiteEnum.CLUBS) {
      return ColorEnum.BLACK;
    } else {
      return ColorEnum.RED;
    }
  }
}
 
class BridgeSuitEnumType extends Enum {
  SPADES: BridgeSuit = new BridgeSuit('Spades', true);
  HEARTS: BridgeSuit = new BridgeSuit('Hearts', true);
  DIAMONDS: BridgeSuit = new BridgeSuit('Diamonds');
  CLUBS: BridgeSuit = new BridgeSuit('Clubs');
 
  constructor() {
    super();
    this.initEnum('BridgeSuit');
  }
 
  // can also have methods on the overall type
  get majors(): BridgeSuit[] {
    return this.values.filter((suit: BridgeSuit ) => suit.isMajor);
  }
}
 
const BridgeSuitEnum: BridgeSuitEnumType = 
  new BridgeSuitEnumType();
 
console.log(BridgeSuitEnum.HEARTS.color.toString()); // Color.RED
console.log(BridgeSuitEnum.HEARTS.isMajor); // true
// ```

## More information

* The directory [test/](test) contains examples.
* See [ngrx-example-app-enums](https://github.com/LMFinney/ngrx-example-app-enums) for a more complicated implementation supporting an [@ngrx](https://github.com/ngrx/store) app.