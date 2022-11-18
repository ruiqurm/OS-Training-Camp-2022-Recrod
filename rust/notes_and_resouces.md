# Rust

- Learn Rust With Entirely Too Many Linked Lists(`https://rust-unofficial.github.io/too-many-lists/`)
- The Rust Programming Language(`https://doc.rust-lang.org/book/`)
- Rust By Example
- Rustlings

# Naming convention

# Reference 和 智能指针

生命周期比较乱，这里特开一节

## Reference

- mut ref
- dangling ref: 指向null或者非法内存的指针
- ref指向slice或者trait object

## 生命周期

生命周期记号念作`tick` 。表示传入的变量的生命周期要大于等于函数生命周期

### 函数生命周期

- any reference *must* have an annotated lifetime.
- any reference being returned *must* have the same lifetime as an input or be `static`.

```rust
fn smallest<'a>(v: &'a [i32]) -> &'a i32{
	let mut s=  &v[0];
	for r in &v[1..]{
		if *r < *s {s = r;}
	}
	s
}
fn main(){
	{
		let arr = [9,4,1,0,1,4,9];
		let s=  smallest(&arr);
		assert_eq!(*s,0);
	}
	assert_eq!(*s,0); // 报错
}

```

### **[Coercion](https://doc.rust-lang.org/rust-by-example/scope/lifetime/lifetime_coercion.html#coercion)**

长的生命周期可以变成短的生命周期。

```rust
fn multiply<'a>(first: &'a i32, second: &'a i32) -> i32 {
    first * second
}

// `<'a: 'b, 'b>` reads as lifetime `'a` is at least as long as `'b`.
// Here, we take in an `&'a i32` and return a `&'b i32` as a result of coercion.
fn choose_first<'a: 'b, 'b>(first: &'a i32, _: &'b i32) -> &'b i32 {
    first
}
```

- 调用multiply不一定要传入两个同样生命周期的，这里会自动转化。
- choose_first a的生命周期要大于等于b

### **[Elision](https://doc.rust-lang.org/rust-by-example/scope/lifetime/elision.html#elision)省略**

我们经常不用写生命周期标记，是因为编译器会自动帮我们加上。

有三条规则：

1. input parameter: 有n个参数就有n个生命周期变量
2. 如果只有一个生命周期参数，那么输出的生命周期和输入相同
3. 如果有多个生命周期，且其中有一个`self` 或者`&mut self` ，输出的生命周期和self相同

## 智能指针

### Box

### Rc

引用计数

Rc::clone(&a) 可以增加引用计数

### RefCell

- 和Box不同，在运行时才会检查borrow
- 单线程使用

### Rc<RefCell<T>>

### 智能指针的选择

- `Rc<T>` enables multiple owners of the same data; `Box<T>` and `RefCell<T>` have single owners.
- `Box<T>` 编译期允许mut或者immut的borrow ; `Rc<T>` 编译期只允许immutable的borrow; `RefCell<T>` 运行期允许mut或者immut的borrow
- `RefCell<T>` 运行期mutable borrows ，甚至可以修改内部内容即使是immutable的

# **enum**

rust 的enum可以放struct，因此十分灵活。

```
enum WebEvent {
    // An `enum` may either be `unit-like`,
    PageLoad,
    PageUnload,
    // like tuple structs,
    KeyPress(char),
    Paste(String),
    // or c-like structures.
    Click { x: i64, y: i64 },
}

// A function which takes a `WebEvent` enum as an argument and
// returns nothing.
fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("page loaded"),
        WebEvent::PageUnload => println!("page unloaded"),
        // Destructure `c` from inside the `enum`.
        WebEvent::KeyPress(c) => println!("pressed '{}'.", c),
        WebEvent::Paste(s) => println!("pasted \"{}\".", s),
        // Destructure `Click` into `x` and `y`.
        WebEvent::Click { x, y } => {
            println!("clicked at x={}, y={}.", x, y);
        },
    }
}

fn main() {
    let pressed = WebEvent::KeyPress('x');
    // `to_owned()` creates an owned `String` from a string slice.
    let pasted  = WebEvent::Paste("my text".to_owned());
    let click   = WebEvent::Click { x: 20, y: 80 };
    let load    = WebEvent::PageLoad;
    let unload  = WebEvent::PageUnload;

    inspect(pressed);
    inspect(pasted);
    inspect(click);
    inspect(load);
    inspect(unload);
}
```

- 在一个`scope`里使用`use`可以省写很多东西

```
fn main() {
    use crate::WebEvent::*;
    let pressed = KeyPress('x');
    // `to_owned()` creates an owned `String` from a string slice.
    let pasted  = Paste("my text".to_owned());
    let click   = Click { x: 20, y: 80 };
    let load    = PageLoad;
    let unload  = PageUnload;

    inspect(pressed);
    inspect(pasted);
    inspect(click);
    inspect(load);
    inspect(unload);
}
```

### **enum实现的链表**

## **变量绑定**

### **freeze**

外层是mut，内层是immutable

## **类型**

### **强制类型转换**

- 使用`as`

- `unsafe`

  下面这种更快，但是可能会溢出

  ```
  unsafe {
      // 300.0 is 44
      println!("300.0 is {}", 300.0_f32.to_int_unchecked::<u8>());
      // -100.0 as u8 is 156
      println!("-100.0 as u8 is {}", (-100.0_f32).to_int_unchecked::<u8>());
      // nan as u8 is 0
      println!("nan as u8 is {}", f32::NAN.to_int_unchecked::<u8>());
  }
  ```

### **类型别名**

- 类型一定要用驼峰式（除非是内置类型）

  ```
  type Inch = u64;
  ```

### **`from` 和 `into` trait**

- `from`从目标属性转换到
- 还有`try_from`和`try_into`

### **字符串转数字**

`impl fmt::Display for xxxx`

## **控制流**

### **loop**

- 可以break或者continue到`'label`上，注意一定要加`'`前缀

```
    'outer: loop {
        println!("Entered the outer loop");

        'inner: loop {
            println!("Entered the inner loop");

            // This would break only the inner loop
            //break;

            // This breaks the outer loop
            break 'outer;
        }

        println!("This point will never be reached");
    }

    println!("Exited the outer loop");
```

- break可以返回一个值

  ```
  fn main(){
      let mut cnt = 0;
      let val = loop{
          cnt += 1;
          if cnt == 8{
              break cnt*2;
          }
      };
      println!("{}",val);
  }
  ```

### **for**

`a..b`是`[a,b)`,`a..=b`是`[a,b]`

```
for i in 1..11{
    println!("{}",i);
}
for i in 1..=10{
    println!("{}",i);
}
```

### **迭代器**

- iter() 拷贝
- into_iter() 移动。注意一旦调用此迭代器，原变量就不能用了
- iter_mut() 可以修改

### **match**

### **tuple**

```
fn main() {
    let triple = (0, -2, 3);

    match triple {
        // Destructure the second and third elements
        (0, y, z) => println!("第一个是0, `y` is {:?}, and `z` is {:?}", y, z),
        (1, ..)  => println!("第一个是1,其他任意"),
        _      => println!("剩余情况"),
    }
}
```

### **enum**

很好用

```
enum Color {
    // These 3 are specified solely by their name.
    Red,
    Blue,
    Green,
    // These likewise tie `u32` tuples to different names: color models.
    RGB(u32, u32, u32),
    HSV(u32, u32, u32),
    HSL(u32, u32, u32),
    CMY(u32, u32, u32),
    CMYK(u32, u32, u32, u32),
}

fn main() {
    let color = Color::RGB(122, 17, 40);
    // TODO ^ Try different variants for `color`

    println!("What color is it?");
    // An `enum` can be destructured using a `match`.
    match color {
        Color::Red   => println!("The color is Red!"), // 单色情况
        Color::Blue  => println!("The color is Blue!"),// 单色情况
        Color::Green => println!("The color is Green!"),// 单色情况
        Color::RGB(r, g, b) =>
            println!("Red: {}, green: {}, and blue: {}!", r, g, b),
        Color::HSV(h, s, v) =>
            println!("Hue: {}, saturation: {}, value: {}!", h, s, v),
        Color::HSL(h, s, l) =>
            println!("Hue: {}, saturation: {}, lightness: {}!", h, s, l),
        Color::CMY(c, m, y) =>
            println!("Cyan: {}, magenta: {}, yellow: {}!", c, m, y),
        Color::CMYK(c, m, y, k) =>
            println!("Cyan: {}, magenta: {}, yellow: {}, key (black): {}!",
                c, m, y, k),
        // Don't need another arm because all variants have been examined
    }
}

```

### **指针和引用**

- `` 解指针,`&,ref`取引用，`ref mut`

### **guards**

也可以加个if判断

```
let pair  = (1,2);
match pair {
    (x,y) if x==y =>println!("x==y"),
    _ => println!("x!=y"),
}
```

### **binding**

使用`@`来绑定数据和变量。

如果是区间，只能用`..=`

```
let x = 2;
match x{
    n @1.=10 => println!("0<{}<=10",n),
    _ =>println!("{}>10",n),
}
```

### **`if let/while let`**

```
let number = Some(7);
if let Some(i) = number {
    println!("Matched {:?}!", i);
}
```

## **function**

### **associated functions**

为特定类型定义的函数

可以看成工厂函数

### **methods**

在特定实例上调用的associated functions

```
use std::fmt;
struct Point{
	x:f64,
	y:f64,
}
impl Point{
	fn origin() ->Point{
		Point{x:0.0,y:0.0}
	}
	fn new(x: f64, y: f64) -> Point {
        Point { x: x, y: y }
    }
    // method
    fn dist(&self,p:Point) -> f64 {
        ((self.x - p.x)*(self.x - p.x)+(self.y - p.y)+(self.y - p.y)).sqrt()
    }
}
impl fmt::Display for Point {
	fn fmt(&self,f:&mut fmt::Formatter)->fmt::Result{
		write!(f,"{},{}",self.x,self.y)
	}
}
fn main() {
	let p1 = Point::origin();
	let p2 = Point::new(2.0, 2.0);
	println!("{}",p2.dist(p1));
}
```

### **闭包**

和c++类似，但是使用`||`来捕获

### **闭包类型**

- `Fn`：按引用捕获

- `FnMut`：按可变引用捕获

- `FnOnce`：按值捕获

  ```
  fn fn_once<F>(func: F) where F: FnOnce() {
      func();
  }
  ```

- 这个主要用于传递闭包作为参数时使用

- 函数也可以像闭包一样传递

- [https://huonw.github.io/blog/2015/05/finding-closure-in-rust/](https://huonw.github.io/blog/2015/05/finding-closure-in-rust/)

### **闭包作为返回值**

```
fn create_fn() -> impl Fn() {
    let text = "Fn".to_owned();
    move || println!("This is a: {}", text)
}

fn create_fnmut() -> impl FnMut() {
    let text = "FnMut".to_owned();
    move || println!("This is a: {}", text)
}

fn create_fnonce() -> impl FnOnce() {
    let text = "FnOnce".to_owned();
    move || println!("This is a: {}", text)
}

fn main() {
    let fn_plain = create_fn();
    let mut fn_mut = create_fnmut();
    let fn_once = create_fnonce();

    fn_plain();
    fn_mut();
    fn_once();
}
```

### **高阶函数和函数式**

```
fn main() {
    let upper = 1000;
    // Functional approach
    let sum_of_squared_odd_numbers: u32 =
        (0..).map(|n| n * n)                             // All natural numbers squared
             .take_while(|&n_squared| n_squared < upper) // Below upper limit
             .filter(|&n_squared| n_squared%2==1)     // That are odd
             .fold(0, |acc, n_squared| acc + n_squared); // Sum them
    println!("functional style: {}", sum_of_squared_odd_numbers);
}
```

### **发散函数**

和返回`()`不同，发散函数不返回值

```
fn foo() -> ! {
    panic!("This call never returns.");
}
```

## **module和crate**

### **区别**

crate是编译的独立单元；crate由多个module组成

### **module**

- `self`同层

  `self::function()`

- `super`上一层

- `pub` public

- 

### **crate**

### **创建库**

`rustc --crate-type=lib xxx.rs`

### **使用库**

`rustc executable.rs --extern rary=library.rlib --edition=2018 && ./executable`

### **cargo**

### **dependency**

```
[dependencies]
# 三种方式
clap = "2.27.1" # from crates.io
rand = { git = "https://github.com/rust-lang-nursery/rand" } # from online repo
bar = { path = "../bar" } # from a path in the local filesystem
```

`[dev-dependencies]`：仅在测试中使用的依赖

### **testing**

- `unit`

  - mod要带`#[cfg(test)]`；fn要带`#[test]`

  - 常用macro

    `assert!`,`assert_eq!`,`assert_ne!`

  - 如果需要带返回值，可以这样：

  ```
  fn sqrt(number: f64) -> Result<f64, String> {
      if number >= 0.0 {
          Ok(number.powf(0.5))
      } else {
          Err("negative floats don't have square roots".to_owned())
      }
  }
  
  #[cfg(test)]
  mod tests {
      use super::*;
  
      #[test]
      fn test_sqrt() -> Result<(), String> {
          let x = 4.0;
          assert_eq!(sqrt(x)?.powf(2.0), x); //注意这里有一个问号
          Ok(())
      }
  }
  
  ```

  - 如果测试需要崩溃，加上`#[should_panic]`

    ```
    #[cfg(test)]
    mod tests {
        use super::*;
    
        #[test]
        fn test_divide() {
            assert_eq!(divide_non_zero_result(10, 2), 5);
        }
    
        #[test]
        #[should_panic]
        fn test_any_panic() {
            divide_non_zero_result(1, 0);
        }
    
        #[test]
        #[should_panic(expected = "Divide result is zero")]
        fn test_specific_panic() {
            divide_non_zero_result(1, 10);
        }
    }
    ```

- Documentation testing

  - 嵌入文档中的测试
  - `/// First line is a short summary describing function.////// The next lines present detailed documentation. Code blocks start with/// triple backquotes and have implicit `fn main()` inside/// and `extern crate <cratename>`. Assume we're testing `doccomments` crate:////// ```/// let result = doccomments::add(2, 3);/// assert_eq!(result, 5);/// ```pub fn add(a: i32, b: i32) -> i32 { a + b}`

- Integration testing

  - 外部测试，通常放在`tests`文件夹里
  - 宏惯例和单元测试一样

### **attribute 宏属性**

命名惯例：

- `#![crate_attribute]` 对全crate生效
- `#[item_attribute]` 对整个item或者module生效

书写规范：

- `#[attribute = "value"]`
- `#[attribute(key = "value")]`
- `#[attribute(value)]`

### **`dead_code`**

```
#[allow(dead_code)]
fn unused_function() {}
```

### **`#![crate_type = ""]`和`#![crate_name = ""]`**

- 当使用cargo时，无效

### **`#[cfg(...)]`**

- `#[cfg(target_os = "linux")]`
- `#[cfg(not(target_os = "linux"))]`
- 

## **generic**

- rust中定义tuple和泛型很像，要区别：

  对于`impl`，也是类似。

  ```
  struct S(A);       // concrete type
  struct S2<A>(A); // Generic type
  impl<T> GenericVal<T> {}
  ```

- 显式调用泛型和隐式调用

  ```
  struct SGen<T>(T); // Generic type `SGen`
  fn generic<T>(_s: SGen<T>) {}
  fn main() {
      // Explicitly specified type parameter `char` to `generic()`.
      generic::<char>(SGen('a'));
      // Implicitly specified type parameter `char` to `generic()`.
      generic(SGen('c'));
  }
  ```

### **泛型trait**

```
// Non-copyable types.
struct Empty;
struct Null;

// 定义一个析构掉self和_的接口
trait DoubleDrop<T> {
    fn double_drop(self, _: T);
}

// 为泛型T提供DoubleDrop的实现
impl<T, U> DoubleDrop<T> for U {
    fn double_drop(self, _: T) {}
}

fn main() {
    let empty = Empty;
    let null  = Null;

    empty.double_drop(null);

    empty;// 这里会提示已经move
    null;
}

```

### **泛型约束**

泛型通常使用trait来约束

```
fn printer<T: Display>(t: T) {
    println!("{}", t);
}
```

比如，这样T要求是有`Display`的类型。

- 约束是空的也行

- 多约束

  ```
  trait A{
      fn print_a(self);
  }
  trait B{
      fn print_b(self);
  }
  #[derive(Copy, Clone)]
  struct Example;
  impl A for Example{
      fn print_a(self){
          println!("A");
      }
  }
  impl B for Example{
      fn print_b(self){
          println!("B");
      }
  }
  fn foo<T:A+B+Copy>(t:T){
      t.print_a();
      t.print_b();
  }
  fn main() {
      let e = Example{};
      foo(e);
  }
  ```

### **where**

where可以更清晰地表达约束。有时候必须使用where

```
impl <A: TraitB + TraitC, D: TraitE + TraitF> MyTrait<A, D> for YourType {}

// Expressing bounds with a `where` clause
impl <A, D> MyTrait<A, D> for YourType where
    A: TraitB + TraitC,
    D: TraitE + TraitF {}
```

### **associated type**

这个例子是为了解决泛型C依赖A,B，需要重复写地问题。

比如：

```
struct Container(i32, i32);
trait Contains<A, B> {
    fn contains(&self, _: &A, _: &B) -> bool; // Explicitly requires `A` and `B`.
    fn first(&self) -> i32; // Doesn't explicitly require `A` or `B`.
    fn last(&self) -> i32;  // Doesn't explicitly require `A` or `B`.
}
impl Contains<A, B> for Container{
   // ..... 略
}
fn difference<A, B, C>(container: &C) -> i32 where
    C: Contains<A, B> {
    container.last() - container.first()
}
// 这里difference不需要用到A,B，但是又必须知道A,B的类型才能写出C的泛型约束
```

要解决这个问题，可以这样：

```
trait Contains {
    type A;
    type B;// 加入A,B类型
	fn contains(&self, _:&Self::A, _:&Self::B) -> bool;

    fn first(&self) -> i32;
    fn last(&self) -> i32;
}
fn difference<C: Contains>(container: &C) -> i32 { ... }
```

### **Phantom type parameters幻影类型参数**

`PhantomData`指的是不在运行期而是编译期检查的变量。

实践中，可能遇到只与结构体本身有关（而与成员无关）的类型。这时，可以用`PhantomData`（零空间）。

```
use std::marker::PhantomData;
#[derive(PartialEq)] // Allow equality test for this type.
struct PhantomTuple<A, B>(A,PhantomData<B>);

#[derive(PartialEq)] // Allow equality test for this type.
struct PhantomStruct<A, B> { first: A, phantom: PhantomData<B> }
fn main() {
    let _tuple1: PhantomTuple<char, f32> = PhantomTuple('Q', PhantomData);
    let _tuple2: PhantomTuple<char, f64> = PhantomTuple('Q', PhantomData);
    // Compile-time Error! Type mismatch so these cannot be compared:
    //println!("_tuple1 == _tuple2 yields: {}",
    //          _tuple1 == _tuple2);
}

```

## **scoping**

### **borrow**

- 可以有无限个immutable borrow，但只能有一个mutable borrow

  ```
  struct Point { x: i32, y: i32, z: i32 }
  fn main() {
      let mut point = Point { x: 0, y: 0, z: 0 };
  
      let borrowed_point = &point;
      let another_borrow = &point;
  
      println!("Point has coordinates: ({}, {}, {})",
                  borrowed_point.x, another_borrow.y, point.z);
  	// 有immutable borrow，不能再mutable borrow
      // let mutable_borrow = &mut point;
      println!("Point has coordinates: ({}, {}, {})",
                  borrowed_point.x, another_borrow.y, point.z);
  	// 后面没有出现这个immutable borrow了，因此可以mutable borrow
      let mutable_borrow = &mut point;
  
      // Change data via mutable reference
      mutable_borrow.x = 5;
      mutable_borrow.y = 2;
      mutable_borrow.z = 1;
  }
  ```

- 通过解包修改局部值

```
#[derive(Clone, Copy)]
struct Point { x: i32, y: i32 }

fn main() {
    let point = Point { x: 0, y: 0 };
	let mut mutable_point = point;
    {
        let Point { x: _, y: ref mut mut_ref_to_y } = mutable_point;
        *mut_ref_to_y = 1;
    }
```

### **生命周期**

### **显式生命周期声明**

格式和泛型很像，区别在于用`'`打头。

生命周期声明表示变量一定要存活过当前函数。

下面的功能是一个简单的传递所有权。

```
fn test<'a, 'b>(arg: &'a i32)
   -> &'b i32
    where 'a:'b
{
    &arg
}
fn main() {
    let x = 1;
    let y = test(&x);
    println!("{}",y);
}

// 或者这样
fn test<'a>(arg: &'a i32)
   -> &'a i32
{
    &arg
}
fn main() {
    let x = 1;
    let y = test(&x);
    println!("{}",y);
}
```

- 类似的，method，struct，trait，bounds都可以加上生命周期的约束

- 带生命周期的约束：

  表示t必须超过a的生命周期

  ```
  fn print_ref<'a, T>(t: &'a T) where
      T: Debug + 'a {
      println!("`print_ref`: t is {:?}", t);
  }
  ```

- 长生命周期可以兼容为短生命周期

- `static`是最长生命周期

  `let s: &'static str = "hello world";`

  可以用`staic`声明使得变量生命周期变化，也可以改变类型为`&static T`实现

  ```
  static A:i32 = 18;
  fn main(){
      let B:&staitc i32 = 18;
  }
  ```

## **trait**

### **通过trait返回基类**

```
struct Sheep {}
struct Cow {}
trait Animal {
    fn noise(&self) -> &'static str;
}

impl Animal for Sheep {
    fn noise(&self) -> &'static str {
        "baaaaah!"
    }
}

impl Animal for Cow {
    fn noise(&self) -> &'static str {
        "moooooo!"
    }
}

fn random_animal(random_number: f64) -> Box<dyn Animal> {
    if random_number < 0.5 {
        Box::new(Sheep {})
    } else {
        Box::new(Cow {})
    }
}
```

### **符号重载**

`use std::ops`

```
use std::ops;
struct Foo;
struct Bar;

#[derive(Debug)]
struct FooBar;

#[derive(Debug)]
struct BarFoo;

impl ops::Add<Bar> for Foo {
    type Output = FooBar;
    fn add(self, _rhs: Bar) -> FooBar {
        println!("> Foo.add(Bar) was called");
        FooBar
    }
}
fn main() {
    println!("Foo + Bar = {:?}", Foo + Bar);
}
```

### **drop**

析构trait

### **iterator**

可迭代属性

- `struct Fibonacci { curr: u32, next: u32,}impl Iterator for Fibonacci{ type Item = u32; fn next(&mut self) ->Option<Self::Item>{ let new_next = self.curr + self.next; self.curr = self.next; self.next = new_next;  Some(self.curr) }}`
- trait内置的方法
  - `take(n)` 遍历到n
  - `skip(n)` 跳过n
  - `iter()` 取得迭代器

### **泛型和`impl Trait`的互相转写**

有时候需要的泛型约束仅仅是一个trait，此时可以不用写作泛型，而是写成`impl Trait`的形式

这里Trait也可以用+的形式

### **作为参数**

```
struct A;
trait Simple {
    fn print(&self);
}
impl Simple for A{
    fn print(&self){
        println!("A");
    }
}
fn test1<T:Simple>(a:&T){
    a.print();
}
fn test2(a:&impl Simple){
    a.print();
}
fn main(){
    let a = A{};
    test1(&a);
    test2(&a);
}
```

### **作为返回值**

作为返回值也是同理，这里略。

### **copy和clone**

- copy是隐式的，告诉编译器传参默认语义从move变为copy，只是简单的memcpy，不能重新实现。
  - 只有所有成员都实现了copy，整个对象才能copy
- clone是显式的，可能需要深拷贝和重新实现。

### **trait继承**

```
trait Person {
    fn name(&self) -> String;
}
trait Student: Person {
    fn university(&self) -> String;
}
// student同样也有person的属性
```

### **消除重叠特征**

如果两个trait有同样的名字，那么使用的时候需要显式的表述trait：

```
< Struct as Trait>::method(&object)
```

比如：

```
struct Person{
    name :&'static str,
    age : i32,
}
trait GetName {
    fn get(&self)->&'static str;
}
trait GetAge{
    fn get(&self)->i32;
}
impl GetName for Person{
    fn get(&self)->&'static str{
        self.name
    }
}
impl GetAge for Person{
    fn get(&self)->i32{
        self.age
    }
}
fn main(){
    let a = Person{name:"hello",age:20};
    println!("name:{},age:{}",<Person as GetName>::get(&a),<Person as GetAge>::get(&a));
}
```

### **宏**

rust的宏有AST，而不是简单的字符串替换。

宏的用途：

- 简化重复代码
- 可以用来做DSL
- 变量化接口。例如需要接受多个参数。

### **格式**

```
macro_rules! create_function {
    // 下面这行是参数，参数要用$开头
    // ident -> identity变量/函数名。
    ($func_name:ident) => {
        // 宏的主体，下面是实际的代码
        fn $func_name() {
            println!("You called {:?}()",
                     stringify!($func_name));
        }
    };
}
// 产生两个上面格式的函数
create_function!(foo);
create_function!(bar);
```

除了`ident`外，还有许多`designators`:`block`,`expr`,`item`

### **支持重载**

```
macro_rules! test {
    ($left:expr; and $right:expr) => {
        println!("{:?} and {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left && $right)
    };
    // ^ each arm must end with a semicolon.
    ($left:expr; or $right:expr) => {
        println!("{:?} or {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left || $right)
    };
}
// 调用时，格式是这样的：
// 注意; and 和 ; or
fn main(){
    test!(1i32 + 1 == 2i32; and 2i32 * 2 == 4i32);
    test!(true; or false);
}
```

### **variadics**

- 重复参数的记号和正则表达式类似，使用`+`或者``

```
macro_rules! find_min {
    // Base case:
    ($x:expr) => ($x);
    // `$x` followed by at least one `$y,`
    ($x:expr, $($y:expr),+) => (
        // Call `find_min!` on the tail `$y`
        std::cmp::min($x, find_min!($($y),+))
    )
}

fn main() {
    println!("{}", find_min!(1u32));
    println!("{}", find_min!(1u32 + 2, 2u32));
    println!("{}", find_min!(5u32, 2u32 * 3, 4u32));
}

```

### **DSL**

也可以用来设计DSL

```
macro_rules! calculate {
    (eval $e:expr) => {{
        {
            let val: usize = $e; // Force types to be integers
            println!("{} = {}", stringify!{$e}, val);
        }
    }};
}

fn main() {
    calculate! {
        eval 1 + 2 // hehehe `eval` is _not_ a Rust keyword!
    }

    calculate! {
        eval (1 + 2) * (3 / 4)
    }
}
```

## **错误处理**

### **panic! 宏**

- panic后，程序结束
- 也可以用`expect`接收panic

### **option**

- `option`,`Some`,`unwrap`

- 在返回`Some`或者`Result`的函数,`unwrap`可以用`?`替换

  ```
  fn next_birthday(current_age: Option<u8>) -> Option<String> {
  	// If `current_age` is `None`, this returns `None`.
  	// If `current_age` is `Some`, the inner `u8` gets assigned to `next_age`
      let next_age: u8 = current_age?;
      Some(format!("Next year I will be {}", next_age))
  }
  ```

- `option`有一个`map`的方法，可以用于处理`Some->Some`和`None->None`的简单情况。可以将多个`match`拆开来，这样用map组合更加清晰。

- `option`还有一个`and_then`方法。这个方法会把`Option`那一层去掉。只留下值或者None

### **Result**

result是一个`value,error`的元组，和Go语言中返回错误的机制很像。

### **map**

result也有map

```
// 如果逐一处理两种情况，就会看起来很复杂
// parse会返回一个Option
fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    match first_number_str.parse::<i32>() {
        Ok(first_number)  => {
            match second_number_str.parse::<i32>() {
                Ok(second_number)  => {
                    Ok(first_number * second_number)
                },
                Err(e) => Err(e),
            }
        },
        Err(e) => Err(e),
    }
}

fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    first_number_str.parse::<i32>().and_then(|first_number| {
        second_number_str.parse::<i32>().map(|second_number| first_number * second_number)
    })
}

// 当然也可以在出现错误的时候就返回
fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    let first_number = match first_number_str.parse::<i32>() {
        Ok(first_number)  => first_number,
        Err(e) => return Err(e), // 直接返回
    };

    let second_number = match second_number_str.parse::<i32>() {
        Ok(second_number)  => second_number,
        Err(e) => return Err(e),
    };

    Ok(first_number * second_number)
}

// 上面的有一个语法糖
fn multiply(first_number_str: &str, second_number_str: &str) -> Result<i32, ParseIntError> {
    let first_number = first_number_str.parse::<i32>()?; //直接使用?
    let second_number = second_number_str.parse::<i32>()?;

    Ok(first_number * second_number)
}
```

### **result的别名**

如果需要返回的错误类型是一样的，可以定义一个新类型：

```
use std::num::ParseIntError;

type AliasedResult<T> = Result<T, ParseIntError>;// 定义新类型
fn multiply(first_number_str: &str, second_number_str: &str) -> AliasedResult<i32> {
	// 略
}
```

### **result混合（多个）错误类型**

- 给result套一层option

- `map_or`： 如果对象是None，做默认处理，否则

  `opt.map_or(Ok(None), |r| r.map(Some))`

- 写一个自定义类型，包括所有的异常。

### **自定义错误类型**

```
use std::fmt;

type Result<T> = std::result::Result<T, DoubleError>;

#[derive(Debug, Clone)]
struct DoubleError;

impl fmt::Display for DoubleError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "invalid first item to double")
    }
}
```

### **保留运行时错误**

```
type Result<T> = std::result::Result<T, Box<dyn error::Error>>;
#[derive(Debug, Clone)]
struct EmptyVec;

impl fmt::Display for EmptyVec {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "invalid first item to double")
    }
}

impl error::Error for EmptyVec {} // 只要给自定义错误添加这个trait即可

fn double_first(vec: Vec<&str>) -> Result<i32> {
    vec.first()
        .ok_or_else(|| EmptyVec.into()) // Converts to Box
        .and_then(|s| {
            s.parse::<i32>()
                .map_err(|e| e.into()) // Converts to Box
                .map(|i| 2 * i)
        })
}
// 也可以这样写：
fn double_first(vec: Vec<&str>) -> Result<i32> {
    let first = vec.first().ok_or(EmptyVec)?;
    let parsed = first.parse::<i32>()?;
    Ok(2 * parsed)
}
```

### **遍历result**

直接遍历可能遇到失败的对象。

- `filter_map()`过滤掉结果为None
- `collect()` 只要有fail就终止
- `partition()` 把正常和异常的数据分成两个序列。

## **常用std 类型**

### **Vec**

- 常用创建方法：

  ```
  let collected_iterator: Vec<i32> = (0..10).collect();
  let mut xs = vec![1i32, 2, 3];
  ```

### **Strings**

- rust中有`str`和`String`两种类型

  `&str`是`String`的切片，`String`是utf-8的字符串

- 如果需要非utf-8的字符串，可以用bytestring

### **hashmap**

```
use std::collections::HashMap;
fn main(){
    let mut contacts = HashMap::new();

    contacts.insert("Daniel", "798-1364");
    contacts.insert("Ashley", "645-7689");
    contacts.insert("Katie", "435-8291");
    contacts.insert("Robert", "956-1745");

    // Takes a reference and returns Option<&V>
    match contacts.get(&"Daniel") {
        Some(&number) => println!("Calling Daniel: {}", call(number)),
        _ => println!("Don't have Daniel's number."),
    }
}
```

使用`#[derive(PartialEq, Eq, Hash)]`可以实现hash

### **hashset**

集合，提供交集，并集，差集和对等差分等操作

### **RC,reference count**

- 记录对一个类型的引用计数

### **ARC，Atomic Reference Counted**

- 适用于多线程的情况。

## **std 其他常用**

### **thread**

```
use std::thread;

const NTHREADS: u32 = 10;

// This is the `main` thread
fn main() {
    // Make a vector to hold the children which are spawned.
    let mut children = vec![];

    for i in 0..NTHREADS {
        // Spin up another thread
        children.push(thread::spawn(move || {
            println!("this is thread number {}", i);
        }));
    }

    for child in children {
        // Wait for the thread to finish. Returns a result.
        let _ = child.join();
    }
}
```

### **channel**

线程间通信使用。

```
let (tx, rx): (Sender<i32>, Receiver<i32>) = mpsc::channel();
```

### **path**

`std::path::Path`

