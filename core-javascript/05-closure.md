# Closure

## Closure란?

- _"클로저는 함수와 함수가 선언될 당시의 lexical environment의 조합(combination)이다." ([MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Closures))_
- '함수가 선언될 당시의 lexical environment'란 함수의 `outerEnvironmentReference`가 참조하는 외부 실행 컨텍스트의 `LexicalEnvironment`를 의미
- `outerEnvironmentReference`를 통해 외부 실행 컨텍스트의 `LexicalEnvironment`에 접근하는 것은 내부 함수가 외부 함수에 선언된 변수를 참조하는 경우 밖에는 없다.
- 즉, 클로저는 **어떤 함수에서 선언한 변수를 참조하는 내부 함수에서만 발생하는 현상**이다.

## Closure 동작 원리

- Closure는 **외부 함수에 선언된 지역 변수를 참조하는 내부 함수를 외부로 전달할 때 외부 함수의 실행 컨텍스트가 종료된 후에도 외부 함수에 선언된 지역 변수에 접근할 수 있는 매커니즘**
    - 내부 함수가 외부로 전달되는 경우
        1. `return` 문으로 반환
        2. 다른 함수의 callback으로 전달
- 또는, **어떤 함수에서 선언한 변수를 참조하는 내부 함수를 외부로 전달할 때 함수 실행 컨텍스트가 종료된 후에도 해당 변수가 사라지지 않는 현상**을 의미
- 즉, closure는 **함수 실행이 종료된 후에도 함수 지역 변수에 접근 가능한 현상**
- **가비지 컬렉터의 동작**에 의해 실행 종료된 외부 함수의 지역 변수에 접근 가능
    - 내부 함수가 외부 함수에 선언된 지역 변수에 접근하면, `outerEnvironmentReference`를 통해 외부 함수의 `LexicalEnvironment`에 접근하여 해당 변수를 참조하게 됨
    - 외부 함수의 실행 컨텍스트가 종료될 때 내부 함수의 실행 컨텍스트가 종료된다면,
        ```javascript
        function outer() {
            let a = 1;
            function inner() {
                console.log(++a);
            }
            inner();
        };
        outer();
        ```
        1. 외부 함수의 `LexicalEnvironment`에 대한 참조가 해제되고,
        2. 동시에 해당 변수의 참조도 해제되어,
        3. 해당 변수가 가비지 컬렉션 대상이 되어 메모리에서 제거될 수 있음
    - 외부 함수의 실행 컨텍스트가 종료될 때 내부 함수의 실행 컨텍스트가 유지된다면,
        ```javascript
        function outer() {
            let a = 1;
            function inner() {
                console.log(++a);
            }
            inner();
            // or
            return inner();
        };
        outer();
        ```
        1. 내부 함수가 외부 함수 밖으로 전달되어 실행 컨텍스트가 유지되고,
        2. 외부 함수의 `LexicalEnvironment`에 저장된 변수에 대한 참조가 유지되어,
        3. **해당 변수는 GC 되지 않고 메모리에 유지됨**
        4. 이후 내부 함수를 실행하면 메모리에 남아있는 외부 함수 변수에 접근 가능

## Closure와 메모리 관리

- Closure는 실행 종료된 함수의 지역 변수에 접근할 수 있도록 의도적으로 메모리를 사용하는 것
- 의도하지 않은 메모리 사용을 의미하는 '메모리 누수(leak)'와는 다르다.
- Closure에 의해 사용된 메모리를 더 이상 사용되지 않을 때 해제시켜 주는 과정은 필요하다.
- JavaScript는 참조 카운트가 0이 되면 GC에 의해 메모리가 해제되므로, **closure를 발생시키는 내부 함수에 대한 참조를 끊어서** 메모리를 정리할 수 있음
    ```javascript
    let outer = (function () {
        let a = 1;
        let inner = function () {
            return ++a;
        }
        return inner;
    })();
    console.log(outer()); // 2
    console.log(outer()); // 3
    outer = null; // inner 함수 참조를 끊어서 메모리 해제
    ```

## Closure 활용 사례

- 함수 실행이 종료되어도 변수를 메모리에 유지시키는 특성을 활용

### 콜백 함수 내부에서 외부 데이터 사용하기

```javascript
const fruits = ["apple", "banana", "peach"];
const $ul = document.createElement("ul");
fruits.forEach(function (fruit) {
    const $li = document.createElement("li");
    $ul.addEventListener("click", function () {
        alert(`Choose ${fruit}`);
    });
    $ul.appendChild($li);
});
document.body.appendChild($ul);
```

- `fruits.forEach()`의 callback 함수가 실행될 때마다 `fruits` 변수를 가진 새로운 실행 컨텍스트 생성
- `addEventListener()`의 callback 함수는 외부의 `fruit` 변수를 참조
- 이 때, **`forEach()`의 callback 함수에 대해 closure 생성**
- 이후 `$li`에 `click` event가 발생하면 **`forEach()`의 callback 함수는 종료되었지만 event handler에서 `fruit` 변수의 값에 접근할 수 있음**

<br/>

```javascript
// ...

const alertFruitBuilder = function (fruit) {
    return function () {
        alert(`Choose ${fruit}`);
    };
}

fruits.forEach(function (fruit) {
    const $li = document.createElement("li");
    $ul.addEventListener("click", alertFruitBuilder(fruit));
    $ul.appendChild($li);
});
```

- Event handler 함수를 재사용하기 위해 고차 함수 활용
- `alertFruitBuilder`가 반환하는 내부 함수는 외부 변수 `fruit`를 참조하여 closure를 형성
- Event handler가 실행될 때 `fruit` 변수가 메모리에 유지되어 값에 접근 가능

### 접근 권한 제어 (정보 은닉)

- **정보 은닉(information hiding)** : module의 내부 로직의 외부 노출을 최소화하여 module 간 결합도를 낮추고 유연성을 높이기 위한 기법
- JavaScript는 변수에 `public`, `private` 등 접근 권한을 부여할 수 없음
- 외부로 전달되는 내부 함수가 참조하는 외부 변수의 값에 접근 가능한 closure의 특성을 활용하면 변수에 접근 권한을 부여할 수 있음
- Closure가 형성될 조건을 만족하기 위해 객체 대신 함수를 만들고, 필요한 변수만 반환하도록 구현
    ```javascript
    function createCar() {
        let fuel = Math.ceil(Math.random() * 10 + 10);
        const power = Math.ceil(Math.random() * 3 + 2);
        let moved = 0;
        return {
            get moved() {
                return moved;
            },
            run() {
                const km = Math.ceil(Math.random() * 6);
                const wasteFuel = km / power;
                if (fuel < wasteFuel) {
                    console.log("이동 불가");
                    return;
                }

                fuel -= wasteFuel;
                moved += km;
            }
        };
    };

    const car = createCar();
    ```
    - `fuel`, `power` 변수는 반환되는 객체의 method들이 직접 반환하지 않으므로 외부에서 값에 접근 불가 (private member로 설정)
    - `moved` 변수는 `moved()` method를 통해 직접 반환되므로 외부에서 값에 접근 가능 (public member)
    - 이 때, `fuel`, `power`, `moved` 등 변수들은 **closure에 의해 메모리에 값이 유지**되므로 `createCar()` 함수의 실행이 종료된 후에도 `moved()`와 `run()` method가 실행될 때 값에 접근 가능

### 부분 적용 함수

```javascript
function partial() {
    var originalPartialArgs = arguments;
    var func = originalPartialArgs[0];
    if (typeof func !== "function") {
        throw new Error("The first argument is not a function");
    }

    return function () {
        var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
        var restArgs = Array.prototype.slice.call(arguments);
        return func.apply(this, partialArgs.concat(restArgs));
    };
}

function add() {
    var result = 0;
    for (var i = 0; i < arguments.length; i++) {
        result += arguments[i];
    }
    return result;
}

var addPartial = partial(add, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9, 10)); // 55
```

- **부분 적용 함수(partially applied function)** : `n`개의 인자를 받는 함수에 `m`개의 인자만 넘겨서 기억시켰다가, 나중에 `n-m`개의 인자를 넘기면 원래 함수의 실행 결과를 반환하는 함수
- `partial()` 함수는 `originalPartialArgs`와 `func` 지역 변수를 참조하는 내부 함수를 반환
- 이후 반환된 내부 함수를 실행하면 **`partial()` 함수 종료 후에도 closure에 의해 메모리에 남아있던 `originalPartialArgs`와 `func` 변수의 값에 접근 가능**
- 부분 적용 함수의 '미리 일부 인자를 넘겨서 기억하게 만들어 두고 **추후 필요한 시점에 기억했던 인자들을 사용**해서 실행'하는 개념이 closure의 정의에 부합

<br/>

```javascript
function debounce(eventName, func, wait) {
    var timeoutId = null;
    return function (event) { // event handler로 사용
        var self = this;
        clearTimeout(timeoutId);
        timeoutId = setTimeout(func.bind(self, event), wait);
    };
}

document.body.addEventListener("mousemove", debounce("move", handleMove, 500));
```

- 비슷한 원리로 `debounce()` 함수 구현 가능
- `debounce()` 함수가 반환하는 내부 함수는 `mousemove` event handler로 사용됨
- `debounce()` 함수가 반환하는 내부 함수가 참조하는 `eventName`, `func`, `wait`, `timeoutId` 변수들은 closure로 처리됨
- 추후 `mousemove` event가 발생해서 `debounce()`가 반환하는 내부 함수가 실행되면, **이미 실행 종료된 `debounce()` 함수의 지역 변수인 `eventName`, `func`, `wait`, `timeoutId`에 접근해서 값을 사용**할 수 있음

### 커링 함수

```javascript
function curry5(func) {
    return function (a) {
        return function (b) {
            return function (c) {
                return function (d) {
                    return function (e) {
                        return func(a, b, c, d, e);
                    }
                }
            }
        }
    }
}

// or

const curry5 = func => a => b => c => d => e => func(a, b, c, d, e);

// usage

const getMax = curry5(Math.max);    // function (a) { ... }
const getMaxWith1 = getMax(1);      // function (b) { ... }
const getMaxWith2 = getMaxWith1(2); // function (c) { ... }
const getMaxWith3 = getMaxWith2(3); // function (d) { ... }
const getMaxWith4 = getMaxWith3(4); // function (e) { ... }
const result = getMaxWith4(5);      // 5 (`Math.max(1, 2, 3, 4, 5)` 가 실행됨)
```

- **커링 함수(currying function)** : 여러 개의 인자를 받는 함수를 인자 1개만 받는 함수로 나눠서 순차적으로 호출할 수 있도록 체인 형태로 구성한 함수
- 마지막 인자를 넘겨서 실행할 때까지 처음에 전달한 함수가 실행되지 않음 -> **지연 실행(lazy execution)**
- 제일 마지막에 반환되는 함수는 이전에 거쳐온 모든 외부 함수들의 parameter에 대해 closure를 형성함
- 제일 마지막에 반환되는 함수는 내부 함수가 아닌 함수의 실행 결괏값을 반환하므로 더 이상 closure가 형성되지 않음