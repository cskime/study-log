# Execution Context

## Execution context

- 실행할 코드에 제공할 **환경 정보**들을 모아놓은 **객체**
- 자바스크립트 엔진은 '**동일한 환경**'에 있는 코드들을 실행할 때 필요한 환경 정보들을 모아서 context를 구성
    - Context를 만드는 환경
        - 전역 공간
        - `eval()`
        - **함수**
        - **block** (`{}`, ES6+)
    - Context에 저장되는 환경 정보
        - `VariableEnvironment` : 현재 context 내부의 식별자 정보 및 외부 환경 정보
        - `LexicalEnvironment` : `VariableEnvironment`와 동일하지만 변경 사항이 실시간 반영됨
        - `ThisBinding` : `this` 식별자가 참조하는 대상 객체
- 생성한 context는 call stack에 넣어서 관련 코드 실행
    - Call stack에 새 context가 추가되면 이전에 실행하던 코드를 중단하고, context와 관련된 코드를 실행
    - Call stack은 가장 위에 있는 context를 우선적으로 실행하여 **전체 코드의 환경과 순서 보장**
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
- 실행 context가 생성될 때(최초 실행 시)의 snapshot을 유지하기 때문에, 코드가 실행되며 식별자 정보가 변경되어도 반영되지 않음
- 전역 실행 context는 `VariableEnvironment` 대신 **전역 객체(global object) 또는 호스트 객체(host object)** 를 사용
    - 브라우저 : `window`
    - Node.js : `global`

### environmentRecord와 호이스팅(hoisting)

- `environmentRecord`에는 매개변수, 함수, `var` 변수 등 식별자 정보가 저장됨
- JavaScript engine은 **코드를 실행하기 전에 context 내부 코드를 순서대로 확인**하며 식별자들을 수집해서 `environmentRecord`에 저장
- 즉, JavaScript engine이 **식별자들을 최상단으로 끌어올린 다음에 코드를 실행**하는 것 처럼 동작함 -> "**호이스팅(hoisting)**"
    - 실제로 식별자가 끌어올려지는 것은 아님
    - 동작의 이해를 돕기 위해 사용하는 가상의 개념
- 호이스팅 규칙
    - 변수 식별자의 값을 할당하는 코드는 끌어올려지지 않는다.
    - 함수 식별자는 함수 선언 전체가 끌어올려진다.
- 매개변수가 있을 때 호이스팅 동작 예시
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
- 함수 선언이 있을 때 호이스팅 동작 예시
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
            // `function b() {}` 함수 선언은 변수에 함수 표현식을 할당한 것처럼 동적
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

## LexicalEnvironment

- 최초 실행 시 `VariableEnvironment`를 그대로 복사해서 `LexicalEnvironment` 생성
- 코드가 실행되는 동안 변경 사항은 `LexicalEnvironment`에만 반영됨