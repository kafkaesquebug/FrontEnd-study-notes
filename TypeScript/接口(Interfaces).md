# 接口(Interfaces)

### 为什么需要接口？

TypeScript的核心原则之一是对值所具有的*结构*(shape)进行类型检查。接口的作用：第一，为类型命名；第二，为本文件内的代码与第三方代码定义类型契约。

<br/>

### 接口如何工作

示例：

```typescript
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

type checker会检查`printLabel`的调用，`printLabel`有一个参数（名为`label`，类型为`string`）。实际上我们传入的对象会包含更多的属性（例如上方代码中的`myObj`），但是编译器只会检查那些必需的属性（有时，也不会这么宽松）。

利用接口重写上方示例，这次使用接口来描述必须包含一个`label`属性且类型为`string`：

```typescript
interface LabeledValue { // 定义接口
  label: string;
}

function printLabel(labeledObj: LabeledValue) { // 将接口用于描述传参的名字与类型
  console.log(labeledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

注：type checker不会检查属性的顺序。

<br>

### 可选属性

接口里的属性不全是必须的，在属性名后加`?`可将属性变为可选属性。__option bags__

```typescript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  let newSquare = { color: "white", area: 100 };
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

let mySquare = createSquare({ color: "black" });
```

可选属性的好处之一是对可能存在的属性进行预定义；好处之二是如果出现不存在于接口中的属性时，可以捕获到错误。示例如下：

```typescript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  let newSquare = {color: "white", area: 100};
  if (config.color) {
    // Error: Property 'clor' does not exist on type 'SquareConfig'
    newSquare.color = config.clor;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

let mySquare = createSquare({color: "black"});
```

<br>

### 只读属性

这些加了readonly的属性只能在刚刚创建时修改：

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}
// 对象字面量赋值之后也无法改变
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

TypeScript的`ReadonlyArray<T>`类型与`Array<T>`相似，只是把所有可变方法去掉了，因此数组创建后再也不能被修改：

```typescript
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // 把整个ReadonlyArray赋值到一个普通数组也不可以,但可以用类型断言重写

let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;

a = ro as number[];
```



### 额外的属性检查

前面我们知道了可以在只定义了`{ label: string; }`时多传递属性size`{ size: number; label: string; }`，也可以定义可选属性，但当两者一起使用时会造成额外的属性检查。

```typescript
interface SquareConfig { // option bags
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  return {
    color: config.color || "red",
    area: config.width ? config.width * config.width : 20,
  };
}

let mySquare = createSquare({ colour: "red", width: 100 }); // 额外传入colour属性

// Argument of type '{ colour: string; width: number; }' is not assignable to parameter of type 'SquareConfig'.
// Object literal may only specify known properties, but 'colour' does not exist in type 'SquareConfig'. Did you mean to write 'color'?
```

注：额外的属性检查(excess property checks)功能原意是用于检查拼写错误，例如上例中将color拼写为colour的情况。

绕开这些检查非常简单。 最简便的方法是使用类型断言：

```typescript
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

然而，最佳的方式是添加一个字符串索引签名，前提是你能够确定这个对象可能具有一些额外属性。

```typescript
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any; // 字符串索引签名(string index signature)
}
```

最后一种跳过检查的方法：将这个对象赋值给一个另一个变量： 因为`squareOptions`不会经过额外属性检查，所以编译器不会报错。

```typescript
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

<br>

### 函数类型

使用接口表示函数类型，我们需要给接口定义一个调用签名。 它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。

```typescript
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

下例展示了如何创建一个函数类型的变量，并将一个同类型的函数赋值给这个变量：

```typescript
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    let result = source.search(subString);
    return result > -1;
}
```

对于函数类型的类型检查来说，函数的参数名不需要与接口里定义的名字相匹配。 比如：

```typescript
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean { // 函数的参数名不需要与接口里定义的名字相匹配
    let result = src.search(sub);
    return result > -1;
}
```

函数的参数会逐个进行检查，要求对应位置上的参数类型是兼容的。 如果你不想指定类型，TypeScript的类型系统会推断出参数类型，因为函数直接赋值给了`SearchFunc`类型变量。 函数的返回值类型是通过其返回值推断出来的（此例是`false`和`true`）。 如果让这个函数返回数字或字符串，类型检查器会警告我们函数的返回值类型与`SearchFunc`接口中的定义不匹配。

<br>

### 可索引的类型

可索引类型具有一个*索引签名*，它描述了对象索引的类型，还有相应的索引返回值类型。例子：

```typescript
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

上面例子定义了`StringArray`接口，它具有索引签名。 这个索引签名表示了当用`number`去索引`StringArray`时,会得到`string`类型的返回值。

共有支持两种索引签名：字符串和数字。__注意，可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型。__这是因为当使用`number`来索引时，JavaScript会将它转换成`string`然后再去索引对象。 也就是说用`100`（一个`number`）去索引等同于使用`"100"`（一个`string`）去索引，因此两者需要保持一致。

```typescript
class Animal {
    name: string;
}

class Dog extends Animal {
    breed: string;
}
// 错误：使用'string'索引，有时会得到Animal!
interface NotOkey {
    [x: number]: Animal;
    [x: string]: Dog;
}
```

字符串索引签名能够很好的描述`dictionary`模式，并且它们也会确保所有属性与其返回值类型相匹配。因为字符串索引声明了`obj.property`和`obj["property"]`两种形式都可以。 下面的例子里，`name`的类型与字符串索引类型不匹配，所以类型检查器给出一个错误提示：

```typescript
interface NumberDictionary {
    [index: string]: number;
    length: number; // 可以，length是number类型
    name: string;   // 错误，name的类型与索引类型返回值的类型不匹配
}
```

最后，可以将索引签名设置为只读，这样就防止了给索引赋值：

```typescript
interface ReadonlyStringArray {
    readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice", "Bob"];
myArray[2] = "Mallory"; // error!
// 不能设置myArray[2]，因为索引签名是只读的
```

<br>

### 类类型

#### 实现接口

TypeScript也能够用Interface来明确强制一个类去符合某种契约。

```typescript
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

也可以在接口中描述一个方法，在类里实现它，如同下面的`setTime`方法一样：

```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

接口描述了类的公共部分，而不是公共和私有两部分。 它不会帮你检查类是否具有某些私有成员。

<br>

### <font color="purple">类静态部分与实例部分的区别??</font>

类具有两个类型：静态部分的类型和实例的类型。当你用构造器签名去定义一个接口并试图定义一个类去实现这个接口时会得到一个错误：

```typescript
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

这里因为当一个类实现了一个接口时，只对其实例部分进行类型检查。 constructor存在于类的静态部分，所以不在检查的范围内。因此，应该直接操作类的静态部分。

看下面的例子，定义了两个接口，`ClockConstructor`为构造函数所用和`ClockInterface`为实例方法所用。 为了方便定义一个构造函数`createClock`，它用传入的类型创建实例。

```typescript
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick();
}

function createClock(ctor: ClockConstructor, hour: number, minute: number):
ClockInterface {
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}

class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);
```

因为`createClock`的第一个参数是`ClockConstructor`类型，在`createClock(AnalogClock, 7, 32)`里，会检查`AnalogClock`是否符合构造函数签名。

<br>

### 继承接口

和类一样，接口也可以相互继承。 一个接口可以继承多个接口，创建出多个接口的合成接口。

```typescript
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

<br>

### <font color="purple">混合类型??</font>

接口能够描述JavaScript里丰富的类型。 一个例子是，一个对象可以同时做为函数和对象使用，并带有额外的属性。

```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter()
c(10);
c.reset();
c.interval = 5.0;
```

<br>

### 接口继承类

当接口继承了一个类类型时，它会继承类的成员但不包括其实现。 就好像接口声明了所有类中存在的成员，但并没有提供具体实现一样。 接口同样会继承到类的private和protected成员。 这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现（implement）。

当你有一个庞大的继承结构时这很有用，但要指出的是你的代码只在子类拥有特定属性时起作用。 这个子类除了继承至基类外与基类没有任何关系。 例：

```typescript
class Control {
    private state: any;   
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}

class TextBox extends Control {

}

// Error: Property 'state' is missing in type 'Image'.
class Image implements SelectableControl {
    select() { }
}

class Location {

}
```

在上面的例子里，`SelectableControl`包含了`Control`的所有成员，包括私有成员`state`。 因为`state`是私有成员，所以只能够是`Control`的子类们才能实现`SelectableControl`接口。 因为只有`Control`的子类才能够拥有一个声明于`Control`的私有成员`state`，这对私有成员的兼容性是必需的。

在`Control`类内部，是允许通过`SelectableControl`的实例来访问私有成员`state`的。 实际上，`SelectableControl`就像`Control`一样，并拥有一个`select`方法。 `Button`和`TextBox`类是`SelectableControl`的子类（因为它们都继承自`Control`并有`select`方法），但`Image`和`Location`类并不是这样的。