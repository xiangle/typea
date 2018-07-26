功能强大JS数据模型验证、处理工具

### Install

```
npm install check-data
```

### 示例

```js
let sample = {
   "name": "test",
   "num": "123456789987",
   "ObjectId": "59c8aea808deec3fc8da56b6",
   "files": ["abc.js", "null", "edb.js"],
   "user": {
      "username": "莉莉",
      "age": 18,
      "address": [
         { "city": "深圳" },
         { "city": "北京" }
      ],
   },
   "money": "2"
}

let { mongoId, email } = Check.types

let { error, data } = Check(sample, {
   "ObjectId": mongoId,
   "name": String,
   "email": email,
   "num": String,
   "files": [String],
   "user": {
      "username": "莉莉",
      "age": Number,
      "address": [
         { "city": String },
         { "city": "北京" }
      ],
   },
   "money": "2"
})
```

### 特性

* 借鉴于mongoosejs数据模型表达式，采用全镜像的数据模型设计，相比其它数据验证器拥有更好的数据结构表现能力和聚合能力。

* 支持对象和数组的无限嵌套，只管按你的数据结构去建模就好了，不必担心数据复杂度、层级深度的问题。

* 可以直接复制你的数据进行快速建模，只需要将值替换为类型后就得到了一个基础的验证模型，甚至有时候连值都不用不替换。

* check-data不仅仅只是数据验证器，同时还拥有很好的数据处理能力，可以在验证前、后灵活的进行数据扩展。另外，基于js对象的树状结构让代码看起来高度类聚，大大降低了碎片化率。

* 拥有足够的容错能力，在验证期间你几乎不需要使用try/catch来捕获异常，返回值中的path错误定位信息可以帮助快速追踪错误来源。

* 当内置数据类型无法满足需求时，可以通过check.use()方法扩展自定义的数据类型。


### 验证模式

check-data支持常规、严格、宽松三种验证模式，多数情况下只需要使用常规模式即可。

> 引入严格模式和宽松模式的主要原因是为了弥补js对象结构对自身的表达分歧，在数组、对象结构中包含子表达式时没有额外的结构来定义空值。

#### 常规模式

常规模式下默认只对allowNull为false的节点强制执行非空验证，默认对包含子表达式的数组、对象结构体执行强制非空验证。

#### 严格模式

严格模式下默认会为所有节点强制执行非空验证，除非明确声明allowNull为true。

#### 宽松模式

宽松模式下不会对包含子表达式的数组、对象结构体进行强制非空验证。

```js
let Check = require('check-data')

// 常规模式
let { error, data } = Check(data, options, extend)

// 严格模式
let { error, data } = Check.strict(data, options, extend)

// 宽松模式
let { error, data } = Check.loose(data, options, extend)
```

### 输入参数

*  `data` * - 待验证数据，允许任意数据类型

*  `options` * - 待验证数据的结构镜像验证表达式，参考[验证表达式](#验证表达式)。

*  `extend` *Objcte* - 数据扩展选项，根据输入数据生成新的数据结构（可选）

*  `extend.$name` *Function* - 数据扩展函数，基于已验证的数据构建新的数据结构，输出结果将以函数名作为key保存到返回值的data中。函数中this和第一个入参指向data（已存在的同名属性值会被函数返回值覆盖）

*  `extend.$name` * - 除函数外的其它任意数据类型，在已验证的数据结构上添加新的属性或覆盖已存在的同名属性

### 返回值

> 返回值是基于约定的对象结构，error和data属性不会同时存在，验证成功返回data，验证失败返回error和msg

*  `data` * - 经过验证、处理后导出数据，仅保留options中定义的数据结构，未定义的部分会被忽略。内置空值过滤，自动剔除对象、数组中的空字符串、undefind值。

*  `error` *String* - 验证失败时返回的错误信息，包含错误的具体位置信息，仅供开发者调试使用

*  `msg` *String* - 验证失败后返回的错误信息，相对于error而言，msg对用户更加友好，可直接在客户端显示

### options验证表达式

options数据验证表达式支持无限嵌套，不管你的数据层级有多深。

type作为验证表达式的内部保留关键字，应尽量避免在入参中包含同名的type属性，否则可能导致验证结果出现混乱。

整体数据结构与待验证数据结构基本保持一致，除了使用type对象表达式不得不增加额外的数据结构。验证表达式中判断一个对象节点是否为验证选项的唯一依据是检查对象中是否包含type属性，如果没有type则被视为对象结构。

options中支持值表达式，可以对表达式节点直接赋值，实现输入、输出的完全匹配或部分匹配，在用于对象动态断言时很方便。

当使用数组表达式时，需要区分单数和复数模式，单数时多个同级子节点会共用一个子表达式，通常用于验证具有相似数据结构的子集。复数时为精确匹配模式，可以完整定义每个子集。

#### 通用选项

* `type` * - 数据类型

* `default` * - 空值时的默认赋值，优先级高于allowNull

* `allowNull` *Boolean* - 是否允许为空，当值为false时强制进行非空验证。

* `set` *Function* - 赋值函数，用于对输入值处理后再输出赋值，函数中this指向原始数据data，当值为空时不执行。

* `ignore` *Array* - 忽略指定的值，当存在匹配项时该字段不会被创建。如忽略空值，通过[null, ""]重新定义空值。

* `and` *Array、Function* - 声明节点的依赖关系，限制所有依赖的参数都不能为空。如参数a必须依赖参数b构成一个完整的数据，那么就要将参数名b加入到and数组中建立依赖关系。除了数组表达式外，还支持通过函数表达式动态生成依赖数组。

* `or` *Array、Function* - 与and相似，区别是只要满足任意一个依赖不为空即可。

* `name` *String* - 节点名称，为参数名定义一个更易于理解的别名，在返回错误描述文本中会优先使用该别名替换属性名

#### 专用选项

> 针对不同的数据类型，会有不同的可选参数，选项如下

##### String

* `minLength` *Number* - 限制字符串最小长度

* `maxLength` *Number* - 限制字符串最大长度

* `reg` *RegExp* - 正则表达式验证

* `in` *Array* - 匹配多个可选值中的一个

##### Number

> 内置类型转换，允许字符串类型的纯数字

* `min` *Number* - 限制最小值

* `max` *Number* - 限制最大值

* `in` *Array* - 匹配多个可选值中的一个

##### Array

* `minLength` *Number* - 限制数组最小长度

* `maxLength` *Number* - 限制数组最大长度

##### Object、Date、Boolean、Function

> 无专用选项


#### 其它数据类型

其它类型通过Check.types获取，types中内置了以下常见类型

##### email

验证Email

##### mobilePhone

验证手机号

##### mongoId

验证mongodb中的ObjectId


### 扩展自定义数据类型

验证器中仅内置了一部分常用的数据类型，如果不能满足你的需求，可以通过Check.use()自行扩展。

check-data依赖validator库，你可以使用Check.use()搭配validator来定制自己的数据类型。


> 当定义的数据类型不存在时则创建，已存在时则合并，新的验证函数会覆盖内置的同名验证函数。

```js
Check.use(name, options)
```

* `name` *Function, Symbol, String* - 类型Key（必填）

* `options` *Object* - 类型选项（必填）

* `options.type` *Function* - 数据类型验证函数（必填）

* `options.$name` *Function* - 自定义验证函数（可选）


```js
Check.use('int', {
   type({ data }) {
      if (Number.isInteger(data)) {
         return { data }
      } else {
         return { error: '必须为int类型' }
      }
   },
   max({ data, option: max }) {
      if (data > max) {
         return { error: `不能大于${max}` }
      } else {
         return { data }
      }
   }
})
```

### 可复用验证器

schema用于创建可复用的验证器，相比每次都将验证表达式作为一次性消耗品，schema是更好的选择，在环境允许的情况下应优先使用schema模式。

> schema的定义应该在应用启动时被执行，而不是运行时。目的是通过预先缓存一部分静态数据，从而减少运行时的内存和计算开销。

```js
Check.schema(options, extend)
```

* `options` * - 验证表达式

* `extend` Object - 数据扩展选项

```js
let schema = Check.schema({
   a: {
      a1: String,
      a2: String
   },
   b: 2,
   c: Number,
})

let sample = {
   a: {
      a1: "jj",
      a2: "12",
   },
   b: 2,
   c: 888,
}

let { error, data } = schema(sample)
```

### 参考示例

#### 数组验证

```js
let sample = {
   a: ['xx', 'kk'],
   b: [666, 999, 88,],
   c: [{ a: 1 }, { a: 2 }, { b: '3' }],
   d: [
      {
         d1: 666,
         d2: "888"
      },
      999,
      [
         {
            xa: 1,
            xb: [1, 2, 3],
         },
         {
            xa: 9,
            xb: [2, 4, 3],
         }
      ],
      "hello"
   ],
   e: [1, 2, 3],
}

let { error, data } = Check(sample, {
   a: [String],
   b: [{
      "type": Number,
      "allowNull": false
   }],
   c: [{ a: Number, b: Number }],
   d: [
      {
         d1: 666,
         d2: String
      },
      Number,
      [
         {
            xa: Number,
            xb: [{
               "type": Number,
               "allowNull": false
            }],
         }
      ],
      String
   ],
   e: Array
})
```

#### 对象验证

```js
let sample = {
   a: {
      a1: 1,
      a2: 12,
   },
   b: 99,
   f(a, b) {
      return a + b
   },
}

let { error, data } = Check(sample,
   {
      a: {
         a1: {
            type: Number,
            allowNull: false
         },
         a2: 12
      },
      b: 99,
      f: {
         type: Function,
         set(func) {
            return func(1, 1)
         }
      },
   }
)
```

#### and依赖验证

```js
let { error, data } = Check({
   "username": "莉莉",
   "addressee": "嘟嘟",
}, {
   "username": {
      "type": String,
      "and": ["addressee", "address"],
   },
   "addressee": {
      "type": String,
      "allowNull": true
   },
   "address": {
      "type": String,
      "allowNull": true,
      and(value){
         if (value === 1) {
            return ["addressee", "address"]
         } else if (value === 2) {
            return ["username", "xx"]
         }
      },
   }
})
```

#### or依赖验证

```js
let { error, data } = Check({
   "username": "莉莉",
   "addressee": "嘟嘟",
}, {
   "username": {
      "type": String,
      "or": ["addressee", "address"]
   },
   "addressee": {
      "type": String,
      "allowNull": true
   },
   "address": {
      "type": String,
      "allowNull": true
   }
})
```

#### 扩展类型验证

```js
Check.use('int', {
   type({ data }) {
      if (Number.isInteger(data)) {
         return { data }
      } else {
         return { error: '必须为int类型' }
      }
   },
})

let { mongoId, email, mobilePhone, int } = Check.types

let { error, data } = Check(
   {
      "id": "5968d3b4956fe04299ea5c18",
      "mobilePhone": "18555555555",
      "age": 20,
   }, 
   {
      "id": mongoId,
      "mobilePhone": mobilePhone,
      "age": int
   }
)
```

#### 综合示例

```js
let sample = {
   "name": "测试",
   "num": "123456789987",
   "ObjectId": "59c8aea808deec3fc8da56b6",
   "files": ["abc.js", "334", "null", "666", "12"],
   "user": {
      "username": "莉莉",
      "age": 18,
      "address": [
         {
            "city": "深圳",
         },
         {
            "city": "北京",
         }
      ],
   },
   "list": [
      {
         "username": "吖吖",
         "age": {
            "kk": [{ kkk: 666 }]
         },
      },
      {
         "username": "可可",
         "age": {
            "kk": [
               { kkk: 666 },
               { kkk: 999 }
            ]
         },
      }
   ],
   "money": "2",
   "guaranteeFormat": 0,
   "addressee": "嘟嘟",
   "phone": "18565799072",
   "coupon": "uuuu",
   "integral": {
      "lala": "168",
      "kaka": 6,
   },
   "search": "深圳",
   "searchField": "userName",
   "email": "xxx@xx.xx",
   "arr": ['jjsd', 'ddd']
}

let { mongoId, email, mobilePhone } = Check.types

let { error, data } = Check(sample,
   {
      "name": {
         "type": String,
         "name": "名称",
         "allowNull": false,
         "default": "默认值"
      },
      "num": {
         "type": Number,
         "value": 666,
      },
      "ObjectId": mongoId,
      "user": {
         "username": "莉莉",
         "age": Number,
         "address": [
            {
               "city": String,
            },
            {
               "city": "北京",
            }
         ],
      },
      "list": [
         {
            "username": String,
            "age": {
               "kk": [{ kkk: Number }]
            },
         }
      ],
      "money": {
         "type": Number,
         "min": 1,
         "in": [1, 2],
      },
      "files": [{
         "type": String,
         "allowNull": false,
      }],
      "guaranteeFormat": {
         "type": Number,
         "to": Boolean,
      },
      "addressee": String,
      "search": "深圳",
      "phone": {
         "type": mobilePhone
      },
      "coupon": {
         "type": String,
         set(value) {
            return { "$gt": value }
         }
      },
      "integral": {
         "lala": {
            "type": Number,
         },
         "kaka": {
            "type": Number,
            "allowNull": false,
            "in": [1, 3, 8, 6],
         }
      },
      "email": {
         "type": email,
         set(value) {
            return [value, , null, , undefined, 666]
         }
      },
      "arr": [String],
   },
   {
      filter({ email, integral }) {
         return {
            "email": email
         }
      },
      where({ email, integral }) {
         return {
            "integral": integral
         }
      }
   }
)
```