(1) `package.json` 생성

```
npm init -y
```

(2) 필요 라이브러리 설치

```
npm i webpack webpack-cli css-loader style-loader mini-css-extract-plugin -D
```

(3) `index.html` 생성

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>CSS & Libraries Code Splitting</title>
  </head>
  <body>
    <header>
      <h3>CSS Code Splitting</h3>
    </header>
    <div>
      <!-- 웹팩 빌드 결과물이 잘 로딩되면 아래 p 태그의 텍스트 색깔이 파란색으로 표시됨 -->
      <p>This text should be colored with blue after injecting CSS bundle</p>
    </div>
    <!-- 웹팩의 빌드 결과물을 로딩하는 스크립트 -->
    <script src="./dist/bundle.js"></script>
  </body>
</html>
```

(4) `base.css` 생성

```css
p {
  color: blue;
}
```

(5) index.js 생성 후, `base.css` import.

```js
import "./base.css";
```

(6) `webpack.config.js` 추가

```js
const path = require("path");

module.exports = {
  mode: "none",
  entry: "./index.js",
  output: {
    filename: "bundle.js", // [chunkhash] - 194d9c955e5a082b2b89
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
};
```

> `mode` 속성은 웹팩 버전 4 이상에서 추가된 속성이라고 한다.<br />
> 웹팩으로 빌드를 할 때 `development`, `production`, `none` 모드를 설정할 수 있다.

#### 위 설정까지 마친 후의 웹팩 빌드 결과물 확인.

<img width="783" src="https://user-images.githubusercontent.com/19165916/201088124-a1c9570b-7c07-428d-9995-a86e2f24c909.png" />

> `<head>`태그 안에 `base.css`가 `<style>`로 들어가있다.<br /> > `bundle.js`에서는 무슨일이 일어난걸까...?

#### 변환된 결과물, `bundle.js` 살펴보기.

```js
var ___CSS_LOADER_EXPORT___ =
  _node_modules_css_loader_dist_runtime_api_js__WEBPACK_IMPORTED_MODULE_1___default()(
    _node_modules_css_loader_dist_runtime_noSourceMaps_js__WEBPACK_IMPORTED_MODULE_0___default()
  );
// Module
___CSS_LOADER_EXPORT___.push([module.id, "p {\n  color: blue;\n}\n", ""]);
```

> 잘은 모르겠지만, CSS LOADER를 통해 base.css에 작성했던 태그가 들어가고 있는 걸 확인할 수 있다.<br />
> 모르긴 몰라도 `.js`에서 import했을 때, 결과물에 어떻게 들어가는지를 확인했고 그러기 위해서는 loader가 필요하다는 것을 확인할 수 있었다.

#### `module:`을 주석한 후 webpack 빌드.

```
$ npm run build
> code-splitting@1.0.0 build
> webpack

ERROR in ./base.css 1:2
Module parse failed: Unexpected token (1:2) // ❌
You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. (적절한 로더를 추가해달라) See https://webpack.js.org/concepts#loaders
> p {
|   color: blue;
| }
 @ ./index.js 1:0-20

webpack compiled with 1 error
```

#### `css-loader` 먼저 적용 후, 결과 확인.

<img width="470" src="https://user-images.githubusercontent.com/19165916/201096677-fd96d49b-2c87-4eea-813a-c802b6dbde20.png"><br />

> p태그의 blue가 적용되지 않은 걸 확인할 수 있다.<br /> > `css-loader`를 적용하고, build에서는 오류가 나지 않았지만, 스타일이 적용되지 않는다는 걸 확인했다.

#### `style-loader`도 적용해서 다시 결과 확인해보기.

이제 다시 `style-loader`를 넣어 적용하고, build해보자.

```js
module: {
  rules: [
    {
      test: /\.css$/,
      use: ["css-loader", "style-loader"],
    },
  ],
},
```

```
ERROR in ./base.css
Module build failed (from ./node_modules/css-loader/dist/cjs.js):
CssSyntaxError // ❌😳

(2:7) /Users/.../code-splitting/base.css Unknown word // ❌

  1 |
> 2 |       import API from "!./node_modules/style-loader/dist/runtime/injectStylesIntoStyleTag.js";
    |       ^
  3 |       import domAPI from "!./node_modules/style-loader/dist/runtime/styleDomAPI.js";
  4 |       import insertFn from "!./node_modules/style-loader/dist/runtime/insertBySelector.js";
 @ ./index.js 1:0-20

webpack compiled with 1 error
```

> ❌ 뭔가, css부터 에러가 난다.

```js
module: {
  rules: [
    {
      test: /\.css$/,
      use: ["style-loader", "css-loader"],
    },
  ],
},
```

> `style-loader`와 `css-loader`의 순서를 바꿔서 다시 실행해보면, 잘 실행된다.<br />

이를 통해 `style-loader`의 역할을 추측해볼 수 있는데,<br />
`css-loader`는 css가 웹팩 안으로(!)들어갈 수 있게 한거고,<br />
`style-loader`는 웹팩 안으로 들어간 스타일을 `<head>`안에 인라인 스타일로 넣어주는 역할을 한다.<br />

이를 통해 loader의 순서도 연관이 있다는 걸 알 수 있었다.<br />
위에도 적었지만, loader는 '오른쪽에서 왼쪽 순서'로 적용이 된다.

```js
// 또 다른 샘플 코드 (.scss)
module: {
  rules: [
    {
      test: /\.scss$/,
      use: ["style-loader", "css-loader", "sass-loader"],
    },
  ],
},
```

> scss파일을 css로 바꾸고, css 파일을 웹팩으로 넣은 후 head태그 안에 넣는 동작. (오른쪽에서 왼쪽)

<br />

(7) css파일을 별도로 분리하기 위해 `MiniCssExtractPlugin` 플러그인 설정을 추가.

```js
// webpack.config.js
var path = require("path");
var MiniCssExtractPlugin = require("mini-css-extract-plugin"); // ✅

module.exports = {
  mode: "none",
  entry: "./index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [{ loader: MiniCssExtractPlugin.loader }, "css-loader"], // ✅
      },
    ],
  },
  plugins: [new MiniCssExtractPlugin()], // ✅
};
```

<img width="320" src="https://user-images.githubusercontent.com/19165916/201104434-f9d5fd17-4a94-4cd9-a84b-c1d03af69957.png"><br />

> 아까와는 다르게 `dist` 디렉토리에 main.css가 생긴 걸 확인할 수 있다.

#### `plugins`를 빼먹어도 에러메시지로 알려준다.

```
ERROR in ./base.css
Module build failed (from ./node_modules/mini-css-extract-plugin/dist/loader.js):
Error: You forgot to add 'mini-css-extract-plugin' plugin (i.e. `{ plugins: [new MiniCssExtractPlugin()] }`), please read https://github.com/webpack-contrib/mini-css-extract-plugin#getting-started
    at Object.pitch (/Users/myeonghyeonjeong/Documents/learning/LearnWebpack/code-splitting/node_modules/mini-css-extract-plugin/dist/loader.js:87:14)
 @ ./index.js 1:0-20

webpack compiled with 1 error
```

#### plugins의 역할

플러그인을 적용하고 build를 한 후 `main.css`가 따로 추출 되었지만,<br />
라이브 서버로 index.html을 열어보면 적용되지 않은 모습을 확인할 수 있다. 스타일을 직접 `<link>`로 적용시켜줘야 한다.

```html
...
  <title>CSS & Libraries Code Splitting</title>
  <link rel="stylesheet" href="./dist/main.css" />
</head>
...
```

이처럼 플러그인은 결과물에 대해 무언가를 바꿔서 제공해줄 수 있다는 걸 알 수 있었다.

<br />

### plugin

플러그인은 웹팩의 기본적인 동작에 추가적인 기능을 제공하는 속성이다. (마치 VSCode의 Extensions나 Chrome의 확장프로그램 같이)<br />

로더와 비교해보면 로더는 파일을 해석하고 변환하는 과정에 관여하는 반면, 플러그인은 해당 결과물의 형태를 바꾸는 역할을 한다고 보면 된다.

```js
// webpack.config.js
module.exports = {
  plugins: [],
};
```

플러그인의 배열에는 '생성자 함수로 생성한 객체 인스턴스'만 추가될 수 있다.

```js
const webpack = require("webpack");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  plugins: [new HtmlWebpackPlugin(), new webpack.ProgressPlugin()],
};
```

> `HtmlWebpackPlugin`: 웹팩으로 빌드한 결과물로 HTML 파일을 생성해주는 플러그인<br /> > `webpack.ProgressPlugin`: 웹팩의 빌드 진행율을 표시해주는 플러그인

#### 자주 사용하는 플러그인

> [https://webpack.js.org/plugins/](https://webpack.js.org/plugins/)

- [split-chunks-plugin](https://webpack.js.org/plugins/split-chunks-plugin/)
- [clean-webpack-plugin](https://www.npmjs.com/package/clean-webpack-plugin)
- [image-webpack-loader](https://github.com/tcoopman/image-webpack-loader)
- [webpack-bundle-analyzer-plugin](https://github.com/webpack-contrib/webpack-bundle-analyzer)

<br />

### 주요 속성 4가지 리뷰 및 정리

<img src="https://user-images.githubusercontent.com/19165916/201109633-5445012e-b89d-4e4c-987e-72c6ad39230b.png" /><br />

- entry: 웹팩을 실행할 대상 파일. 진입점.
- output: 웹팩의 결과물에 대한 정보를 입력하는 속성. 일반적으로 `filename`과 `path`를 정의한다.
- loader: css, 이미지와 같은 자바스크립트가 아닌 파일을 웹팩이 해석할 수 있게 추가하는 속성. (오른쪽에서 왼쪽 순으로 적용된다)
- plugin: 웹팩으로 변환한 파일에 '추가적인 기능'을 더하고 싶을 때 사용하는 속성. (변환 과정 전반에 대한 제어권을 갖고 있다 / 전반적인 과정(loader)에도 관여할 수 있다. 도식은 설명상...)

> 이외에도 [resolve](https://webpack.js.org/configuration/resolve/#root), [devServer](https://webpack.js.org/configuration/dev-server/#root), [devtool](https://webpack.js.org/configuration/devtool/#devtool) 속성에 대해 알고 있으면 좋다.

<br />

### 웹팩 설정 파일 설정 및 변경할 때 참고할 자료

- [Loaders](https://webpack.kr/loaders/)
- [Plugins](https://webpack.kr/plugins/)
