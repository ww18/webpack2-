边学边翻译的，如有不当之处请指出

### 开始部分就不多说了

### 安装
#### 前期准备
在开始之前确保安装了最新的nodejs。目前的LTS是一个理想的出发点，如果你用旧版本的nodejs可能会遇到各种各样的问题。

#### 本地安装（当前项目安装）
`npm install webpack --save-dev`

`npm install webpack@<version> --save-dev`

如果你项目中使用的是npm 的script， 则npm会尝试寻找你本地安装的有用的webpack
`
"scripts": {`
    `"start": "webpack --config mywebpack.config.js"`
`}`

这是标准并推荐的做法。

#### 全局安装
`npm install webpack -g`
现在webpack命令可以全局使用

#### 提醒
尽量使用稳定版本

### 升级的问题就不多说了

### 代码分割
代码分割是webpack最令人信服的功能之一。它允许你的代码分割成各种的块，然后按需加载，比如用户去到一个匹配的route，或是用户触发了一个事件，
这需要较小的代码块，允许你根据优先级控制资源。如果运用得当，你的应用系统的加载时间将会有很大的提升。

#### 缓存和并行负载的资源分割
##### 包资源分割（外部依赖资源分割）

一个典型的应用程序可以依赖于许多第三方库的框架/功能需求。与应用程序代码不同，这些库中的代码不经常更改。

如果我们以它们自己代码块的形式保存这些代码，与应用程序代码分开，我们可以利用浏览器的缓存机制来缓存这些文件，以便在最终用户的机器上使用更长的时间。

为此，vendor文件名的［hash］部分必须保持常数，而不受应用中代码的影响。学习[如何分割库代码](https://webpack.js.org/guides/code-splitting-libraries/)
使用`CommonsChunkPlugin`插件

##### css分割

你可能也想分割样式块到单独的文件（或模块），而独立于应用逻辑。这样做的话提高了缓存，并且可以并行的加载应用程序的代码，这样的话预防了FOUC（无样式内容的闪现）。

学习如何分割css，使用`ExtractTextWebpackPlugin`插件

#### 按需的代码分割
虽然前一类的资源分割需要用户在配置中预先指定拆分点，也可以在应用程序代码中创建动态拆分点。
这可以用于更精细的分块编码，例如，按我们的申请航线或按照预测用户行为。这允许用户加载非必需资产的需求
`Code Splitting - Using import()` – ECMAScript
`Code Splitting - Using require.ensure` – CommonJS 

### css的代码分割
使用webpack处理css文件，需要将css文件引入到js文件中，就像引入其他的文件一样，使用`css-loader`，将css作为js模块输出，或是选择`ExtractTextWebpackPlugin`
将css提取并生成css文件

#### 引入css文件
和js引入其他模块一样引入css文件，如在vendor.js文件中
`import 'bootstrap/dist/css/bootstrap.css';`
#### 使用css-loader
像下面这样在webpack.config.js文件中配置

`module.exports = {`
   ` module: {`
        `rules: [{`
            `test: /\.css$/,`
            `use: 'css-loader'`
        `}]`
    `}`
`}`
这样配置的结果是css和js打包到一起了。

这有缺点，你将无法利用浏览器的能力异步并行加载CSS，而是要你的页面不得不等到你的整个JavaScript加载完成，才能应用css到dom中。
`ExtractTextWebpackPlugin`这个插件能够解决这个问题

#### 使用ExtractTextWebpackPlugin
安装`ExtractTextWebpackPlugin`插件
`npm i --save-dev extract-text-webpack-plugin`
在webpack中使用该插件需要三步
+`var ExtractTextPlugin = require('extract-text-webpack-plugin');`
`module.exports = {`
    `module: {`
         `rules: [{`
             `test: /\.css$/,`
-            `use: 'css-loader'`
+            `use: ExtractTextPlugin.extract({`
+                `use: 'css-loader'`
+            `})`
         `}]`
     `},`
+    `plugins: [`
+        `new ExtractTextPlugin('styles.css'),`
+    `]`
`}`
以上三个步后，会生成一个新的css文件，并将其作为一个单独的html标签，插入到html中。
