# Vue + Webpack 环境搭建

### A：项目初始化

1. 创建文件夹 vue_project


2. 命令行进入文件夹目录，执行命令

   ```shell
   npm init -y
   ```

### B：安装依赖包、搭建vue基础编译环境

1. 安装 webpack、vue、vue-loader

   ```shell
   npm install vue --save
   npm install webpack vue-loader --save-dev
   npm install css-loader vue-template-compiler --save-dev
   ```


2. 在文件目录中创建一下目录

   ```shell
   ./webpack.config.js
   ./src/app.vue
   ./src/index.js
   ```

3. 在`./src/app.vue`文件中添加以下内容

   ```html
   <template>
     <div class="test">
       {{text}}
     </div>
   </template>
   <script>
   export default {
     data() {
       return {
         text: "Hello world"
       };
     }
   };
   </script>
   <style scoped>
   .test {
     color: red;
   }
   </style>
   ```

4. 在`./src/index.js`中添加以下内容

   ```javascript
   import Vue from "vue";
   import App from "./app.vue";

   const root = document.createElement("div");
   document.body.appendChild(root);

   new Vue({
     render: h => h(App)
   }).$mount(root);
   ```

5.  在`./webpack.config.js`中添加以下内容

   ```javascript
   const path = require("path");
   const config = {
     mode: "development",
     entry: path.resolve(__dirname, "./src/index.js"),
     output: {
       filename: "bundle.js",
       path: path.resolve(__dirname, "./dist")
     },
     module: {
       rules: [
         {
           test: /\.vue$/,
           loader: "vue-loader"
         }
       ]
     }
   };
   module.exports = config;
   ```

6.  在`package.json`的`scripts`节点中添加以下内容

   ```javascript
   "build": "webpack --config webpack.config.js"
   ```

在执行完上述步骤之后，回到命令行运行下面命令：

```shell
npm run build
```

此时 webpack 4会提示需要安装`webpack-cli`，输入`yes`安装完继续打包编译即可

完成之后项目根目录会出现`dist/bundle.js`文件，即说明vue基础环境搭建成功

### C：配置CSS、图片等loader

1. 安装`style-loader ` `url-loader` `file-loader`

   ```shell
   npm install style-loader url-loader file-loader --save-dev
   ```

2. 添加测试文件

   1. 在`src`文件夹中以下文件夹

      ```
      assets/styles
      assets/images
      ```

   2. 在`styles`文件夹中新增`test.css`文件，添加以下内容

      ```css
      body {
        color: red;
        background-image: url('../images/done.svg')
      }
      ```

   3. 在`images`文件夹中添加对应的图片文件

3. 在`webpack.config.js`的`module.rules`节点中追加以下内容

   ```javascript
   {
       test: /\.css/,
       use: ["style-loader", "css-loader"]
   },
   {
       test: /\.(jpg|gif|png|jpeg|svg)/,
       use: [
           {
               loader: "url-loader",
               options: {
                   limit: 1024,
                   name:'[name].[ext]'
               }
           }
       ]
   }
   ```

4. 在`index.js`中添加测试代码(保证对应路径有对应文件，否则会报错)

   ```javascript
   import './assets/styles/test.css'
   import './assets/images/bg.jpeg'
   ```

5. 运行`npm run build`进行打包编译，成功之后`dist`文件夹中应该会出现

   ```
   bundle.js
   bg.jpeg
   ```

   两个文件，如果只有`bundle.js`文件，则可能`bg.jpeg`文件太小，被转为`base64`编码放在`bundle.js`文件中，可考虑换图片试试看

### D：配置devServer和html-webpack-plugin

1. 安装`html-webpack-plugin`

   ```shell
   npm install html-webpack-plugin --save-dev
   ```

2. `webpack.config.js`中配置html-webpack-plugin

   引入`html-webpack-plugin`，在`const path = require("path");`下面添加以下内容:

   ```javascript
   const htmlWebpackPlugin = require('html-webpack-plugin')
   ```

   在`webpack.config.js`中的`config`新增属性`plugins`

   ```javascript
   plugins:[
       new htmlWebpackPlugin()
   ]
   ```

   这时候重新回命令行执行

   ```shell
   npm run build
   ```

   **在`dist`文件夹中除了`bundle.js`文件和图片文件，还有`index.html`文件则说明`html-webpack-plugin`配置成功**

3. 安装`cross-env`和`webpack-dev-server`

   ```shell
   npm install cross-env --save
   npm install webpack-dev-server --save-dev
   ```

4. 修改`package.json`

   将原来：

   ```javascript
   "build": "webpack --config webpack.config.js"
   ```

   修改为：

   ```javascript
   "build": "cross-env NODE_ENV=production webpack --config webpack.config.js",
   ```

   并且在下面继续添加另一个执行命令：

   ```javascript
   "dev": "cross-env NODE_ENV=development webpack-dev-server --config webpack.config.js"
   ```

5. 修改`webpack.config.js`

   引入`webpack`

   ```javascript
   const webpack = require("webpack");
   ```

   判断当前环境(通过 `cross-env` 传进来的环境变量判断)

   ```javascript
   const isDev = process.env.NODE_ENV === "development";
   ```

   在`config.plugins`中添加一个插件，用于配置全局变量

   ```javascript
   new webpack.DefinePlugin({
       "process.env": isDev ? '"development"' : '"production"'
   })
   ```

   判断如果为开发环境，则进行`webpack-dev-server`

   ```javascript
   if (isDev) {
     config.devServer = {
       port: 8081,
       host: "0.0.0.0",
       contentBase: path.resolve(__dirname, "./dist"),
       overlay: {
         warnings: true,
         errors: true
       }
     };
     config.plugins.push(
       new webpack.HotModuleReplacementPlugin(),
       new webpack.NoEmitOnErrorsPlugin()
     );
   }
   ```

6. 运行`devServer`

   ```shell
   npm run dev
   ```

   如果运行没有报错，则打开浏览器，访问地址`http://0.0.0.0:8081/`



### E：完整的`webpack.config.js`文件

```javascript
const path = require("path");
const htmlWebpackPlugin = require("html-webpack-plugin");
const webpack = require("webpack");

const isDev = process.env.NODE_ENV === "development";

const config = {
  mode: "development",
  entry: path.resolve(__dirname, "./src/index.js"),
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "./dist")
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: "vue-loader"
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"]
      },
      {
        test: /\.(jpg|gif|png|jpeg|svg)/,
        use: [
          {
            loader: "url-loader",
            options: {
              limit: 1024,
              name: "[name].[ext]"
            }
          }
        ]
      }
    ]
  },
  plugins: [
    new webpack.DefinePlugin({
      "process.env": isDev ? '"development"' : '"production"'
    }),
    new htmlWebpackPlugin()
  ]
};

if (isDev) {
  //开发环境
  config.devServer = {
    port: 8081,
    host: "0.0.0.0",
    contentBase: path.resolve(__dirname, "./dist"),
    overlay: {
      warnings: true,
      errors: true
    }
  };
  config.plugins.push(
    new webpack.HotModuleReplacementPlugin(),//热更新
    new webpack.NoEmitOnErrorsPlugin() //报错不退出
  );
}

module.exports = config;

```

