# This

## `this`가 결정되는 방식

- `this`는 함수가 호출될 때(실행 context가 생성될 때) 결정되어 binding 됨

### 전역 공간에서 this

- 전역 공간에서 `this`는 **전역 객체**
- 브라우저 : `window` 객체
- Node.js : `global` 객체

> **전역 공간에서 전역 변수의 성질**
> 
> - 전역 변수는 전역 객체(`window` 또는 `global`)를 통해서도 접근할 수 있다.
>     ```javascript
>     var a = 1;
>     console.log(a); // 1
>     console.log(window.a); // 1
>     console.log(this.a); // 1
>     ```
>     - JavaScript의 모든 변수는 실행 context 생성 시 `LexicalEnvironment`의 `environmentRecord` 객체상에 등록된다.
>     - 전역 공간에서 실행 context는 `LexicalEnvironment`가 전역 객체를 참조하는 `GlobalEnv`를 참조한다.
>     - 따라서, 전역 변수를 선언하면 전역 context의 `LexcialEnvironment`의 property로 등록되어 전역 객체를 통해 접근할 수도 있다.
>     - 또한, 전역 공간에서 `this`가 전역 객체를 참조하므로 `this`를 통해서 전역 변수에 접근할 수도 있다.
> - 전역 변수를 선언하는 것과 전역 객체에 property를 추가하는 것의 차이
>     - 전역 객체에 직접 추가한 전역 변수에 대해 `delete` 연산자를 사용하면 전역 객체에서 전역 변수 property가 삭제된다.
>         ```javascript
>         window.a = 1;
>         delete window.a;
>         console.log(a, window.a, this.a); // Uncaught ReferenceError: a is not defined
>         ```
>     - 전역 변수에 `delete` 연산자를 적용하면 전역 객체의 property가 **삭제되지 않는다.**
>         ```javascript
>         var a = 1;
>         delete a; // `window.`이 생략된 것과 같으므로 `delete` 연산자를 사용할 수 있다.
>         console.log(a, window.a, this.a); // '1 1 1'이 정상 출력된다. (property가 삭제되지 않았다.)
>         ```
>     - 전역 변수 선언 시 전역 객체의 property로 자동 추가될 때 **`configurable` 속성이 `false`로 설정**되기 때문에 변경 및 삭제되지 않는다.

### 함수 또는 method 내부에서 this

- 함수와 method의 차이
    - 함수와 method는 '독립성'에 차이가 있음
    - 함수는 그 자체로(독립적으로) 기능을 수행하지만, method는 자신을 호출한 대상 객체에 관련된 동작을 수행
- JavaScript는 `this` binding으로 함수와 method의 차이를 구현함
    - 함수로서 호출했는지 method로서 호출했는지 여부만 알면 `this`에 binding되는 대상을 정확히 예측할 수 있음
    - 함수를 호출할 때 `this`는 **전역 객체**
        - 함수로서의 호출은 호출 주체에 대한 정보를 알 수 없음
        - 실행 context가 활성화될 때 `this`가 지정되지 않았다면 전역 객체로 binding 함
        - 따라서, 함수 호출 시 `this`는 전역 객체로 binding
    - Method를 호출할 때 `this`는 **호출한 객체**
        - Method와 가장 가까이 있는 점(`.`)의 앞(또는, 대괄호(`[method]`) 앞) 부분에 해당하는 객체가 binding됨
            ```javascript
            var obj = {
                inner: {
                    someMethod: function () { console.log(this); },
                }
            };

            // 아래 두 줄은 모두 `obj.inner` 객체를 console에 출력
            obj.inner.someMethod();  
            obj.inner[someMethod]();
            ```
        - 함수가 선언된 주변 환경(e.g. method 내부인지, 함수 내부인지 등)은 중요하지 않음
        - **Method 호출 시점에 호출 주체가 누구인지가 중요**
- `this`의 설계상 오류
    - 호출 주체가 없을 때 `this`에 전역 객체를 binding하는 동작은 자연스럽지 않음
    - 호출 주체가 없을 때 **호출 당시의 주변 환경으로부터 `this`를 상속**받아서 사용할 수 있어야 함
    - ES5 까지는 지역 변수 `self`에 외부 함수의 `this` 참조를 복사하는 방법으로 해결
        ```javascript
        var obj = {
            outer: function () {
                /* 1. 내부 함수 `inner1`를 단독으로 호출할 때,
                 *    `inner1` 내부의 `this`는 `inner1` 함수에 binding된 `this`를 참조하므로 전역 객체 출력
                 */
                console.log(this); // `outer` method를 호출한 주체인 `obj` 객체 출력
                var inner1 = function () {
                    console.log(this); // 호출 주체가 없으므로 전역 객체 출력 (e.g. window, global 등)
                }
                inner1();

                /* 1. 내부 함수 `inner1`가 `this` 대신 외부 함수 `outer`의 `this`를 `self` 변수를 통해 참조하므로
                 *    `inner1`를 단독으로 호출해도 외부 함수의 `this`를 상속받는 것 처럼 동작한다.
                 */
                var self = this;
                var inner2 = function () {
                    console.log(self); // `outer` method에 binding된 `this`를 참조하므로 `obj` 객체 출력
                }
                inner2();
            }
        };
        obj.outer();
        ```
    - ES6 부터는 자기 자신의 실행 context에 `this`를 binding 하지 않는 화살표 함수를 사용해서 해결
        ```javascript
        var obj = {
            outer: function () {
                console.log(this); // `outer` method를 호출한 주체인 `obj` 객체 출력
                var inner = () => {
                    console.log(this); // 화살표 함수는 `this` binding을 갖지 않으므로 `outer`의 `this`를 참조
                }
                inner();
            }
        };
        obj.outer();
        ```

### Callback 함수에서 this

- Callback 함수 : 다른 함수 또는 method의 argument로 전달해서 **제어권을 넘겨준** 함수
- 다른 함수 또는 method가 callback 함수의 제어권을 가지고 실행 시점이나 `this` binding 등을 결정함
- Callback 함수의 `this` binding을 결정하지 않는 경우 (`this`가 전역 객체)
    - `setTimeout()`
    - `forEach()`
- Callback 함수의 `this` binding을 내부에서 결정해 주는 경우
    - `addEventListener()` : `this`에 handler를 등록한 event가 발생한 HTML element를 binding
        ```javascript
        const element = document.body.querySelector("#a");
        element.addEventListener("click", function (event) {
            console.log(this, event); // `this`가 `element`로 binding 됨
        });
        ```

### 생성자 함수에서 this

- 함수를 `new` keyword와 함께 호출하면 생성자 함수로 동작
- 생성자 함수는 **생성자 함수의 `prototype` property를 참조하는 `__proto__` property를 가진 객체**를 만들고 `this`에 binding
- 생성자 함수 내부에서 `this`를 통해 이 객체 instance에 접근해서 property 및 method들을 추가
- 생성자 함수는 `return`문이 없으면 암묵적으로 `this`를 반환 => **객체를 생성**해 주는 함수로서 동작