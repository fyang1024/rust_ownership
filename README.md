### Rust 编程语言中的所有权、引用、可变引用概念小结

- #### 所有权 (Ownership)

  所有权是 Rust 的核心概念，它让 Rust 不需要垃圾回收机制而保证内存安全。关于所有权的规则如下：

  - Rust 中的每一个值都有一个变量表示，该变量就是该值的所有者
  - 每个值在一个时间只能存在一个所有者
  - 当所有者在作用域结束时，该所有者就被丢弃了，无法在后面的代码中被使用

  这里必须要解释一下作用域了。Rust 用一对花括号限定作用域。一对花括号内，变量的作用域是从创建到花括号结束的地方。作用域是可以嵌套的。

  下面用一段代码和注释来举例说明所有权和作用域的概念。

  ```Rust
  fn foo(s: String) -> String { // 作用域的开始
    println!("{s}");
    s // 返回了字符串所有权
  } // 作用域的结束

  fn bar(s: String) { // 作用域的开始
    println!("{s}");
  } // 作用域的结束

  fn main() { // 大作用域的开始
    let a = 10u32; // a成为了10u32的所有者
    let b = a; // 编译器把a的值拷贝了一份，b成为拷贝的10u32的所有者
    println!("{a}"); // 打印出10
    println!("{b}"); // 打印出另一个10

    let s = String::from("hello"); // s成为了“hello”字符串的所有者
    { // 小作用域的开始
        let x = String::from("world"); // x成了“world”字符串的所有者
        println!("{x}"); // 打印出world
        println!("{s}"); // 大作用域中的s仍在，打印出hello
    } // 小作用域的结束
    // println!("{x}"); // 小作用域结束了，所以x已经被丢弃，这里无法使用x
    println!("{s}"); // 大作用域中的s仍在，打印出hello

    let s1 = String::from("love"); // s1成了“love”字符串的所有者
    let s2 = s1; // s2成了“love”的所有者，s1不再是“love”所有者
    // println!("{s1}"); // 每个值同一时间只能存在一个所有者，s1不再有效
    println!("{s2}"); // s2是“love”所有者，打印出love
    println!("{s2}"); // s2还是“love”所有者，打印出love

    let s3 = foo(s2); // s2失去“love”所有权，s3或者“love”所有权
    // println!("{s2}"); // s2已经失去了“love”所有权
    println!("{s3}"); // s3是“love”所有者，打印出love
    bar(s3); // s3失去了“love”所有权
    // println!("{s3}"); // s3不再拥有“love”
    // 到了这里，所有变量都失去了love
  } //大作用域的结束
  ```

  我们注意到变量类型会影响到变量赋值给另一个变量的行为，有的赋值是值的复制，产生值的副本和副本的所有者，另外的则是移动所有权，而不产生值的副本。
  默认做复制操作的有：

  - 所有的整数类型，比如 u32,
  - 布尔类型，bool
  - 浮点数类型：f32, f64
  - 字符类型 char
  - 由以上类型组成的元组类型 Tuple，如（i32, i32, char）
  - 引用 Reference（后面会介绍）

  其它类型，默认都是所有权的移动

- #### 引用 (Reference)

  引用是一种借出所有权的方式，引用可以指向某个值而没有它的所有权，可以理解为借用。引用默认是不可变引用，也就是说代码不可以通过引用改变引用所指的值。变量前加上&就产生了一个对该变量的引用，比如&x 即是对 x 的引用

  - 一个值在任何时间可以有任意数量的（不可变）引用
  - 引用的引用仍然指向值而非引用

  我们还是用代码和注释来举例说明引用的概念

  ```Rust
  fn main() {
    let a = 10u32; // a是10u32的所有者
    let b = &a; // b是a的引用，a依然是10u32的所有者
    let c = &&&a; // c还是a的引用，a依然是10u32的所有者
    let d = &b; // d也是a的引用，a依然是10u32的所有者
    let e = b; // e复制了b，也是a的引用，a依然是10u32的所有者
    println!("{a}"); // 打印10
    println!("{b}"); // 打印10
    println!("{c}"); // 打印10
    println!("{d}"); // 打印10
    println!("{e}"); // 打印10

    // 下面的代码跟上面的一段本质一样，就不赘述了
    let s1 = String::from("love");
    let s2 = &s1;
    let s3 = &&&s1;
    let s4 = &s2;
    let s5 = s2;
    println!("{s1}");
    println!("{s2}");
    println!("{s3}");
    println!("{s4}");
    println!("{s5}");
  }
  ```

- #### 可变引用 (Mutable Reference)

  可变引用，顾名思义，是可以用于改变所指的值的引用，在变量前加上"&mut "即可产生可变引用。

  - 可变引用必须指向可变变量，可变变量是变量名前有“mut ”的变量，表示该变量的值可以被改变
  - 一个值在任何时间最多有一个可变引用
  - 对一个值的可变引用和引用不能同时存在

  下面用几段代码和注释来举例说明可变引用的概念

  ```Rust
  fn main() {
    let a = 10u32; // a是不可变变量
    let b = &mut a; // 编译出错，不可变变量不能产生可变引用
  }
  ```

  ```Rust
  fn main() {
    let mut a = 10u32; // a是可变变量
    let b = &mut a; // b是a的可变引用
    *b = 20; // 通过b改变了a的值
    println!("{b}"); // 打印出20
    println!("{a}"); // a的值已经从10变为20，打印出20
  }
  ```

  ```Rust
  fn main() {
    let mut a = 10u32; // a是可变变量
    let b = &mut a; // b是a的可变引用
    *b = 20; // 通过b改变了a的值
    println!("{a}"); // 编译出错，因为这一刻a产生了不可变引用，而下一行语句要求a的可变引用b同时存在
    println!("{b}");
  }
  ```

  对比下面的代码

  ```Rust
  fn main() {
    let mut a = 10u32; // a是可变变量
    let b = &mut a; // b是a的可变引用
    *b = 20; // 通过b改变了a的值
    println!("{a}"); // 打印出20，编译器知道后面可变引用b不再需要，已经丢弃了b，此刻20只有一个不可变引用
    // println!("{b}");
  }
  ```

  再举一个无法编译通过的例子

  ```Rust
  fn main() {
    let mut a = 10u32;
    let b = &mut a;
    *b = 20;
    let _c = &a; // 编译出错，因为这一刻a产生了不可变引用，而下一行语句要求a的可变引用同时存在
    println!("{b}");
  }
  ```

  还是一个无法编译的例子

  ```Rust
  fn main() {
    let mut a = 10u32;
    let c = &a; // 产生了a的不可变引用c
    let b = &mut a; // 编译出错，因为编译器看到最后一行仍然需要用c，无法丢弃c，这一行要求产生a的可变引用b，a不能同时有可变引用和不可变引用
    *b = 20;
    println!("{c}");
  }
  ```

  通过上面的例子，我们可以总结出引用的作用域和所有权的作用域是不一样的，编译器会在引用的最后一次使用后丢弃那个引用，由于"对一个值的可变引用和引用不能同时存在", 我们可以得出这样的结论，对同一个所有权的可变引用的作用域不能有任何重叠，对同一个所有权的可变引用和不可变引用的作用域也不能有任何重叠

  我们再看三段代码以加深理解可变引用

  ```Rust
  fn main() {
    let mut a = 10u32; // a是10u32的可变所有者
    let b = &mut a; // b成为a的可变引用
    *b = 20; // 通过可变引用b我们改变了a的值
    println!("{b}"); // 打印出20
    let d = &b; // d成为a的不可变引用，可变引用b被丢弃
    println!("{d}"); // 打印出20
  }
  ```

  ```Rust
  fn main() {
    let mut a = 10u32; // a是10u32的可变所有者
    let b = &mut a; // b成为a的可变引用
    *b = 20; // 通过可变引用b我们改变了a的值
    println!("{b}");
    let d = &mut b; // d和b同时成为a的不可变引用,这违反了一个变量同时只能有一个可变引用的规则，编译出错
    println!("{d}");
  }
  ```

  ```Rust
  fn main() {
    let mut a = 10u32; // a是10u32的可变所有者
    let b = &mut a; // b成为a的可变引用
    *b = 20; // 通过可变引用b我们改变了a的值
    println!("{b}"); // 打印出20，b被丢弃
    let d = &mut a; // d成为a的可变引用
    println!("{d}"); // 打印出20，d被丢弃，a被丢弃
  }
  ```
