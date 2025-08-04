# Data Type

## Data type의 종류

- **기본형(primitive type)** : number, string, boolean, null, undefined, symbol
- **참조형(reference type)** : object (array, function, Set, Map, Date, RegExp, ...)

## 변수 선언과 데이터 할당

- **변수 선언** : 메모리의 일정 공간을 할당받고 식별자(변수 이름)을 붙이는 것
- **데이터 할당** : 데이터를 별도의 메모리 공간에 저장하고 변수에 그 주소값을 저장하는 것
- JavaScript는 데이터의 성질에 따라 메모리 공간을 '**변수 영역**'과 '**데이터 영역**'으로 구분
    - 값을 저장하는데 필요한 메모리 공간이 가변적일 때, 값을 변수에 직접 저장하면 변수 선언을 위해 확보한 메모리 공간의 크기를 늘려야 한다.
    - 그런데, 미리 확보된 메모리 공간을 늘리는 작업을 수행하려면 처리해야 할 연산량이 많아진다.
    - 데이터 변환을 효율적으로 처리하기 위해 JavaScript는 **데이터를 별도의 메모리 공간에 저장하고 변수에는 그 주소값을 저장**한다.
    - 변수는 실제 값보다 상대적으로 작고 고정된 크기의 주소값을 저장하므로 데이터가 변경되어도 효율적으로 처리할 수 있다.
- 변수 `a`에 `abc` 문자열을 할당하는 코드의 동작 방식
    ```javascript
    var a = "abc";
    ```
    1. 변수 영역에 메모리 공간을 할당받고(`0x1003`) `a`라는 식별자(`name`)를 연결
    2. 데이터 영역에 메모리 공간을 할당받고(`0x5005`) `"abc"` 값을 저장
    3. `"abc"`가 저장된 데이터 영역의 주소값(`0x5005`)을 변수 `a` 주소의 값으로 저장
    <img src="images/01-data-type-01.png" width="450" />

## 기본형 데이터와 참조형 데이터

- 변수(variable)와 상수(constant)는 '변수 영역 메모리의 값에 대한 변경 가능성'을 기준으로 구분
- **불변성(immutability)** 은 '**데이터 영역 메모리의 값**에 대한 변경 가능성'을 기준으로 구분
- 즉, 불변성은 '**데이터 영역 메모리의 값을 변경할 수 없다**'는 것을 의미함
- JavaScript의 기본형 데이터는 모두 불변값으로, **값을 변경하려면 항상 새로운 값을 만들어야 한다.**
    - 숫자 `5`를 `7`로 바꾸는 경우, `5`가 저장된 데이터 영역 메모리의 값을 변경하는 것이 아님
    - 숫자 `7` 값을 새로 만들고 다른 데이터 영역 메모리에 저장한 다음 그 주소값을 사용
    - 이 때, JavaScript는 데이터를 만들기 전에 이미 해당 값이 메모리에 존재하는지 먼저 확인함
        - 데이터 영역 메모리에 같은 값이 저장되어 있다면 그 메모리 주소를 **재활용**
        - 데이터 영역 메모리에 같은 값이 없다면 새로 만들어서 저장한 뒤 그 주소를 사용
    - 예시
        ```javascript
        var a = 5;
        var b = 5;
        b = 7;
        ```
        1. 변수 영역에 `a` 이름으로 메모리 공간 할당
        2. 데이터 영역에 숫자 `5`가 없으므로, 새로 저장한 뒤 변수 `a`의 메모리 공간에 그 주소를 저장
        3. 변수 영역에 `b` 이름으로 메모리 공간 할당
        4. 데이터 영역에서 숫자 `5`가 저장된 주소를 변수 `b`의 메모리 공간에 저장
        5. 데이터 영역에 숫자 `7`을 저장한 뒤, 변수 `b`의 메모리 공간의 값을 새로 할당한 주소로 교체
        <img src="images/01-data-type-02.png" width="450" />
- JavaScript의 참조형 데이터(객체)는 값을 직접 변경할 수 있는 '가변값'으로 알려져 있지만, **메모리 상에서는 불변값이다.**
    - JavaScript 객체는 둘 이상의 데이터를 갖는 collection으로, **내부에 포함된 데이터를 위한 별도의 변수 영역**을 가짐
    - 객체의 property 값을 변경하는 것은 '**객체의 변수 영역**' 메모리의 값을 변경하는 것
    - 객체를 저장한 변수에는 객체의 데이터들이 저장된 주소가 저장되고, 이 주소는 직접 변경할 수 없음 → "**데이터 영역에서 객체는 불변값이다.**"
    - 하지만, 객체 변수 영역에서 값을 바꾸면 `obj` 변수 입장에서는 객체가 변경된 것처럼 보인다 → "**변수 영역에서 객체는 가변값이다.**"
    - 예시
        ```javascript
        var obj = { a: 1, b: 'bbb' };
        obj.a = 2;
        ```
        1. 변수 영역에 `obj` 이름으로 메모리 공간 할당
        2. `obj`에 할당하는 값은 객체이므로, property들을 저장하기 위한 별도의 변수 영역 메모리를 할당하고 그 주소를 데이터 영역 메모리에 저장
        3. 변수 `obj` 메모리 공간에는 객체 주소가 저장된 데이터 영역 메모리 주소를 저장
        4. 숫자 `1`과 `bbb` 값을 데이터 영역에서 찾고, 없으면 새로 생성한 뒤 메모리를 할당해서 저장
        5. 객체 변수 영역에 property `a`와 `b`를 위한 메모리를 할당하고, 4번에서 할당한 데이터 영역 메모리 주소를 각각 저장
        <img src="images/01-data-type-03.png" width="700" />
    - 위 예시에서, 숫자 `1`이 저장된 메모리는 참조하는 곳이 없으므로 나중에 가비지 컬렉터에 의해 정리될 수 있다.
- 기본형 데이터와 참조형 데이터의 메모리 동작 방식을 고려하면, **기본형 데이터와 참조형 데이터의 변수 복사 동작에는 차이가 없다.**
    - 변수를 복사했을 때 두 변수가 동일한 '데이터 영역 메모리 주소'를 바라보게 되는 점에서 동일하다.
        ```javascript
        var a = 10; // 변수 `a`에 `10`이 저장된 데이터 영역 메모리 `0x5003` 저장
        var b = a; // 변수 `b`의 변수 영역 메모리 공간에 `0x5003` 주소가 복사됨

        var obj1 = { c: 10, d: "ddd" }; // 변수 `obj1`에 객체 주소가 저장된 데이터 영역 메모리 `0x5004` 저장
        var obj2 = obj1; // 변수 `obj2` // 변수 `obj2`의 변수 영역 메모리 공간에 `0x5004` 주소가 복사됨
        ```
    - 복사된 변수에 값을 변경할 때 차이가 발생한다.
        ```javascript
        b = 15; // `15`를 데이터 영역 메모리를 새로 할당받아 저장하고 변수 `b`에 새 주소를 저장
        console.log(a === b); // false. 변수 `a`와 `b`가 가리키는 데이터 영역 주소가 다르다. (값이 다르다.)

        obj2.c = 20; // 객체 property의 변수 영역 메모리 공간의 값을 변경하는 것
        console.log(obj1 === obj2); // true. 변수 `obj1`과 `obj2`가 여전히 동일한 데이터 영역 주소를 가리킨다.
        ```
    - `obj2` 변수에 새 객체를 할당한다면, `obj2` 변수의 값은 새 객체의 주소가 저장된 데이터 영역 메모리 주소가 되어 `obj1`과 다른 주소를 바라보게 된다.
        ```javascript
        obj2 = { c: 20, d: "ddd" };
        console.log(obj1 === obj2); // false. 이제 변수 `obj1`과 `obj2`가 다른 데이터 영역 주소를 가리킨다.
        ```
- JavaScript의 기본형 및 참조형 값들은 변수에 저장될 때 모두 **주소가 저장된다**. 
    - 기본형 데이터는 그 주소에 실제 값이 들어있고,
    - 참조형 데이터는 그 주소에 내부 값들이 저장된 또다른 주소가 들어 있다. (두 단계를 거침)

## 불변 객체

- 불변값은 **기존 데이터가 변하지 않는 것**을 의미한다. (메모리의 데이터 영역 주소에 저장된 값이 변하지 않는 것)
- JavaScript의 기본형 데이터는 데이터 자체를 변경할 수 없는 **불변값(immutable)** 이다.
- JavaScript의 참조형 데이터(객체)도 데이터 자체를 변경할 수 없는 불변값이지만, **객체 내부 property를 변경할 때는 '가변'** 이다.
- 객체 내부 property에 대한 가변성 때문에 객체를 값으로 전달한 뒤 내부 property를 변경하면 원본 객체가 영향을 받는 문제가 생긴다.
    ```javascript
    let user = { name: "Kim" };

    function changeName(user, newName) {
        let newUser = user;
        newUser.name = newName;
        return newUser;
    }

    let user2 = changeName(user, "Lee");

    console.log(user.name); // Lee <-- 원본 객체의 `name` property의 값이 변경되었다.
    console.log(user2.name); // Lee
    ```
    - `changeName` 함수에서 `newUser`에 `user`를 복사하면, 원본 객체가 저장된 메모리 주소가 복사되어 두 변수가 같은 객체를 가리킴
    - 따라서, `newUser.name`은 `user.name`은 동일한 객체 property를 가리키므로 `newName` 변경사항이 원본 객체에도 반영됨
- 원본 객체의 값을 유지해야 하는 경우, **객체의 내부 property를 변경해야 할 때마다 새로운 객체를 만들어 재할당하도록 강제**하여 불변성을 확보할 수 있다.
    - `changeName` 함수가 `user`를 복사하지 않고 새로운 객체를 생성해서 반환하도록 수정
        ```javascript
        let user = { name: "Kim" };

        // 1. 객체의 property 값을 직접 복사
        function changeName(user, newName) {
            return { name: newName };
        }

        // 2. 객체의 모든 property를 순회하며 복사
        function changeName2(user, newName) {
            var result = {};
            for (const prop = in user) {
                result[prop] = user[prop]
            }

            result.name = newName;
            return result;
        }

        let user2 = changeName(user, "Lee");

        console.log(user.name); // "Kim" <-- 복사본 객체에서 값을 변경해도 원본 객체에 영향을 주지 않는다.
        console.log(user2.name); // "Lee"
        ```
    - 객체의 `name` property를 변경할 떄는 항상 `changeName` 함수를 사용하도록 규칙을 정하고 강제
    - 또는, [immutable.js](https://immutable-js.com/), [baobab.js](https://github.com/Yomguithereal/baobab) 등 라이브러리를 사용해서 **시스템적으로 제약을 거는 것이 더 안전하다.**
- 이 때, 중첩 객체를 갖는 객체의 경우 **얕은 복사가 아닌 깊은 복사를 수행**해야 한다.
    - 얕은 복사(shallow copy) : 바로 아래 단계의 값만 복사하는 방식
    - 깊은 복사(deep copy) : 객체 내부의 모든 값들을 복사하는 방식
    - 중첩 객체를 갖는 객체에 얕은 복사를 수행하면 원본과 복사본이 여전히 동일한 중첩 객체를 참조하므로 복사본을 변경했을 때 원본이 영향을 받는 문제가 여전히 존재한다.
    - 따라서, 객체를 복사할 때 **중첩 객체를 갖는 property에 대해 객체 복사를 재귀적으로 수행**하여 깊은 복사를 구현해야 한다.
        ```javascript
        // 객체의 property를 순회하며 중첩 객체에 대해 재귀적으로 실행되는 함수
        function copyObjectDeep(target) {
            // 1. 객체 및 모든 중첩 객체에 대해 재귀적으로 복사를 수행
            if (typeof target === "object" && target !== null) {
                let result = {};
                for (const prop in target) {
                    result[prop] = copyObjectDeep(target[prop]);
                }
                return result;
            } 
            // 2. 기본값은 그대로 반환
            else {
                return target;
            }
        }

        let obj1 = { 
            a: 1, 
            b: { 
                c: null, 
                d: [1, 2] 
            } 
        };
        let obj2 = copyObjectDeep(obj1);
        obj2.a = 3;
        obj2.b.c = 4;
        obj1.b.d[1] = 3;

        console.log(obj1); // { a: 1, b: { c: null, d: [1, 3] } }
        console.log(obj2); // { a: 3, b: { c: 4, d: [1, 2] } }
        ```
        - `for..in` 반복문을 사용할 때 prototype chaining을 통해 상속받은 property들은 [`Object.hasOwnProperty`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty) method로 filtering 가능
        - 객체의 getter/setter는 [`Object.getOwnPropertyDescriptor`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)를 사용해서 복사 가능
- Method, hidden property(`__proto__`), getter/setter 등을 제외한 순수 데이터만 복사한다면 **JSON 직렬화/역직렬화**를 사용해서 간편하게 깊은 복사를 구현할 수 있다.
    ```javascript
    function copyObjectUsingJSON(target) {
        return JSON.parse(JSON.stringify(target));
    }

    let obj1 = { 
        a: 1, 
        b: { 
            c: null
        } 
    };
    let obj2 = copyObjectUsingJSON(obj1);
    obj2.a = 3;
    obj2.b.c = 4;
    console.log(obj1); // { a: 1, b: { c: null } }
    console.log(obj2); // { a: 3, b: { c: 4 } }
    ```

## undefined와 null

- JavaScript에서 `undefined`와 `null`은 '없음'을 나타내는 값이다.
- `undefined`는 JavaScript 엔진이 **값이 존재하지 않을 때** 자동으로 부여하는 값이다.
    - 값을 대입하지 않은 변수(데이터 영역의 메모리 주소를 지정하지 않은 식별자)에 접근
        - `var` 변수는 데이터 영역의 메모리 주소를 지정하지 않는 식별자에 `undefined`를 할당
        - `const`, `let` 변수는 이러한 식별자에 **`undefined`를 할당하지 않은 채로 초기화를 마치고**, 이후 실제 변수가 평가될 때 `undefined`로 초기화
            - TDZ(Temporary Dead Zone) 형성
            - 즉, **`const`, `let` 변수는 실제 변수가 평가되기 전까지는 접근할 수 없다.**
    - 객체에 존재하지 않는 property에 접근
    - `return` 문이 없거나 호출되지 않는 함수의 실행 결과
- 배열에서 `undefined`의 동작
    - 빈 배열을 만들면 `undefined`가 아닌 `empty`를 요소로 갖는다.
        ```javascript
        let arr1 = new Array(3);
        console.log(arr1); // [empty, empty, empty]

        let arr2 = [undefined, undefined, undefined];
        console.log(arr2); // [undefined, undefined, undefined]
        ```
    - `empty` 요소는 배열의 순회 대상에서 제외되지만, `undefined`는 순회 가능하다.
        ```javascript
        var arr1 = [undefined, 1]; // [undefined, 1]
        var arr2 = [];
        arr2[1] = 1; // [empty, 1];

        arr1.forEach((v, i) => console.log(v, i)); 
        // undefined 0
        // 1 1

        arr2.forEach((v, i) => console.log(v, i)); 
        // 1 1
        ```
        - 배열은 key가 index인 property를 가진 객체임
        - 즉, 배열에서 **`empty`는 객체에 존재하지 않는 property를 나타내는 것**으로, 순회할 수 없는 것이 당연함
        - `undefined`는 값이 `undefined`인 property이기 때문에 순회 가능
- '값이 없음'읆 명시적으로 표현하는 방법
    - `undefined`는 JavaScript engine이 자동으로 할당하기도 하고 개발자가 '값이 없음'을 표현하기 위해 직접 할당할 수도 있음
    - 혼동을 피하기 위해, **개발자가 '값이 없음'을 명시적으로 표현할 때는 `undefined` 대신 `null`을 사용**하고, `undefined`는 JavaScript engine이 반환하는 값으로만 남겨둔다.
    - 값이 `null`인지 확인하는 방법
        - `typeof` 연산자가 `null`값에 대해 `object`를 반환하는 버그때문에, 값이 `null`인지 확인하기 위해 `typeof` 연산자를 사용할 수 없음
        - 동등 연산자(`==`)는 `undefined`와 `null`을 같다고 평가하므로 사용할 수 없음
        - 따라서, **`null`값을 정확히 판단할 때는 일치 연산자(`===`)를 사용한다.**