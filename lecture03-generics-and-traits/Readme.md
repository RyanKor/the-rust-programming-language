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
