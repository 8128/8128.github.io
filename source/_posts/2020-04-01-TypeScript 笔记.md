---
title: TypeScript 笔记
comments: true
toc: true
toc_number: false
copyright: true
mathjax: false
katex: false
hide: false
date: 2020-04-01 00:46:42
tags:
	- typescript
	- javascript
	- note
categories: javascript
description: TypeScript 自学笔记
cover: https://raw.githubusercontent.com/8128/PicGo/master/20200401004938.png
top_img:
keywords:
---

这是个笔记向文档，并非教程

来源：

-  [TypeScript入门教程](https://ts.xcatliu.com/basics)
- [深入理解TypeScript](https://jkchao.github.io/)
- [TypeScript官方教程](https://www.typescriptlang.org/docs/home.html)

# 接口

## 接口

接口运行时的影响为 0。在 TypeScript 接口中有很多方式来声明变量的结构。

下面两个是**等效**的声明, 示例 A 使用内联注解，示例 B 使用接口形式：

```ts
// 示例 A
declare const myPoint: { x: number; y: number };

// 示例 B
interface Point {
  x: number;
  y: number;
}
declare const myPoint: Point;
```

示例 B 的好处在于，如果有人创建了一个基于 `myPoint` 的库来添加新成员, 那么他可以轻松将此成员添加到 `myPoint` 的现有声明中:

```ts
// Lib a.d.ts
interface Point {
  x: number,
  y: number
}
declare const myPoint: Point

// Lib b.d.ts
interface Point {
  z: number
}

// Your code
myPoint.z // Allowed!
```

TypeScript 接口是开放式的，这是 TypeScript 的一个重要原则，它允许你使用接口来模仿 JavaScript 的可扩展性。

## 类可以实现接口

如果你希望在类中使用必须要被遵循的接口（类）或别人定义的对象结构，可以使用 `implements` 关键字来确保其兼容性：

```ts
interface Point {
  x: number;
  y: number;
}

class MyPoint implements Point {
  x: number;
  y: number; // Same as Point
}
```

基本上，在 `implements（实现）` 存在的情况下，该外部 `Point` 接口的任何更改都将导致代码库中的编译错误，因此可以轻松地使其保持同步：

```ts
interface Point {
  x: number;
  y: number;
  z: number; // New member
}

class MyPoint implements Point {
  // ERROR : missing member `z`
  x: number;
  y: number;
}
```

注意，`implements` 限制了类实例的结构，如下所示:

```ts
let foo: Point = new MyPoint();
```

但像 `foo: Point = MyPoint` 这样的代码，与其并不是一回事。

## 注意

### 并非每个接口都是很容易实现的

接口旨在声明 JavaScript 中可能存在的任意结构。

思考以下例子，可以使用 `new` 调用某些内容：

```ts
interface Crazy {
  new (): {
    hello: number;
  };
}
```

你可能会有下面这样的代码：

```ts
class CrazyClass implements Crazy {
  constructor() {
    return { hello: 123 };
  }
}

// Because
const crazy = new CrazyClass(); // crazy would be { hello:123 }
```

你可以使用接口声明所有“疯狂的”的 JavaScript 代码，甚至可以安全地在 TypeScript 中使用它们。但这并不意味着你可以使用 TypeScript 类来实现它们。

# Utility Types

https://www.typescriptlang.org/docs/handbook/utility-types.html#partialt

- Partial\<T>

Constructs a type with all properties of `T` set to optional. This utility will return a type that represents all subsets of a given type.

Example 

```ts
interface Todo {
    title: string;
    description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
    return { ...todo, ...fieldsToUpdate };
}

const todo1 = {
    title: 'organize desk',
    description: 'clear clutter',
};

const todo2 = updateTodo(todo1, {
    description: 'throw out trash',
});
```

- Omit\<T,K>

Constructs a type by picking all properties from `T` and then removing `K`.

Example

```ts
interface Todo {
    title: string;
    description: string;
    completed: boolean;
}

type TodoPreview = Omit<Todo, 'description'>;

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
};
```

- Pick\<T,K>

Constructs a type by picking the set of properties `K` from `T`.

Example

```ts
interface Todo {
    title: string;
    description: string;
    completed: boolean;
}

type TodoPreview = Pick<Todo, 'title' | 'completed'>;

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
};
```

# 联合类型

联合类型（Union Types）表示取值可以为多种类型中的一种。

## 简单的例子

```ts
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
```

```ts
let myFavoriteNumber: string | number;
myFavoriteNumber = true;


// index.ts(2,1): error TS2322: Type 'boolean' is not assignable to type 'string | number'.
//   Type 'boolean' is not assignable to type 'number'.
```

联合类型使用 `|` 分隔每个类型。

这里的 `let myFavoriteNumber: string | number` 的含义是，允许 `myFavoriteNumber` 的类型是 `string` 或者 `number`，但是不能是其他类型。

## 访问联合类型的属性或方法

当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候，我们**只能访问此联合类型的所有类型里共有的属性或方法**：

```ts
function getLength(something: string | number): number {
    return something.length;
}


// index.ts(2,22): error TS2339: Property 'length' does not exist on type 'string | number'.
//   Property 'length' does not exist on type 'number'.
```

上例中，`length` 不是 `string` 和 `number` 的共有属性，所以会报错。

访问 `string` 和 `number` 的共有属性是没问题的：

```ts
function getString(something: string | number): string {
    return something.toString();
}
```

联合类型的变量在被赋值的时候，会根据类型推论的规则推断出一个类型：

```ts
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven';
console.log(myFavoriteNumber.length); // 5
myFavoriteNumber = 7;
console.log(myFavoriteNumber.length); // 编译时报错


// index.ts(5,30): error TS2339: Property 'length' does not exist on type 'number'.
```

上例中，第二行的 `myFavoriteNumber` 被推断成了 `string`，访问它的 `length` 属性不会报错。

而第四行的 `myFavoriteNumber` 被推断成了 `number`，访问它的 `length` 属性时就报错了。

# 泛型

设计泛型的关键目的是在成员之间提供有意义的约束，这些成员可以是：

- 类的实例成员
- 类的方法
- 函数参数
- 函数返回值

## 动机和示例

下面是对一个先进先出的数据结构——队列，在 `TypeScript` 和 `JavaScript` 中的简单实现。

```ts
class Queue {
  private data = [];
  push = item => this.data.push(item);
  pop = () => this.data.shift();
}
```

在上述代码中存在一个问题，它允许你向队列中添加任何类型的数据，当然，当数据被弹出队列时，也可以是任意类型。在下面的示例中，看起来人们可以向队列中添加`string` 类型的数据，但是实际上，该用法假定的是只有 `number` 类型会被添加到队列里。

```ts
class Queue {
  private data = [];
  push = item => this.data.push(item);
  pop = () => this.data.shift();
}

const queue = new Queue();

queue.push(0);
queue.push('1'); // Oops，一个错误

// 一个使用者，走入了误区
console.log(queue.pop().toPrecision(1));
console.log(queue.pop().toPrecision(1)); // RUNTIME ERROR
```

一个解决的办法（事实上，这也是不支持泛型类型的唯一解决办法）是为这些约束创建特殊类，如快速创建数字类型的队列：

```ts
class QueueNumber {
  private data = [];
  push = (item: number) => this.data.push(item);
  pop = (): number => this.data.shift();
}

const queue = new QueueNumber();

queue.push(0);
queue.push('1'); // Error: 不能推入一个 `string` 类型，只能是 `number` 类型

// 如果该错误得到修复，其他将不会出现问题
```

当然，快速也意味着痛苦。例如当你想创建一个字符串的队列时，你将不得不再次修改相当大的代码。我们真正想要的一种方式是无论什么类型被推入队列，被推出的类型都与推入类型一样。当你使用泛型时，这会很容易：

```ts
// 创建一个泛型类
class Queue<T> {
  private data: T[] = [];
  push = (item: T) => this.data.push(item);
  pop = (): T | undefined => this.data.shift();
}

// 简单的使用
const queue = new Queue<number>();
queue.push(0);
queue.push('1'); // Error：不能推入一个 `string`，只有 number 类型被允许
```

另外一个我们见过的例子：一个 `reverse` 函数，现在在这个函数里提供了函数参数与函数返回值的约束：

```ts
function reverse<T>(items: T[]): T[] {
  const toreturn = [];
  for (let i = items.length - 1; i >= 0; i--) {
    toreturn.push(items[i]);
  }
  return toreturn;
}

const sample = [1, 2, 3];
let reversed = reverse(sample);

reversed[0] = '1'; // Error
reversed = ['1', '2']; // Error

reversed[0] = 1; // ok
reversed = [1, 2]; // ok
```

在此章节中，你已经了解在*类*和*函数*上使用泛型的例子。一个值得补充一点的是，你可以为创建的成员函数添加泛型：

```ts
class Utility {
  reverse<T>(items: T[]): T[] {
    const toreturn = [];
    for (let i = items.length; i >= 0; i--) {
      toreturn.push(items[i]);
    }
    return toreturn;
  }
}
```

TIP

你可以随意调用泛型参数，当你使用简单的泛型时，泛型常用 `T`、`U`、`V` 表示。如果在你的参数里，不止拥有一个泛型，你应该使用一个更语义化名称，如 `TKey` 和 `TValue` （通常情况下，以 `T` 作为泛型的前缀，在其他语言如 C++ 里，也被称为模板）

## 误用的泛型

我见过开发者使用泛型仅仅是为了它的 hack。当你使用它时，你应该问问自己：你想用它来提供什么样的约束。如果你不能很好的回答它，你可能会误用泛型，如：

```ts
declare function foo<T>(arg: T): void;
```

在这里，泛型完全没有必要使用，因为它仅用于单个参数的位置，使用如下方式可能更好：

```ts
declare function foo(arg: any): void;
```

## 设计模式：方便通用

考虑如下函数：

```ts
declare function parse<T>(name: string): T;
```

在这种情况下，泛型 `T` 只在一个地方被使用了，它并没有在成员之间提供约束 `T`。这相当于一个如下的类型断言：

```ts
declare function parse(name: string): any;

const something = parse('something') as TypeOfSomething;
```

仅使用一次的泛型并不比一个类型断言来的安全。它们都给你使用 API 提供了便利。

另一个明显的例子是，一个用于加载 json 返回值函数，它返回你任何传入类型的 `Promise`：

```ts
const getJSON = <T>(config: { url: string; headers?: { [key: string]: string } }): Promise<T> => {
  const fetchConfig = {
    method: 'GET',
    Accept: 'application/json',
    'Content-Type': 'application/json',
    ...(config.headers || {})
  };
  return fetch(config.url, fetchConfig).then<T>(response => response.json());
};
```

请注意，你仍然需要明显的注解任何你需要的类型，但是 `getJSON` 的签名 `config => Promise` 能够减少你一些关键的步骤（你不需要注解 `loadUsers` 的返回类型，因为它能够被推出来）：

```ts
type LoadUserResponse = {
  user: {
    name: string;
    email: string;
  }[];
};

function loaderUser() {
  return getJSON<LoadUserResponse>({ url: 'https://example.com/users' });
}
```

与此类似：使用 `Promise` 做为一个函数的返回值比一些如：`Promise` 的备选方案要好很多。

### 配合 axios 使用

通常情况下，我们会把后端返回数据格式单独放入一个 interface 里：

```ts
// 请求接口数据
export interface ResponseData<T = any> {
  /**
   * 状态码
   * @type { number }
   */
  code: number;

  /**
   * 数据
   * @type { T }
   */
  result: T;

  /**
   * 消息
   * @type { string }
   */
  message: string;
}
```

当我们把 API 单独抽离成单个模块时：

```ts
// 在 axios.ts 文件中对 axios 进行了处理，例如添加通用配置、拦截器等
import Ax from './axios';

import { ResponseData } from './interface.ts';

export function getUser<T>() {
  return Ax.get<ResponseData<T>>('/somepath')
    .then(res => res.data)
    .catch(err => console.error(err));
}
```

接着我们写入返回的数据类型 `User`，这可以让 TypeScript 顺利推断出我们想要的类型：

```ts
interface User {
  name: string;
  age: number;
}

async function test() {
  // user 被推断出为
  // {
  //  code: number,
  //  result: { name: string, age: number },
  //  message: string
  // }
  const user = await getUser<User>();
}
```

# TypeScript Joiful

传送门: [joiful](https://github.com/joiful-ts/joiful)

大概就是个拿来validate你数据的东西，看看你输入输出空不空之类的

This lib allows you to apply Joi validation constraints on class properties, by using decorators.

This means you can combine your type schema and your validation schema in one go!

Calling `Validator.validateAsClass()` allows you to validate any object as if it were an instance of a given class.

## Basic Usage

大概就是搞一个decorator然后说明下数据类型和限制

```ts
import * as jf from 'joiful';

class SignUp {
  @jf.string().required()
  username: string;

  @jf
    .string()
    .required()
    .min(8)
  password: string;

  @jf.date()
  dateOfBirth: Date;

  @jf.boolean().required()
  subscribedToNewsletter: boolean;
}

const signUp = new SignUp();
signUp.username = 'rick.sanchez';
signUp.password = 'wubbalubbadubdub';

const { error } = jf.validate(signUp);

console.log(error); // Error will either be undefined or a standard joi validation error
```

## Validate plain old javascript objects

假如之前的object并不是按class继承出来的，你也可以用这个方法来以class判定这个object合不合规矩

Don't like creating instances of classes? Don't worry, you don't have to. You can validate a plain old javascript object as if it were an instance of a class.

```ts
const signUp = {
  username: 'rick.sanchez',
  password: 'wubbalubbadubdub',
};

const result = jf.validateAsClass(signUp, SignUp);
```

## Custom decorator constraints

自定义decorator，让你在任何object上套用验证：

`customDecorators.ts`

```ts
import * as jf from 'joiful';

const password = () =>
  jf
    .string()
    .min(8)
    .regex(/[a-z]/)
    .regex(/[A-Z]/)
    .regex(/[0-9]/)
    .required();
```
`changePassword.ts`

```ts
import { password } from './customDecorators';

class ChangePassword {
  @password()
  newPassword: string;
}
```

## Validating array properties

```ts
class SimpleTodoList {
  @jf.array().items(joi => joi.string())
  todos?: string[];
}
```

To validate an array of objects that have their own joiful validation:

```ts
class Actor {
  @string().required()
  name!: string;
}

class Movie {
  @string().required()
  name!: string;

  @array({ elementClass: Actor }).required()
  actors!: Actor[];
}
```

## Validating object properties

To validate an object subproperty that has its own joiful validation:

```ts
class Address {
  @string()
  line1?: string;

  @string()
  line2?: string;

  @string().required()
  city!: string;

  @string().required()
  state!: string;

  @string().required()
  country!: string;
}

class Contact {
  @string().required()
  name!: string;

  @object().optional()
  address?: Address;
}
```