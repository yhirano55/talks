# webpack, 5min

author: @yhirano55

## Create project and initialize

```sh
$ mkdir ~/src/webpack-demo && cd $_
$ git init
$ echo "node_modules" >> .gitignore
$ git add .gitignore
$ git commit -m "init"
```

## Install webpack

```sh
$ yarn init
$ git add package.json
$ git commit -m "yarn init"
$ yarn add webpack --dev
$ git add .
$ git commit -m "yarn add webpack --dev"
```

## Create app/index.js

```javascript
function component() {
  var elem = document.createElement("div");
  elem.innerHTML = _.join(["hello", "webpack"], ", ");
  return elem;
}

document.body.appendChild(component());
```

## Create index.html

```html
<html>
	<head>
		<title>webpack demo</title>
		<script src="https://unpkg.com/lodash@4.16.6"></script>
	</head>
	<body>
		<script src="app/index.js"></script>
	</body>
</html>
```

and `open index.html`

```sh
$ git add .
$ git commit -m "app init"
```

## Using webpack

add lodash to package.json:

```sh
$ yarn add lodash
$ git add .
$ git commit -m "yarn add lodash"
```

modify `app/index.js`:

```sh
% git diff app/index.js
diff --git a/app/index.js b/app/index.js
index e1bd23d..d4c7ef9 100644
--- a/app/index.js
+++ b/app/index.js
@@ -1,3 +1,5 @@
+import _ from "lodash";
+
 function component() {
   var elem = document.createElement("div");
   elem.innerHTML = _.join(["hello", "webpack"], ", ");
```

modify `index.html`:

```sh
% git diff index.html
diff --git a/index.html b/index.html
index ef8be2c..7c6cb40 100644
--- a/index.html
+++ b/index.html
@@ -1,9 +1,8 @@
 <html>
        <head>
                <title>webpack demo</title>
-               <script src="https://unpkg.com/lodash@4.16.6"></script>
        </head>
        <body>
-               <script src="app/index.js"></script>
+               <script src="dist/bundle.js"></script>
        </body>
 </html>
```

then run command:

```sh
$ ./node_modules/.bin/webpack app/index.js dist/bundle.js
```

```sh
$ git add .
$ git commit -m "modify app"
```

## Configure webpack

add `webpack.config.js`:

```javascript
var path = require('path');

module.exports = {
  entry: './app/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

then you can build through `./node_modules/.bin/webpack --config webpack.config.js`, but too long to input.

edit `package.json`:

```sh
% git diff package.json
diff --git a/package.json b/package.json
index 2dc4d1c..03b1fa5 100644
--- a/package.json
+++ b/package.json
@@ -9,5 +9,8 @@
   },
   "dependencies": {
     "lodash": "^4.17.4"
+  },
+  "scripts": {
+    "build": "webpack"
   }
 }
```

then you can build `yarn build`

## Using loaders

install loaders:

```sh
$ yarn add style-loader css-loader url-loader --dev
```

edit `webpack.config.js`:

```javascript
var path = require('path');

module.exports = {
  entry: './app/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    loaders: [
      {
        test: /\.css$/, loaders: ["style-loader", "css-loader"]
      },
      {
        test: /\.jpg$/, loaders: ["url-loader"]
      }
    ]
  }
};
```

then add `app/style.css`:

```css
body {
  background: #222;
  color: #f0f0f0;
}
```

and add `photo.jpg`.

## Using Stylesheet and Image

edit `app/index.js`;

```javascript
import _ from "lodash";
import style from "./style.css";
import photo from "./photo.jpg";

function component() {
  var elem = document.createElement("div");
  elem.innerHTML = _.join(["hello", "webpack"], ", ");
  return elem;
}

function image() {
  var elem = document.createElement("img");
  elem.src = photo;
  return elem;
}

document.body.appendChild(component());
document.body.appendChild(image());
```

then `yarn build && open index.html`

## So...what's webpack???

webpack is **module bundler**.

> - Bundles ES Modules, CommonJS and AMD modules (even combined).
> - Can create a single bundle or multiple chunks that are asynchronously loaded at runtime (to reduce initial loading time).
> - Dependencies are resolved during compilation, reducing the runtime size.
> - Loaders can preprocess files while compiling, e.g. TypeScript to JavaScript, Handlebars strings to compiled functions, images to Base64, etc.
> - Highly modular plugin system to do whatever else your application requires.

source: webpack's [README](https://github.com/webpack/webpack)

1. モジュールの依存解決
2. npm modulesの依存解決
3. チャンクに分割して、必要なときに必要なファイルを非同期で読み込む
4. 中間言語のトランスパイルや画像ファイルのbase64形式への変換
5. assetsにdigestの付与もできる

## Repository

[yhirano55/webpack-demo](https://github.com/yhirano55/webpack-demo)

ありがとうございました:innocent:
