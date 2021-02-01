# queryString.parse()

参考：

[Node.js v15.6.0 Documentation: Query string](https://nodejs.org/api/querystring.html#querystring_querystring_parse_str_sep_eq_options)

[Node.js | querystring.parse() Method](https://www.geeksforgeeks.org/node-js-querystring-parse-method/)

<br>

将一个URL查询字符串(URL query)解析成一个包含查询URL的键(key)和对值(pari values)的对象(object)。返回的对象不继承JavaScript对象的原型，因此通常的Object方法无法工作。在解析过程中，除非有替代的字符编码格式，否则会假定UTF-8编码格式。要解码其他字符编码，必须指定decodeURIComponent选项。

<br>

### 语法

```typescript
querystring.parse(str[, sep[, eq[, options]]])
```

str\<string>: 要解析的URL query string

sep\<string>: 用于分隔查询字符串中的键(key)和值对(value pairs)的子串。默认值：'&'。

eq\<string>: 用于分隔查询字符串中的键(key)和值(values)的子字符串。默认值：'='。

options\<Object>: 

​        decodeURIComponent\<Function>: 对查询字符串中的百分比编码字符(percent-encoded characters)进行解码时要使用的函数。默认值: querystring.unescape()。

​        maxKeys\<number>:  指明转化的最大键数，0表示没有最大键数限制。默认值: 1000

返回值：返回一个对象，该对象拥有从查询字符串中解析出的键和值对。

<br>

### 举例

`'foo=bar&abc=xyz&abc=123'`将被转化为：

```typescript
{
    foo: 'bar',
    abc: ['xyz', '123']
}
```

querystring.parse()方法返回的对象并不原型地(prototypically )继承自JavaScript对象。这意味着典型的对象方法，如obj.toString()、obj.hasOwnProperty()和其他方法没有被定义，也无法使用。

默认情况下，查询字符串中的百分比编码字符(percent-encoded characters)将被假定为使用UTF-8编码。如果使用了其他的字符编码，则需要指定一个替代的 decodeURIComponent 选项。

<br>

`Input - 1`

```typescript
// Import the querystring module 
const querystring = require("querystring"); 

// Specify the URL query string 
// to be parsed 
let urlQuery = 
"username=user1&units=kgs&units=pounds&permission=false"; 

// Use the parse() method on the string 
let parsedObject = querystring.parse(urlQuery); 

console.log("Parsed Query:", parsedObject); 

// Use the parse() method on the string 
// with sep as `&&` and eq as `-` 
urlQuery = 
"username-user1&&units-kgs&&units-pounds&&permission-false"; 
parsedObject = querystring.parse(urlQuery, "&&", "-"); 

console.log("\nParsed Query:", parsedObject);
```

`Ouput - 1`

```typescript
Parsed Query: [Object: null prototype] {
  username: 'user1',
  units: [ 'kgs', 'pounds' ],
  permission: 'false'
}

Parsed Query: [Object: null prototype] {
  username: 'user1',
  units: [ 'kgs', 'pounds' ],
  permission: 'false'
}
```

<br>

`Input - 2`

```typescript
// Import the querystring module 
const querystring = require("querystring"); 
  
// Specify the URL query string 
// to be parsed 
let urlQuery =  
  "user=admin&articles=1&articles=2&articles=3&access=true"; 
  
// Use the parse() method on the string 
// with default values 
let parsedObject = querystring.parse(urlQuery, "&", "="); 
  
console.log("Parsed Query:", parsedObject); 
  
// Use the parse() method on the string 
// with maxKeys set to 1 
parsedObject =  
  querystring.parse(urlQuery, "&", "=", { maxKeys: 1 }); 
  
console.log("\nParsed Query:", parsedObject); 
  
// Use the parse() method on the string 
// with maxKeys set to 2 
parsedObject =  
  querystring.parse(urlQuery, "&", "=", { maxKeys: 2 }); 
  
console.log("\nParsed Query:", parsedObject); 
  
// Use the parse() method on the string 
// with maxKeys set to 0 (no limits) 
parsedObject =  
  querystring.parse(urlQuery, "&", "=", { maxKeys: 0 }); 
  
console.log("\nParsed Query:", parsedObject);
```

`Output - 2`

```typescript
Parsed Query: [Object: null prototype] {
  user: 'admin',
  articles: [ '1', '2', '3' ],
  access: 'true'
}

Parsed Query: [Object: null prototype] { user: 'admin' }

Parsed Query: [Object: null prototype] 
              { user: 'admin', articles: '1' }

Parsed Query: [Object: null prototype] {
  user: 'admin',
  articles: [ '1', '2', '3' ],
  access: 'true'
}
```

