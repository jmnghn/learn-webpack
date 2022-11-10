## "프론트엔드 개발자를 위한 웹팩" 학습 log

<br />

### 웹팩에서의 모듈

웹팩에서 지칭하는 모듈이라는 개념은 위와 같이 자바스크립트 모듈에만 국한되지 않고 웹 애플리케이션을 구성하는 모든 자원을 의미한다. 웹 애플리케이션을 제작하려면 HTML, CSS, JavaScript, Images, Font 등 많은 파일들이 필요하다. 이 파일 '하나하나가 모두 모듈'이다.

<br />

### 웹팩의 등장 배경

웹팩이 등장한 이유는 크게 3가지 이다.

#### 1. 파일 단위의 자바스크립트 모듈 관리

```js
let a = 10;
console.log(a); // 10

function logText() {
  console.log(a); // 10
}
```

```html
<html>
  <head>
    <!-- ... -->
  </head>
  <body>
    <!-- ... -->
    <script src="./app.js"></script>
    <script src="./main.js"></script>
  </body>
</html>
```

```js
// app.js
let num = 10;
function getNum() {
  console.log(num);
}
```

```js
// main.js
let num = 20;
function getNum() {
  console.log(num);
}
```

```html
<html>
  <head>
    <!-- ... -->
  </head>
  <body>
    <!-- ... -->
    <script src="./app.js"></script>
    <script src="./main.js"></script>
    <script>
      getNum(); // 20
    </script>
  </body>
</html>
```

> 파일 단위로 변수를 관리하고 싶은 욕구, 자바스크립트 모듈화에 대한 욕구를 예전까진 AMD, Common.js와 같은 라이브러리로 풀어왔다.

#### 2. 웹 개발 작업 자동화 도구

- 코딩 시 브라우저 새로 고침
- HTML, CSS, JS 압축
- 이미지 압축
- CSS 전처리기 변환

등

> Grunt와 Gulp 같은 것들의 등장.

#### 3. 웹 애플리케이션의 빠른 로딩 속도와 높은 성능

일반적으로 특정 웹 사이트를 접근할 때 5초 이내로 웹 사이트가 표시되지 않으면 대부분의 사용자들은 해당 사이트를 벗어나거나 집중력을 잃게 된다.

그래서 웹 사이트의 로딩 속도를 높이기 위해 많은 노력들이 있었다. 그 중 대표적인 노력이 브라우저에서 서버로 요청하는 파일 숫자를 줄이는 것이다. 이를 위해 앞에서 살펴본 웹 태스트 매니저를 이용해 파일들을 압축하고 병합하는 작업들을 진행했다.

뿐만 아니라 초기 페이지 로딩 속도를 높이기 위해 나중에 필요한 자원들은 나중에 요청하는 레이지 로딩 (Layzy Loading)이 등장했다.

웹팩은 기본적으로 필요한 자원은 미리 로딩하는게 아니라 그 때 그 때 요청하자는 철학을 갖고 있다.

> 한편으로는 이제는 너무 당연(!)해지고, 또 새로운 웹 성능지표와 그를 위한 노력들이 있구나...

<br />

### 웹팩으로 해결하려는 문제?

위 웹팩의 등장 배경에서도 살펴봤지만 웹팩에서 해결하고자 하는 기존의 문제점은 다음과 같이 4가지다.

- 자바스크립트 변수 유효 범위
- 브라우저별 HTTP 요청 숫자의 제약
- 사용하지 않는 코드의 관리
- Dynamic Loading & Lazy Loading 미지원

#### 자바스크립트 변수 유효 범위

웹팩은 변수 유효 범위의 문제점을 [ES6의 Modules 문법](https://babeljs.io/docs/en/learn#modules)과(ESM) 웹팩의 번들링으로 해결한다.

#### 브라우저별 HTTP 요청 숫자의 제약

TCP 스펙에 따라 브라우저에서 한 번에 서버로 보낼 수 있는 HTTP 요청 숫자는 제약되어 있다. 아래의 표는 최신 브라우저 별 최대 HTTP 요청 횟수다.

| 브라우저        | 최대 연결 횟수 |
| --------------- | -------------- |
| 크롬            | 6              |
| 사파리          | 6              |
| 파이어폭스      | 6              |
| 오페라          | 6              |
| 안드로이드, ios | 6              |

> 22년 6월 15일 - 익스플로러 공식 지원 종료<br />

따라서, 'HTTP 요청 숫자를 줄이는 것'이 웹 애플리케이션의 성능을 높여줄 뿐만 아니라 사용자가 사이트를 조작하는 시간을 앞당겨 줄 수 있다.

> 클라이언트에서 서버에 HTTP 요청을 보내기 위해서는 먼저 TCP/IP가 연결되어야 한다.

웹팩을 이용해 여러 개의 파일을 하나로 합치면, 위와 같은 브라우저별 HTTP 요청 숫자 제약을 피할 수 있다. ('웹팩-소개-영상'에도 나온 내용)

#### Dynamic Loading & Lazy Loading 미지원

[Require.js](https://requirejs.org/)와 같은 라이브러리를 쓰지 않으면 동적으로 원하는 순간에 모듈을 로딩하는 것이 불가능 했다. 그러나 이젠 웹팩의 Code Splitting 기능을 이용하여 원하는 모듈을 원하는 타이밍에 로딩할 수 있다.
