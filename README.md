

边学边翻译的，如有不当之处请指出

## 目录
- [安装](#安装)
- [代码分割](#代码分割)
- [代码分割-css](#代码分割-css)
- [代码分割-第三方库](#代码分割-第三方库)
- [代码分割-使用import()](#代码分割-使用import)
- [分割代码-使用require.ensure](#分割代码-使用requireensure)
- [为生产环境打包](#为生产环境打包)
- [缓存](#缓存)
- [开发](#开发)

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

### 代码分割-第三方库

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
 
### 代码分割-使用import()
 
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

### 分割代码-使用require.ensure

在这节中我们将讨论webpack如何使用require.ensure分割代码。

webpack在构建require.ensure代码的时候是静态解析的。在回调函数中依赖的或require进来的模块，都会被加入到一个新的块，这个新的块会被写入到一个新的文件中，并通过webpack的jsonp按需加载。语法如下：

    require.ensure(dependencies: String[], callback: function(require), chunkName: String)
  
#### dependencies    
    
这是一个字符串数组，是在回调函数中的所有代码都可以执行之前需要声明并获得的模块。

#### callback
当依赖加载完成之后运行的回调函数。require函数最为参数传递到回调函数中，在函数体内部可以通过require获取更多的模块。

#### chunkName
这个参数是为这个require.ensure创建的chunk名字。通过传递相同的名字给不同的require.ensure，我们能够将它们的代码合并到一个文件中。从而使浏览器加载一个包即可。

#### 例子
让我们看一下下面的文件目录

    .
    ├── dist
    ├── js
    │   ├── a.js
    │   ├── b.js
    │   ├── c.js
    │   └── entry.js
    └── webpack.config.js
    
entry.js

    require('./a');
    require.ensure(['./b'], function(require){
        require('./c');
        console.log('done!');
    });
    
a.js

    console.log('***** I AM a *****');

b.js

    console.log('***** I AM b *****');

c.js

    console.log('***** I AM c *****');
    
webpack.config.js

    var path = require('path');
    var htmlWebpackPlugin = require('html-webpack-plugin');

    module.exports = function(env) {
        return {
            entry: './js/entry.js',
            output: {
                filename: 'bundle.js',
                path: path.resolve(__dirname, 'dist'),
                //publicPath: 'https://cdn.example.com/assets/',
                // tell webpack where to load the on-demand bundles. 

                //pathinfo: true,
                // show comments in bundles, just to beautify the output of this example.
                // should not be used for production.
            },
            //添加index文件生成代码，能更清楚的明白
            plugins:[
                new htmlWebpackPlugin({
                    fileName: 'index.html',
                    template: 'index.html'
                })
            ]
        }
    }
注意：在使用代码分割中，output.publicPath是一个很重要的参数，它告诉webpack从哪里按需加载打包生成的文件。
在这个项目中运行webpack，我们发现生成了两个js文件，bundle.js和0.bundle.js。entry.js和a.js打包成了一个文件bundle.js ，bundle.js

    /******/ (function(modules) { // webpackBootstrap
    //webpack bootstrap code...

    /******/     // __webpack_public_path__
    /******/     __webpack_require__.p = "https://cdn.example.com/assets/";

    // webpack bootstrap code...
    /******/ })
    /******/ ([
    /* 0 */
    /* unknown exports provided */
    /* all exports used */
    /*!*****************!*\
      !*** ./js/a.js ***!
      \*****************/
    /***/ (function(module, exports) {

    console.log('***** I AM a *****');


    /***/ }),
    /* 1 */,
    /* 2 */,
    /* 3 */
    /* unknown exports provided */
    /* all exports used */
    /*!*********************!*\
      !*** ./js/entry.js ***!
      \*********************/
    /***/ (function(module, exports, __webpack_require__) {

    __webpack_require__(/*! ./a */ 0);
    __webpack_require__.e/* require.ensure */(0).then((function(require){
        __webpack_require__(/*! ./c */ 2);
        console.log('done!');
    }).bind(null, __webpack_require__)).catch(__webpack_require__.oe);


    /***/ })
    /******/ ]);
    
该代码中我们看到 __webpack_require__.p ，对应的是上面的output.publicPath这个配置

b.js 和 c.js 打包成了 0.bundle.js

    webpackJsonp([0],[
    /* 0 */,
    /* 1 */
    /* unknown exports provided */
    /* all exports used */
    /*!*****************!*\
      !*** ./js/b.js ***!
      \*****************/
    /***/ (function(module, exports) {

    console.log('***** I AM b *****');


    /***/ }),
    /* 2 */
    /* unknown exports provided */
    /* all exports used */
    /*!*****************!*\
      !*** ./js/c.js ***!
      \*****************/
    /***/ (function(module, exports) {

    console.log('***** I AM c *****');
    /***/ })
    ]);
我们看到html文件中只注入了bundle.js文件。当我们打开浏览器的时候，webpack将按需加载0.bundle.js文件。
我们自己写的时候可以先不用publicPath。
reuqire.ensure的注意事项：
1.以上代码运行后只在console中打印出了a，c，b并没有打印出来，要想打印出b还需要在代码中require一下b。
2.第一个参数可以为空。

### 为生产环境打包

这一章将讲解如何使用webpack为生产环境打包

#### 自动打包方式

运行`webpack -p`（或等价的 `webpack --optimize-minimize --define process.env.NODE_ENV="'production'"`）。执行下列步骤：

- 使用UglifyJsPlugin压缩文件
- 运行LoaderOptionsPlugin插件
- 设置node环境变量

##### 压缩
webpack配备了`UglifyJsPlugin`这个插件，为了压缩输出。这个插件支持所有的UglifyJs的配置。指定`--optimize-minimize`在terminal中，相当于在webpack.config.js中设置了如下配置：

    // webpack.config.js
    const webpack = require('webpack');

    module.exports = {
      /*...*/
      plugins:[
        new webpack.optimize.UglifyJsPlugin({
          sourceMap: options.devtool && (options.devtool.indexOf("sourcemap") >= 0 || options.devtool.indexOf("source-map") >= 0)
        })
      ]
    };
    
##### source Maps
我们鼓励您在生产中启用源映射。它们对于调试和运行基准测试非常有用。webpack可以在代码块和分割文件中源映射。

在你的配置，使用devtool对象设置源地图类型。我们目前支持七种类型的源地图。您可以在我们的配置文档页中找到更多关于它们的信息。
其中一个很好的选择是使用`cheap-module-source-map`，简化了源映射到每行的单一映射.

##### node的环境变量
运行` webpack -p` (or `--define process.env.NODE_ENV="'production'"`)相当于以下面的方式调用 DefinePlugin :

    // webpack.config.js
    const webpack = require('webpack');

    module.exports = {
      /*...*/
      plugins:[
        new webpack.DefinePlugin({
          'process.env.NODE_ENV': JSON.stringify('production')
        })
      ]
    };
    
这部分对于配置开发环境和生产环境很有用，可以参考API。

#### 手动配置webpack的不同环境
当我们在不同的环境中有多种配置时，最简单的方法就是为每个环境编写单独的JS文件。例如:
dev.js

    module.exports = function (env) {
      return {
        devtool: 'cheap-module-source-map',
        output: {
            path: path.join(__dirname, '/../dist/assets'),
            filename: '[name].bundle.js',
            publicPath: publicPath,
            sourceMapFilename: '[name].map'
        },
        devServer: {
            port: 7777,
            host: 'localhost',
            historyApiFallback: true,
            noInfo: false,
            stats: 'minimal',
            publicPath: publicPath
        }
      }
    }
    
prod.js

    module.exports = function (env) {
      return {
        output: {
            path: path.join(__dirname, '/../dist/assets'),
            filename: '[name].bundle.js',
            publicPath: publicPath,
            sourceMapFilename: '[name].map'
        },
        plugins: [
            new webpack.LoaderOptionsPlugin({
                minimize: true,
                debug: false
            }),
            new webpack.optimize.UglifyJsPlugin({
                beautify: false,
                mangle: {
                    screw_ie8: true,
                    keep_fnames: true
                },
                compress: {
                    screw_ie8: true
                },
                comments: false
            })
        ]
      }
    }
    
webpack.config.js

    function buildConfig(env) {
      return require('./config/' + env + '.js')(env)
    }

    module.exports = buildConfig;
    
从我们的package.json，我们建立我们的应用程序使用webpack，命令是这样的：

    "build:dev": "webpack --env=dev --progress --profile --colors",
    "build:dist": "webpack --env=prod --progress --profile --colors",
    
你可以看到，我们传递环境变量到webpack.config.js文件中。从那里我们使用了一个简单的开关，通过简单的判断加载正确的JS文件环境。  
以上的配置文件写的不完整，没有entry选项，所以我们自己做例子的时候直接看下面的就行。

一种先进的方法是有一个基本的配置文件，存放所有常用的功能，又有环境的具体文件和简单的使用“合并”，合并webpack配置。这能够帮助我们避免重复代码。
例如，你可以有你所有的基础配置，如解决您的JS，TS，PNG，JPEG，JSON等。在一个共同的基础文件如下：
base.js

    module.exports = function() {
        return {
            entry: {
                'polyfills': './src/polyfills.ts',
                'vendor': './src/vendor.ts',
                'main': './src/main.ts'

            },
            output: {
                path: path.join(__dirname, '/../dist/assets'),
                filename: '[name].bundle.js',
                publicPath: publicPath,
                sourceMapFilename: '[name].map'
            },
            resolve: {
                extensions: ['.ts', '.js', '.json'],
                modules: [path.join(__dirname, 'src'), 'node_modules']

            },
            module: {
                rules: [{
                    test: /\.ts$/,
                    use: [
                        'awesome-typescript-loader',
                        'angular2-template-loader'
                    ],
                    exclude: [/\.(spec|e2e)\.ts$/]
                }, {
                    test: /\.css$/,
                    use: ['to-string-loader', 'css-loader']
            }, {
                test: /\.(jpg|png|gif)$/,
                use: 'file-loader'
            }, {
                test: /\.(woff|woff2|eot|ttf|svg)$/,
                use: {
                  loader: 'url-loader',
                  options: {
                    limit: 100000
                  }
                }
            }],
        },
        plugins: [
            new ForkCheckerPlugin(),

            new webpack.optimize.CommonsChunkPlugin({
                name: ['polyfills', 'vendor'].reverse()
            }),
            new HtmlWebpackPlugin({
                template: 'src/index.html',
                chunksSortMode: 'dependency'
            })
        ],
    };
    }
    
然后将此基础配置与特定环境下的配置文件使用“webpackMerge”。让我们看一个例子，我们合并我们的产品文件，上面所提到的，这个基础配置文件使用“webpackMerge”：

prod.js (updated)

    const webpackMerge = require('webpack-merge');

    const commonConfig = require('./base.js');

    module.exports = function(env) {
        return webpackMerge(commonConfig(), {
            plugins: [
            new webpack.LoaderOptionsPlugin({
                minimize: true,
                debug: false
            }),
            new webpack.DefinePlugin({
                'process.env': {
                    'NODE_ENV': JSON.stringify('production')
                }
            }),
            new webpack.optimize.UglifyJsPlugin({
                beautify: false,
                mangle: {
                    screw_ie8: true,
                    keep_fnames: true
                },
                compress: {
                    screw_ie8: true
                },
                comments: false
            })
        ]
    })
    }
我们注意到在prod.js文件中有三个主要的更新，
- 'webpack-merge' 和'base.js'.
- 我们将output属性移到了base.js文件中，只是强调一点，我们的output属性是公用的属性，所以我们重构了prod.js文件，然后将属性移到了base.js文件中。
- 我们使用`DefinePlugin`定义了生产环境的 process.env.NODE_ENV环境变量。当我们在生产环境下打包时，process.env.NODE_ENV的值是 production。同样，我们可以管理我们选择的各种变量，具体的环境。

然而，在你所有的环境中选择什么是共用的取决于你自己。我们刚刚只是演示了一些在我们建立应用时典型的共用部分。
主要知道webpackMerge的用法

### 缓存
使用webpack使静态资源长期缓存有以下方法：
- 为内容相关的每个文件添加[chunkhash]
- 将minifest提取为单独的文件
- 确保包涵结构代码的入口文件在有相同依赖的情况下不会改变hash值（？）
更多的优化设置

- 在html需要资源时使用编译器统计文件名字
- 在加载资源之前生成manifest JSON文件，并将其内联到html页中。

#### 问题
我们代码改变后会生成新的文件，但是如果文件名不变的话，浏览器可能会取缓存中的文件，而会产生问题，但是如果每次都生成新的文件的话那缓存又没有用了。

#### 解决：每个文件生成唯一的哈希值
如果文件的内容没有改变，我们怎么才能产生相同的文件名呢？例如，当没有更新依赖项时，只需要重新打包应用程序代码，而不必重新打包依赖文件。
解决方法是我们使用[chunkhash]替换[hash]值。(但是我们在项目中一般是添加文件的后缀，而不是每次都生成新的文件。)

    module.exports = {
      /*...*/
      output: {
        /*...*/
    -   filename: "[name].[hash].js"
    +   filename: "[name].[chunkhash].js"
      }
    };
    
此配置也将创建2个文件，但在这种情况下，每个文件将得到自己独特的哈希。

注意在开发环境中不要使用[chunkhash]，因为会增加编译时间。分开开发和生产环境的配置能够使用不同的名字，开发环境是[name].js ，生产环境是 [name].[chunkhash].js。这里有个问题，这样的话每次生产环境就会有一个新的文件，如果发布了很多次就会有很多个文件，不做处理吗？

#### 确定的hash值

为了压缩生成的文件，webpack使用标实符替换模块的名字。通过编译，标实符被生成并映射到生成的文件名字中，然后放到一个名字为manifest的js对象中。为了保存生成中的标实符，webpack提供了`NamedModulesPlugin`插件为开发环境，`HashedModuleIdsPlugin`插件为生产环境。

这节是上面 代码分割-第三方库 的详细介绍或者说是升级版
按以上配置还是有相同的问题，就是无论修改了文件的哪部分，所有的文件都重新打包一次，名字改变，缓存失效。为了解决这个问题，我们需要使用`ChunkManifestWebpackPlugin`这个插件，这将提取清单到一个单独的JSON文件。这个通过一个变量替换了manifest。但是我们能够做到更好，我们可以通过`CommonsChunkPlugin`提取运行时文件到一个单独的入口。下面是一个更新的webpack.config.js，将我们建立目录清单和运行时产生的文件.

    // webpack.config.js
    var ChunkManifestPlugin = require("chunk-manifest-webpack-plugin");

    module.exports = {
      /*...*/
      plugins: [
        /*...*/
        new webpack.optimize.CommonsChunkPlugin({
          name: ["vendor", "manifest"], // vendor libs + extracted manifest
          minChunks: Infinity,
        }),
        /*...*/
        new ChunkManifestPlugin({
          filename: "chunk-manifest.json",
          manifestVariable: "webpackManifest"
        })
      ]
    };
（自己的： 这一段中最后一个生成json文件的试了一下去掉后没有影响啊，还有html文件中头部放js代码不知道干什么用的，还有上面的这些还不能完全做到hash值不变，需要再加上两个插件，

    var path = require("path");
    var webpack = require("webpack");
    var ChunkManifestPlugin = require("chunk-manifest-webpack-plugin");
    var WebpackChunkHash = require("webpack-chunk-hash");

    module.exports = {
      entry: {
        vendor: "./src/vendor.js", // vendor reference file(s)
        main: "./src/index.js" // application code
      },
      output: {
        path: path.join(__dirname, "build"),
        filename: "[name].[chunkhash].js",
        chunkFilename: "[name].[chunkhash].js"
      },
      plugins: [
        new webpack.optimize.CommonsChunkPlugin({
          name: ["vendor", "manifest"], // vendor libs + extracted manifest
          minChunks: Infinity,
        }),
        new webpack.HashedModuleIdsPlugin(),
        new WebpackChunkHash(),
        new ChunkManifestPlugin({
          filename: "chunk-manifest.json",
          manifestVariable: "webpackManifest"
        })
      ]
    };
这样就完美了，这样修改代码只会改变代码部分的hash值，而不会改变公共部分的值。

#### 内连Manifest代码
将manifest文件和webpack的运行时文件内连到html中，用来减少http请求。这个功能依赖服务器的配置。但是如果你的项目不依赖于服务器渲染，那我们就能生成一个简单的index.html文件

### 开发
在这一页中，我们将解释如何开始开发和如何选择三个工具之一开发。这是假设你已经有了一个WebPACK的配置文件。在产品阶段不会使用这些工具。

#### 调整编译器

一些文本编辑器具有“安全写入”功能，默认情况下启用此功能。作为一个结果，保存的文件将不总是在编译结果。
每一个编译器有不同的方式去禁用它，最公用的方式是：

    Sublime Text 3 - Add "atomic_save": false to your user preferences.
    IntelliJ - use search in the preferences to find "safe write" and disable it.
    Vim - add :set backupcopy=yes in your settings.
    WebStorm - uncheck Use "safe write" in Preferences > Appearance & Behavior > System Settings
    
#### 文件地图
当js抛出了一个错误，我们很想知道是那个文件的哪一行代码导致了这个错误。如果webpack将所有文件都打包生成新的文件后，我们就很不方便去追踪错误。文件地图就尝试去解决这个错误。这里有很多不同的配置，每个都有优点也有缺点，开始的时候我们使用下面这个：

    devtool: "cheap-eval-source-map"
    
#### 选择一个工具
webpack能够使用watch模式，在这种模式下，webpack监听所有的文件，当文件变化时重新编译。 webpack-dev-server提供了一个简单的方式，使用开发服务器快速重新编译。如果你已经有了开发服务器，但是想足够灵活的话， webpack-dev-middleware 这个可以作为中间件。
webpack-dev-server 和 webpack-dev-middleware编译后保存到内存中，意味着不会保存到硬盘中，这样能够使编译更快，也不会产生更多的文件。
在大多数情况下，你会想用webpack-dev-server，因为它是最容易上手，并提供多功能开箱。

#### webpack 的监听模式
webpack的监听模式监听文件的变化。如果检测到任何更改，它将再次运行编译。在编译的时候如果想要看到进度条的话：

    webpack --progress --watch
观看模式没有假设一个服务器，所以你需要提供自己的。一个简单的服务器是`serve`，安装`npm i --save-dev serve`之后，在目录下运行``npm bin`/serve``。还有更简单的方式，在package.json文件中设置scripts

    ...
    "scripts": {
      "start": "serve"
    }
    ...
你可以通过运行`npm start` 在你的项目中的目录中启动服务器。每次编译后，您将需要手动刷新浏览器以查看更改。

注意：`--single`这个命令对于单页面应用很有用

#### chrome工作区的监听模式
如果设置了chrome为监听模式，那么不必刷新浏览器，需要设置webpack的配置来适应。`devtool: "inline-source-map"`。
这有一些陷阱关于使用工作区监听模式：
- 大块（如常见的块，超过1MB），重建可能导致页面空白，这将迫使你刷新浏览器。
- 小的块要比大的块编译的更块，因为 `inline-source-map` 会用base64的编码处理源码，所以会慢。（？）

#### webpack-dev-server
webpack-dev-server提供了一个服务器可以自动加载，并且很容易配置。前期准备，确保你有一个index.html文件指向你的包。假设output.filename是bundle.js。用npm安装`webpack-dev-server`。

    npm install webpack-dev-server --save-dev
当安装成功之后可以使用如下命令`webpack-dev-server --open`。
注意： 如果运行报错的话，可以运行如下命令`node_modules/.bin/webpack-dev-server`。可以将命令添加到package.json中的scripts命令中，这样可以优化代码。`"scripts": { "start": "webpack-dev-server" }`。上面的命令会自动打开浏览器，地址是：http://localhost:8080。
修改任意文件，并保存，我们会看到重新编译了，编译完成之后浏览器会自动刷新。如果什么都没有发生的话，我们需要添加`watchOptions`配置。
现在我们可以让它自动加载了，还可以更进一步： 热模块更换。这是一个能使模块互换而不刷新页面的接口。可以参考如何配置HMR部分。
默认使用的是内连模式，这种模式注入客户端-需要重新加载和显示错误-在您的绑定中。内嵌模式你会得到你的工具生成错误和警告的控制台。webpack-dev-server能够做很多事情，比如代发请求去后台服务器，对于更多配置参看devServer documentation.

#### webpack-dev-middleware
webpack-dev-middleware工作连接基于中间件堆栈。这会很有用，如果你已经有了node服务器或是你想完全控制你的服务器。这个中间件会将编译文件放到内存中，当编译进行的时候，会将编译文件推迟到文件编译完成才开始请求。这是为高级用户准备的。webpack-dev-server更容易使用。
安装这个插件：

    npm install express webpack-dev-middleware --save-dev
安装完成后我们会进行如下配置使用：middle.ware.js

    var express = require("express");
    var webpackDevMiddleware = require("webpack-dev-middleware");
    var webpack = require("webpack");
    var webpackConfig = require("./webpack.config");

    var app = express();
    var compiler = webpack(webpackConfig);

    app.use(webpackDevMiddleware(compiler, {
      publicPath: "/" // Same as `output.publicPath` in most cases.
    }));

    app.listen(3000, function () {
      console.log("Listening on port 3000!");
    });
依赖我们配置的output.publicPath 和 output.filename，在http://localhost:3000/bundle.js  应该能获取到bundle.js文件。默认情况下watch监听模式是开启的，我们还可以使用lazy mode，这将只编译请求的入口点，在请求入口文件的时候编译：

    app.use(webpackDevMiddleware(compiler, {
      lazy: true,
      filename: "bundle.js" // Same as `output.filename` in most cases.
    }));
执行`node middle.ware.js`，然后在浏览器中打开http://localhost:3000/ ，我们会看到页面，并且它也会自动刷新
还有更多的配置，参看devServer documentation.

### Development - Vagrant
如果你有一个更先进的项目和使用Vagrant在虚拟机中运行你的开发环境中，你也会想在虚拟机中运行webpack

#### 配置项目
开始的时候确保Vagrantfile 有一个固定的IP

    Vagrant.configure("2") do |config|
      config.vm.network :private_network, ip: "10.10.10.61"
    end
下一步在项目中安装webpack和webpack-dev-server 。确保有webpack.config.js文件，如果还没有的话，使用下面最简单的配置

    module.exports = {
      context: __dirname,
      entry: "./app.js"
    };
创建一个index.html文件，script标签引入编译完成的文件，如果没有配置output.filename，则文件名字为bundle.js

    <!DOCTYPE html>
    <html>
      <head>
        <script src="/bundle.js" charset="utf-8"></script>
      </head>
      <body>
        <h2>Heey!</h2>
      </body>
    </html>
注意：我们需要创建一个app.js文件。

#### 运行服务器
我们运行下面命令去开启服务器：

    webpack-dev-server --host 0.0.0.0 --public 10.10.10.61:8080 --watch-poll
默认情况下我们只能访问localhost，因为我们需要从主机上访问，所以我们需要改变`--host`像上面那样。

webpack-dev-server包涵一个脚本在生成的文件中，这个脚本连接到一个WebSocket，从而使文件发生变化的时候去重新加载。这里的`--public`是告诉脚本去哪里寻找WebSocket，这个服务器将默认使用8080端口，所以我们在这里也这么指定。

`--watch-poll`这个确保webpack能发现文件中的变化，默认情况下webpack监听文件系统的事件触发，但是虚拟机在这方面有很多问题，所以才使用的`--watch-poll`。http://10.10.10.61:8080 这个服务器目前是可用的，如果我们修改了app.js，会自动加载。

#### Nginx的高级用法
为了模拟更真实的生产环境，我们需要用nginx代理webpack-dev-server。在nginx的配置文件中添加如下信息：

    server {
      location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        error_page 502 @start-webpack-dev-server;
      }

      location @start-webpack-dev-server {
        default_type text/plain;
        return 502 "Please start the webpack-dev-server first.";
      }
    }

这proxy_set_header行是很重要的，因为这个配置允许WebSockets能正确工作。启动webpack-dev-server的命令修改成下面这样：

    webpack-dev-server --public 10.10.10.61 --watch-poll
这个是我们只能访问127.0.0.1，但是也没关系，nginx能够使它在你的主机上是可用的。

#### 结论
我们将Vagrant box设置一个可访问的IP地址，然后使webpack-dev-server可以公开访问，然后就可以通过浏览器访问了。这样我们就解决了一个经常遇到的问题，这个问题是VirtualBox不能发出文件系统事件，导致服务器在文件更改后无法重新加载。
    };一个
    };
