# 泛型(Generics)

### 一句话概括为什么要使用泛型

方便控件的复用。

<br/>

### 定义泛型

最简单的例子，一个返回输入值的函数：

```typescript
function identity(arg: number): number { // 第二个number代表返回的类型
  return arg;
}
```

上方函数只能输入number类型。

<br/>

将number改为any：

```typescript
function identity(arg: any): any {
  return arg;
}
```

上方函数可以输入任何(any)类型，但是返回时无法得知输入的具体类型。

<br/>

为了得知输入的具体类型，可以使用一个`type varialble`：

```typescript
function identity<T>(arg: T): T {
  return arg;
}
```

上方函数中的`T`可以捕获输入的类型。

<br/>

### 传入参数

第一种方法：传入所有参数，包括类型参数：

```typescript
let output = identity<string>("myString");
//       ^ = let output: string
```

上方指定了`T`是`string`类型。

<br/>

第二种方法：利用`类型推论`，编辑器自动确定类型：

```typescript
let output = identity("myString");  // type of output will be 'string'
```

<br/>

### 使用泛型

使用泛型时，编辑器要求你必须把这个参数当作可能的任意类型。举例：

```typescript
function identity<T>(arg: T): T {
  return arg;
}
```

如果要打印`T`的长度，直接调用length属性是不行的：

```typescript
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length); // Property 'length' does not exist on type 'T'.
  return arg;
}
```

因为有可能`T`并没有length属性。

<br/>

如果操作的`T`是数组，则可以用如下方式创建数组并使用length属性：

```typescript
function loggingIdentity<T>(arg: T[]): T[] {
  console.log(arg.length);
  return arg;
}
```

可以这样理解：泛型函数`loggingIdentity`的接收类型参数`T`，实际传入的参数是`arg`，`arg`是一个数组，里面的元素类型都是`T`，返回的也是元素类型都是`T`的数组。如果传入一个数字数组，那么返回也是一个数字数字，`T`为number类型。这样增加了灵活性。

<br/>

该函数也可以改成如下形式：

```typescript
function loggingIdentity<T>(arg: Array<T>): Array<T> {
  console.log(arg.length); // Array has a .length, so no more error
  return arg;
}
```

<br/>

### 泛型类型

创建泛型函数的方法如下：

```typescript
function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```

也可以使用不同的参数名，比如把T改为U：

```typescript
function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: <U>(arg: U) => U = identity; // change T to U
```

也可以将泛型作为对象字面类型的调用签名：

```typescript
function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: { <T>(arg: T): T } = identity; // as a call signature
```

<br/>

接下来就可以写泛型接口了：

```typescript
interface GenericIdentityFn {
  <T>(arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

也可以把泛型参数当作整个接口的一个参数：

```typescript
interface GenericIdentityFn<T> { // interface arg
  (arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity; // number
```

___Docs: "This makes the type parameter visible to all the other members of the interface."???___

上例不再是泛型函数，而是把非泛型函数签名作为泛型类型一部分。 使用`GenericIdentityFn`时还得传入一个类型参数来指定泛型类型（例如上例的`number`），锁定了之后代码里使用的类型。

<br/>

### 泛型类

话不多说，直接举例：

```typescript
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};
```

```typescript
// T as string type
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

alert(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

泛型类指的是实例部分的类型，所以类的静态属性不能使用这个泛型类型。

<br/>

### 泛型限制

前面关于length属性的例子，我们可以限制函数处理带有length属性的所有类型。

```typescript
interface Lengthwise { // 描述约束条件的接口
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T { // extends ...
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

<br/>

### 在泛型中使用类型参数

声明一个类型参数，且它被另一个类型参数所约束。

比如，现在我们想要从某对象里获取某属性。 并且确保这个属性存在于对象`obj`上，因此需要在这两个类型之间使用约束。

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // okay
getProperty(x, "m"); // error: Argument of type 'm' isn't assignable to 'a' | 'b' | 'c' | 'd'.
```

<br/>

### 在泛型里使用class(待理解)

使用泛型创建工厂函数时，需要引用构造函数的类类型：

```typescript
function create<T>(c: { new (): T }): T {
  return new c();
}
```

一个更高级的例子，使用原型属性推断并约束构造函数与类实例的关系`?`：

```typescript
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function createInstance<A extends Animal>(c: new () => A): A {
    return new c();
}

createInstance(Lion).keeper.nametag;  // typechecks!
createInstance(Bee).keeper.hasMask;   // typechecks!
```



