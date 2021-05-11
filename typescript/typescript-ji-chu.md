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

TS 的对象类型跟JS的类似，有 `object` 





