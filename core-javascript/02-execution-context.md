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

## 번외: VariableEnvironment는 왜 필요할까?

- `VariableEnvironment`는 현재 컨텍스트의 환경 정보들을 저장하고 snapshot을 생성하여 함수 실행 중에도 값이 변하지 않는다고 설명한다.
- 그리고 함수가 실행되는 동안에는 주로 `LexicalEnvironment`가 사용된다고 한다.
- `VariableEnvironment`는 값이 변하지 않는다고 하니까 거의 사용하지 않을 것 처럼 보이는데, 왜 필요한 것일까?

### ES3 와 ES6+ 에서 실행 Context의 차이

- ES3 에서는 함수 호출 시 생성되는 실행 context에서 `VariableObject`(변수 객체)라고 부르는 곳에 아래 정보를 저장한다.
    - `arguments` 객체
    - 지역 변수 (값을 `undefined`로 초기화 해 둔 상태)
    - this binding (`this` 참조 객체 저장)
    - scope 정보
- ES6+ 에서는 `VariableObject` 하나만 사용하는 방식에서 `VariableEnvironment`와 `LexicalEnvironment` 두 가지 환경 정보를 분리해서 저장한다.
    - 각 환경에는 두 가지 정보가 저장된다.
    - Outer Environment Reference : 상위 scope의 `LexicalEnvironment` 참조 저장 (Lexical Nesting Structure)
    - Environment Records : 변수, 함수 식별자 저장
        - Declarative Environment Records : 일반적인 변수, 함수, `this` binding 저장
        - Object Environment Records : `with`, `catch`문 안에서 선언한 변수, 함수 저장

### 실행 Context를 `VariableEnvironment`와 `LexicalEnvironment`로 구분하는 이유

1. `var` 변수와 `let`, `const` 변수의 동작 차이
    - `var` 변수는 **functional scope**를 갖기 때문에 함수 내부의 다른 block scope에서도 동일한 환경 record로 접근하면 된다.
    - `VariableEnvironment`는 이렇게 함수 안에서 단일 환경 record를 제공하는 목적으로 사용된다.
    - 하지만, `let`과 `const`는 **block scope**를 갖기 때문에 **함수 내부의 block마다 독립적인 환경 record에 저장되어야 한다.**
    - `LexicalEnvironment`는 이렇게 **실행 context 안에서 block 마다 여러 개의 환경 record를 제공**하기 위해 사용된다.
2. `var` 변수와 `let`, `const` 변수의 생성 과정 차이
    - 변수는 3가지 단계를 거쳐 생성된다.
        1. 선언 단계 (Declaration phase) : 변수를 환경 정보에 등록
        2. 초기화 단계 (Initialization phase) : `LexicalEnvironment`에 등록된 변수에 메모리를 할당하고 `undefined`로 초기화 (Lexical Binding)
        3. 할당 단계 (Assignment phase) : 변수에 값 할당 (`undefined` 또는 그 외에 다른 값)
    - `var` 변수는 1번과 2번 단계가 동시에 일어난다.
        - 먼저 `VariableEnvironment`에 식별자가 저장된다.
        - `VariableEnvironment`를 사용해서 함수 레벨의 `LexicalEnvironment`를 생성할 때(instantiate) `var` 변수의 메모리 할당 및 `undefined` 초기화가 일어난다.
        - 그래서, 변수 선언문 이전에 접근해도 `undefined`를 반환할 뿐 error가 발생하지 않는다.
    - `let`과 `const` 변수는 1번과 2번 단계가 개별적으로 일어난다.
        - `LexicalEnvironment` 생성 시 식별자를 저장한다. 식별자를 등록했지만 아직 메모리를 할당하지 않은 상태이다.
        - 변수 선언문이 실행되는 시점에 메모리 할당 및 `undefined` 초기화가 일어난다.
        - 그래서, 변수 선언문 이전에 접근하면 error가 발생한다. => **TDZ(Temporal Dead Zone) 형성**

### 결론

- `VariableEnvironment`와 `LexicalEnvironment`를 구분하는 이유는 ES6+ 에서 새롭게 추가된 `let`과 `const` 변수의 동작 방식을 지원하면서도 이전에 사용하던 `var` 변수와 함수 선언문의 동작에 대해 호환성을 제공하기 위함이다.
    - `VariableEnvironment` : functional scope를 갖는 `var` 변수를 관리하는 목적
    - `LexicalEnvironment` : block scope를 갖는 `let`과 `const` 변수, 함수 선언 등을 관리하는 목적
- `let`과 `const` 변수는 `var` 변수가 functional scope를 갖는 것에서 비롯된 문제들을 해결하기 위해 도입되었다. 따라서, `VariableEnvironment`와 `LexicalEnvironment`는 이 문제들을 해결하기 위한 구체적인 방법으로 볼 수 있겠다.
- JavaScript는 `VariableEnvironment`와 `LexicalEnvironment`를 분리해서 사용하여 다양한 scope(functional 및 block scope)를 지원할 수 있게 되었다.
- `LexicalEnvironment`에서 `var` 변수를 초기화한다는 내용도 있어서, 대부분의 일반적인 상황에서 동작하는 방식이라고 이해하면 될 것 같다.
- 이 동작 방식은 block scope가 도입된 ES6+ 모던 JavaScript의 동작이다. Block scope 이전에는 다르게 동작할 수 있다.

### 참고

- [Velog | VariableEnvironment는 왜 필요할까](https://velog.io/@bapbodanbbang/Variable-Environment%EB%8A%94-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%A0%EA%B9%8C)
- [[JavaScript] ES6의 Execution Context(실행 컨텍스트)의 동작 방식과 Lexical Nesting Structure(Scope chain)](https://m.blog.naver.com/dlaxodud2388/222655214381)
- [Variable Environment vs. Lexical Environment](https://seholee.com/blog/varaible-environment-vs-lexical-environment/)


## [Claude Q&A](https://claude.ai/share/4449c15e-c6e5-4f8e-8de1-c47f60b658dd)

### 실행 Context의 생성 시점

- 실행 context는 **함수 호출 시점**에 생성된다.
    ```javascript
    function myFunction() {
        console.log('함수 실행');
        var x = 10;
    }

    // 이 시점에서는 실행 컨텍스트 생성 안됨 (함수만 정의)
    console.log('함수 호출 전');

    // 이 시점에서 실행 컨텍스트 생성!
    myFunction(); 

    // 함수 실행 완료 후 실행 컨텍스트 소멸
    ```
- 실행 context 생성 단계
    1. 생성 단계 (Creation phase)
        - `VariableEnvironment` 및 `LexicalEnvironment` 설정
        - `this` binding 결정
        - Hoisting 처리
    2. 실행 단계 (Execution phase)
        - 실제 코드 실행
        - 변수에 값 할당

### `outerEnvironmentReference` 설정 시점 (Scope chain 결정 시점, lexical scoping)

- **결정 시점** : 함수 정의 시점 (함수를 정의하는 코드 실행)
    - 함수의 `[[Environment]]` 슬롯에 현재 scope의 `LexicalEnvironment` 참조 저장
- **설정 시점** : 함수 호출 시점 (함수 자체를 실행) 
    - 함수의 `[[Environment]]`에 저장된 참조를 실행 context의 `outerEnvironmentReference`에 저장

### "VariableEnvironment에 담기는 내용은 LexicalEnvironment와 같지만 최초 실행 시의 스냅샷을 유지한다는 점이 다릅니다."

- 이 문장만 보면 `VariableEnvironment`에 저장된 변수(식별자)의 값은 실행 context 생성 이후 바뀌지 않는다는 것 처럼 이해된다.
- 하지만, `VariableEnvironment`가 저장하고 있는 변수의 값이 바뀌지 않는다는 의미가 아니라 `VariableEnvironment`에 저장된 객체 자체가 바뀌지 않는다는 의미이다.
- 함수 내부에서 block을 실행할 때, 함수 실행 context의 `LexicalEnvironment`는 block의 `LexicalEnvironment`로 교체되어 실행된다.
- [Inflearn | '실행 컨텍스트 안의 VariableEnvironment 를 존재 이유가 궁금합니다.' 질문의 답변](https://www.inflearn.com/community/questions/359757?focusComment=151373)에 따르면,
    - 내부 block scope의 실행이 종료되면 원래 상태의 `LexicalEnvironment`로 복구할 때 `VariableEnvironment`가 사용된다고 한다.
    - Claude 설명과 합쳐 보면,
        1. 함수 scope는 `VariableEnvironment`와 `LexicalEnvironment`를 하나씩 가지고 있다.
        2. Block scope는 독립적인 `LexicalEnvironment`를 갖는다.
        3. Block이 실행될 때, 함수의 `LexicalEnvironment`에 할당된 객체는 block scope의 `LexicalEnvironment`로 교체된다.
        4. 함수 내부에서 block 내부의 코드를 실행할 때, 해당 block의 `LexicalEnvironment`를 가지고 코드를 실행하게 된다.
        5. Block 실행이 종료되면 함수의 `LexicalEnvironment`를 block 실행 이전으로 돌려야하는데, 이 때 `VariableEnvironment`를 사용한다. 
            - `VariableEnvironment`는 함수 실행 context 생성 시 만들어진 뒤 변경되지 않으므로 함수의 원래 환경 정보를 가지고 있다.

### "실행 컨텍스트를 생성할 때 VariableEnvironment에 정보를 먼저 담은 다음, 이를 그대로 복사해서 LexicalEnvironment를 만들고, 이후에는 LexicalEnvironment를 주로 활용하게 됩니다."

- ES6+ 부터 `var` 변수도 `LexicalEnvironment`를 사용해서 찾도록 매커니즘이 변경되었다.
- 실행 context에서 식별자를 해석하고 참조하는 주체가 `LexicalEnvironment`인 것
- `let`이나 `const` 같은 변수를 많이 사용하기 때문이 아니다.