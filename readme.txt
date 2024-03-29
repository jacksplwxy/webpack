*常规页面引入多个js文件带来的问题：
·多个js文件加载导致多个http请求，页面速度变慢
·单个js文件中无法弄清楚哪些变量或方法来自哪个文件
·js文件之间的依赖关系复杂，依赖顺序容易出错
·文件未压缩导致加载过慢


*关于模块：
·文档：https://www.webpackjs.com/concepts/modules/
·文档：https://www.webpackjs.com/api/module-methods/


*环境：
·安装node环境
·npm init使项目符合node规范，以便webpack打包项目
·避免使用全局安装webpack，因为不同版本的webpack版本不兼容。npm uninstall webpack webpack-cli -g
·webpack-cli使我们可以在命令行中直接使用webpack命令
·局部安装：
  -- npm install webpack --save-dev
  -- npm install webpack-cli --save-dev //webpack-cli使我们在命令行中可以运行webpack命令
·npx webpack -v：查看当前项目下的webpack版本
·使用：
  -- npx webpack index.js  //用webpack翻译index.js文件
  -- 代码参考：src/lesson1


*webpack是什么？
·webpack是什么:webpack是一个模块打包工具
·能够识别以下规范的模块代码：
  -- ES Moudule：import a from './a.js' export default a
  -- Common JS：var a=require('./a.js')  module.exports=a
  -- CMD
  -- ADM
·借助loader，webpack可以实现打包css、png等任何文件
·webpacak的本质是基于nodejs编写
·作用：
  -- 打包多个js，减少http请求
  -- 明确文件之间的依赖关系（易读）
  -- 简化文件之间的依赖顺序（易于调试）
·打包输出信息：
  -- hash：hash值，本次打包的唯一值
  -- version：打包用到webpack版本
  -- time：打包耗时
  -- asset：打包完成后生成的文件名称
  -- size：文件大小
  -- chunks：打包生成的文件的id值
  -- chunk names：打包入口的文件名

*配置文件webpack.config.js：
·webpack 开箱即用，可以无需使用任何配置文件。然而，webpack 会假定项目的入口起点为 src/index，然后会在 dist/main.js 输出结果，并且在生产环境开启压缩和优化。
·webpack.config.js是webpack的配置文件
·修改默认配置文件为webpackconf.js：npx webpack --config webpackconf.js
·模式(mode)：production、development。默认production，代码被压缩


*loader：
·webpack默认只能打包js文件
·文件(图片)打包：  
  -- file-loader：
     -- 文件打包
     -- 作用：将文件移动到dist文件夹下 → 将文件随机命名或自定义 → 将文件名作为返回值返回
  -- url-loader：
     -- 与file-loader相似
     -- 可以将文件打包为base64
     -- limit: 满足限制文件大小才进行打包，以免js过大，影响加载
        {
            loader: 'url-loader',
            options: {
                name: '[name]_[hash].[ext]',
                outputPath: 'images/',
               limit: 10240
            }
		    }
·样式打包：
  -- postcss-loader：自动添加css的浏览器厂商前缀，如-webkit-、-ms-等
  -- sass-loader：将sass文件翻译为css文件
  -- css-loader：分析出css文件之间的import引用关系，并将他们打包成一个css文件
  -- style-loader：将css-loader打包出的文件内容挂在到html下的head中
  -- 配置：
     {
         test: /\.scss$/,
         use: [
             'style-loader', 
             {
               loader:'css-loader', 
               options:{
                 importLoaders:2,  //当引入的css文件存在@import其他css文件,该配置能使引入的其他css文件也进行postcss-loade和sass-loader这2个loader打包
                 modules:true,  //开启css模块化打包，防止全局引入的css影响其他模块的css,注意代码也要调整，参考《5、loader--css-modules》
               }
             },
             'sass-loader',
             'postcss-loader'
         ]
     }
  -- loader的执行顺序：从下至上、从右至左
·babel：
  -- 高版本js代码打包
  -- 配置前需安装@babel/core、babel-loader、@babel/preset-env、@babel/polyfill
     -- babel-loader：babel在webpack中的加载器
     -- @babel/core：babel核心库，帮助babel识别es代码内容，并将es代码翻译为AST,再根据语法转换规则将AST转换为低版本es代码
     -- @babel/preset-env：es6转化为es5的翻译规则
     -- @babel/polyfill：
        -- @babel/preset-env只是提供了语法转换的规则，但是它并不能弥补浏览器缺失的一些新的功能，如一些内置的方法和对象，如Promise,Array.from等，此时就需要polyfill来做js得垫片，弥补低版本浏览器缺失的这些新功能
        -- 在全局位置单独引入：import "@babel/polyfill"，如果配置了useBuiltIns: 'usage' ，可以不用单独引入
        -- polyfill是全局配置，会污染其他包。对于不想污染其他包的时候可以用@babel/plugin-transform-runtime。故开发类库，第三方模块或者组件库时使用transform-runtime，平常的项目使用babel-polyfill即可
  -- 使用babel-loader加载js文件
      { 
          test: /\.js$/,  //只对js文件打包
          exclude: /node_modules/,  //排除node_modules文件夹下的文件打包
          loader: 'babel-loader', //使用babel-loader进行js文件加载
      }
  -- babel配置：
     	presets: [
        [
          "@babel/preset-env",  //将es6转es5的核心包
          {
            targets: {
              chrome: ">67",  //对于chrome浏览器大于67版本就支持的功能不进行es版本的转换
            },
            useBuiltIns: 'usage'  //只对业务代码中用到的特性进行打包，未用到的部分不打包，以减小打包体积。该配置会按需自动引入@babel/polyfill。
          }
        ],
        "@babel/preset-react" //支持打包react代码
	    ]
  -- 文档：
      -- 《babel官网——在webpack中的配置》：https://www.babeljs.cn/setup#installation
      -- 《了解babel：polyfill、loader、 preset-env及 core之间的关系》：https://zhuanlan.zhihu.com/p/138108118


*plugins：
·plugins可以在webpack运行到某个时刻的时候帮你做一些事情
·html-webpack-plugin：
  -- 在打包结束后自动生成一个html文件，并把打包生成的js自动引入进来
  -- 可以配置指定模板来生成html文件
·clean-webpack-plugin：
  -- 重新打包前，删除原来的打包文件夹
  -- 可以配置需要删除的文件夹
·HotModuleReplacementPlugin：
  -- webpack开箱即用插件
  -- 它允许在运行时更新各种模块，而无需进行完全刷新。即页面不刷新的情况下，更新代码并生效。
  -- 在webpack.config.js中配置：
     -- plugins: [new webpack.HotModuleReplacementPlugin()]
     -- devServer: {hot: true,hotOnly: true}
  -- 在js中使用：vue-loader实现了以下代码，所以看上去vue自带热更新效果
     import number from './number'
     number()
     if(module.hot){
       module.hot.accept('./number',()=>{
         documnet.body.removeChild(document.getElementById('number'))
         number()
       })
     }
·SplitChunksPlugin：
  -- wenpack开箱即用插件
  -- 背景：在多入口文件打包的时候，出现了重复引入第三方库的问题。在webpack4之前，都是利用 CommonsChunkPlugin 插件来进行公共模块抽取。到了webpack4之后，利用了SplitChunksPlugin插件来进行公共模块抽取
  -- 该组件是Code Splitting的基础组件
  -- 默认配置说明：
      module.exports = {
        optimization: {
          splitChunks: {
            chunks: 'async',     // 代码分割时对异步代码生效，all：所有代码有效，inital：同步代码有效
            minSize: 30000,      // 代码分割最小的模块大小，引入的模块大于30000B也就是30kb 才做代码分割
            maxSize: 0,      // 代码分割最大的模块大小，大于这个值要进行代码分割，一般使用默认值
            minChunks: 1,      // 引入的次数大于等于1时才进行代码分割
            maxAsyncRequests: 6,      // 最大的异步请求数量,也就是同时加载的模块最大模块数量
            maxInitialRequests: 4,      // 入口文件做代码分割最多分成4个js文件
            automaticNameDelimiter: '~',      // 文件生成时的连接符
            automaticNameMaxLength: 30,      // 自动生成的文件名的最大长度
            cacheGroups: {  //缓存组，默认分为vendors和default两个组。webpack会先缓存代码，待分析完所有代码再决定根据代码是否满足组条件决定打包到哪个组中
              vendors: {  // vendors缓存组
                test: /[\\/]node_modules[\\/]/,      // 位于node_modules中的模块做代码分割
                priority: -10      // 根据优先级决定打包到哪个组里，例如一个node_modules中的模块进行代码分割，既满足vendors，又满足default，那么根据优先级会打包到vendors组中。
              }, 
              default: {      // default缓存组。没有test表明所有的模块都能进入 default 组，但是注意它的优先级较低。
                priority: -20,      //  根据优先级决定打包到哪个组里,打包到优先级高的组里。
                reuseExistingChunk: true      // 如果一个模块已经被打包过了,那么再打包时就忽略这个上模块
              }
            }
          }
        }
      } 
  -- 其他配置：
     -- optimization.splitChunks.cacheGroups.vendors.filename:"vendor.js"
        在分组中可以人为地规定打包后文件的名字，在 vendor 分组中添加 filename = "vendor.js" 之后，在 vendor 分组中打包后文件的名字都是 vendor.js 
  -- 文档：
     -- 《webapck4抽取公共模块“SplitChunksPlugin”》：https://www.cnblogs.com/xieqian/p/10973039.html
·mini-css-extract-plugin：
  -- css单独打包插件
  -- 该插件不支持热更新，所以不在开发环境中使用，影响开发效率，只在生产环境中使用
  -- 配置时，注意Three Shaking配置影响打包，因为它开启了optimization.usedExports:true。只需在package.json下的sideEffects需配置*.css
  -- 配合插件optimize-css-assets-webpack-plugin，可以实现css的压缩


*webpack核心概念和配置：
·entry：
  -- 打包入口，支持多页面的多个入口
  -- 文档：https://www.webpackjs.com/configuration/entry-context/#entry
·output：
  -- filename:此选项决定了每个输出bundle的名称。这些bundle将写入到output.path选项指定的目录下。
  -- publicPath：path是webpack所有文件的输出的路径，必须是绝对路径；publicPath并不会对生成文件的路径造成影响，主要是对你的页面里面引入的资源的路径做对应的补全，常见的就是css文件里面引入的图片
  -- chunkFilename:异步引入的代码的打包文件名
  -- 文档：https://www.webpackjs.com/configuration/output/
·devtool:
  -- source-map：开启sourceMap,调试时可以从打包后的文件对应到源码位置
  -- cheap-module-eval-source-map：开发环境中推荐的模式，提示比较全，且打包速度快。
     -- 其中cheap：表示只映射到源码的多少行，不映射到多少列（因为多少列通常意义不太大），这样可以很大提升打包速度
     -- 其中module：可以映射到第三方模块源码中
     -- 其中eval：eval方式生成sourceMap，速度快
  -- cheap-module-source-map：线上生产环境调试时推荐的模式，提示效果比cheap-module-eval-source-map更好
  -- 文档：https://www.webpackjs.com/configuration/devtool/
·devServer：
  -- 基本功能：启动一个本地服务器，监听文件变化自动打包，并且自动刷新浏览器。而webpack --watch只会监听变化自动打包这一个功能。
  -- 配置：
  	devServer: {	//启动devServer服务器
		    contentBase: './dist',	//指定服务器根路径
		    open: true,	//自动打开浏览器，自动访问服务器地址
		    port: 8080,	//指定服务端口
        proxy:{ //启动代理
          '/api':'http://localhost:3000'  //当访问url中包含api，则跳转指定代理url
        },
        historyApiFallback:true //自动将404响应跳转到首页入口文件 index.html,常用于单页应用路由跳转配置
	  }
·Three Shaking：
  -- 一个文件中，只打包用到了的模块，未使用的部分shaking掉
  -- 只支持ES Module的引入
  -- 如何配置：
     -- development模式下：
        1、在webpack.config.js下：optimization.usedExports:true
        2、可选项：package.json下：sideEffects:["@babel/polly-fill","*.css"] //忽略@babel/polly-fill、任何css模块
     -- production模式下：只需要在package.json下配置sideEffects，默认开启Three Shaking功能
·development模式和produciton模式：
  -- development模式较produciton模式区别：
     -- 启动了一个服务器
     -- 重新打包
     -- 热更新
     -- sourceMap级别不同
     -- 代码未做压缩
  -- 实际开发中，可以将生产环境配置和开发环境配置分成两个文件，并且将公共配置单独提出来，生成：webpack.common.js、webpack.dev.js、webpack.prod.js
  -- npm命令行改为：
     -- "dev": "webpack-dev-server --config ./build/webpack.dev.js",
     -- "build": "webpack --config ./build/webpack.prod.js"
·Code Splitting：
  -- Code Splitting：代码分割
  -- 背景：
     -- 当打包体积很大的第三方库时，打包生成的文件也会很大，导致js加载过慢，页面渲染不及时问题
     -- 当我们修改一部分业务代码重新打包时，浏览器又得重新加载打包后的整个文件
     -- 多入口打包文件时，公共库被重复打包
  -- webpack实现代码分割的3种方式：
     -- webpack的手动代码分割解决方案：
        -- 手动将第三方库也作为一个打包入口，html多个js引入
        -- 分析：该方案需手动配置，麻烦且易出错
     -- webpack的自动分割方案：
        -- 配置:
           1、在webpack.config.js下：optimization.splitChunks.chunks:'all'
           2、在webpack.config.js下，optimization.splitChunks.cacheGroups:{vendors:false,default:false}
        -- 分析:
           -- 该方案可以允许我们编写同步代码，webpack自动切割第三方库，html同步引入多个js
           -- 该代码分割能力来自插件：SplitChunksPlugin
     -- webpack + babel方案：
        -- 配置:
           1、安装babel-plugin-dynamic-import-webpack
           2、在.babelrc下引入：plugins: ["@babel/plugin-syntax-dynamic-import"]
           3、编写代码：
              function getComponent(){
                return import('lodash').then({default:_})=>{
                 var element =document.createElement('div')
                 element.innerHTML=_.join(['jack','ya'],'**')
                 return element
                }
              }
              getComponent().then(ele=>{
                document.body.appendChild(ele)
              })
        -- 分析:
           -- 该方案为异步加载方案，使用时才加载，webpack无需任何配置，会自动对异步加载的文件进行切割
           -- 异步加载能力来自babel插件babel-plugin-dynamic-import-webpack
           -- 代码分割能力来自webpack插件SplitChunksPlugin
        -- 异步打包模块命名：
           1、代码第二行：return import('lodash').then({default:_})=>{
              变为：return import(/* webpackChunkName:"lodash" */ 'lodash').then({default:_})=>{
           2、由于babel-plugin-dynamic-import-webpack不支持/* webpackChunkName:"lodash" */这种语法，可以用@babel/plugin-syntax-dynamic-import替换掉该插件,此时打包将生成一个名vendors-lodash.js
           3、将vendors-lodashs.js改为lodash.js：
              利用插件SplitChunksPlugin：在webpack.config.js下，optimization.splitChunks.cacheGroups:{vendors:false,default:false}
  -- chunk：代码块。代码被分割后，每一个代码块就可以称为一个chunk；chunk是webpack根据功能拆分出来的，chunk包含着module，可能是一对多也可能是一对一，chunk包含三种情况，就是Entry配置（通过配置多个entry文件来实现）、动态/按需加载（通过写代码时主动使用import()或者require.ensure来动态加载）、抽取公共代码（使用splitChunks配置来提取公共代码）三种实现代码拆分的情况。
  -- bundle与chunk：bundle是webpack打包之后的各个文件，一般就是和chunk是一对一的关系，bundle就是对chunk进行编译压缩打包等处理之后的产出
  -- 打包分析：https://www.webpackjs.com/guides/code-splitting/#bundle-%E5%88%86%E6%9E%90-bundle-analysis-
·懒加载：
  -- 懒加载：用到的时候才进行资源加载
  -- 实现懒加载的方案有3种：
     -- 通过ES6的import实现懒加载
        function getComponent(){
          return import(/* webpackChunkName:"lodash" */ 'lodash')=>{
            var element =document.createElement('div')
            element.innerHTML=_.join(['jack','ya'],'**')
            return element
          }
        }
     -- 通过ES7的async实现懒加载
        async function getComponent(){
          const {default:_}=await import(/* webpackChunkName:"lodash" */ 'lodash')
          var element =document.createElement('div')
          element.innerHTML=_.join(['jack','ya'],'**')
          return element
        }
     -- 通过webpack的require.ensure实现懒加载（弃用了）
        require.ensure(['lodash'], function() {
            var _ = require('lodash')
            var element =document.createElement('div')
            element.innerHTML=_.join(['jack','ya'],'**')
            return element
        }, 'lodash')
        -- require.ensure实现懒加载原理：require.ensure把一些js模块给独立出一个个js文件，然后需要用到的时候，在创建一个script对象，加入到document.head对象中即可，浏览器会去请求这个js文件
·页面代码使用率：
  -- 查看使用率：打开一个网站 → F12 → ctr+shift+p → 输入coverage → Show Coverage → 刷新即可 
  -- 提升使用率的方式：只加载核心展示代码，异步加载未使用的代码（即懒加载）
        document.addEventListenter('click',()=>{
          import('./click.js').then(({default:func})=>{
            func()
          })
        }) 
  -- prefetch和preload：
     -- 异步加载导致加载速度变慢，体验差。可以通过preload/prefetch解决
     -- preload：
        -- preload加载的资源是在浏览器渲染机制之前进行处理的，并且不会阻塞onload事件；
        -- preload可以支持加载多种类型的资源，并且可以加载跨域资源；
        -- preload加载的js脚本其加载和执行的过程是分离的。即preload会预加载相应的脚本代码，待到需要时自行调用
     -- prefetch：
        -- prefetch加载的资源可以获取当前或非当前页面所需要的资源
        -- 能将其放入缓存至少5分钟（无论资源是否可以缓存）
        -- 当页面跳转时，未完成的prefetch请求不会被中断
     -- prefetch和preload区别：preload主要用于预加载当前页面需要的资源；而prefetch主要用于加载将来页面可能需要的资源
     -- prefetch使用实例：
            document.addEventListenter('click',()=>{
              import(/* webpackPrefetch:true */ './click.js').then(({default:func})=>{
                func()
              })
            }
·cacheing：
  -- 背景：浏览器会缓存静态资源，只要资源文件名不变，浏览器就会直接从缓存中读取资源。这样会导致项目上线后，用户浏览器缓存了文件，但是我们更新了代码，用户并不能请求到最新的文件。解决这个问题的办法是资源名后跟一个hash值，例如main.dfsjkdfjk23j2k3j23.js
  -- webpack配置实现在资源名上加hash，只要我们更新代码，就会生成一个新的hash文件，并且webpack会自动更新新文件的引用：
     output:[
       filename:'[name].[contenthash].js'
       chunkFilename:'[name].[contenthash].js'
     ]
  -- 老版本webpack还需配置：
      optimization:{
        runtimeChunk:{
          name:'runtime'
        }
      }
·css单独打包：
  -- webpack默认会将css打包进入js文件中
  -- css文件单独打包出来需使用插件：mini-css-extract-plugin
·shimming：
  -- shimming：垫片，补充原来全局环境功能不足的情况
  -- 背景：假如一个第三方库的运行需要依赖jquery，但是它的package.json并未配置jquery依赖。这时我们需要全局引入jquery，实现类似@babel/polly-fill
  -- webpack全局引入第三方库的配置：
     1、引入webpack：
        const webpack = require('webpack')
     2、配置webpack自带插件：ProvidePlugin
        plugins: [
          new webpack.ProvidePlugin({
            $: 'jquery',
            _join: ['lodash', 'join']
          })
        ]
·env：
  -- env：环境全局对象
  -- 使用实例：
      module.exports = (env) => {
        if(env && env.production) {
          return merge(commonConfig, prodConfig);
        }else {
          return merge(commonConfig, devConfig);
        }
      }
      "scripts": {
          "dev": "webpack-dev-server --config ./build/webpack.common.js",
          "build": "webpack --env.production --config ./build/webpack.common.js"
      }
·resolve：


*各类场景配置的最佳实践：
·库的配置：
  -- 库的打包通常需要满足其他用户多种引入方式，例如import、require、<script>等
  -- 配置：
      {
          externals: 'lodash',//忽略lodash的打包
          output: {
              path: path.resolve(__dirname, 'dist'),
              filename: 'library.js',
              library: 'library',  //标签<script src='./library'></script>引入方式 
              libraryTarget: 'umd'  //通用模块引入方式
          }
      }
·pwa的配置
·TypeScript的配置：
·Eslint的配置：
·多页面打包配置：
  -- 多页面，即多个html页面
·Vue Cli3的配置：
  -- 通过创建vue.config.js文件进行配置
  -- cli3的配置是在webpack基础上进行了一层封装，简化了webpack打包配置，具体配置参考：https://cli.vuejs.org/zh/config/#%E5%85%A8%E5%B1%80-cli-%E9%85%8D%E7%BD%AE
  -- cli3同样可以进行webpack的原生配置，就是通过configureWebpack这个配置项进行，文档：https://cli.vuejs.org/zh/config/#configurewebpack


*提升webpack打包速度：
·跟上技术的迭代：Node、Npm、Yarn：
  -- node运行速度提升，依赖node的webpack速度自然提升
  -- 新版本的包管理工具能够更快分析包的依赖和引入
·在尽可能少的模块上应用Loader：
  -- 例如增加exclude配置，可以免去加载node_modules中的js文件
      rules: [{ 
        test: /\.js$/, 
        exclude: /node_modules/, 
        loader: 'babel-loader',
      }]
·plugin尽可能精简，并确保可靠：
  -- 例如开发环境下，无需对代码进行压缩，所以无需配置压缩插件
  -- 选择性能比较好的插件
·resolve参数合理配置：
  -- 不要配置过多查找，影响性能
·使用DllPligin插件提升打包速度：
  -- 第三方模块只打包一次
     -- 使用add-asset-html-webpack-plugin插件将第三方模块打包到一个统一文件中，并暴露到全局变量中
     -- 使用DllReferencePlugin插件实现第三方模块引入是从打包的统一文件中进行，而不是node_modules中
·控制包文件大小：
  -- 配置Three Shaking
  -- 配置SplitChunksPlugin
·thread-loader,parallel-webpack,happypack多进程打包
·合理使用sourceMap
·结合stats分析打包结果
·开发环境内存编译
·开发环境无用插件剔除


*webpack的原理：
·原理：
  1、解析webpack配置参数，合并从shell传入和webpack.config.js文件里配置的参数，生产最后的配置结果。
  2、注册所有配置的插件，好让插件监听webpack构建生命周期的事件节点，以做出对应的反应。
  3、从配置的entry入口文件开始解析文件构建AST语法树，找出每个文件所依赖的文件，递归下去。
  4、在解析文件递归的过程中根据文件类型和loader配置找出合适的loader用来对文件进行转换。
  5、递归完后得到每个文件的最终结果，根据entry配置生成代码块chunk。
  6、输出所有chunk到文件系统。
  文档：https://blog.csdn.net/sinat_17775997/article/details/89413142
·如何编写一个loader
  -- 编写一个简单的loader--replaceLoader.js：
      module.exports = function(source) { //注意此处不能写成箭头函数
        return source.replace('jacksplwxy', 'xiaoya');
      }
  -- 使用该loader:
     webpack.conf.js下
      module: {
        rules: [{
          test: /\.js/,
          use: [path.resolve(__dirname,'./loaders/replaceLoader.js')]
        }]
      }
  -- 文档：
     -- 《loader API》：https://www.webpackjs.com/api/loaders/
·如何编写一个plugin：
  -- 实例：一个可以在html文件中插入自定义代码的插件 https://github.com/jacksplwxy/inject-body-compaticity-webpack-plugin
  -- 原理：在Webpack运行的生命周期中会广播出许多事件，Plugin可以监听这些事件，在合适的时机通过Webpack提供的API改变输出结果。
  -- 一个最基础的Plugin的代码是这样的：
      class BasicPlugin{
        // 在构造函数中获取用户给该插件传入的配置
        constructor(options){
        }
        // Webpack 会调用BasicPlugin实例的apply方法给插件实例传入compiler对象
        apply(compiler){
          compiler.plugin('compilation',function(compilation) {
          })
        }
      }
      // 导出 Plugin
      module.exports = BasicPlugin;
  -- 在使用上面的Plugin时，相关配置代码如下：
      const BasicPlugin = require('./BasicPlugin.js');
      module.export = {
        plugins:[
          new BasicPlugin(options),
        ]
      }
  -- webpack运行plugin原理：
     Webpack启动后，在读取配置的过程中会先执行new BasicPlugin(options)初始化一个BasicPlugin获得其实例。
     在初始化compiler对象后，再调用basicPlugin.apply(compiler)给插件实例传入compiler对象。
     插件实例在获取到compiler对象后，就可以通过compiler.plugin(事件名称, 回调函数) 监听到Webpack广播出来的事件。
     并且可以通过compiler对象去操作Webpack。
  -- Compiler和Compilation：
     -- Compiler：包含了Webpack环境所有的的配置信息，包含options，loaders，plugins这些信息，这个对象在Webpack启动时候被实例化，它是全局唯一的，可以简单地把它理解为Webpack实例；
     -- Compilation:包含了当前的模块资源、编译生成资源、变化的文件等。当Webpack以开发模式运行时，每当检测到一个文件变化，一次新的Compilation将被创建。Compilation对象也提供了很多事件回调供插件做扩展。通过Compilation也能读取到Compiler对象。
     -- Compiler和Compilation的区别在于：Compiler代表了整个Webpack从启动到关闭的生命周期，而Compilation只是代表了一次新的编译。
  -- 文档：
     -- 《Webpack原理-编写Plugin》：https://segmentfault.com/a/1190000012840742
·如何编写一个简易webpack：
  -- 本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。
  -- 打包流程：
     -- 1、利用babel完成代码转换,并生成单个文件的依赖
     -- 2、生成依赖图谱
     -- 3、生成最后打包代码
  -- 源码：《27、make-webpack》
  -- 文档： 
     -- 《实现一个简单的Webpack》：https://zhuanlan.zhihu.com/p/76969308



*webpack的打包策略：


*总结：
  由于浏览器的版本的差异、业务的差异、开发和生产环境的差异，导致前端开发过程中不得不去处理这些问题，但人工处理会带来耗费精力、效率低下、配置遗漏、容易出错、缺乏行业最佳实践等问题，而webpack能够很好的自动化处理这些问题，是前端工业化的必然产物。
  另一方面，由于开发场景的复杂性，导致webpack配置复杂是必然性的。同时，由于技术的进步和最佳实践的积累，webpack版本不断升级也是必然性的。
  相对于以前前端的刀耕火种，webpack主要为前端开发带来了如下变化：
  1、模块化开发：
     虽然以前有一些sea.js这类的社区方案，但大部分的开发者还是更习惯通过script标签引入js文件。这样带来两个非常大的问题。一是必须要严格管理好文件的依赖关系，先后顺序一定不能反；二是前面的依赖文件的内容很容易污染后面的文件，导致根本不清楚变量或方法来自哪个文件；这两点都给可维护性带来非常大的挑战。
     而webpack原生支持node模块，而且也可以通过babel支持es6模块。
  2、模块编译：
     对于使用es6、ts、less、sass等这类新式语法的项目，我们通常都要阅读并引入大量配置文件，并且需要手动进行编译，这些繁琐的东西通常往往会导致我们不乐意采用这些新式语法，毕竟没人愿意在没有麻烦出现之前创造一堆麻烦。
     而webpack通过loader对模块的源代码进行转换，只需要我们对需要打包的文件进行统一且简单的配置即可，极大的降低新式语法的使用门槛
  3、本地服务：
     由于chrome对file协议和http协议处理文件的效果并不一致，所以在进行前端开发时通常都要将文件部署到tomcat或apache这类服务器上再进行开发，以求获得最真实的效果，这样就带来了配置服务器和项目部署的麻烦。
     而webpack通过webpack-dev-server插件提供快速启动前端本地服务的能力，为前端展示开发效果带来便利。
     本地服务除了展示前端代码的真实效果外，还提供了2个非常重要的功能：热更新和代理。热更新能自动刷新页面，极大提高开发效率；而代理通过简单配置即可解决跨域问题。
  4、提供source map
      babel、文件打包、 Tree Shaking、代码分割(Code Splitting)
      webpack打包策略
      提升打包速度



*许可协议：
  本项目所有文档遵循CC-BY-SA 4.0协议（ https://creativecommons.org/licenses/by-sa/4.0/deed.zh ）。使用者可以对本创作进行转载、节选、混编、二次创作，可以将其运用于商业用途，唯须署名作者，并且采用本创作的内容必须同样采用本协议进行授权


*更多前端工程化内容：
·npm：https://github.com/jacksplwxy/npm
·webpack：https://github.com/jacksplwxy/webpack
·代码质量：https://github.com/jacksplwxy/Good-Code
·前端自动化测试：https://github.com/jacksplwxy/AutoTest
·前端性能优化：https://github.com/jacksplwxy/FrontEndPerformanceOptimization
·web安全：https://github.com/jacksplwxy/security
·持续集成/持续部署：https://github.com/jacksplwxy/CI-CD




