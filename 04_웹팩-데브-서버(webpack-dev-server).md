## 웹팩 데브 서버(Webpack Dev Server)

### 웹팩 데브 서버가 필요한 이유

코드를 수정했을 때, `npm run build → 새로고침`과 같은 반복적인 과정의 불편을 덜 수 있게...

<br />

### Webpack Dev Server

웹팩의 빌드 대상 파일이 변경되었을 때, 매번 웹팩 명령어를 실행하지 않고 코드만 변경하고 저장하면 웹팩으로 빌드한 후 브라우저를 새로고침 한다.

매번 명령어를 치거나 브라우저를 새로 고침하는 번거로움과 그 시간뿐 아니라 웹팩 빌드 시간 또한 줄여주기 때문에 웹팩 기반의 웹 애플리케이션 개발에 필수로 사용된다.

<br />

#### 웹팩 데브 서버의 특징

웹팩 데브 서버는 일반 웹팩 빌드와 다른점이 있다. 먼저 명령어를 보자.

```json
"scripts": {
  "dev": "webpack-dev-server",
  "build": "webpack"
}
```

웹팩 데브 서버를 실행하여 웹팩 빌드를 하는 경우에는 빌드한 결과물이 파일 탐색기나 프로젝트 폴더에서 보이지 않는다. 좀 더 구체적으로 얘기하자면 웹팩 데브 서버로 빌드한 결과물은 '메모리에 저장'되고 파일로 생성하지는 않기 때문에 '컴퓨터 내부적으로는 접근할 수 있지만' 사람이 직접 눈으로 보고 파일을 조작할 순 없다.<br />

그러므로 웹팩 데브 서버는 '개발할 때만 사용'하다가 개발이 완료되면 웹팩 명령어를 통해 '결과물'을 파일로 생성해야 한다.

> 컴퓨터 구조 관점에서 파일 입출력보다 메모리 입출력이 더 빠르고 컴퓨터 자원을 덜 소모한다.<br />
> file-level이 아닌 in-memory-level에서 실행된다...! >.<

<br />

#### 웹팩 데브 서버 튜토리얼

> 웹팩 데브 서버로 빌드한 결과물이 파일 탐색기에서 보이진 않지만 정상적으로 애플리케이션에 로딩되어 돌아가는 것에 주의하자.

실습 절차<br />

(1) `package.json` 생성 <br />

```sh
npm init -y
```

(2) 아래 명령어로 필요 라이브러리 설치 <br />

```sh
npm i webpack webpack-cli webpack-dev-server html-webpack-plugin -D
```

(3) `package.json` 파일에서 아래와 같이 `scripts` 속성에 커스텀 명령어를 추가한다. <br />

```json
{
  // ...
  "scripts": {
    "dev": "webpack serve"
  }
}
```

(4) 프로젝트 루트 레벨에 `index.html` 파일 생성 후 내용을 추가한다.<br />

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Webpack Dev Server</title>
  </head>
  <body>
    <!-- 빌드 결과물이 정상적으로 로딩되면 아래 div 태그의 텍스트가 변경됨 -->
    <div class="container">TBD..</div>
    <!-- HTML Webpack Plugin에 의해 웹팩 빌드 내용이 아래에 추가됨 -->
  </body>
</html>
```

(5) 프로젝트 루트 레벨에 `index.js` 파일을 생성하고 아래 내용을 추가한다. <br />

```js
const div = document.querySelector(".container");
div.innerText = "Webpack loaded!!!";
```

(6) 웹팩 설정 파일 `webpack.config.js`를 생성하고 아래 내용을 추가한다.

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "none",
  entry: "./index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  devServer: {
    port: 9000,
    // overlay, ... 등등
  },
  plugins: [
    new HtmlWebpackPlugin({
      // index.html 템플릿을 기반으로 빌드 결과물을 추가해줌
      template: "index.html",
    }),
  ],
};
```

(7) 명령어 입력 창에 `npm run dev`를 입력하여 웹팩 데브 서버를 실행한다.<br />
(8) localhost:9000에 접속 후 화면에 'Webpack loaded!!' 문구를 확인한다.

<img width="800" src="https://user-images.githubusercontent.com/19165916/201138258-e58a01c1-7a30-4cc1-b696-b8610c7bbc58.png" /><br />
<img width="280" src="https://user-images.githubusercontent.com/19165916/201138273-c1c7a639-72ed-4b26-8dd2-44db961a0620.png" /><br />

> 에디터 어디에서도 dist/bundle.js를 찾아볼 수 없지만,<br />
> 개발자도구 `network`탭에서는 이를 불러오고 있다. (in-memory)

<img width="1162" src="https://user-images.githubusercontent.com/19165916/201140079-0ef92dfe-0a0f-48da-9e4a-a88d0d6bd2f5.png"><br />

> HtmlWebpackPlugin이 `<script>`를 추가한 걸 확인할 수 있다.
