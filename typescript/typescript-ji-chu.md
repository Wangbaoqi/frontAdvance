# TypeScript 基础

### 基本类型

TS 除了支持 JS 所有[基本类型](../javascript-advance/javascript-basic/data-type.md)之外，还有一些其他的基本类型：

* void 没有任何类型 例如函数没有返回值时，其返回值类型为void
* any 任何类型 动态的
* enum 枚举类型
* never 表示永不存在的值的类型

```typescript
// void 函数没有返回值
function foo():void {
  conosle.log('void')
}
// 变量为void类型 只能赋值 undefined 或者 null
let unable:void = undefined;
let unable:void = null; 
```

#### Any 类型

any 类型跟现在JS类型效果是一致的，类型是动态的，两者之间的区别是类型的判断的时机不一致（JS是在运行时，TS是静态的）

```typescript
// 变量为 any
let foo: any = 2;
foo = '2';

// 数组子项类型不同
let arr: any[] = [1, '2', true, {}];
```

#### Enum 类型

enum 类型是对JS类型的补充，可以跟后端语言一样，为一组数值赋予友好的名字

```typescript
enum Color { Red = 1, Green = 2, Blue = 3}
let c: Color = Color.Red;
```

#### Never 类型

`never` 类型表示那些永不存在的值的类型。可以应用到抛出异常的函数或者不会有返回值的函数。

`never`是任何类型的子类型，可以赋值给任何类型。

```typescript
// 抛出异常的函数
function error(err: string): never {
 throw new Error(err)
}
// 返回never类型的函数 必须无法到达终点
function loop(): never {
  while(true) {}
}
```

### 对象类型

TS 的对象类型跟JS的类似，有 `object` 、`array` 、`class` 等。

```typescript
let obj: object = {};
let obj: {name: string, age: number} = {name: 'nate', age: 20};
// 数组定义方式1
let arr: number[] = [1,2,3];
// 数组定义方式2
let arr: Array<number> = [1,2,3];
// 数组中不同类型定义
let arr: (number | string)[] = [1, '3', 2];

class Person {
  name: string,
  age: number
}
const perList: Person[] = [
  new Person(),
  {
    name: 'nate',
    age: 20
  }
]
```

#### 类型别名

```typescript
type User = {
  name: string,
  age: number
}
const users: User[] = [{name: 'nate', age: 20}];
```

#### Tuple 元组类型

元组类型表示一个已知数量和类型的数组，其定义的类型顺序也是一致的。

```typescript
const tuArr: [string, number] = ['1', 2]; 
tuArr[0]; // '1'
tuArr[1]; // 2
tuArr[2] = 4; // ok
tuArr[3] = false; // error
```

#### Function 函数类型

函数的使用跟JS是一致的，在TS中是给函数增加了一静态类型，这样在开发的函数的时候可以减少很多的类型错误。

**函数返回类型**

在TS中，函数是具有返回类型的（具有返回值），比如 `number` 、 `string` 等

```typescript
// 函数返回 number类型
function add(x: number, y: number): number {
  return x + y;
}
// 没有返回值
function foo(): void {}

// 类型推断 入参都是 number类型的 则返回一定是number类型的
function add(x: number, y: number) {
  return x + y
}

// 函数表达式
// (s: sring) => string 表示bar是一个函数类型 入参和返回都是string类型的
const bar: (s: sring) => string = (s) => { 
  return s;
}: 
```

### interface 接口





```typescript
interface Person {
  // readonly name: string,
  name: string,
  // age 可选属性
  age?: number,
  [propName: string]: any,
  say(): string
}

// 接口继承
interface Teacher extends Person {
  teach(): string
}

// 接口函数类型
interface Say {
  (s: string): string
}


const getName = (person: Person): void => {
  console.log(person.name)
}
const setName = (person: Person, name: string): void => {
  person.name = name
}

const person = {
  name: 'nate'
}
```









