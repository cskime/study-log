# Summary

## 1. Data Type

### 기본형 및 참조형 데이터의 메모리 동작 방식

- JavaScript는 변수를 선언하고 값을 할당하는 과정에서 메모리를 두 가지 영역으로 나누어 사용함
    - **변수 영역** : 변수에 할당되는 영역
    - **데이터 영역** : 실제 데이터를 저장하는 영역
- 일반적인 변수는 변수 영역에 메모리가 할당되고, 할당되는 값이 저장된 데이터 영역의 주소가 저장됨
- 기본형 데이터가 메모리에 저장되는 과정
    <br><img src="./images/01-data-type-01.png" width="400px" /><br>
    1. 변수는 메모리의 변수 영역에 할당하고
    2. 값은 메모리의 데이터 영역에 할당해서 저장한 뒤
    3. 값이 저장된 데이터 영역 주소값을 변수에 저장
- 객체 등 참조형 데이터가 메모리에 저장되는 과정
    <br><img src="./images/01-data-type-03.png" width="650px" /><br>
    1. 객체에 선언된 property들을 별도의 변수 영역에 할당하고,
    2. Property의 값은 데이터 영역에 저장한 뒤 그 주소값을 property의 변수 영역에 저장한 뒤,
    3. 객체의 property들이 저장된 변수 영역 메모리 주소를 데이터 영역에 저장해서,
    4. 3번의 데이터 영역 주소값을 객체를 저장하는 변수의 변수 영역 메모리에 저장
- 기본적으로 값은 데이터 영역에 저장되고 그 주소값이 변수에 할당되므로, 이미 데이터 영역에 존재하는 값은 메모리를 새로 할당하지 않고 기존 메모리 주소를 재활용할 수 있음
    <br><img src="./images/01-data-type-02.png" width="400px" /><br>

### 불변성

- JavaScript에서 불변(immutable)은 **데이터 영역에 할당된 메모리에 저장된 값이 변경되지 않는 것**을 의미
- 기본형 데이터는 모두 불변값으로, 데이터 영역에서 메모리에 저장된 값을 변경하지 않고 항상 새로 메모리를 할당해서 저장한다.
- 객체 등 참조형 데이터 자체는 불변이지만 property는 변수 영역에 할당되므로 가변이다.
- 따라서, 객체 불변성을 확보하려면 항상 새로운 객체를 생성하는 방식으로 복사해야 한다.
    ```javascript
    function changeName(user, newName) {
        const newUser = user;        // ❌ 원본과 동일한 객체를 참조하여 원본 값에 영향을 줌
        const newUser = { ...user }; // ✅ 새로운 객체를 만들기때문에 원본에 영향을 주지 않음

        newUser.name = newName;
        return newUser;
    }
    ```
- Property가 객체를 값으로 갖는 경우, 중첩 객체들까지 깊은 복사가 이루어져야 한다.
    - 깊은 복사를 수행하는 함수를 직접 구현
        ```javascript
        function deepCopy(obj) {
            if (typeof obj === "object" && obj !== null) {
                const result = {};
                for (const key in obj) {
                    result[key] = deepCopy(obj[key]);
                }
                return result;
            } else {
                return obj;
            }
        }
        ```
        - [Object.hasOwnProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)를 사용해서 상속받은 property 제외 가능
        - [Object.getOwnPropertyDescriptor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)를 사용해서 getter/setter 복사 가능
    - Method, hidden property, getter/setter 등을 제외한 순수 데이터만 복사한다면 JSON 직렬화/역직렬화를 통해 간단하게 구현 가능
        ```javascript
        function copy(obj) {
            return JSON.parse(JSON.stringify(obj));
        }
        ```

### undefined와 null

- `undefined` : JavaScript engine이 '값이 존재하지 않음'을 표현하기 위해 자동으로 부여하는 값
- `null` : 개발자가 명시적으로 '값이 없음'을 나타낼 때 사용하는 값
- 변수 선언 시 `undefined` 초기화 비교
    - `var`는 선언한 변수에 값이 할당되지 않아도 `undefined`로 즉시 초기화하므로, 초기화 구문이 실행되기 이전에 변수에 접근해도 `undefined`를 반환할 뿐 error가 발생하지 않았음
    - `let`, `const`는 변수 초기화 구문이 실제로 실행되기 전까지 `undefined`로 초기화하지 않으므로, 초기화 구문이 실행되기 이전에 변수에 접근하면 error 발생 -> **TDZ(Temporary Dead Zone) 형성**
    - `var`, `let`, `const` 변수 모두 호이스팅(hoisting)이 일어나지만, `undefined`로 초기화되는 시점이 달라서 동작 방식에 차이가 발생한다.
- 배열에서 `undefined`의 동작
    - `undefined`는 '값이 존재하지 않음'을 나타내지만 배열은 `undefined`를 값으로 취급
    - 배열도 객체의 일종이므로, 값이 `undefined`로 초기화 된 property를 갖기 때문
- `null`을 정확하게 체크하는 방법
    - `typeof` 연산자는 `null` 값에 대해 `object`를 반환하는 버그가 있음
    - `==` 연산자는 `undefined`와 `null`을 같다고 평가하므로, 피연산자가 `null`이 아닐 수 있음
    - **`===` 연산자는 정확히 `null`인 경우에만 같다고 평가**