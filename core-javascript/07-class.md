# Class

## Class와 Instance

- Class : 공통 속성을 모아서 추상화한 것
- Instance : class의 구체적인 예시

## JavaScript에서 Class

- JavaScript는 prototype 기반 언어이지만 class 관점에서 비슷하게 해석할 수 있음
  ```javascript
  // Class
  function Rectangle(width, height) {
    this.width = width;
    this.height = height;
  }
  
  // Instance(or prototype) method
  Rectangle.prototype.getArea = function () {
    return this.width * this.height;
  }

  // Static method
  Rectangle.isRectangle = function (instance) {
    return instance instanceof Rectangle && 
      instance.width > 0 && 
      instance.height > 0;
  }

  // Instance
  const rect = new Rectangle(3, 4);
  console.log(rect.getArea()); // ✅ 12
  console.log(rect.isRectangle(rect)); // ❌ error
  console.log(Rectangle.isRectangle(rect)); // ✅ true
  ```
  - Class -> 생성자 함수
  - Instantiate -> `new` keyword
  - Instance member -> `prototype` 객체에 선언해서 instance가 접근할 수 있는 member
  - Static method -> 생성자 함수에 선언해서 instance가 접근할 수 없는 member
- JavaSscript는 instance에서 직접 method를 정의할 수 있기 때문에 'instance member' 보다 '**prototype member**'로 부르는게 일반적

## 상속

- Prototype chaining을 활용해서 상속을 구현할 수 있음
- Prototype chain을 형성하는 가장 쉬운 방법은 하위 class의 `prototype`에 상위 class의 instance를 부여하는 것
  ```javascript
  function Grade(...args) {
    // 유사 배열 생성
    for (const i = 0; i < args.length; i++) {
        this[i] = args[i];
    }
    this.length = args.length;
  }

  // 배열을 prototype으로 연결해서 `Array.prototype`을 상속받음
  Grade.prototype = [];

  const grade = new Grade(100, 80);

  // Instance에서 `Array.prototype`에서 상속받은 `push()` method 접근 가능
  grade.push(90);
  console.log(grade); // Grade { 0: 100, 1: 80, 2: 90, length: 3 }
  ```
- 하지만, 이 방법은 구조적 안정성이 떨어지고 class의 추상성을 해침
  - `Grade` class의 예시에서,
    ```javascript
    delete grade.length;
    grade.push(70);
    console.log(grade); // Grade { 0: 70, 1: 80, 2: 90, length: 1 }
    ```
    1. `Grade` instance는 여전히 배열이 아닌 일반 객체의 성질을 갖고 있으므로 `length` property를 삭제할 수 있음
    2. `push()` method를 호출하면 이제 `grade` instance가 아닌 `Grade.prototype`에 할당된 빈 배열의 `length`를 사용
    3. `Array.prototype.push()`는 `length` 값에 해당하는 index로 값을 저장한 뒤 `length`를 1 증가시키도록 동작함
    4. 결과적으로, `grade`의 `0` index의 값 `100`을 `70`으로 덮어쓰고 빈 배열의 `length`를 `0`에서 `1`로 증가시키도록 동작
  - 즉, **class에 있는 값이 instance의 동작에 영향을 줘서 문제가 발생함**
    - 상위 class에 해당하는 배열이 `length` 등의 구체적인 데이터를 가지고 있음
    - `grade` instance만 `length` 속성을 가졌다면 예상치 못한 동작이 일어나는 대신 오류를 감지할 수 있을 것
- 이런 문제를 방지하려면 **class는 instance가 사용할 method만을 갖는 추상적인 '틀'로서만 작성되어야 함**

### Class가 구체적인 데이터를 갖지 않도록 만드는 방법

```javascript
function Rectangle(width, height) {
    this.width = width;
    this.height = height;
}
Rectangle.prototype.getArea = function () {
    return this.width * this.height;
}

function Square(width) {
    Rectangle.call(this, width, width); // Rectangle 상속
}
```

1. `prototype`이 가진 property들을 일일이 삭제한 뒤 `Object.freeze()`로 새로운 property를 추가할 수 없도록 설정
    ```javascript
    // 1. Rectangle.prototype을 상속받기 위해 instance 할당
    Square.prototype = new Rectangle();

    // 2. `undefined`로 초기화된 `Rectangle` instance의 속성 삭제
    delete Square.prototype.width;
    delete Square.prototype.height;

    // 3. `Rectangle` instance에 다른 구체적인 데이터를 추가할 수 없도록 제한
    Object.freeze(Square.prototype);
    ```
2. 상위 class와 하위 class 사이에 빈 `Bridge` instance를 사용
    ```javascript
    // 1. 빈 생성자 함수 `Bridge` 정의
    function Bridge() {}

    // 2. `Bridge.prototype`을 상위 class의 `prototype`으로 교체
    Bridge.prototype = Rectangle.prototype;

    // 3. 하위 class의 `prototype`에 `Bridge`의 instance를 할당
    Square.prototype = new Bridge();
    Object.freeze(Square.prototype);
    ```
    - 처음부터 구체적인 데이터를 갖지 않는 빈 instance를 할당해서 property 삭제 과정을 생략
    - `Bridge.prototype`을 상위 class의 `prototype`으로 교체해서 상속 관계는 유지
3. `Object.create()` 사용
    ```javascript
    Square.prototype = Object.create(Rectangle.prototype);
    Object.freeze(Square.prototype);
    ```
    - 하위 class의 `prototype`이 상위 class의 `prototype`을 직접 참조
    - 상위 class의 instance가 없으므로 prototype chain 상에 다른 구체적인 데이터가 생성되지 않아서 안전

### 하위 class에서 생성자(constructor) 복구

- 상속을 구현하면서 하위 class의 `prototype`이 상위 class의 `prototype`을 가리키게 됨
- 그 과정에서 하위 class의 `constructor`가 상위 class를 가리키는 문제가 있음
- 하위 class의 `constructor`를 직접 변경해야 함
    ```javascript
    Square.prototype.constructor = Square;
    ```

## ES6의 class와 상속

- ES6 부터 class 문법이 도입됨
- ES5와 ES6에서 class 구현 비교
    - ES5
        ```javascript
        // Class
        function Person(name) {
            this.name = name;
        }

        // Static method
        Person.staticMethod = function () {
            return this.name + " staticMethod";
        }

        // Instance(prototype) method
        Person.prototype.method = function () {
            return this.name + " method";
        }

        // Instantiate
        const person = new Person("kim");
        ```
    - ES6 -> `class` keyword 사용
        ```javascript
        // Class
        class Person {
            constructor (name) {
                this.name = name;
            }

            // Static method
            static staticMethod() {
                return this.name + " staticMethod";
            }

            // Instance(prototype) method
            method() {
                return this.name + " method";
            }
        }

        // Instantiate
        const person = new Person("kim");
        ```
- ES5와 ES6에서 상속 비교
    - ES5
        ```javascript
        function Rectangle(width, height) {
            this.width = width;
            this.height = height;
        }
        Rectangle.prototype.getArea = function () {
            return this.width * this.height;
        }

        function Square(width) {
            Rectangle.call(this, width, width); // 상위 class 생성자 사용
        }
        Square.prototype = Object.create(Rectangle.prototype); // 상위 class member 상속
        Square.prototype.constructor = Square; // constructor 복구
        Object.freeze(Square.prototype);
        ```
    - ES6 -> `extends` keyword 사용
        ```javascript
        class Rectangle {
            constructor (width, height) {
                this.width = width;
                this.height = height;
            }
            getArea() {
                return this.width * this.height;
            }
        }

        class Square extends Rectangle {
            constructor (width) {
                super(width, width); // 상위 class 생성자 사용
            }
            getArea() { // 상위 class의 `getArea`를 상속받지 않고 override
                console.log("size is :", super.getArea()); // 상위 class의 member에 접근
            }
        }
        ```