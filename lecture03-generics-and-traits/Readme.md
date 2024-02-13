# Chapter 3. Generics & Traits

## Generic

- 제네릭은 컨셉의 복제를 효율적으로 다루기 위한 도구로서, 구체화된 타입이나 다른 속성들에 대하여 추상화된 대리인의 역할을 수행.

- 제네릭은 함수, 구조체, 열거형, 메소드를 정의할 때 사용할 수 있음.

## Generic Implementations

- struct & enums의 generic 타입을 impl 에서 사용하려면 impl block 사용할 때 함께 선언이 필요함.

```rust
impl<T, E> Result<T, E> {
    fn is_ok(&self) -> bool {
        match *self {
            Ok(_) => true,
            Err(_) => false,
        }
    }
}
```

## Traits

- 앞 부분에서 impl block에서 함수를 사용하는 것은 다 좋은데 정형화 되어 있지 않고, 어떤 타입이 무엇을 할 수 있는지에 대해 추상적으로 알 길이 현재 없음

- 트레잇(trait) 이란 러스트 컴파일러에게 특정한 타입이 갖고 다른 타입들과 함께 공유할 수도 있는 기능에 대해 말하는 것.

- 사용 방법은 implementation과 비슷하지만, 다르다.

  - Trait는 trait 라는 키워드를 사용하여 선언하며, trait 블럭 안에 공통 메서드의 원형(method signature)을 갖는다.

```rust
trait PrettyPrint {
    fn format(&self) -> String;
}
```

- Trait을 impl 내부에서 사용하고 싶으면, impl Trait for Type을 사용함.

  - 만약 trait 안에 여러 개의 메서드가 있으면 (그리고 trait의 디폴트 구현이 없으면) 모든 trait 메서드를 해당 타입에서 구현하여야 한다.

- impl block 하나 당 Trait 하나임.

- self/&self 모두 impl 내부에서 사용 가능.

## Generic Functions

- 함수의 generic 타입 당연히 선언 가능.

- 아래 예제는 `<T, U>` 를 선언해서 foo 라는 함수의 타입 매개 변수를 선언한 것.

  - x:T, y:U

```rust
fn foo<T, U>(x: T, y: U) {
    // ...
}
```

## Generic with Trait Bounds

- Trait Bound는 함수의 파라미터나 리턴에 트레이트를 전달하기 위한 표현 방식으로 제네릭을 사용하여 표현

- 모든 타입 받는 게 아니라, 제네릭 타입 사용해서 타입의 범위 지정 가능.

- 2개 이상의 Trait bound 사용할 땐 `+` 연산자를 사용해서 복수개의 제네릭 타입 설정 가능.

- 아직 negative trait bound는 구현 안됨.

  - 예를 들어, T라는 제네릭 타입에 대해 Clone 타입은 못 온다고 하는 등...

- 구조체도 trait bound 사용 가능.

- 구조체 헤더와 impl 블록 헤더에 모든 제네릭 타입을 선언을 해야함.

- 이 때, impl block header에만 trait bounds를 지정해주면 됨.

  - 이는 각각 다른 특성 바운드를 가진 구조체에 대해 여러 개의 임팩트를 갖고자 하는 경우에 유용

```rust
enum Result<T, E> {
   Ok(T),
   Err(E),
}
trait PrettyPrint {
   fn format(&self) -> String;
}
impl<T: PrettyPrint, E: PrettyPrint> PrettyPrint for Result<T, E> {
   fn format(&self) -> String {
      match *self {
         Ok(t) => format!("Ok({})", t.format()),
         Err(e) => format!("Err({})", e.format()),
      }
   }
}
```

- `Self(대문자)`는 `self(소문자)`를 참조하는 특수 타입임.

```rust
enum Result<T, E> { Ok(T), Err(E), }
// This is not the trait Rust actually uses for equality
trait Equals {
   fn equals(&self, other: &Self) -> bool;
}
impl<T: Equals, E: Equals> Equals for Result<T, E> {
   fn equals(&self, other: &Self) -> bool {
      match (*self, *other) {
         Ok(t1), Ok(t2) => t1.equals(t2),
         Err(e1), Err(e2) => e1.equals(e2),
         _ => false
      }
   }
}
```

## Inheritance

- 어떤 trait는 impl 실행이 되기 전에 다른 trait을 요구하는 상황이 있음.

  - 예를 들어, Eq 라는 trait는 반드시 PartialEq가 impl 되어야하고, Copy는 Clone이 필요함.

- 아래 예제에서는 Child Trait는 반드시 Parent implementation이 필요함.

```rust
trait Parent {
    fn foo(&self) {
        // ...
    }
}
trait Child: Parent {
    fn bar(&self) {
        self.foo();
        // ...
    }
}
```

## Default Methods

- Trait을 구현할 때, 메소드들에 대한 기본 impl 값을 갖는 게 가능함.

- 기본 impl이 제공되면, implementor trait은 해당 메소드 정의할 필요가 없음.

- trait의 default implementation 구현은 trait 블록에 작성하면 됨.

- 해당 trait의 implementor는 기본 값을 overwrite하면 됨.

  - 단, `ne` 정의 금지. eq와 ne 사이 관계 위배할 여지 높음.

```rust
trait PartialEq<Rhs: ?Sized = Self> {
    fn eq(&self, other: &Rhs) -> bool;
    fn ne(&self, other: &Rhs) -> bool {
        !self.eq(other)
    }
}
trait Eq: PartialEq<Self> {}
```

## Deriving

- 많은 특성은 매우 간단해서 컴파일러가 대신 구현해 주는 경우가 많음.

- `#[derive(...)]` 속성은 컴파일러에 사용자가 지정한 모든 특성에 대한 default implementation을 삽입하도록 지시합니다.

```rust
#[derive(Eq, PartialEq, Debug)]
enum Result<T, E> {
   Ok(T),
   Err(E)
}
```

- 아래 코어 trait을 삽입하는 게 가능함.

```rust
Clone, Copy, Debug, Default, Eq,
Hash, Ord, PartialEq, PartialOrd.
```

- custom traits을 deriving하는 것은 러스트 1.6 에서 무척 불안정한 feature임.

- 항상 deriving이 잘 동작하는 게 아님.

- 모든 멤버가 해당 특성을 파생할 수 있는 경우에만 데이터 유형에서 특성을 파생할 수 있습니다.
  예를 들어, f32는 Eq가 아니므로 f32만 포함된 구조체에서는 Eq를 파생할 수 없습니다.

## Clone

```rust
pub trait Clone: Sized {
    fn clone(&self) -> Self;
    fn clone_from(&mut self, source: &Self) { ... }
}
```

- T 타입 유형의 값을 복제하는 방법을 정의하는 Trait

- 소유권 문제 해결 가능

  - 소유권 가져오거나 대여라기보다는 값 자체를 복제해서 새로 생성

```rust
#[derive(Clone)] // without this, Bar cannot derive Clone.
struct Foo {
    x: i32,
}
#[derive(Clone)]
struct Bar {
    x: Foo,
}
```

```rust
pub trait Copy: Clone { }
```

- 그래서 `Copy`를 사용하는 것은 소유권 이동에 관한 의미를 갖는 것이 아닌 복제에 대한 의미를 내포함.

- 타입들은 비트 복사에 의해 복제되는 것이 되어야하고(memcpy) 참조 타입은 복제될 수 없음.

- marker trait: 추가적인 메서드를 구현하지 않고 대신 동작을 정의.

- 일반적으로 복사할 수 있는 유형은 복제하는 것이 좋음.

## Debug

```rust
pub trait Debug {
    fn fmt(&self, &mut Formatter) -> Result;
}
```

- `{:?}` 사용할 때 결과 값 정의하는 옵션

- 디버깅 결과 값을 생성하고, pretty print는 안됨.

- 디버깅할 땐 driving 항상 사용해야됨.

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}
let origin = Point { x: 0, y: 0 };
println!("The origin is: {:?}", origin);
// The origin is: Point { x: 0, y: 0 }
```

- 디버깅할 땐, 타입의 기본 값이 정의되어 있어야함.

```rust
pub trait Default: Sized {
    fn default() -> Self;
}
```

## Eq vs PartialEq

```rust
pub trait PartialEq<Rhs: ?Sized = Self> {
    fn eq(&self, other: &Rhs) -> bool;
    fn ne(&self, other: &Rhs) -> bool { ... }
}
pub trait Eq: PartialEq<Self> {}
```

- `==` 연산자 사용해서 값의 동등을 비교함.

- `PartialEq` 는 부분적인 등가 관계 표현

  - 대칭: if a==b then b==a
  - 추이적 성질(전이?): if a==b and b==c then a==c

- `ne`는 eq와 반대로 부정 동등 비교를 의미

- Eq는 등가 관계 표현

  - partialEq를 포함해서 자기 자신과의 비교도 포함됨
  - 재귀적: a == a

- marker trait: 추가적인 메서드를 구현하지 않고 대신 동작을 정의.

## Hash

```rust
pub trait Hash {
    fn hash<H: Hasher>(&self, state: &mut H);
    fn hash_slice<H: Hasher>(data: &[Self], state: &mut H)
        where Self: Sized { ... }
}
```

- Hash 타입.

- 위에서 H 타입 매개변수는 해시 값을 연산하는데 사용되는 추상 해시임.

- 만약 Eq Trait 실행하면 추가적인 중요한 속성 알 수 있음.

```rust
k1 == k2 -> hash(k1) == hash(k2)
```

## Ord vs. PartialOrd

```rust
pub trait PartialOrd<Rhs: ?Sized = Self>: PartialEq<Rhs> {
    // Ordering is one of Less, Equal, Greater
    fn partial_cmp(&self, other: &Rhs) -> Option<Ordering>;
    fn lt(&self, other: &Rhs) -> bool { ... }
    fn le(&self, other: &Rhs) -> bool { ... }
    fn gt(&self, other: &Rhs) -> bool { ... }
    fn ge(&self, other: &Rhs) -> bool { ... }
}
```

- 정렬된 순서의 값과 비교하는 Trait임.

- 비교는 아래 조건들을 반드시 충족할 수 있어야함.

  - 비대칭 : if a < b then !(a > b), as well as a > b implying !(a < b); and

  - 전이 : a < b and b < c 면 a < c. ==와 > 연산자에도 동일 적용.

- lt, le, gt, ge 모두 parctial_cmp 기반으로 default implementation이 존재함.

```rust
pub trait Ord: Eq + PartialOrd<Self> {
    fn cmp(&self, other: &Self) -> Ordering;
}
```

- Ord는 전체 순서를 구성하는 타입에 대한 Trait.

- Ord trait에 대한 조건을 충족하면 형성되는 순서가 전체 순서가 됨.

- 이 trait이 derived 되면, 사전적인 순서로 값이 정렬됨.

## Associated Types

- Rust Book에서 가져온 Graph trait을 확인.

```rust
trait Graph<N, E> {
    fn edges(&self, &N) -> Vec<E>;
    // etc
}
```

- N과 E는 제네릭 타입이지만, 그래프에서 의미를 갖지 않음.

- 그리고 그래프 trait을 사용하는 모든 함수는 N과 E에 대해 제네릭 타입으로 선언이 되어야함.

- 이러산 상황에 대한 솔루션이 Associated type

- `type` 이라 선언되어 있는 trait block 내부 값이 trait의 내부에 있는 제네릭 타입과 연관되어 있음.

- trait의 implementor는 연관된 유형이 무엇에 해당하는지 지정할 수 있습니다.

```rust
trait Graph {
  type N;
  type E;
  fn edges(&self, &Self::N) -> Vec<Self::E>;
}
impl Graph for MyGraph {
  type N = MyNode;
  type E = MyEdge;
  fn edges(&self, n: &MyNode) -> Vec<MyEdge> { /*...*/ }
}
```

- 예시로, rust 표준 라이브러리에서 반복자(Iterator)같은 trait은 Item과 연관된 타입을 정의함.

- `Iterator::next` 같은 trait은 `Option<Self::Item>`을 반환함.

  - 이러한 Trait은 사용자가 반복자가 컬렉션을 순회하면서 어떤 타입을 클라이언트가 얻을지 쉽게 구체화할 수 있음.

## Trait Scope

- Foo라는 trait을 프로그램에서 정의했다고 가정.

- 러스트에서 어떤 타입을 이 trait에서 실행하든, 우리가 소유하고 있지 않은 타입에 대해서도 implement가 가능하다.

- 하지만 이건 무척 안좋은 예제이므로, 최대한 피하는 게 좋다.

```rust
trait Foo {
   fn bar(&self) -> bool;
}
impl Foo for i32 {
    fn bar(&self) -> bool {
        true
    }
}
```
