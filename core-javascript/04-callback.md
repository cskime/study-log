# Callback Function

## Callback 함수와 제어권

- 제어권 : 함수의 호출 시점, 전달 인자, `this`를 결정하는 책임
- Callback 함수 : 전달 인자로 다른 함수에 전달하는 함수
- Callback 함수의 호출 주체는 인자로 전달한 함수
- 즉, callback 함수는 다른 함수에 전달되면서 **제어권도 함께 위임**함

### 함수 호출 시점 제어

```javascript
let count = 0;
function callbackFunction() {
    console.log(count);
    if (++count > 4) {
        clearInterval(timer);
    }
};
const timer = setInterval(callbackFunction, 300);
```

- `setInterval(callback, delay[, param1, param2, ...])` 함수는 `delay`마다 `callback` 함수를 실행하도록 **호출 시점을 제어**

### 전달 인자 제어

```javascript
function callbackFunction(currentValue, index, array) {
    console.log(currentValue, index, array);
    return currentValue + 10;
}
const newArray = [10, 20, 30].map(callbackFunction);
console.log(newArray); // [20, 30, 40]
```

- `Array.prototype.map(callback[, thisArg])` 함수는 `callback` 함수를 호출할 때 **전달되는 인자의 위치와 종류를 제어**
- `map()` 함수는 `callback` 함수로 '현재 값', 'index', '대상 배열' 값을 순서대로 전달
- `callback` 함수는 `map()`이 전달하는 인자들을 순서대로 받아서 사용해야 함

### `this` 제어

- Callback 함수는 기본적으로 함수로 호출되기 때문에, `this`를 직접 지정하지 않으면 전역 객체를 참조
- 제어권을 넘겨받는 함수가 callback 함수의 `this` 참조를 제어하는 방법
    1. `this`를 전역 객체로 설정
        ```javascript
        setTimeout(function () {
            console.log(this); // Window { ... } (전역 객체)
        }, 300);
        ```
        - `setTimeout(callback, delay[, param1, param2, ...])`과 같은 함수는 `thisArg`를 설정하는 방법을 제공하지 않음
        - `callback` 함수에 `this`를 설정하려면 `bind()` method를 사용해서 `this` 참조를 할당한 뒤 callback으로 전달 ([참고](https://developer.mozilla.org/ko/docs/Web/API/Window/setTimeout#this_%EB%AC%B8%EC%A0%9C))
    2. `thisArg`를 전달 인자로 받아서 `call`/`apply` method로 callback의 `this`를 설정
        ```javascript
        Array.prototype.map = function (callback, thisArg) {
            const mappedArray = [];
            for (const i = 0; i < this.length; i++) {
                const mappedValue = callback.call(
                    thisArg || window, 
                    this[i], // 현재 값
                    i,       // index
                    this     // 현재 배열
                );
                mappedArray[i] = mappedValue;
            }
            return mappedArray;
        };
        ```
        - Prototype에 등록한 함수의 `this`는 공통 instance를 참조
        - `call()` method로 `callback` 함수의 `this` 참조 설정 (`thisArg` or 전역 객체)
        - e.g. `forEach(callback[, thisArg])`, `map(callback[, thisArg])` 등
    3. 내부적으로 호출 주체를 `this`로 설정
        ```html
        <html>
            <head>...</head>
            <body>
                <button id="a">Click</button>
                <script>
                    document.body
                        .querySelector("#a")
                        .addEventListener("click", function (event) {
                            console.log(this); // <button id="a">Click</button>
                        });
                </script>
            </body>
        </html>
        ```

## Callback 함수는 함수다 (`this` 유실)

```javascript
const obj = {
    values: [1, 2, 3],
    logValues: function (value, index) {
        console.log(this, value, index);
    }
};

obj.logValues(1, 2); // { values: [1, 2, 3], logValues: f } 1 2
[4, 5, 6].forEach(obj.logValues); // `this`가 `obj` 객체가 아님
// Window { ... } 4 0
// Window { ... } 5 1
// Window { ... } 6 2
```

- 객체의 method를 method로서 호출하면 `this`는 호출 주체인 객체를 참조
- 객체의 method를 callback으로 전달하면 함수가 전달되는 것이므로, callback이 실행될 때는 함수로 실행됨
    - 함수 또는 method를 호출하는 주체에 따라 `this`가 결정됨

## Callback 함수 내부의 `this`에 다른 값 바인딩

- 과거에는 `self` 변수에 `this` 참조를 저장하고, `this` 대신 `self` 변수를 사용하는 클로저를 만드는 방식을 사용
    ```javascript
    const obj = {
        name: "obj",
        func: function () {
            const self = this;
            return function () {
                console.log(self.name);
            };
        }
    };
    const callback = obj.func;
    setTimeout(callback, 1000); // "obj"
    ```
    - `this`를 직접 사용하면 `setTimeout` 내부에서 `callback`이 실행될 때 호출 주체가 없으므로 `obj` 객체가 아닌 전역 객체로 설정될 것
- ES5 부터는 `bind()` method를 사용
    ```javascript
    const obj = {
        name: "obj",
        func: function () {
            console.log(this.name);
        }
    };
    setTimeout(obj.func.bind(obj), 1000); // "obj"
    ```

## Callback 지옥과 비동기 제어

- 비동기 코드 : 현재 실행 중인 코드(또는 함수)의 완료 여부와 무관하게 즉시 다음 코드로 넘어가는 코드
    - `setTimeout`, `addEventListener`, `XMLHttpRequest`, `fetch` 등
    - 별도의 요청을 보내거나, 실행을 대기하거나, 보류하는 코드
- 비동기 코드가 이전의 비동기 코드 실행 결과에 의존하는 경우, 감당하기 힘들 정도로 코드 들여쓰기 수준이 깊어져서 가독성이 떨어지고 수정이 어려워지는 문제 발생
    ```javascript
    setTimeout(() => { // first
        // ...
        setTimeout(() => { // second
            // ...
            setTimeout(() => { // third
                // ...
                setTimeout(() => { // fourth
                   // ... 
                }, 500);
            }, 500);
        }, 500);
    }, 500);
    ```
- Callback hell 문제를 해결하는 방법
    1. Callback으로 전달하는 익명 함수를 모두 기명 함수로 변경
        ```javascript
        function first() {
            // ...
            setTimeout(second, 500);
        }
        function second() 
            // ...
            setTimeout(third, 500);
        }
        function third() {
            // ...
            setTimeout(fourth, 500);
        }
        // ...

        setTimeout(first, 500);
        ```
    2. `Promise와` `Generator` 사용 (ES6+)
        - `Promise` : `resolve` 또는 `reject` 함수가 실행되어야 다음 코드(`then()`, `catch()` 등)가 실행되는 chaining 문법 사용
            ```javascript
            new Promise((resolve) => {
                setTimeout(() => { // first
                    // ...
                    resolve();
                }, 500);
            }).then(() => {
                return new Promise((resolve) => { // second
                    setTimeout(() => {
                        // ...
                        resolve();
                    }, 500);
                });
            }).then(...);
            ```
        - `Generator` : Generator 함수(`function*`)에서 `yield` keyword로 실행 흐름을 제어하고 `next()` method로 비동기 코드 실행
            ```javascript
            function execute() {
                setTimeout(() => {
                    maker.next();
                }, 500);
            }

            const generator = function* () {
                const result1 = yield execute();
                const result2 = yield execute();
                const result3 = yield execute();
                // ...
            }
            const maker = generator();
            maker.next();
            ```
    3. `async`/`await` 사용 (ES2017+)