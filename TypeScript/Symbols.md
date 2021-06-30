# Symbols

自ECMAScript 2015起，`symbol`成为了一种新的原生类型，就像`number`和`string`一样。

`symbol`类型的值是通过`Symbol`构造函数创建的。

```typescript
let sym1 = Symbol();

let sym2 = Symbol("key"); // 可选的字符串key
```

Symbols是不可改变且唯一的。

```typescript
let sym2 = Symbol("key");
let sym3 = Symbol("key");

sym2 === sym3; // false, symbols是唯一的
```

像字符串一样，symbols也可以被用做对象属性的键。

```typescript
let sym = Symbol();

let obj = {
    [sym]: "value"
};

console.log(obj[sym]); // "value"
```

Symbols也可以与计算出的属性名声明相结合来声明对象的属性和类成员。

```typescript
const getClassNameSymbol = Symbol();

class C {
    [getClassNameSymbol](){
       return "C";
    }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

<br>

## (众所周知的Symbols)[https://typescript.bootcss.com/symbols.html]

