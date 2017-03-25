# Type

## Primitive

- number, string, boolean
- undefined, null

값이 할당되지 않은 변수는 타입이 undefined이고 값도 undefined

```javascript
typeof NaN === typeof 1; // NaN의 타입은 number
typeof 1 === 'number'; // true

typeof 'abc' === 'string'; // true

typeof true === 'boolean'; // true

typeof undefined === 'undefined'; // true

typeof abc === 'undefined' // true, 정의하지 않은 변수를 typeof 해도 에러 안나고 undefined 반환 

// null의 실제 타입은 null 이지만,
// typeof 연산자로는 object로 나옴
// null이 빈 객체를  '참조' 하므로
// 최초 JS 만들때 이렇게 정했다고 함
// 변수가 빈 객체를 가리키게 하려면 null로 명확하게 초기화
typeof null === typeof {}; // true
typeof null === 'object'; // true

var str = 'string'; // undefined
str[0]; // 's'
str[0] = 'S'; // 'S' 
console.log(str); // 'string', 즉, string은 불변

NaN === NaN; // false, 자기 자신과 === 하는데 false가 나오는 유일한 값
Number.NaN === NaN; // NaN의 값은 Number.NaN과 같지만 식의 값은 false로 나옴
isNaN(NaN); // true
isNaN(Number.NaN); // true

null === undefined; // false
null == undefined; // true
// undefined는 null과 초기화되지 않은 변수를 비교할 목적으로 ECMA=262 3판에서 추가
```
- JavaScript의 모든 타입은 Boolean 값으로 변환될 수 있다.
	- false가 되는 값 : 빈문자열, 0, NaN, null, undefined 
	- 그 외는 모두 true

- 숫자로 자동 변환 시도 순서
	- valueOf() 반환값
		- valueOf()가 NaN을 반환하면 toString() 반환값

## Reference

### Object

```javascript
console.dir({});
// 출력 결과
Object
  - __proto__: Object (Object라고 표시되지만 실제 값은 Object.prototype임)

// [[Prototype]] 확인
var obj = {};
obj.__proto__ === Object.prototype; // true
obj.__proto__ === Object; // false
Object.prototype.__proto__ === null; // true, 그냥 스펙에서 이렇게 정함
```

JS에서 Primitive type을 제외한 모든 데이터는 Object.

- 모든 객체는 [[Prototype]] 속성을 가진다.
- [[Prototype]] 속성은 일반적으로 어떤 함수 객체의 prototype 속성이 가리키는 객체를 가리킨다.(아래 Function 참고)
- 모든 객체는 [[Prototype]] 속성이 가리키는 객체 A의 속성과 메서드에 접근 및 사용할 수 있다.(프로토타입 상속)
- 모든 객체는 [[Prototype]] 속성이 가리키는 객체 A의 [[Prototype]] 속성이 가리키는 객체 B의 속성과 메서드에 접근 및 사용할 수 있다.(프로토타입 체이닝)
- [[Prototype]]이 가리키는 객체 A를 다른 객체 C로 변경할 수 있다. 즉, [[Prototype]]에 다른 객체를 할당할 수 있으며, 이 방식으로 부모를 선택할 수 있다.

#### Array

```javascript
console.dir([]);
// 출력 결과
Array[0]
  - length: 0
  - __proto__: Array[0] (Array[0]라고 표시되지만 실제 값은 Array.prototype임)

// [[Prototype]] 확인
var arr = [];
arr.__proto__ === Array.prototype; // true
Array.prototype.__proto__ === Object.prototype; // true
```
JS에서 Array 객체는 length 속성을 중심으로 동작한다.
- 모든 Array 객체는 length 속성을 가진다.
- length 속성은 Array의 최대 index + 1의 값을 가진다.
- length 속성을 가진 모든 객체는 유사 배열 객체로서 apply/call 을 통해 push(), shift() 등 Array.prototype의 메서드를 적용할 수 있다.

```javascript
var pseudoArr = {};
pseudoArr.length = 0;
Array.prototype.push.call(pseudoArr, 0);
console.log(pseudoArr); // Object {0: 0, length: 1}
```

#### Function

```javascript
console.dir(function() {});
// 출력 결과
function anonymous()
  arguments: null
  caller: null
  length: 0
  name: ""
  prototype: Object
    constructor: function()
    __proto__: Object (Object라고 표시되지만 실제로는 Object.prototype)
  __proto__: function() (function()라고 표시되지만 실제로는 Function.prototype)
  <function scope>
    Global: window

// [[Prototype]] 확인
var func = function() {};
func.__proto__ === Function.prototype; // true
func.__proto__ === Function; // false
Function.prototype.__proto__ === Object.prototype; // true, 그냥 스펙에서 이렇게 정함

// prototype 확인
func.prototype === Function.prototype; // false
typeof func.prototype === 'object'; // true
typeof func.prototype === 'function'; // false
console.dir(func.prototype);
// 출력 결과
func
  constructor: function()
  __proto__: Object (Object라고 표시되지만 실제로는 Object.prototype)
```

- 모든 함수는 객체다. 따라서 함수도 [[Prototype]] 속성을 가진다.
- 함수 A가 정의될 때 함수의 ECMAScript의 표준 속성인 length와 prototype 속성이 생기고,
    - 비표준 속성인 arguments, caller, name 속성도 생긴다.
    - scope도 함께 생긴다.
- A의 [[Prototype]]은 Function.prototype이 가리키는 객체를 가리킨다.
    - 즉 A.\__proto\__와 Function.prototype은 동일한 객체를 가리킨다.
    
            A.__proto__ === Function.prototype

- 함수 A가 정의될 때, A.prototype이 가리키는 객체 B도 함께 생성된다.
- 프로토타입 객체 B는 함수 객체가 아니다.
    - 프로토타입 객체 B도 객체이므로 [[Prorotype]] 속성을 가지며, B의 [[Prototype]] 속성은 Object.prototype을 가리킨다.
    - 프로토타입 객체 B는 constructor라는 속성을 가지며, B의 constructor 속성은 함수 객체 A를 가리킨다.
    - 프로토타입 객체 B는 constructor라는 이름과는 다르게 constructor가 가리키는 함수 A에 의해 생성되는 것이 아니라 Object()에 의해 생성되며, 
        - 생성된 이후 contstructor에 함수 객체 A가 할당되는 것이다.

- 일반적으로 어떤 함수 객체의 prototype 속성이 가리키는 객체는 함수가 아니지만, 예외적으로 Function.prototype이 가리키는 객체는 함수다.    

        typeof A.prototype === 'object'
        typeof Function.prototype === 'function'
- Function.prototype이 가리키는 객체가 함수이기 때문에 아래와 같은 여러가지 논리적 모순이 생기지만, 구버전과의 호환성 유지를 위해 ECMAScript 6에서도 Function.prototype은 함수로 지정
    - Function.prototype이 가리키는 객체가 함수라면, 함수의 표준 속성인 prototype을 가져야하므로 Function.prototype.prototype도 있어야 하고, Function.prototype.prototype.prototype도 있어야 하는 무한 문제가 생기지만,
        - Function.prototype이 가리키는 객체는 함수이면서도 예외적으로 prototype 속성이 없다라고 스펙에 명시하는 것으로 무한 문제 해결.
    - Function.prototype이 가리키는 객체는 모든 함수의 부모 역할을 하는 객체라서, 임의의 함수 A에 대해

        `A.__proto__ === Function.prototype`이 성립한다.
        - Function.prototype이 가리키는 객체도 함수이므로 `Function.prototype.__proto__ === Function.prototype`이 성립해야하고, 이는 Function.prototype이 가리키는 객체 자신의 부모이자 자식이 된다는 이상한 논리를 형성하지만,
            - `Function.prototype.__proto__ === Object.prototype`라고 스펙에 명시하는 것으로 해결.
    - 이 밖에도 Function.prototype이 가리키는 객체에 대해 스펙에서 명시한 것은
        - Function.prototype이 가리키는 객체가 함수이므로 Function.prototype()와 같이 실행할 수도 있는데, 이렇게 실행되면 무조건 undefined를 반환
        - Function.prototype.length의 값은 0
        - Function.prototype.name의 값은 빈문자열

- var objectFromA = new A(); 와 같이 생성자 방식으로 objectFromA라는 객체를 생성하면 objectFromA.\__proto\__는 A.prototype을 가리킨다. 

#### RegExp

#### Date

# 실행 컨텍스트

>An execution context is a specification device that is used to track the runtime evaluation of code by an ECMAScript implementation.
>실행 컨텍스트는 ECMAScript 구현에 의한 코드의 실시간 평가를 추적하는데 사용하는 명세 장치다.
>실행 컨텍스트는 실행 가능한 자바스크립트 코드 블록이 실행되는 환경이다.

실행 컨텍스트는 세 가지 경우에만 생성된다.

- 전역 코드
- eval()로 실행되는 코드
- 함수의 호출

생성된 실행 컨텍스트는 실행 컨텍스트 스택에 쌓이고, 위 세 가지 경우가 종료되면 해당 실행 컨텍스트는 스택에서 빠져나간다.

```javascript
// 실행 컨텍스트의 구조를 알아보기 위한 예제 코드 
function foo(i) {
    var a = 'hello';
    var b = function privateB() {

    };
    function c() {

    }
}
foo(22);

// 코드로 표현한 실행 컨텍스트의 구조
executionContext = {
    scopeChain: { ... },
    variableObject: {
        arguments: {
            0: 22,
            length: 1
        },
        i: 22,
        c: pointer to function c()
        a: undefined,
        b: undefined
    },
    this: { ... }
}
```

## 함수의 호출

함수가 호출되면 함수의 내용을 실행하기 전에 아래와 같은 일이 순서대로 일어난다.

0. 실행 컨텍스트 생성
1. 변수 객체 생성
2. arguments 객체 생성
3. 스코프 체인 생성
4. 변수 생성
5. this 바인딩
6. 코드 실행

## 변수 객체 생성

변수 객체는 arguments 객체, 변수 등을 담는 컨테이너다.
변수 객체는 전역 객체와 활성 객체를 통칭하며, 일부에서는 변수 객체 대신 활성 객체로 표기하기도 한다.
변수 객체는 함수의 실행이 끝나면 소멸한다.

```javascript
variableObject = {
    arguments: {
    },
    var1: value1,
    var2: value2,
    var3: value3,
    ...
}
```

변수 객체에는 변수와 함수가 저장되지만, 괄호 안에 정의된 함수는 변수 객체에 저장되지 않는다.
```javascript
function funcDef() {}
(function funcInParen() {});

console.log(funcDef); // funcDef() {} 표시
console.log(funcInParen); // Uncaught ReferenceError: funcInParen is not defined(…)

```

## arguments 객체 생성

함수 호출 시 넘겨받은 인자를 저장한다.
primitive 형이면 값을 복사하고, 참조면 주소값을 저장한다.

## 스코프 체인 객체 생성

스코프 체인 객체 [[Scope]]는 변수 객체를 원소로 가지고 있는 링크드 리스트다.

[[Scope]] 자체는 함수의 호출이 아니라 함수의 정의 시에 이미 생성되어, 
함수의 정의 당시의 실행 컨텍스트 내에 있는 [[Scope]]를 가리키고 있다.

즉, 어떤 함수 A가 정의된다는 것은, 함수 A를 정의하는 코드가 실행된다는 의미고,
함수 A를 정의하는 코드를 실행할 수 있는 실행 컨텍스트 B(함수 A의 실행 컨텍스트가 아닌)가 이미 존재한다는 것을 의미한다.
함수 A를 정의하는 코드가 실행될 때 함수 A의 [[Scope]]는 실행 컨텍스트 B에 있는 [[Scope]]를 가리킨다.

함수 A가 호출되어 함수 A의 실행 컨텍스트 C가 생기면, 함수 A의 정의 당시의 실행 컨텍스트 B에 있는 [[Scope]]가 가리키고 있던 모든 변수 객체의 참조를 새로운 링크드 리스트 D에 복사하고, A의 호출에 의해 생성된 실행 컨텍스트 C 내의 변수 객체의 참조를 D에 추가한 후, A의 실행 컨텍스트인 C에 있는 [[Scope]]는 D를 가리키게 된다.

A에서는 링크드 리스트 D에 담긴 모든 변수 객체에 담겨있는 모든 변수, 즉, A를 포함하여 A를 감싸고 있는 영역의 모든 변수와 함수에 접근할 수 있다.  

## 변수 생성

변수의 참조만 생성되어 메모리에 잡히고, 값은 undefined로 초기화된다.
변수의 값은 함수가 실행되면서 할당식을 만나면 다시 할당된다.

## this 바인딩

this 바인딩은 뉴쩜콜라

다른 함수 A의 파라미터로 전달되는 함수 B에 기술되어 있는 this는 기본적으로는 중첩 함수로서 전역 객체가 할당되지만 대부분 A에서 call(), apply(), bind()를 통해 B의 this에 어떤 값을 바인딩하므로, 소스를 보거나 직접 테스트 해보지 않는한 알 수 없다.

# Chain

JavaScript에는 두 종류의 Chain이 존재한다.
하나는 Prototype Chain이고, 다른 하나는 Scope Chain이다.

## Prototype Chain

프로토타입 체인은 객체의 **속성(Property)**과 상속과 관련이 있다. 
따라서 `.`이나 `[]`를 통해 접근할 수 있는 속성을 찾을 때 프로토타입 체인이 사용되며, 
`.`이나 `[]`가 아니라 `var`로 선언된 **변수는 프로토타입 체인을 통해서는 접근할 수 없고, 스코프 체인을 통해서만 접근할 수 있다.**

어떤 객체 A에서 사용되는 a라는 속성이 A에 없으면, A.\__proto\__ 객체에서 a를 찾고, 없으면 A.\__proto\__.\__proto\__에서 a를 찾고..를 반복해서 Object.prototype 까지 찾는다.

## Scope Chain

스코프 체인은 함수의 변수(Variable) 유효범위와 관련이 있다. 
따라서, 함수 내에 사용되는 변수의 참조를 찾을 때 스코프 체인이 사용되며,   
`.`이나 `[]`로 지정된 **속성은 스코프 체인을 통해서는 접근할 수 없고, 프로토타입 체인을 통해서만 접근할 수 있다.**

어떤 함수 A에서 사용되는 a라는 변수가 A에서 선언되지 않았으면, A를 감싸고 있는 함수 AA에서 a를 찾고, 없으면 AA를 감싸고 있는 AAA함수에서 a를 찾고..를 반복해서 전역 객체까지 찾는다. {{{왜 전역 객체까지 찾지???}}} 전역 객체에 선언된 변수는 전역 객체의 속성이 되며, 전역 객체는 Object.prototype을 상속받으므로 여기서부터는 프로토타입 체인이 적용되어 Object.prototype 까지 찾는다.

{{맹 왈}}
스코프체인을 찾다 글로벌 vo에서도 못 찾으면, 기명 함수 이름을 저장해놓은 특수 객체를 뒤지게 되어 있는데, 이 특수 객체를 뒤지는 것 부터는 스코프체인이 아니라 프로토타입 체인을 탄다.
즉, 기명 함수 이름 저장 특수 객체를 뒤지다 없으면 그 객체의 [[Prototype]]인 Object.prototype을 뒤지게 된다는.. => 아니다!!

```javascript
window.a = 3;
Window.prototype.a = 2;
Object.prototype.a = 1;
var test = function a() {
　　console.log(a);
};
test(); // function a가 출력된다.

// function a()를 function b()로 바꿔서 실행하면 3이 출력
// window.a = 3 을 지우고 실행하면 2가 출력
// Window.prototype.a = 2를 지우고 실행하면 1이 출력
```


참고 : http://dmitrysoshnikov.com/ecmascript/chapter-4-scope-chain/#two-dimensional-scope-chain-lookup

- Variable Object(변수 객체)는 ECMA 5에서 Variable Environment라는 걸로 바뀌었다.
참고 : http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-2-lexical-environments-ecmascript-implementation/#variable-environment


전역 객체에 선언된 변수는 전역 객체의 속성이 된다.

```javascript
var x = 3;
console.log(x === this.x); // 전역 공간에서의 this는 전역 객체를 가리킨다.
```

# Lexical Environment

별도로 정리 예정



# HTML - JavaScript

## &lt;script>

- defer : 파일은 즉시 다운로드하지만, 스크립트 실행을 `</html>` 이후로 지연, 로딩 순서는 태그 순서대로
	- 스펙대로라면 DomContentLoaded 이벤트보다 먼저 실행되어야 하지만 현실은 그렇지 않음
- async : 파일은 즉시 다운로드하지만, 스크립트 실행을 지연하는 것은 defer와 비슷하나 로딩 순서를 보장하지 않음   

- 기본적으로는 스크립트 태그 내의 내용을 내려받고, 파싱해서 해석을 완료할 때까지 페이지 렌더링이 멈추기 때문에, `<head>` 안에 두는 것은 좋지 않다. `</body>` 바로 앞에 두자. 

## 브라우저 모드

- 쿽스 모드 : ie5인것 처럼 행동, 비표준
- 표준 모드
- `<!DOCTYPE>`을 명시하지 않으면 쿽스 모드로 동작

## "use strict"

함수 내의 첫 줄에서 쓰면 그 함수만 스트릭트 모드 적용

- 정의하지 않은 변수에 값 할당 시 에러
- 8진수 에러
- 
