## 웹팩 튜토리얼 1 - 웹팩 적용전

```
npm init -y
npm i webpack webpack-cli -D
npm i lodash
```

```
make index.html
make /src/index.js
```

<br />

## 웹팩 튜토리얼 2 - 웹팩 적용 후

- `/src/index.js`에 `import _ from "lodash";`추가
- index.html에 `<script src="./dist/main.js"></script>` 추가 (기존 lodash를 load하는 스크립트 모두 제거)
- `package.json`, `"scripts"` - `"build": "webpack"` 추가
- `npm run build`
  - `/dist/main.js` 생성됨.

<br />

### 웹팩 튜토리얼 3 - mode 적용

```
$ npm run build

> getting-started@1.0.0 build
> webpack // ✅ package.json에서 설정한 명령어를 확인할 수 있다.

asset main.js 69.5 KiB [emitted] [minimized] (name: main) 1 related asset
runtime modules 1010 bytes 5 modules
cacheable modules 532 KiB
  ./src/index.js 268 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 531 KiB [built] [code generated]

WARNING in configuration // ✅ (중요) - 'none' | 'development' | 'production'
The 'mode' option has not been set, webpack will fallback to 'production' for this value.
Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/configuration/mode/
```

> `/dist/main.js`도 난독화 되어 있다.

- `package.json`, scripts 수정 `"build": "webpack --mode=none"`

```
$ npm run build
> getting-started@1.0.0 build
> webpack --mode=none

asset main.js 536 KiB [emitted] (name: main)
runtime modules 1.25 KiB 6 modules
cacheable modules 532 KiB
  ./src/index.js 268 bytes [built] [code generated]
  ./node_modules/lodash/lodash.js 531 KiB [built] [code generated]
webpack 5.74.0 compiled successfully in 137 ms
```

> WARNING 메세지가 사라진 걸 확인할 수 있다.<br /> > `/dist/main.js`또한 알아볼 수 있는 형태로 변환되어 있다.

<br />

## 웹팩 튜토리얼 4 - 웹팩 설정 파일 적용

원하는 설정을 위해 매번 명령어에 입력하는 게 아닌...

```
$ "webpack --mode=none --entry=src/index.js --output=public/output.js"
```

`webpack.config.js`

```js
var path = require("path");

module.exports = {
  mode: "none",
  entry: "./src/index.js",
  output: {
    filename: "main.js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

<br />

## 웹팩 튜토리얼 5 - 웹팩 설정 파일 설명

> [Node.js path API 문서](https://nodejs.org/api/path.html)

<br />

## 웹팩 튜토리얼 6 - 튜토리얼 소스 분석

<br />

## 웹팩 튜토리얼 7 - 웹팩 변환 전후 결과 비교

before(cdn)와 비교해봐도 개발자도구 네트워크 탭에서의 request 차이.<br />
(만약 사용하는 라이브러리의 수가 더 많아진다면...?)

<br />

## 웹팩 튜토리얼 8 - 웹팩 빌드 결과 파일 분석
