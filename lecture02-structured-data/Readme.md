# Chapter 02. Structured Data

## Struct

- 러스트에서 손쉽게 Struct 데이터 타입을 만드는 2가지 방법이 있음.

  - Structs: C언어에서 사용하는 형태의 구조체

  - Enums: OCaml 언어 타입에서 사용하는 것과 비슷하고, 데이터는 여러 타입 중 하나가 될 수 있음.

- Struct와 Enum에는 데이터 타입에 대한 메서드를 정의하는 구현 블록(`impl`, 이후 impl이라 표기)이 하나 이상 있을 수 있습니다.

- Struct 타입 작성 규칙

  - 선언하는 타입 이름은 CamelCase, 내부의 필드 선언에는 snake_case를 사용함.

- Struct 내부의 field 접근할 때는 `.`을 사용해서 호출한다.

- Struct는 부분적으로 초기화하는 것이 안될 수 있음 (원문에서 may be accessed with dot notation)

  - 모든 필드 값을 초기화할 때 정의하거나 나중에 초기하화하기 위해 초기화가 안된 값을 선언하는 방식을 사용한다.

```rust
let mut p = Point { x: 19, y: 8 };
p.x += 1;
p.y -= 1;
```

- Struct는 field 단위로 mut을 선언할 수 있음.

- 한 가지 명심할 것은 mutability는 variable binding의 속성이라 타입의 가변성을 선언하기 위한 목적으로 사용하지 않음.

  - 때문에 Field 수준의 mutability는 Cell Type을 통해 작성될 수 있음.

```rust
struct Point {
    x: i32,
    mut y: i32, // Illegal!
}
```

- Struct는 모듈 이름으로 네임스페이스가 지정됩니다.

  - 예를 들어, foo라는 모듈 내부에 Point라는 Struct를 선언했다면, `foo::Point`로 접근 가능.

- Struct의 field들은 기본적으로 Private임.

  - public하게 사용하려면 pub 예약어를 사용해서 외부에서 접근하게 변경할 수 있음.

- Private field는 Struct가 선언된 모듈 내에서만 액세스할 수 있습니다.

- 아래 예시는 `mod` 밖에서 Point struct에 접근할 때 발생하는 에러 예시임.

```rust
mod foo {
    pub struct Point {
        pub x: i32,
        y: i32,
    }
}
fn main() {
    let b = foo::Point { x: 12, y: 12 };
    //      ^~~~~~~~~~~~~~~~~~~~~~~~~~~
    // error: field `y` of struct `foo::Point` is private
}
```

- 반대로 같은 mod 내부에 있으면 접근 가능함.

```rust
mod foo {
    pub struct Point {
        pub x: i32,
        y: i32,
    }
    // Creates and returns a new point
    pub fn new(x: i32, y: i32) -> Point {
        Point { x: x, y: y }
    }
}
```

- match 문을 사용해서 구조체 분해 (JS에선 비구조화 할당이라 부름)

```rust
pub struct Point {
    x: i32,
    y: i32,
}
match p {
    Point { x, y } => println!("({}, {})", x, y)
}
```

- match 문을 Struct에 사용할 때 몇 가지 팁

  - Field를 비구조화할 때 순서는 필요 없음.

  - 중괄호 안에 필드를 나열하여 해당 변수 이름에 구조체 멤버를 바인딩합니다.

    - 바인딩된 변수를 변경하려면 struct_field: new_var_binding을 사용합니다.

  - 필드 생략: 이름 없는 모든 필드를 무시하려면 ...을 사용합니다. (Javascript에서 rest parameter에 해당.)

- 반대로 어떤 Struct를 초기화할 때, 다른 구조체 `s`에서 특정 또는 전체 필드를 가져올 때 `..`를 사용합니다.

- 그래서 초기화할 때 지정하지 않은 모든 필드는 대상 Struct에서 복사됩니다.

- 초기화한 Struct는 복사한 구조체와 동일한 타입이어야합니다.

  - 다른 타입의 Struct에서 같은 유형의 필드를 복사할 수 없음.

```rust
struct Foo { a: i32, b: i32, c: i32, d: i32, e: i32 }
let mut x = Foo { a: 1, b: 1, c: 2, d: 2, e: 3 };
let x2 = Foo { e: 4, .. x };
// Useful to update multiple fields of the same struct:
x = Foo { a: 2, b: 2, e: 2, .. x };
```

## Tuple Struct

- Tuple Struct 형태의 선언도 가능

  - 개별 field 접근할 땐 `x.0, x.1` 형태로 접근.

```rust
struct Color(i32, i32, i32);
let mut c = Color(0, 255, 255);
c.0 = 255;
match c {
    Color(r, g, b) => println!("({}, {}, {})", r, g, b)
}
```

- Tuple Struct는 별칭이 아닌 새로운 타입을 생성할 때 유용함.

- 아래 예제에서 두 타입은 구조적으로는 동일하지만, 등가(equatable)가하지 않음. (These two types are structurally identical, but not equatable.)

```
참고 자료

Identical은 두 객체가 완전히 같은 메모리 위치를 가리키는 것을 의미합니다. 이는 구조적으로나 값적으로 동일한 것을 나타냅니다. 예를 들어, 두 객체가 같은 메모리 주소를 가리킨다면, 이들은 identical합니다.

반면에 Equatable은 두 객체가 동등함을 비교할 수 있는 프로토콜입니다. 즉, 두 객체가 동일한 값을 가지고 있는지를 비교할 수 있습니다. Equatable 프로토콜을 채택한 타입은 해당 프로토콜을 준수하여 동등성을 비교할 수 있습니다. 예를 들어, 두 객체가 같은 값으로 동등하다면, 이들은 equatable합니다.
```

```rust
// Not equatable
struct Meters(i32);
struct Yards(i32);
// May be compared using `==`, added with `+`, etc.
type MetersAlias = i32;
type YardsAlias  = i32;
```

## Unit Structs (Zero-Sized Types)

- Zero Size의 Struct 구성이 가능함.

  - 즉, No Field!

- 다른 데이터 구조에서 타입을 "marker"하는 형태로 사용 가능.

  - 예를 들어 컨테이너가 저장하는 데이터의 유형을 표시하는 데 유용합니다.

```rust
struct Unit;
let u = Unit;
```

## Enums

- 여러 가지 중 하나일 수 있는 일부 데이터를 표현하는 방법

- Java, C#, C++에서보다 강력하게 사용 가능.

- 각 enum은 Sturct와 동일하게 사용 가능.

  - Unit Variant
  - Struct Variant
  - Tuple Variant

- Struct와 동일하게 Resultish::\* 형태로 모듈 호출하는 것처럼 사용.

```rust
enum Resultish {
    Ok,
    Warning { code: i32, message: String },
    Err(String)
}

match make_request() {
    Resultish::Ok =>
        println!("Success!"),
    Resultish::Warning { code, message } =>
        println!("Warning: {}!", message),
    Resultish::Err(s) =>
        println!("Failed with error: {}", s),
}
```

## Recursive Types

- 아래 예제처럼 함수형 스타일의 `List`라는 타입을 선언할 수 있음.

```rust
enum List {
    Nil,
    Cons(i32, List),
}
```

- 문제는 이러한 정의는 컴파일 시 크기가 무한대로 커지는 상황이 발생함.

- 구조체와 열거형은 기본적으로 인라인으로 저장되므로 재귀적이지 않을 수 있음.

- 즉, 명시적으로 지정하지 않는 한 요소는 참조로 저장되지 않음.

- 컴파일러는 이 문제를 해결하는 방법을 알려주지만, 상자는 무엇일까요? (정말 원문 그대로 box를 의미)

### 참고자료: Boxes

- box(소문자)란 Rust에서 데이터를 Heap에 할당하는 한 가지 방법 중 하나임.

- Box\<T\>(대문자)는 정확하게 하나의 owner를 갖고 있는 heap pointer임.

  - Box는 유일하게 데이터를 소유하고 있으며, alias를 사용할 수 없음.

- Box들은 scope를 벗어나면 소멸됨.

- Box 생성할 땐 `Box::new()` 사용.

```
let boxed_five = Box::new(5);
enum List {
    Nil,
    Cons(i32, Box<List>), // OK!
}
```

## Methods

```rust
impl Point {
    pub fn distance(&self, other: Point) -> f32 {
        let (dx, dy) = (self.x - other.x, self.y - other.y);
        ((dx.pow(2) + dy.pow(2)) as f32).sqrt()
    }
}
fn main() {
    let p = Point { x: 1, y: 2 };
    p.distance();
}
```

- method는 impl 블록 내에서 structs or enums를 실행할 수 있음.

- Field 접근하는 것처럼 `.`을 사용해서 접근이 가능함.

- pub을 사용해서 impl 내부 값에 접근 가능.

- impl 자체는 pub을 선언해서 사용하지 않음.

- 열거형에 대해 구조체와 똑같은 방식으로 작업함.

- 메서드의 첫 번째 인수인 self는 메서드에 필요한 소유권의 종류를 결정합니다.

- `&self`: 메서드가 값을 차용합니다.

  - 다른 소유권 모델이 필요하지 않는 한 이 인수를 사용합니다.

- `&mut self`: 메서드가 값을 가변적으로 차용합니다.

  - 함수는 호출되는 구조체를 수정해야 합니다.

- `self`: 메서드가 소유권을 가집니다.

  - 함수는 값을 소비하고 다른 것을 반환할 수 있습니다.

```rust
impl Point {
    fn distance(&self, other: Point) -> f32 {
        let (dx, dy) = (self.x - other.x, self.y - other.y);
        ((dx.pow(2) + dy.pow(2)) as f32).sqrt()
    }
    fn translate(&mut self, x: i32, y: i32) {
        self.x += x;
        self.y += y;
    }
    fn mirror_y(self) -> Point {
        Point { x: -self.x, y: self.y }
    }
}
```

- `distance`는 필드에 액세스해야 하지만 수정할 수는 없습니다.

- `translate`는 구조체 필드를 수정합니다.

- `mirror_y`는 완전히 새로운 구조체를 반환하여 이전 구조체를 사용합니다.

## Associated Functions

```rust
impl Point {
    fn new(x: i32, y: i32) -> Point {
        Point { x: x, y: y }
    }
}
fn main() {
    let p = Point::new(1, 2);
}
```

- Associated Funtions: 메서드와 비슷하지만 self를 사용하지 않음.

  - namespacing syntax인 `Point::new()`를 사용함.
    - Point.new() 형태로 사용하지 않음.
    - 자바의 Static method와 비슷함.

- 생성자(Constructor)와 유사한 동작을 하는 함수의 이름은 `new` 임.

  - 러스트에서는 생성자라는 고유한 개념도 없고, 자동 생성도 없음.

## Implementations

- rust에서 methods, associated funtions, 그리고 함수는 일반적으로 overloaded 될 수 없음.

  - function overloading or method overloading: 구현이 다른 동일한 이름의 함수를 여러 개 생성하는 기능

  - 예: `Vec::new()`와 `Vec::with_capacity(capacity: usize)`는 모두 Vec의 생성자

- 메소드는 상속되지 않을 수 있음.

  - 대신 Rust Struct & Enums가 반드시 구성되어야함.
  - Traits를 사용하면 기본 상속이 존재함.

## Patterns

- `...`를 사용해서 value의 범위 지정 가능.

  - numerics & chars

- 변수 바인딩과 같이 값에 바인딩하려면 `_`를 사용하고 바인딩을 삭제

```rust
let x = 17;
match x {
    0 ... 5 => println!("zero through five (inclusive)"),
    _ => println!("You still lose the game."),
}
```

## match:References

- `&`와 `ref`는 동일한 표현임.

- mut에 대한 참조를 얻을 땐, `ref mut`를 사용한다.

```rust
let mut x = 17;
match x {
    ref r if x == 5 => println!("{}", r),
    ref mut r => *r = 5
}
```

## if-let Statement

- if let 문법은 if와 let을 조합하여 하나의 패턴만 매칭 시키고 나머지 경우는 무시하는 값을 다루는 덜 수다스러운 방법을 제공

- if let을 이용하는 것은 사용자가 덜 타이핑하고, 덜 들여 쓰기 하고, 보일러 플레이트 코드를 덜 쓰게 된다는 뜻

## while-let Statement

- if let과 비슷하지만, 조건이 일치하지 않을 때까지 반복합니다.

## Inner Bindings

- 더 복잡한 데이터 구조의 경우 `@`를 사용하여 내부 요소에 대한 변수 바인딩을 생성.

## Lifetimes

- Ownership을 이해하기 위한 퍼즐 중 하나: Lifetime

- Lifetime은 일반적으로 꽤 가파른 학습 곡선을 가지고 있습니다.

- 필요한 경우 나중에 이 과정의 후반부에서 더 넓은 범위로 다시 다룰 수 있습니다.

- Lifetime을 이해하기 위한 시나리오

      - 리소스를 획득
      - 리소스에 대한 참조를 빌려줌.
      - 리소스를 다 사용했다고 판단하여 리소스를 할당 해제
      - 하지만 여전히 리소스에 대한 참조를 보유는 유지되고 있는데, 이를 본 소유자가 사용하기로 결정
      - 충돌 발생

- Rust가 이 시나리오를 불가능하게 만든다고 이미 말씀드렸지만, 어떻게 불가능한지는 설명하지 않았음.
- 컴파일러에게 3단계가 4단계 이전에 절대 발생하지 않는다는 것을 증명해야함.
