# Execution Context

## 실행 컨텍스트 (Execution context)

- 실행할 코드에 제공할 **환경 정보**들을 모아놓은 **객체**
- 자바스크립트 엔진은 **동일한 환경에 있는 코드들을 실행할 때 필요한 환경 정보**들을 모아서 context를 구성
    - Context를 만드는 환경
        - 전역 공간
        - `eval()`
        - **함수**
        - **block** (`{}`, ES6+)
    - Context에 저장되는 환경 정보
        - 실행 context 객체가 활성화되는 시점에 3가지 정보 수집
        - `VariableEnvironment` : 현재 context 내부의 식별자 정보 및 외부 환경 정보
        - `LexicalEnvironment` : `VariableEnvironment`와 동일하지만 변경 사항이 실시간 반영됨
        - `ThisBinding` : `this` 식별자가 참조하는 대상 객체
    - Context 생성 및 실행
        - **함수가 호출되는 시점**에 context를 생성하고 call stack에 push
        - 전역 context는 script가 최초 실행되는 시점에 생성되어 call stack에 push됨
        - Call stack에 새 context가 추가되면 **이전에 실행하던 context의 코드 실행을 중단**하고 새로 추가된 context의 코드를 실행
        - JavaScript engine은 call stack의 가장 위에 있는 context를 우선적으로 실행하여 **전체 코드의 환경과 순서 보장**
- 실행 예시
    ```javascript
    var a = 1;
    function outer () {
        function inner() {
            console.log(a);
            var a = 3;
        }
        inner();
        console.log(a);
    }
    outer();
    console.log(a);
    ```
    1. 전역 코드를 실행할 때 (1) 전역 공간에 대한 context를 생성하고 (2) call stack에 추가해서 실행
    2. `outer` 함수를 호출하면,
        1. 전역 코드 실행이 중단되고 
        2. `outer` 함수에 대한 context를 생성한 뒤 call stack에 넣고
        3. `outer` 함수의 코드 실행
    3. `inner` 함수를 호출하면, 
        1. `outer` 코드 실행이 중단되고
        2. `inner` 함수에 대한 context를 생성한 뒤 call stack에 넣고
        3. `inner` 함수의 코드 실행
    3. `inner` 함수 실행이 끝나면, 
        1. `inner` 함수의 context가 call stack에서 제거되고
        2. `outer` 함수에서 실행이 중단된 지점부터 코드 실행 재개
    4. `outer` 함수 실행이 끝나면,
        1. `outer` 함수의 context가 call stack에서 제거되고
        2. 전역 코드에서 실행이 중단되 지점부터 실행 재개
    5. 전역 코드 실행이 끝나면 전역 context가 call stack에서 제거되고 실행 종료

## VariableEnvironment

- 현재 context 내부의 식별자 정보(`environmentRecord`)와 외부 환경 정보(`outerEnvironmentReference`)를 저장하고 있는 객체
- 실행 context가 생성될 때(최초 실행 시)의 snapshot을 유지하기 때문에, **코드가 실행되며 식별자 정보가 변경되어도 반영되지 않음**
- 전역 실행 context는 `VariableEnvironment` 대신 **전역 객체(global object) 또는 호스트 객체(host object)** 를 사용
    - 브라우저 : `window`
    - Node.js : `global`

## LexicalEnvironment

- 최초 실행 시 `VariableEnvironment`를 그대로 복사해서 `LexicalEnvironment` 생성
- 코드가 실행되는 동안 변경 사항은 `LexicalEnvironment`에만 반영됨

### environmentRecord와 호이스팅(hoisting)

- `environmentRecord` : 실행 컨텍스트의 매개변수, 함수, `var` 변수, 함수 등 식별자 정보를 저장하는 객체
    - JavaScript engine은 **코드를 실행하기 전에 context 내부 코드를 순서대로 확인**하며 식별자들을 수집해서 `environmentRecord`에 저장
    - 변수 : 식별자만 저장
      - `var` : 식별자를 저장하면서 `undefined`로 초기화하므로, 초기화 구문 이전에도 접근 가능 (`undefined` 반환)
      - `let`, `const` : 식별자를 저장하고 초기화 구문을 실행하기 전까지는 'uninitialized' 상태이므로 접근 불가 (TDZ 형성)
    - 함수 선언 : 함수 이름으로 된 식별자를 저장하고, 그 식별자를 함수 표현식으로 초기화
- **호이스팅(hoisting)** : `environmentRecord` 객체를 사용하는 JavaScript engine의 동작을 개념적으로 설명한 것
    - JavaScript engine은 `environmentRecord` 식별자를 먼저 수집한 다음 코드를 실행
    - 코드가 실행되기 전에 **선언된 식별자들이 코드 최상단으로 끌어올려지는 것 처럼** 동작 (실제로 끌어올려지는 것이 아님)
    - 호이스팅 규칙
        - 변수는 선언문만 호이스팅되고, 할당문은 호이스팅 되지 않는다. => `environmentRecord`에는 식별자만 수집되기 때문
        - 함수 선언문은 함수 선언 전체가 끌어올려진다. => 함수 자체가 '선언' 이므로 예외적인 동작
- 매개변수가 있는 함수에서 호이스팅 동작
    ```javascript
    function a(x) {
        console.log(x);
        var x;
        console.log(x);
        var x = 2;
        console.log(x);
    }
    a(1);

    // [ 예상 실행 결과 ]
    //
    // 1
    // undefined
    // 2
    ```
    1. `a` 함수를 실행하면 실행 context가 생성되고 내부에 `environmentRecord`를 가진 `VariableEnvironment` 및 `LexicalEnvironment` 생성
    2. `a` 함수의 context 내부에서 사용된 식별자들을 순서대로 모아서 `environmentRecord`에 저장 -> 호이스팅
        ```javascript
        function a() {
            /* environmentRecord에 저장된 정보 */
            var x; // parameter
            var x; // local variable 1
            var x; // local variable 2

            // ...
        }
        ```
    3. `environmentRecord`에 저장된 식별자들을 사용해서 실제 코드 실행
        ```javascript
        function a() {
            // ...

            /* 실제 코드 실행 */
            x = 1; // argument 초기화
            console.log(x);
            console.log(x);
            x = 2; // local variable 값 할당
            console.log(x);
        }

        // [ 실제 실행 결과 ]
        //
        // 1
        // 1
        // 2
        ```
- 내부에 함수 선언문이 있는 함수에서 호이스팅 동작
    ```javascript
    function a() {
        console.log(b);
        var b = "bbb";
        console.log(b);
        function b() {}
        console.log(b);
    }
    a(); // (1)

    // [ 예상 실행 결과 ]
    //
    // undefined
    // bbb
    // function b() {}
    ```
    1. `a` 함수를 실행하면 실행 context가 생성되고 내부에 `environmentRecord`를 가진 `VariableEnvironment` 및 `LexicalEnvironment` 생성
    2. `a` 함수의 context 내부에서 사용된 식별자들을 순서대로 모아서 `environmentRecord`에 저장 -> 호이스팅
        ```javascript
        function a() {
            /* environmentRecord에 저장된 정보 */
            var b;
            // `function b() {}` 함수 선언은 변수에 함수 표현식을 할당한 것처럼 동작
            var b = function b () { }

            // ...
        }
        ```
    3. `environmentRecord`에 저장된 식별자들을 사용해서 실제 코드 실행
        ```javascript
        function a() {
            // ...

            /* 실제 코드 실행 */
            console.log(b); // (1)
            b = "bbb";
            console.log(b); // (2)
            console.log(b); // (3)
        }

        // [ 실제 실행 결과 ]
        //
        // function b() {}
        // bbb
        // bbb
        ``` 
        1. 함수 식별자 `b`가 변수 식별자 `b`보다 나중에 선언되었으므로, `(1)`에서는 함수가 출력됨
        2. `(2)`는 직전에 `b` 식별자의 값을 `"bbb"`로 변경했으므로 `bbb`가 출력됨
        3. `(3)`은 이전에 `b` 식별자의 값이 변경되지 않았으므로 동일하게 `bbb`가 출력됨

### 함수 선언문과 함수 표현식

- 함수 선언문 : 함수 정의부만 존재하고 별도의 할당 명령이 없는 것
- 함수 표현식 : 함수를 변수에 할당하는 것
- 함수 표현식은 변수 식별자를 사용하므로 함수 이름이 없어도 됨
    - '기명 함수 표현식'과 '익명 함수 표현식'으로 구분
    - 과거에는 익명 함수 표현식에서 `name` property는 `undeinfed` 또는 `unnamed` 값을 반환하여 함수 이름을 가져올 수 없었기 때문에, 디버깅 시 어떤 함수인지 추적할 때는 기명 함수 표현식이 유리했음
    - 하지만, 모던 자바스크립트는 `name` property에서 **익명 함수 표현식의 변수 식별자를 반환**하기 때문에 기명 함수 표현식의 장점이 사라짐
    - 함수 표현식을 재귀 호출할 때, 외부의 변수 식별자를 사용해도 되는데 굳이 기명 함수 표현식을 사용할 이유가 없다고 보기도 함 -> **익명 함수 표현식을 주로 사용**
- 호이스팅 관점에서 함수 선언문과 표현식 비교
    - 아래와 같은 코드가 있을 때,
        ```javascript
        console.log(sum(1, 2));
        console.log(multiply(3, 4));

        function sum (a, b) {
            return a + b;
        }

        var multiply = function (a, b) {
            return a * b;
        };
        ```
    - 함수 선언문은 함수 전체가 호이스팅되지만, 함수 표현식은 변수 식별자만 호이스팅 됨
        ```javascript
        // 함수 선언문은 함수 전체가 호이스팅 됨
        var sum = function sum(a, b) {
            return a + b;
        }

        // 함수 표현식은 변수 식별자만 호이스팅 됨
        var multiply; 

        // `sum` 식별자에는 함수가 저장되어 있으므로 정상적으로 호출 가능
        // => 함수 선언문이 선언 위치에 상관 없이 호출 가능한 이유
        console.log(sum(1, 2));

        // `multiply` 식별자는 초기화 코드를 만나지 못했으므로 `undefined`가 저장됨
        // `undefined`에 호출자를 사용했으므로 error 발생
        console.log(multiply(3, 4));

        // 함수 표현식 할당은 실제 코드 실행이 도착한 순간 이루어짐
        // 단, 이 코드에서 실행 시점이 error가 발생한 뒤 이므로 실제로는 실행되지 않을 것
        multiply = function (a, b) {
            return a * b;
        };
        ```
- **함수 선언문보다 함수 표현식 사용을 권장한다.**
    - 함수 선언문은 선언 위치에 관계 없이 어디서든 접근할 수 있으므로, 코드 흐름이 일반적이지 않다.
    - 함수 표현식은 선언 위치 이후로만 접근할 수 있으므로 '선언한 뒤 사용한다'는 일반적인 흐름으로 사용할 수 있다.
    - 동일한 이름을 가진 함수를 중복 선언하면서 발생할 수 있는 문제를 함수 표현식이 해결할 수 있지만, 엄격 모드에서는 같은 이름으로 함수를 선언할 수 없으므로 큰 효과는 없는 것 같다.

### Scope와 outerEnvironmentReference

- Scope : 식별자에 대한 유효 범위 (식별자에 접근할 수 있는 범위)
    - ES5 까지는 함수에 의해서만 scope가 생성됨 (함수 scope)
    - ES6+ 부터는 block에 대해서도 scope가 생성됨 (block scope)
        - 단, `var` 변수는 함수 scope만 가짐
        - `let`과 `const` 변수, strict mode에서 함수 선언 등이 block scope를 가짐
- Scope chain : 식별자 유효 범위를 안에서부터 바깥으로 차례대로 검색해 나가는 것
    - 현재 실행 context의 `environmentRecord`에서 먼저 식별자 탐색
    - 식별자가 없다면 `outerEnvironmentReference`를 통해 외부 `environmentRecord`에서 식별자 탐색
    - 전역 context까지 반복해서 식별자를 탐색하고, 찾지 못하면 `undefined` 반환
- `outerEnvironmentReference` : 호출된 함수가 **선언된 위치**에서의 `LexicalEnvironment` 참조
    - `outerEnvironmentReference`는 함수 선언문 코드가 실행되는 시점에 **결정됨** (함수 선언문 코드를 실행한 실행 context의 `LexicalEnvironment`)
    - 해당 함수가 호출되는 시점에 실행 context가 생성되면서 `LexicalEnvironment`의 `outerEnvironmentReference`에 **설정됨**
- **변수 은닉화(variable shadowing)**
    - Scope chain 상에서 동일한 이름의 식별자가 두 개 이상 선언될 때, 현재 scope에서 가장 가까운 scope에 선언된 식별자 외에는 접근할 수 없음
    - 함수 내부에서 특정 식별자에 접근하려고 할 때, 현재 실행 context의 `LexicalEnvironment`부터 시작하여 **scope chain을 따라 해당 식별자가 존재하는 가장 가까운 `LexicalEnvironment`를 찾는다.**
- 전역 변수와 지역 변수
    - 전역 변수(global variable) : 전역 scope에서 선언한 변수
    - 지역 변수(local variable) : 함수 scope (함수 내부)에서 선언한 변수
    - 지역 변수로 선언하면 전역 scope에서 접근할 수 없으므로 코드 안정성이 더 좋기 때문에 **전역 변수 사용을 최소화하는 것이 좋다.**
    - 전역 변수 사용을 최소화하기 위한 방법
        - 즉시 실행 함수 (IFE, Immediately invoked Function Expression)
        - namespace
        - Module pattern
        - Sandbox pattern