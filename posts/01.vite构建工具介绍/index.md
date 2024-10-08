# Vite构建工具介绍

## 构建工具

### 什么是构建工具

&gt;  浏览器它只认识html，css，js

企业级项目里都可能具备哪些功能？

1. `typescript`：使用`tsc`工具将ts代码转换为`js`代码；
2. `React/Vue`：安装`react-compiler/vue-compiler`，将我们写的`jsx`文件或者`.vue`文件转换为render函数；
3. `less/sass/postcss/component-style`：我们有需要安装`less-loader,sass-loader`等一系列编译工具转换为css代码；
4. 语法降级：`babel`可以将es的新语法转换旧版浏览器可以接受的语法；
5. 体积优化：`uglifyjs`可以将我们的代码进行压缩变成体积更小性能更高的文件。

&gt; 以上稍微改一点，就会很麻烦，所以我们希望有一个构建工具可以将以上工具全部集成到一起，实现上述功能，我们只需要关注我们写的代码即可。即构建工具可以帮我们自动去tsc，react-comiler，less，babel，uglifyjs全部走一遍，让我们不用每次关心我们的代码在浏览器运行。

&gt; 打包：将我们写的浏览器不认识的代码，交给构建工具进行编译处理的过程就叫做打包，打包完成以后会给我们一个浏览器可以认识的文件。

构建工具承担了以下脏活累活：

1. 模块化开发支持：支持直接从node_modules里引入代码 &#43; 多种模块化支持
2. 处理代码兼容性：比如babel语法降级，less，ts语法转换（**不是构建工具做的，构建工具将这些语法对应的处理工具集成进来自动化处理**）
3. 提高项目性能：压缩文件，**代码分割**
4. 优化开发体验：
   * 构建工具会帮你自动监听文件的变化，当文件变化以后自动帮你调用对应的集成工具进行重新打包，然后再浏览器重新运行（整个过程叫做热更新，hot replacement）
   * 开发服务器：跨域的问题，用`react-cli create-react-element vue-cli`解决跨域的问题 

&gt; 我们只需要首次给构建工具提供一个配置文件（这个配置文件也不是必须的，没有它也会默认处理），有了集成的配置文件以后，我们就可以在下次需要更新的时候调用一次对应的命令即可，如果再结合热更新，我们就更加不需要管任何东西，这就是构建工具去做的事情，**它让我们不用关心生产的带啊吗也不用关心代码如何在浏览器运行，只需要关心我们的开发怎么写的爽怎么写就好了**。

### 主流构建工具

* `webpack`
* `vite`
* `parcel`
* `esbuild`
* `rollup`
* `grunt`
* `gulp`

&gt; 国内主流还是`webpack`，`vite`和`esbuild`。

### vite相较于webpack的优势

&gt; 然而，当我们开始构建越来越大型的应用时，需要处理的 JavaScript 代码量也呈指数级增长。包含数千个模块的大型项目相当普遍。基于 JavaScript 开发的工具就会开始遇到性能瓶颈：&lt;font color=&#34;red&#34;&gt;通常需要很长时间（甚至是几分钟！）才能启动开发服务器，即使使用模块热替换（HMR），文件修改后的效果也需要几秒钟才能在浏览器中反映出来。&lt;/font&gt;如此循环往复，迟钝的反馈会极大地影响开发者的开发效率和幸福感。

起因：我们的项目越大，构建工具（webpack）所要处理的js代码就越多【跟webpack的一个构建过程（工作流程有关）】

造成的结果：构建工具需要很长时间才能启动开发服务器（把项目跑起来）

```shell
yarn start
yarn dev

npm run dev
npm run start
```

&gt; webpack不能改，如果要改，则会动到webpack的大动脉。
&gt;
&gt; ```js
&gt; // 这段代码最终回到浏览器里去运行
&gt; const lodash = require(&#34;lodash&#34;); // commonjs 规范
&gt; import Vue from &#34;vue&#34;; // es6 module
&gt; 
&gt; // webpack是允许我们这么写的，webpack会这样转换
&gt; const lodash = webpack_require(&#34;loadsh&#34;);
&gt; const Vue = webpack_require(&#34;vue&#34;);
&gt; ```
&gt;
&gt; webpack的编译原理，AST抽象语法分析的工具，分析出js文件有哪些导入导出操作
&gt;
&gt; &lt;font color=&#34;red&#34;&gt;构建工具是运行在服务端的&lt;/font&gt;
&gt;
&gt; ```js
&gt; (function(modules) {
&gt;     function webpack_require() {}
&gt;     // 入口是index.js
&gt;     // 通过webpack的配置文件得来的：webpack.config.js ./src/index.js
&gt;     modules[entry](webpack_require);
&gt; }, ({
&gt;     &#34;index.js&#34;: (webpack_require) =&gt; {
&gt;         const lodash = webpack_require(&#34;lodash&#34;);
&gt; 		const Vue = webpack_require(&#34;vue&#34;);
&gt;     }
&gt; }))
&gt; ```
&gt;
&gt; 因为webpack支持模块化，它一开始就必须要统一模块化代码，所以意味着它需要将所有的依赖读一遍。
&gt;
&gt; vite会不会直接把webpack干翻？vite是基于es modules的，侧重点不一样，webpack更多的关注兼容性，而vite关注浏览器端的开发体验。

## vite启动项目初体验

### 你必须要理解的vite脚手架和vite

&gt; vite官网搭建vite项目文档教程：[https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project](https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project)

比如我们敲了```yarn create vite```

1. 帮我们全局安装了一个东西：`create-vite`（vite的脚手架）
2. 直接运行这个`create-vite` bin目录下的一个执行配置

&lt;font color=&#34;red&#34;&gt;误区：认为官网中使用对应的yarn create构建项目的过程也是vite在做的事情&lt;/font&gt;

&gt; 我们之前接触过vue-cli，create-vite和vite的关系是：create-vite内置了vite。使用vue-cli会内置webpack

先学习的就是vite，暂时不会使用`yarn create vite my-vue-app --template vue`。vue-cli可以和webpack分的很清楚。

&gt; vue-cli给我们提供已经精装修的模板。
&gt;
&gt; 我们自己搭建一个项目：下载vite，vue，post-css，less，label
&gt;
&gt; vue-cli/create-vite给了一套精装修的模板：什么都下好了，并且给你做了最佳实践的配置

### vite开箱即用

&gt; 开箱即用（out of box）：不需要做任何额外的配置就可以使用vite来帮你处理构建工作

在默认情况下，我们的esmodule去导入资源的时候，要么是绝对路径，要么是相对路径，既然我们现在的最佳实践是node_modules，那么为什么es官方在我们导入非绝对路径和非相对路径的资源的时候不默认帮我们搜寻node_modules？

&gt; 浏览器环境中的安全性原因是一个主要考虑因素。如果浏览器默认搜索`node_modules`目录，那么恶意的代码可能会利用这个功能访问和执行不受信任的模块代码，从而导致安全风险。通过限制导入路径，浏览器可以更好地控制模块的来源和访问权限。
&gt;
&gt; 另外，性能也是一个重要的考虑因素。浏览器默认只支持绝对路径和相对路径的导入，可以在编译时静态解析模块依赖关系，从而提高加载和执行模块的效率。如果浏览器要搜索`node_modules`目录，可能需要进行额外的文件系统操作和路径解析，增加了加载模块的时间和资源消耗。

### vite的预加载

```js
import _ from &#34;lodash&#34;; // lodash可能也import了其他的东西

import _vite_cjsImport0_lodash from &#34;/node_modules/.vite/deps/loadsh.js?v=ebe57916&#34;;
```

在处理的过程中如果说看到了有非绝对路径或者相对路径的引用，则会尝试开启路径补全

找寻依赖的过程是自当前目录依次向上查找的过程，直到搜寻到根目录或者搜寻到对应依赖为止 `/user/node_modules/lodash, ../`

&gt; 区分生产环境和开发环境：
&gt;
&gt; `yarn dev` ----&gt; 开发（每次依赖预构建所重新构建的相对路径都是正确的）
&gt;
&gt; vite会全权交给一个叫做rollup的库去完成生产环境的打包
&gt;
&gt; 缓存 ----&gt;
&gt;
&gt; 实际上vite在考虑另外一个问题的时候就顺便把这个问题解决了
&gt;
&gt; 
&gt;
&gt; commonjs 规范的导出 module.exports
&gt;
&gt; 有的包是以commonjs规范的格式导出 axios
&gt;
&gt; **依赖预构建**：首先vite会找到对应的依赖，然后调用esbuild（对js语法进行处理的一个库），将其他规范的代码转换成esmodule规范，然后放到当前目录下的node_modules/.vite/deps，同时对esmodule规范的各个模块进行统一集成

所以它解决了3个问题：

1. 不同的第三方包会有不同的导出格式（这个是vite没法约束人家的事情）
2. 对路径的处理上可以直接使用.vite/deps，方便路径重写
3. 叫做网络多包传输的性能问题（也是原生esmodule规范不敢支持node_modules的原因之一），有了依赖预构建以后无论它有多少的额外的export和import，vite都会尽可能的将他们集成最后只生成一个或者几个模块

`vite.config.js === webpack.config.js`

### vite配置文件处理细节

1. vite配置文件的语法提示

   1. 如果你使用的是webstorm，那你可以得到很好的语法补全
   2. 如果你使用的是vscode或者其他编辑器，则需要做一些特殊处理

2. 关于环境的处理

   &gt; 过去我们使用webpack的时候，我们需要区分配置文件的一个环境：
   &gt;
   &gt; * `webpack.dev.config`
   &gt; * `webpack.prod.config`
   &gt; * `webpack.base.config`
   &gt; * `webpackmerge`

### vue环境变量配置

&gt; 环境变量：会根据当前的代码环境产生值的变化的变量就叫做环境变量
&gt;
&gt; 代码环境：
&gt;
&gt; 1. **开发环境**：开发环境是开发人员进行软件开发和调试的地方。在开发环境中，开发人员可以进行代码编写、调试、测试和验证。这个环境通常是本地的开发机器，开发人员可以通过使用 Vite 的开发服务器（dev server）来提供实时的重新加载（live reloading）和模块热替换（hot module replacement）功能，以便更快地进行开发和调试。
&gt; 2. **测试环境**：测试环境是用于进行软件测试的环境。在测试环境中，开发人员和测试人员可以对软件进行不同类型的测试，例如单元测试、集成测试和端到端测试。在测试环境中，可以使用 Vite 构建工具生成测试所需的构建文件，并在模拟的环境中进行测试。
&gt; 3. **预发布环境**：预发布环境是在软件发布之前进行最后测试和验证的环境。在预发布环境中，可以对软件进行更全面的测试，以确保它符合发布的质量标准。这个环境通常是一个类似于生产环境的环境，但在实际发布之前，可能会使用一些模拟数据和模拟系统进行测试。
&gt; 4. **灰度环境**：灰度环境是在软件发布后逐步向用户群体推出新功能或更新的环境。在灰度环境中，新的软件版本或功能将部署到一小部分用户中，以便测试其稳定性和兼容性。这个环境类似于生产环境，但只有一部分用户能够访问新的功能或更新。
&gt; 5. **生产环境**：生产环境是最终向用户提供服务的环境。在生产环境中，已经通过了开发、测试、预发布和灰度环境的验证，并且已准备好为最终用户提供稳定、可靠的服务。在生产环境中，通常会使用 Vite 构建工具生成用于部署的生产级别的构建文件，并进行必要的优化和压缩，以提供最佳的性能和用户体验。

我们在和后端同学对接的时候，前端在开发环境中请求的后端API地址和生产环境请求的后端API地址是一个吗？肯定不是一个

* 开发和测试：http://test.api/
* 生产：https://api/

&gt; 在vite中的环境变量处理：
&gt;
&gt; vite内置了dotenv这个第三方库，会自动读取.env文件，并解析这个文件中的对应的环境变量，并将其注入到process对象下（但是vite考虑到和其他配置的一些冲突问题，它不会直接注入到process对象下）
&gt;
&gt; 涉及到vite.config.js中的一些配置：
&gt;
&gt; * root
&gt; * envDir：用来配置当前环境变量的文件地址
&gt;
&gt; vite给我们提供了一些补偿措施：我们可以调用vite的loadEnv来手动确认env文件
&gt;
&gt; porcess.cwd方法：返回node进程的工作目录

&gt; 小知识：为什么vite.config.js可以书写成esmodule的形式，这是因为vite他在读取这个vite.config.js的时候会率先node去解析文件语法，如果发现你是esmodule规范则会直接将你的esmodule规范替换成common js规范

&gt; .env：所有环境都需要用到的环境变量
&gt;
&gt; .env.development：开发环境需要用到的环境变量（默认情况下vite将我们的开发环境取名为development）
&gt;
&gt; .env.production：生产环境需要用到的环境变量（默认情况下vite将我们的生产环境取名为production）
&gt;
&gt; 
&gt;
&gt; yarn dev --mode development 会将mode设置为development传递进来
&gt;
&gt; 当我们调用loadenv的时候，它会做如下几件事情：
&gt;
&gt; 1. 直接找到.env文件不解释，并解析其中的环境变量，放到一个对象里
&gt;
&gt; 2. 会将传进来的mode这个变量的值进行拼接：`env.development`，并根据我们提供的目录去取对应配置文件并进行解析，并放进一个对象
&gt;
&gt; 3. 我们可以理解为
&gt;
&gt;    ```js
&gt;    const baseEnvConfig = 读取.env配置
&gt;    const modeEnvConfig = 读取env相关配置
&gt;    const lastEnvConfig = {...baseEnvConfig, ...modeEnvConfig}
&gt;    ```

如果是客户端，vite会将对应的环境变量注入到import.meta.env里去

vite做了一个拦截，为了防止我们将隐私性的变量直接送到import.meta.env中，所以做了一层拦截，如果你的环境变量不是以VITE开头的，他就不会帮你注入到客户端中去，如果我们想要更改这个前缀，可以去使用envPrefix配置。

## vite 原理篇

### vite是怎么让浏览器可以识别.vue文件呢

Vite 是一个现代化的前端构建工具，它使用了一种名为单文件组件（Single File Components）的技术来让浏览器能够识别和加载 .vue 文件。

在传统的前端开发中，浏览器无法直接识别和加载 .vue 文件，因为 .vue 文件包含了 HTML、CSS 和 JavaScript 代码，而浏览器只能识别和执行 JavaScript 文件。

Vite 利用了构建工具和打包器的能力，在开发阶段将 .vue 文件转换为浏览器可以识别的形式。它借助特定的编译器（如 Vue 编译器）将 .vue 文件的模板、样式和脚本部分分别提取出来，并将它们转换为浏览器可以理解的代码。

具体地说，Vite 使用了名为 &#34;vue-loader&#34; 的工具来解析和转换 .vue 文件。这个工具会解析 .vue 文件的内容，提取出其中的模板、样式和脚本，并将它们转化为独立的代码块。然后，Vite 使用浏览器原生的 ES 模块导入机制，通过 `&lt;script type=&#34;module&#34;&gt;` 标签将这些代码块加载到浏览器中。

当浏览器遇到 `&lt;script type=&#34;module&#34;&gt;` 标签时，它会将标签中的 JavaScript 代码作为一个独立的模块加载和执行。Vite 利用这个特性，将 .vue 文件中的模板、样式和脚本部分分别作为独立的模块加载到浏览器中，并在浏览器中动态组合它们，构建出最终的组件。

### 使用path.resolve的原因

&gt; 在使用路径时，尽量使用 `path.resolve` 方法可以确保路径的可靠性和跨平台性。以下是几个使用 `path.resolve` 方法的好处：
&gt;
&gt; 1. 处理相对路径和绝对路径：`path.resolve` 方法可以接受多个参数，将它们解析为一个绝对路径。这意味着你可以使用相对路径作为参数，并将其解析为相对于当前工作目录的绝对路径。这对于确定准确的文件路径非常有用。
&gt; 2. 解决跨平台路径问题：在不同的操作系统中，对于路径分隔符和路径表示法有所差异。使用 `path.resolve` 方法可以确保生成的路径在不同的操作系统下都是有效的，因为它会自动根据当前操作系统调整路径分隔符和表示法。
&gt; 3. 处理路径拼接和规范化：`path.resolve` 方法会将传入的路径片段进行拼接，并返回一个规范化的路径。这意味着它会解析和处理路径中的 `..`、`.` 等相对路径符号，确保生成的路径是规范化的、干净的路径。
&gt; 4. 确保路径的存在性：使用 `path.resolve` 方法生成的路径是确保存在的，它不会检查路径是否有效或文件是否存在，但会确保路径的格式正确。这可以帮助你在操作文件系统时提供正确的路径，避免出现错误或异常。

## vite与css

### 在vite中处理css

&gt; vite天生就支持对css文件的直接处理

1. vite在读取到main.js文件中引用到了index.css
2. 直接使用fs模块去读取index.css中文件内容
3. 直接创建一个style标签，将index.css中文件内容直接copy到style标签里
4. 将style标签插入到index.html的head中
5. 将css文件中的内容直接替换为js脚本（方便热更新或者css模块化），同时设置`Content-Type`为js，从而让浏览器以JS脚本的形式来执行该css后缀的文件

&gt; 场景：
&gt;
&gt; * 一个组件最外层的元素类名一般取名： wrapper
&gt; * 一个组件最底层的元素类名一般取名：footer
&gt;
&gt; 但你取了footer这个名字，别人因为没有看过你这个组件的源代码，也可能去取名footer这个类名，最后可能会导致样式被覆盖（因为类名重复），这就是我们在协同开发很容易出现的问题

`cssmodule`就是来解决这个问题的：

1. module.css（module是一种约定，表示需要开启css模块化）
2. 他会将你的所有类名进行一定规则的替换（将footer替换为_footer_i22st_1）
3. 同时创建一个映射对象{ `footer: &#34;_footer_i22st_1&#34;`}
4. 将替换过后的内容塞进style标签里然后放入到head标签中（能够读到index.html的文件内容）
5. 将componentA.module.css内容全部抹除，替换为JS脚本
6. 将创建的映射对象在脚本中默认导出

### css文件类型

&gt; 1. CSS（.css）：CSS 是层叠样式表的标准文件格式，它使用类似于选择器和属性的语法来描述网页的样式。CSS 是前端开发中最常见的样式表语言，浏览器原生支持。
&gt; 2. LESS（.less）：LESS 是 CSS 的拓展，它引入了变量、嵌套规则、Mixin（混入）等功能，以简化 CSS 的编写和维护。LESS 文件需要在开发阶段通过 LESS 编译器转换为标准的 CSS 文件，然后在浏览器中加载。
&gt; 3. SCSS/SASS（.scss/.sass）：SCSS（Sassy CSS）和 SASS（Syntactically Awesome Style Sheets）也是 CSS 的拓展，提供了类似 LESS 的功能，如变量、嵌套规则和 Mixin。SCSS 与 SASS 的语法略有不同，但都需要通过编译器将其转换为标准的 CSS 文件。

```scss
/* SCSS */
.container {
  width: 100%;

  .header {
    background-color: #333;
    color: #fff;
    padding: 10px;
  }

  .content {
    padding: 20px;

    p {
      margin-bottom: 10px;
    }

    a {
      color: #f00;
      text-decoration: none;

      &amp;:hover {
        text-decoration: underline;
      }
    }
  }

  .footer {
    background-color: #333;
    color: #fff;
    padding: 10px;
  }
}
```

在这个示例中，`.container` 是最外层的容器选择器，它包含了 `.header`、`.content` 和 `.footer` 子选择器。通过嵌套定义，我们可以更直观地表示这些选择器之间的层次结构。

此外，嵌套定义还可以减少重复代码的编写。在上述示例中，`.header` 和 `.footer` 具有相同的背景颜色、文字颜色和内边距，通过嵌套定义，我们只需在父选择器中指定一次即可，避免了重复的样式声明。

另外，嵌套定义还可以方便地应用伪类和伪元素样式。在示例中，嵌套定义了 `a` 元素的样式，并使用 `&amp;:hover` 表示其悬停状态下的样式，这样可以更直观地表示选择器之间的关系。

通过使用嵌套定义，我们可以更清晰地组织和维护样式代码，减少了冗余和重复的工作，提高了代码的可读性和可维护性。

### vite.config.js中css配置

&gt; 在vite.config.js中我们通过css属性去控制整个vite对于css的处理行为

#### module篇

```js
css: { 
    
    modules: { 
    }
}
css: { // 对css的行为进行配置
  // modules配置最终会丢给postcss modules
  modules: { // 是对css模块化的默认行为进行覆盖
    localsConvention: &#39;camelCase&#39;, // 修改生成的配置对象的 key 的展示形式为驼峰命名
    scopeBehaviour: &#39;global&#39;, // 配置当前的模块化行为为全局化
    generateScopedName: &#39;[name]__[local]___[hash:base64:5]&#39;, // 指定生成的类名的命名规则
    hashPrefix: &#39;my-app&#39;, // 生成的 hash 的前缀
    globalModulePaths: [&#39;path/to/global/styles&#39;] // 不参与 CSS 模块化的路径
  }
}
```

* `localsConvention`：修改生成的配置对象的key的展示形式（驼峰还是中划线形式）
* `scopeBehaviour`：配置当前的模块化行为是模块化还是全局化（有hash就是开启了模块化的一个标志，因为它可以保证产生不同的hash值来控制我们的样式类名不被覆盖）
* `generateScopedName`：`[name_[local]_[hash:5]]`，指定生成的类名的命名规则（可以配置为函数，也可以配置成字符串规则）
* `hashPrefix`：生成的hash会根据你的类名进行生成，如果想要你生成的hash更加的独特一点，你可以配置hashPrefix，你配置的这个字符串会参与到最终的hash生成
* `globalModulePaths`：代表你不想参与到css模块化的路径

#### preprocessorOption篇

&gt; 主要是用来配置css预处理的一些全局参数

```js
css: {
  preprocessorOptions: {
    // 配置 CSS 预处理器的全局参数
  }
}
```

在 `preprocessorOptions` 中，你可以配置 CSS 预处理器的一些全局参数，具体参数的配置取决于你使用的预处理器（如 Sass、Less 等）。这里可以配置一些通用的选项，比如：

- `additionalData`：额外的全局样式数据，可以在每个 CSS 文件的顶部注入，例如共享的变量、混合器等。
- `sass`：用于配置 Sass 预处理器的选项，如 `sass` 选项中的 `indentedSyntax` 表示是否使用缩进语法。
- `less`：用于配置 Less 预处理器的选项，如 `less` 选项中的 `javascriptEnabled` 表示是否启用 Less 中的 JavaScript 表达式。

#### postcss篇

```js
import autoprefixer from &#39;autoprefixer&#39;;

export default {
  // ...
  css: {
    postcss: {
      plugins: [
        autoprefixer(), // 配置 PostCSS 插件，例如 Autoprefixer
        // 其他的 PostCSS 插件...
      ]
    }
  }
}
```

在 `postcss` 配置项中，你可以指定要使用的 PostCSS 插件。在示例中，我们使用了一个常见的插件 Autoprefixer，它用于自动添加 CSS 浏览器前缀，以提供跨浏览器兼容性。

你可以根据需要，将其他的 PostCSS 插件添加到 `plugins` 数组中。这些插件可以用于执行各种 CSS 处理任务，例如压缩、优化、转换等。

需要注意的是，为了使用特定的 PostCSS 插件，你需要在项目中安装这些插件的相应依赖，并在 `vite.config.js` 文件中进行正确的导入和配置。

此外，你还可以在 `postcss` 配置项中设置其他选项，例如 `config`、`sourceMap` 等，以满足特定的需求。具体的配置选项和语法规则可以参考 PostCSS 插件的文档或相关资源。

## vite相关知识

### vite加载静态资源

&gt; 什么是静态资源？
&gt;
&gt; 静态资源是指不需要经过构建处理的文件，例如图片、视频、字体等，除了动态API以外，百分之九十九的资源都被视作静态资源

vite对静态资源基本上是开箱即用的，除了一些特殊情况（svg）

要加载静态资源，你可以将它们放置在你的项目目录中的任何位置。通常，你可以将这些静态资源放置在你的项目根目录下的 `public` 文件夹中，这是一个预定义的静态资源文件夹。当你在代码中引用这些静态资源时，Vite 会自动将它们提供给你的应用程序。

例如，如果在你的项目中有一个名为 `public` 的文件夹，并且在其中有一个图片文件 `logo.png`，你可以在代码中像下面这样引用它：

```html
&lt;img src=&#34;/logo.png&#34; alt=&#34;Logo&#34; /&gt;
```

Vite 会自动将此路径解析为相应的静态资源，并将其提供给你的应用程序。

需要注意的是，在 Vite 中，你可以使用相对于根目录的绝对路径来引用静态资源，而无需考虑模块化的路径解析。这是因为 Vite 使用自己的开发服务器，能够在运行时动态处理这些静态资源的请求。

对于某些特殊的静态资源，如 SVG 文件，你可能需要额外的配置来确保正确加载。对于 SVG 文件，你可以使用 `@vitejs/plugin-svg` 插件来处理。你可以按照 Vite 官方文档中的说明，添加该插件并进行相应的配置。

### vite在生产环境中对静态资源的处理

&gt; 1. **静态资源的导入和处理：** 在你的代码中，如果有静态资源的导入语句（如图片、字体、CSS 文件等），Vite 会根据这些导入语句自动处理这些资源。
&gt; 2. **资源优化和压缩：** Vite 会对导入的静态资源进行优化和压缩，以减小文件大小并提升加载性能。这包括但不限于压缩图片、压缩和合并 CSS 文件等操作。
&gt; 3. **指纹化文件名：** 为了更好的缓存管理和更新机制，Vite 会为处理后的静态资源生成带有指纹的文件名。这意味着每个文件都会有一个唯一的哈希值作为文件名的一部分，例如 `logo.8e4c5f7b.png`。当文件内容发生变化时，哈希值也会发生变化，从而确保客户端能够正确地缓存和更新静态资源。
&gt; 4. **输出静态资源：** 处理后的静态资源会被输出到构建目录（默认为 `dist`）中。Vite 会根据资源类型生成相应的文件，如图片会生成 `.png`、`.jpg` 等文件，CSS 文件会生成 `.css` 文件等。
&gt; 5. **引用静态资源：** 在你的 HTML 文件或生成的代码中，Vite 会自动更新静态资源的引用路径，以指向构建目录中的正确文件。这样，在生产环境中，你可以直接使用相对于构建目录的路径来引用静态资源，而无需关心开发环境中的模块解析和路径处理。

### vite常用插件

&gt; 插件是什么？
&gt;
&gt; vite会在生命周期的不同阶段中调用不同的插件以达到不同的目的
&gt;
&gt; 生命周期：vite从开始执行到执行结束

#### vite-aliases

&gt; 作用：别名自动生成
&gt;
&gt; 安装：`yarn add vite-aliases -D`
&gt;
&gt; 将其添加到`vite.config.js`中
&gt;
&gt; ```js
&gt; // vite.config.js
&gt; import { ViteAliases } from &#39;vite-aliases&#39;
&gt; 
&gt; export default {
&gt;   plugins: [
&gt;     ViteAliases()
&gt;   ]
&gt; };
&gt; ```

vite-aliases的可选配置项如下：

```js
ViteAliases({
  /**
  * Relative path to the project directory
  */
  dir: &#39;src&#39;,

  /**
  * Prefix symbol for the aliases
  */
  prefix: &#39;~&#39;,

  /**
  * Allow searching for subdirectories
  */
  deep: true,

  /**
  * Search depthlevel for subdirectories
  */
  depth: 1,

  /**
  * Creates a Logfile
  * use `logPath` to change the location
  */
  createLog: false,

  /**
  * Path for Logfile
  */
  logPath: &#39;src/logs&#39;,

  /**
  * Create global project directory alias
  */
  createGlobalAlias: true,

  /**
  * Turns duplicates into camelCased path aliases
  */
  adjustDuplicates: false,

  /**
  * Used paths in JS/TS configs will now be relative to baseUrl
  */
  useAbsolute: false,

  /**
  * Adds seperate index paths
  * approach created by @davidohlin
  */
  useIndexes: false,

  /**
  * Generates paths in IDE config file
  * works with JS or TS
  */
  useConfig: true,

  /**
  * Override config paths
  */
  ovrConfig: false,

  /**
  * Will generate Paths in tsconfig
  * used in combination with `useConfig`
  * Typescript will be auto detected
  */
  dts: false,

  /**
  * Disables any terminal output
  */
  silent: true,

  /**
  * Root path of Vite project
  */
  root: process.cwd()
});
```

#### vite-plugin-html

&gt; 功能：
&gt;
&gt; 1. HTML 压缩能力
&gt; 2. EJS 模版能力
&gt; 3. 多页应用支持
&gt; 4. 支持自定义`entry`
&gt; 5. 支持自定义`template`
&gt;
&gt; 安装：`yarn add vite-plugin-html -D`
&gt;
&gt; 将其添加到`vite.config.js`中

&gt; 用法：
&gt;
&gt; 将EJS标签添加到`index.html`中
&gt;
&gt; ```html
&gt; &lt;head&gt;
&gt;   &lt;meta charset=&#34;UTF-8&#34; /&gt;
&gt;   &lt;link rel=&#34;icon&#34; href=&#34;/favicon.ico&#34; /&gt;
&gt;   &lt;meta name=&#34;viewport&#34; content=&#34;width=device-width, initial-scale=1.0&#34; /&gt;
&gt;   &lt;title&gt;&lt;%- title %&gt;&lt;/title&gt;
&gt;   &lt;%- injectScript %&gt;
&gt; &lt;/head&gt;
&gt; ```
&gt;
&gt; 在`vite.config.js`中配置，该方法可以根据需要引入需要的功能
&gt;
&gt; ```js
&gt; import { defineConfig, Plugin } from &#39;vite&#39;
&gt; import vue from &#39;@vitejs/plugin-vue&#39;
&gt; 
&gt; import { createHtmlPlugin } from &#39;vite-plugin-html&#39;
&gt; 
&gt; export default defineConfig({
&gt;   plugins: [
&gt;     vue(),
&gt;     createHtmlPlugin({
&gt;       minify: true,
&gt;       /**
&gt;        * After writing entry here, you will not need to add script tags in `index.html`, the original tags need to be deleted
&gt;        * @default src/main.ts
&gt;        */
&gt;       entry: &#39;src/main.ts&#39;,
&gt;       /**
&gt;        * If you want to store `index.html` in the specified folder, you can modify it, otherwise no configuration is required
&gt;        * @default index.html
&gt;        */
&gt;       template: &#39;public/index.html&#39;,
&gt; 
&gt;       /**
&gt;        * Data that needs to be injected into the index.html ejs template
&gt;        */
&gt;       inject: {
&gt;         data: {
&gt;           title: &#39;index&#39;,
&gt;           injectScript: `&lt;script src=&#34;./inject.js&#34;&gt;&lt;/script&gt;`,
&gt;         },
&gt;         tags: [
&gt;           {
&gt;             injectTo: &#39;body-prepend&#39;,
&gt;             tag: &#39;div&#39;,
&gt;             attrs: {
&gt;               id: &#39;tag&#39;,
&gt;             },
&gt;           },
&gt;         ],
&gt;       },
&gt;     }),
&gt;   ],
&gt; })
&gt; ```

#### vite-plugin-mock

&gt; mock数据：模拟数据
&gt;
&gt; 前后端一般是并行开发，用户列表（接口文档）
&gt;
&gt; mock数据，去做你前端的工作
&gt;
&gt; 1. 简单方式：直接去写死一两个数据，方便调试。
&gt;    * 缺陷：没法做海量数据测试
&gt;    * 没法获得一些标准数据
&gt;    * 没法去感知http的异常
&gt; 2. mockjs：模拟海量数据的，vite-plugin-mock的依赖项是mockjs

&gt; 安装：
&gt;
&gt; ```shell
&gt; yarn add mockjs -D
&gt; yarn add vite-plugin-mock -D
&gt; ```
&gt;
&gt; 使用方法：[https://github.com/vbenjs/vite-plugin-mock](https://github.com/vbenjs/vite-plugin-mock)

#### 其他插件

[插件地址](https://cn.vitejs.dev/plugins/)



---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.vite%E6%9E%84%E5%BB%BA%E5%B7%A5%E5%85%B7%E4%BB%8B%E7%BB%8D/  

