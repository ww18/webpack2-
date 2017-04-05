

边学边翻译的，如有不当之处请指出

## 目录
- [安装](#安装)
- [代码分割](#代码分割)
- [css的代码分割](#css的代码分割)
- [第三方库的代码分割](#第三方库的代码分割)
- [使用import()进行代码分割](#使用import()进行代码分割)
- [使用require.ensure分割代码](#使用require.ensure分割代码)

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

    module.exports = {
       module: {
            rules: [{
                test: /\.css$/,
                use: 'css-loader'
            }]
        }
    }
这样配置的结果是css和js打包到一起了。

这有缺点，你将无法利用浏览器的能力异步并行加载CSS，而是要你的页面不得不等到你的整个JavaScript加载完成，才能应用css到dom中。
`ExtractTextWebpackPlugin`这个插件能够解决这个问题

#### 使用ExtractTextWebpackPlugin

安装`ExtractTextWebpackPlugin`插件
`npm i --save-dev extract-text-webpack-plugin`

在webpack中使用该插件需要三步

    var ExtractTextPlugin = require('extract-text-webpack-plugin');
    module.exports = {
        module: {
             rules: [{
                 test: /\.css$/,
                 use: 'css-loader'
                 use: ExtractTextPlugin.extract({
                     use: 'css-loader'
                 })
             }]
         },
         plugins: [
             new ExtractTextPlugin('styles.css'),
         ]
    }

以上三个步后，会生成一个新的css文件，并将其作为一个单独的html标签，插入到html中。

### 第三方库的代码分割

一个典型的应用程序使用第三方库的框架/功能需求。使用这些库的特定版本，这里的代码不经常更改。然而，应用程序代码经常更改。

捆绑应用程序代码与第三方库代码将是低效的。这是因为浏览器可以缓存基于缓存头的资源文件，如果文件内容不改变，就可以不需要调用CDN就可以缓存文件。为了利用这一点，我们希望保持第三方库文件的哈希不变，无论应用程序代码如何变化。

只有当我们为第三方库和应用程序代码分开时，我们才能做到这一点。

让我们考虑一个使用momentjs示例的应用程序，一个常用的时间格式库。

安装库文件如下
`npm install --save moment`

在index文件中引入moment库，并console出当前时间
index.js
`var moment = require('moment');
console.log(moment().format());`

我们可以在webpack中做如下配置

    var path = require('path');

    module.exports = function(env) {
        return {
            entry: './index.js',
            output: {
                filename: '[name].[chunkhash].js',
                path: path.resolve(__dirname, 'dist')
            }
        }
    }
在项目中运行webpack，会发现，moment和index.js都打包成了bundle.js，这对于我们的应用是不理想的，如果index.js有改动的话，整个代码块都需要重新build。浏览器也被迫每次都重新加载新的资源，尽管它们它们大部分都没有改变。

#### 多个入口文件
让我们解决以上问题，通过为moment添加一个额外的入口，名字为vendor
var path = require('path');

    module.exports = function(env) {
        return {
            entry: {
                main: './index.js',
                vendor: 'moment'
            },
            output: {
                filename: '[name].[chunkhash].js',
                path: path.resolve(__dirname, 'dist')
            }
        }
    }
    
 通过运行webpack，我们发现生成了两个文件，如果检查这两个文件，我们会发现，每个文件中都有moment，原因在于，moment是一个依赖的主要应用，每个入口将会捆绑自己的依赖。
以上原因我们将需要使用CommonsChunkPlugin 插件
 
#### CommonsChunkPlugin 
这是一个相当复杂的插件。它从根本上允许我们从不同的代码中提取所有公共模块，并将它们添加到公共包中，如果公共包不存在就会创建一个。
我们使用CommonsChunkPlugin，修改webpack文件

    var webpack = require('webpack');
    var path = require('path');

    module.exports = function(env) {
        return {
            entry: {
                main: './index.js',
                vendor: 'moment'
            },
            output: {
                filename: '[name].[chunkhash].js',
                path: path.resolve(__dirname, 'dist')
            },
            plugins: [
                new webpack.optimize.CommonsChunkPlugin({
                    name: 'vendor' // Specify the common bundle's name.
                })
            ]
        }
    }
 
现在在运行webpack，则moment只存在于vendor文件中。
#### 隐含通用供应商块(或第三方库)
可以配置一个commonschunkplugin实例只接受供应商库
 
    var webpack = require('webpack');
    var path = require('path');

    module.exports = function() {
        return {
            entry: {
                main: './index.js'
            },
            output: {
                filename: '[name].[chunkhash].js',
                path: path.resolve(__dirname, 'dist')
            },
            plugins: [
                new webpack.optimize.CommonsChunkPlugin({
                    name: 'vendor',
                    minChunks: function (module) {
                       // this assumes your vendor imports exist in the node_modules directory
                       return module.context && module.context.indexOf('node_modules') !== -1;
                    }
                })
            ]
        };
    }
    
以上配置同样可以生成vendor文件，并且包涵node_modules文件夹中的所有第三方库
 
#### 清单文件
以上的配置中，如果代码改变了，那么两个文件的hash值都变化了，这种情况不是我们想要的。这意味着我们仍然无法获得浏览器缓存的好处，因为每一个版本的更改，浏览器将不得不重新加载文件。

这里的问题是，在每次构建，生成一些Webapck 运行代码，这有助于WebPACK做工作。当有一个单一的文件时，运行时代码在这个文件中。但是当生成多个文件时，运行时代码会被解压到公共模块中，就是vendor文件。

为了防止这种情况，我们需要提取运行时代码到一个单独的清单文件。尽管我们以创建了另一个文件为代价，但是我们获得了vendor文件长期缓存的权益。

    var webpack = require('webpack');
    var path = require('path');

    module.exports = function(env) {
        return {
            entry: {
                main: './index.js',
                vendor: 'moment'
            },
            output: {
                filename: '[name].[chunkhash].js',
                path: path.resolve(__dirname, 'dist')
            },
            plugins: [
                new webpack.optimize.CommonsChunkPlugin({
                    names: ['vendor', 'manifest'] // Specify the common bundle's name.
                })
            ]
        }
    };
注意是[chunkhash]
 
使用我们迄今所学到的，我们也可以实现相同的结果与隐含的通用供应商块。
 
    var webpack = require('webpack');
    var path = require('path');

    module.exports = function() {
        return {
            entry: {
                main: './index.js' //Notice that we do not have an explicit vendor entry here
            },
            output: {
                filename: '[name].[chunkhash].js',
                path: path.resolve(__dirname, 'dist')
            },
            plugins: [
                new webpack.optimize.CommonsChunkPlugin({
                    name: 'vendor',
                    minChunks: function (module) {
                       // this assumes your vendor imports exist in the node_modules directory
                       return module.context && module.context.indexOf('node_modules') !== -1;
                    }
                }),
                //CommonChunksPlugin will now extract all the common modules from vendor and main bundles
                new webpack.optimize.CommonsChunkPlugin({ 
                    name: 'manifest' //But since there are no more common modules between them we end up with just the runtime code included in the manifest file
                })
            ]
        };
    }
 
### 使用import()进行代码分割
 
#### 动态import
目前一个功能性的import()模块加载的语法，在ECMAScript中正在讨论。

ES2015装载规范定义了import()作为负载ES2015模块的动态运行的方法。

webpack将import作为一个分割点，并且将请求的模块放到一个单独的文件中。

import() 使用模块名称作为参数，并返回一个Promise对象: 
import(name) -> Promise

index.js

    function determineDate() {
      import('moment').then(function(moment) {
        console.log(moment().format());
      }).catch(function(err) {
        console.log('Failed to load moment', err);
      });
    }

    determineDate();
注意：import()中的path不能是全部动态的，如import(Math.random())这种写法是错误的。可以是全部静态的，import('./././style.css')，也可以是部分动态的，如import('./'+path+'/style.css')

#### Promise polyfill
import()内部依赖Promise对象。
如果在旧浏览器上使用import，就需要写一些铺垫来是浏览器可以解析Promise，像 es6-promise 或是 promise-polyfill。

在项目的入口处，写上如下代码：

    import Es6Promise from 'es6-promise';
    Es6Promise.polyfill();
    // or
    import 'es6-promise/auto';
    // or
    import Promise from 'promise-polyfill';
    if (!window.Promise) {
      window.Promise = Promise;
    }
    // or ...
    
#### Babel的使用
如果我们想使用Babel中的import，我们需要添加或安装` syntax-dynamic-import`这个插件，虽然它仍然是第3阶段绕过解析器错误。当建议加入规格，这将不再是必要的。

    npm install --save-dev babel-core babel-loader babel-plugin-syntax-dynamic-import babel-preset-es2015
    # for this example
    npm install --save momentindex

index-es2015.js

    function determineDate() {
      import('moment')
        .then(moment => moment().format('LLLL'))
        .then(str => console.log(str))
        .catch(err => console.log('Failed to load moment', err));
    }

    determineDate();

webpack.config.js

    module.exports = {
      entry: './index-es2015.js',
      output: {
        filename: 'dist.js',
      },
      module: {
        rules: [{
          test: /\.js$/,
          exclude: /(node_modules)/,
          use: [{
            loader: 'babel-loader',
            options: {
              presets: [['es2015', {modules: false}]],
              plugins: ['syntax-dynamic-import']
            }
          }]
        }]
      }
    };
    
如果没有使用syntax-dynamic-import这个的话会编译错误，
-Module build failed: SyntaxError: 'import' and 'export' may only appear at the top level, or
-Module build failed: SyntaxError: Unexpected token, expected {

#### 使用async/await
使用ES2017 的async/await 在import()中

    npm install --save-dev babel-plugin-transform-async-to-generator babel-plugin-transform-regenerator babel-plugin-transform-runtime

index-es2017.js

    async function determineDate() {
      const moment = await import('moment');
      return moment().format('LLLL');
    }

    determineDate().then(str => console.log(str));
    
webpack.config.js

    module.exports = {
      entry: './index-es2017.js',
      output: {
        filename: 'dist.js',
      },
      module: {
        rules: [{
          test: /\.js$/,
          exclude: /(node_modules)/,
          use: [{
            loader: 'babel-loader',
            options: {
              presets: [['es2015', {modules: false}]],
              plugins: [
                'syntax-dynamic-import',
                'transform-async-to-generator',
                'transform-regenerator',
                'transform-runtime'
              ]
            }
          }]
        }]
      }
    };
    
这部分是说明js这种写法的打包方式

#### import 取代了 require.ensure？并不全是
好消息：无法加载块现在可以处理，因为它们是基于Promise的。
警告：`require.ensure`允许简单的块的命名通过使用第三个参数，但是importAPI并没有提供这个能力。如果想继续使用这个功能的话，只能使用`require.ensure`。

    require.ensure([], function(require) {
      var foo = require("./module");
    }, "custom-chunk-name");
    
#### System.import 被弃用了

## 拓展阅读回头再看

### 使用require.ensure分割代码
在这节中我们将讨论webpack如何使用require.ensure分割代码
