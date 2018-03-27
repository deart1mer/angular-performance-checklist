# Angular 性能检查表

<img src="./assets/flash.png" width="1000">

> 翻译自 [mgechev](https://github.com/mgechev) 大神的文章
>
> 原文链接：https://github.com/mgechev/angular-performance-checklist

## 介绍

本文档包含一系列实践，可帮助我们提升 Angular 应用程序的性能。 “Angular 性能检查表”涵盖了不同的主题 - 从服务器端预渲染和打包应用程序到运行时性能以及框架执行的变更检测优化。

该文件分为两个主要部分：

* 网络性能 - 列出了将大大改善我们应用程序的加载时间的实践。它们包括延迟和带宽降低的方法。
* 运行时性能 - 提高应用程序运行时性能的实践。它们主要包括更改检测和渲染相关的优化。

有些做法会影响到两个类别，因此可能会有一个小小的交集，但是，用例和含义的差异将被明确提及。

大多数小节列出了与具体实践相关的工具，这些工具可以使我们的开发流程实现自动化，从而提高工作效率。

请注意，大多数实践对于 HTTP/1.1 和 HTTP/2 均有效。通过指定可以应用哪种协议版本，将会提到做出例外的做法。

## 目录

- [Angular 性能清单](#angular-性能清单)
  - [介绍](#介绍)
  - [目录](#目录)
  - [网络性能](#网络性能)
    - [打包](#打包)
    - [压缩代码和死码消除](#压缩代码和死码消除)
    - [删除模板的空格](#删除模板的空格)
    - [Tree-shaking](#tree-shaking)
    - [AoT(Ahead-of-Time) 编译](#aotahead-of-time-编译)
    - [压缩](#压缩)
    - [资源预获取](#资源预获取)
    - [资源懒加载](#资源懒加载)
    - [不要懒加载默认路由](#不要懒加载默认路由)
    - [缓存](#缓存)
    - [使用 Application Shell](#使用-application-shell)
    - [使用 Service Workers](#使用-service-workers)
  - [运行时优化](#运行时优化)
    - [使用 `enableProdMode`](#使用-enableprodmode)
    - [AoT(Ahead-of-Time) 编译](#aotahead-of-time-编译-1)
    - [Web Workers](#web-workers)
    - [服务端渲染](#服务端渲染)
    - [变更检测](#变更检测)
      - [`ChangeDetectionStrategy.OnPush`](#changedetectionstrategyonpush)
      - [分离变更检测器](#分离变更检测器)
      - [Angular Zone 上下文之外执行](#angular-zone-上下文之外执行)
    - [使用纯管道](#使用纯管道)
    - [为 `*ngFor` 指令使用 `trackBy` 选项](#为-ngfor-指令使用-trackby-选项)
- [结论](#结论)
- [贡献](#贡献)
- [License](#license)

## 网络性能

本节中的一些工具仍在开发中，可能会发生变化。 Angular核心团队正在尽可能地为我们的应用程序自动化构建过程，所以很多事情都会透明地发生。

### 打包

打包是一种标准的做法，旨在减少浏览器为了交付用户请求的应用程序而需要执行的请求数量。本质上，打包程序接收输入点列表并生成一个或多个 bundle。这样，浏览器就可以通过只执行一些请求来获取整个应用程序，而不是分别请求每个单独的资源。

随着您的应用程序增长，将所有内容捆绑到一个大型 bundle 中，它们将再次产生反作用。使用 Webpack 探索代码分割技术。

**由于 HTTP/2 [服务器推送](https://http2.github.io/faq/#whats-the-benefit-of-server-push)功能，不需要关注附加的 http 请求。**

**Tooling**

允许我们高效打包我们的应用程序的工具有：

- [Webpack](https://webpack.js.org) - 提供 [tree-shaking](#tree-shaking) 有效的打包。
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - 分割代码的技巧。.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - 使用 http2 分割代码。
- [Rollup](https://github.com/rollup/rollup) - 利用ES2015模块的静态特性，通过 tree-shaking 提供有效的打包。
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 执行大量优化并提供打包支持。最初是用 Java 编写的，因为最近它也有一个 [JavaScript 版本](https://www.npmjs.com/package/google-closure-compiler-js)可以在[这里](https://www.npmjs.com/package/google-closure-compiler-js)找到。
- [SystemJS Builder](https://github.com/systemjs/builder) - 为混合依赖模块树的 SystemJS 提供单文件构建。
- [Browserify](http://browserify.org/).

**资源**



- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### 压缩代码和死码消除

这些做法使我们能够通过减少应用程序的有效负载来最小化带宽消耗。

**工具**

- [Uglify](https://github.com/mishoo/UglifyJS) - 执行压缩比如修改变量，删除注释和空白，删除死代码等。完全用JavaScript编写，为所有流行的任务运行者提供了插件。
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 执行类似于 uglify 压缩类型。在高级模式下，它积极地转换我们程序的AST，以便能够执行更复杂的优化。它也有一个[JavaScript版本](https://www.npmjs.com/package/google-closure-compiler-js)，可以在[这里](https://www.npmjs.com/package/google-closure-compiler-js)找到。 GCC 还支持大部分 ES2015 模块语法，因此可以执行 [tree-shaking](#tree-shaking)。

**资料**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### 删除模板的空格

尽管我们没有看到空格字符（一个匹配`\s`正则表达式的字符），但它仍然是通过网络传输的字节表示的。如果我们将模板中的空白减少到最小，我们将分别能够进一步降低 AoT 代码的包大小。

谢天谢地，我们不必手动执行此操作。 `ComponentMetadata` 接口提供属性 `preserveWhitespaces` ，默认值为 `true` ，因为删除空白总是会影响 DOM 布局。如果我们将该属性设置为 `false`，那么 Angular 将修剪不必要的空白，这将导致进一步缩小包的大小。

- [preserveWhitespaces in the Angular docs](https://angular.io/api/core/Component#preserveWhitespaces)

### Tree-shaking

对于我们应用程序的最终版本，我们通常不使用 Angular 和/或任何第三方库提供的完整代码，即使是我们编写的代码。得益于 ES2015 模块的静态特性，我们能够摆脱我们应用中未引用的代码。

**例子**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```
一旦我们 tree-shaking 和打包 `app.js`，我们会得到：

```javascript
let foo = () => 'foo';
console.log(foo());
```

这意味着未使用的导出的 `bar` 将不会包含在最终 bundle 中。

**工具**

- [Webpack](https://webpack.js.org) - 通过执行 tree-shaking 优化提供高效的打包。应用程序打包后，它不会导出未使用的代码，因此它可以安全地将其视为死代码并由 Uglify 删除。
- [Rollup](https://github.com/rollup/rollup) - 利用 ES2015 模块的静态特性，通过执行高效的 tree-shaking 来打包。
- [Google Closure Compiler](https://github.com/google/closure-compiler) - 执行大量优化并提供打包支持。最初是用 Java 编写的，因为最近它也有一个 [JavaScript 版本](https://www.npmjs.com/package/google-closure-compiler-js)可以在[这里](https://www.npmjs.com/package/google-closure-compiler-js)找到。

*注意:* GCC 目前还不支持 `export *` 。这对于构建 Angular 应用程序非常重要，因为“barrel”模式的使用方法非常繁琐。

**资料**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### AoT(Ahead-of-Time) 编译

广泛的工具（如GCC，Rollup等）的可用性挑战是无法使用它们的功能进行分析 Angular 组件的类 HTML 模板。这使得他们的 tree-shaking 支持效率较低，因为他们不确定哪些指令在模板中被引用。 AoT 编译器通过 ES2015 模块导入将 Angular HTML 模板转换为 JavaScript 或 TypeScript。通过这种方式，我们可以在打包过程中高效地进行 tree-shake，并删除由 Angular、第三方库或我们自己定义的所有未使用的指令。

**工具**

- [@angular/compiler-cli](https://github.com/angular/angular/tree/master/modules/%40angular/compiler-cli) - 代替 [tsc](https://www.npmjs.com/package/typescript) 静态分析我们的应用程序并为组件的模板输出 TypeScript / JavaScript。

**资料**

- ["Ahead-of-Time Compilation in Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### 压缩

压缩响应的有效载荷是减少带宽使用的标准做法。通过指定 `Accept-Encoding` 头的值，浏览器会提示服务器哪些压缩算法在客户机上可用。另一方面，服务器为响应的 `Content-Encoding` 头设置值，以便告诉浏览器选择了哪个算法来压缩响应。

**工具**

这里的工具不是 Angular 特有的，完全取决于我们使用的 Web/应用程序服务器。典型的压缩算法有：

- deflate - 一种数据压缩算法和相关的文件格式，它使用 LZ77 算法和霍夫曼编码的组合。
- [brotli](https://github.com/google/brotli) - 一种通用的无损压缩算法，它使用 LZ77 算法的现代变体，霍夫曼编码和二阶上下文建模的压缩比数据压缩数据，其压缩比可与目前最好的通用压缩方法相媲美。速度与 deflate 类似，但提供更密集的压缩。

*注意:* Brotli [还没广泛支持](http://caniuse.com/#search=brotli).

**资料**

- ["Better than Gzip Compression with Brotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### 资源预获取

资源预获取是改善用户体验的好方法。我们可以预先获取资源（图像，样式，打算[懒加载的模块](#资源懒加载)等）或数据。有不同的预取策略，但其中大部分取决于应用程序的细节。

### 资源懒加载

如果目标应用程序具有数百个依赖关系的庞大代码库，上述做法可能无助于我们将捆绑包合理缩小（合理可能为100K或2M，它又完全取决于业务目标）。

在这种情况下，一个好的解决方案可能是懒惰地加载一些应用程序的模块。例如，假设我们正在建立一个电子商务系统。在这种情况下，我们可能希望独立于面向用户的UI加载管理面板。一旦管理员必须添加新产品，我们希望提供所需的用户界面。这可能只是“添加产品页面”或整个管理面板，具体取决于我们的用例/业务需求。

**工具**

- [Webpack](https://github.com/webpack/webpack) - 允许异步模块加载。

### 不要懒加载默认路由

让我们假设我们有以下路由配置：

```ts
// Bad practice
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard',  loadChildren: './dashboard.module#DashboardModule' },
  { path: 'heroes', loadChildren: './heroes.module#HeroesModule' }
];
```

用户第一次使用 url 为 https://example.com/ 打开应用程序时，它们将被重定向到 `/dashboard` ，这将触发具有路径 `dashboard` 的懒加载路由。为了呈现模块的引导组件，Angular 将不得不下载文件 `dashboard.module` 及其所有依赖项。之后，该文件需要由 JavaScript VM 分析并执行。

在初始页面加载过程中触发额外的 HTTP 请求并执行不必要的计算是一种不好的做法，因为它会减慢初始页面的渲染速度。考虑将默认页面路由声明为非懒惰。

### 缓存

缓存是另一种通常的做法，旨在通过利用启发式的方法来加速我们的应用程序，即如果最近有人要求提供某种资源，那么可能会在不久的将来再次提出要求。

为了缓存数据，我们通常使用自定义缓存机制。为了缓存静态资源，我们可以用 [CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache) 来使用标准浏览器缓存或 Service Worker。

### 使用 Application Shell

为了更快地提高应用程序的性能，请使用 [Application Shell](https://developers.google.com/web/updates/2015/11/app-shell)。

应用程序外壳是我们向用户展示的最低用户界面，以表示他们即将交付应用程序。为了动态生成应用程序外壳，您可以使用 Angular Universal 自定义指令，该指令根据使用的渲染平台来显示元素（即在使用 `platform-server` 时隐藏除 App Shell 外的所有内容）。

**工具**

- [Angular Mobile Toolkit](https://github.com/angular/mobile-toolkit) - 旨在自动管理 Service Worder 的进程。它还包含用于缓存静态资源的 Service Worker，以及用于生成[应用程序外壳](#)的 Service Worker。
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - 对 Angular 的同构 JavaScript 支持。

**资料**

- ["Instant Loading Web Apps with an Application Shell Architecture"](https://developers.google.com/web/updates/2015/11/app-shell)

### 使用 Service Workers

我们可以将 Service Worker 视为位于浏览器中的 HTTP 代理。客户端发送的所有请求首先被 Service Worker 拦截，Service Worder 可以直接处理它们或通过网络传递它们。

**工具**

- [Angular Mobile Toolkit](https://github.com/angular/mobile-toolkit) - 旨在自动管理 Service Worder 的进程。它还包含用于缓存静态资源的 Service Worker，以及用于生成[应用程序外壳](#)的 Service Worker。
- [Offline Plugin for Webpack](https://github.com/NekR/offline-plugin) - Webpack插件，可以将 Service Worker 的支持回退到 AppCache 中。

**资料**

- ["The offline cookbook"](https://jakearchibald.com/2014/offline-cookbook/)

## 运行时优化

本部分包含可以应用的实践，以每秒 60 帧（fps）提供更流畅的用户体验。

### 使用 `enableProdMode`

在开发模式中，Angular 会执行一些额外的检查，以验证执行变更检测不会导致对任何绑定进行任何其他更改。这样框架确保了单向数据流已经被遵循。

为了生产环境，需要禁止这些更改，不要忘记调用 `enableProdMode`。

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

### AoT(Ahead-of-Time) 编译

AoT 不仅可以通过执行 tree-shaking 来实现更高效的打包，还可以提高应用程序的运行时性能。 AoT的替代方案是在运行时执行的即时编译（JiT），因此我们可以通过将编译作为构建过程的一部分来减少呈现应用程序所需的计算量。

**工具**

- [@angular/compiler-cli](https://github.com/angular/angular/tree/master/modules/%40angular/compiler-cli) - 代替 [tsc](https://www.npmjs.com/package/typescript) 静态分析我们的应用程序并为组件的模板输出 TypeScript / JavaScript。
- [angular2-seed](https://github.com/mgechev/angular2-seed) - 一个启动项目，其中包括对 AoT 编译的支持。
- [angular-cli](https://cli.angular.io) 使用 `ng serve --prod` 。

**资料**

- ["Ahead-of-Time Compilation in Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Web Workers

典型的单页面应用程序（SPA）中的常见问题是我们的代码通常运行在单个线程中。这意味着如果我们想要以 60fps 的速度实现流畅的用户体验，那么我们最多可以在 16ms 内执行各个帧之间的渲染，否则它们会下降一半。

在具有巨大组件树的复杂应用程序中，变更检测需要每秒执行数百万次检查，因此不会很难开始丢弃帧。感谢 Angular 的平台不可知性（agnosticism），并且它与DOM体系结构解耦，可以在 Web Worker 中运行我们的整个应用程序（包括更改检测），并让主 UI 线程仅负责渲染。

**工具**

- 核心团队支持允许我们在 Web Worker 中运行应用程序的模块。示例如何使用，可以在[这里](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers)找到。
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) - 一个 webpack 的 Web Worker Loader.

**资料**

- ["Using Web Workers for more responsive apps"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

### 服务端渲染

传统 SPA 的一大问题是，只有在初始渲染所需的全部 JavaScript 可用时才能渲染它们。这导致两个大问题：

- 并非所有搜索引擎都运行与页面关联的 JavaScript，因此他们无法正确地为动态应用的内容进行索引。
- 糟糕的用户体验，因为用户只会看到一个空白/加载中的屏幕，直到与页面相关的JavaScript被下载，解析并执行。

服务器端渲染通过在服务端上预先渲染请求的页面并在初始页面加载期间提供渲染页面的标签与样式来解决此问题。

**工具**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - 对 Angular 的同构 JavaScript 支持。
- [Preboot](https://github.com/angular/preboot) - 库帮助管理从服务器生成的Web视图到客户端生成的Web视图的状态转换（即事件，焦点，数据）。

**资料**

- ["Angular Server Rendering"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angular Universal Patterns"](https://www.youtube.com/watch?v=TCj_oC3m6_U)

### 变更检测

在每个异步事件上 Angular 在整个组件树上执行变更检测。虽然检测变化的代码已针对[内联缓存](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html)进行了优化，但在复杂的应用程序中，这仍然是一项沉重的计算。提高变化检测性能的一种方法是不对基于最近操作不应该改变的子树执行它。

#### `ChangeDetectionStrategy.OnPush`

`OnPush` 变更检测策略允许我们禁用组件树的子树的变更检测机制。通过将任何组件的变更检测策略设置为 `ChangeDetectionStrategy.OnPush` 值，将使变更检测**仅仅**在组件接收到不同输入时执行。当 Angular 将它们与之前输入的引用值进行比较结果为 `false`时，Angular 将认为输入不同。结合[不可变的数据结构](https://facebook.github.io/immutable-js/) `OnPush` 可以为这些“纯”组件带来巨大的性能影响。

**资料**

- ["Change Detection in Angular"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)

#### 分离变更检测器

实施自定义变化检测机制的另一种方式是通过分离（`detach`）和重新连接（`reattach`） 给定组件的变更检测器（CD）。一旦我们分离（`detach`）CD，Angular 将不会执行整个组件子树的检查。

当用户操作或与外部服务的交互触发变更检测的次数超过需求时，通常会使用此实践。在这种情况下，我们可能需要考虑分离变化检测器并仅在需要执行变更检测时重新连接。

#### Angular Zone 上下文之外执行

感谢 [zone.js](https://github.com/angular/zone.js) 的功劳，Angular 的变化检测机制得以触发。 Zone.js 猴子补丁在浏览器中修补所有异步 API 在任何异步回调执行结束时触发更改检测。在**极少数情况**下，我们可能希望给出的代码在 Angular Zone 的上下文之外执行，而无需运行更改检测机制。在这种情况下，我们可以使用 `NgZone` 实例的 `runOutsideAngular` 方法。

**例子**

在下面的代码片段中，您可以看到使用此实践的组件示例。当调用 `_incrementPoints` 方法时，组件将每10ms开始递增 `_points` 属性（默认情况下）。增加会出现动画。因为在这种情况下，我们不希望触发整个组件树的更改检测机制（每隔10ms），我们可以在 Angular Zone 的上下文之外运行 `_incrementPoints` 并手动更新DOM（请参阅 `points` 的 setter）。

```ts
@Component({
  template: '<span #label></span>'
})
class PointAnimationComponent {

  @Input() duration = 1000;
  @Input() stepDuration = 10;
  @ViewChild('label') label: ElementRef;

  @Input() set points(val: number) {
    this._points = val;
    if (this.label) {
      this.label.nativeElement.innerText = this._pipe.transform(this.points, '1.0-0');
    }
  }
  get points() {
    return this._points;
  }

  private _incrementInterval: any;
  private _points: number = 0;

  constructor(private _zone: NgZone, private _pipe: DecimalPipe) {}

  ngOnChanges(changes: any) {
    const change = changes.points;
    if (!change) {
      return;
    }
    if (typeof change.previousValue !== 'number') {
      this.points = change.currentValue;
    } else {
      this.points = change.previousValue;
      this._ngZone.runOutsideAngular(() => {
        this._incrementPoints(change.currentValue);
      });
    }
  }

  private _incrementPoints(newVal: number) {
    const diff = newVal - this.points;
    const step = this.stepDuration * (diff / this.duration);
    const initialPoints = this.points;
    this._incrementInterval = setInterval(() => {
      let nextPoints = Math.ceil(initialPoints + diff);
      if (this.points >= nextPoints) {
        this.points = initialPoints + diff;
        clearInterval(this._incrementInterval);
      } else {
        this.points += step;
      }
    }, this.stepDuration);
  }
}
```

**警告**：**仅在当你确定你在做什么的时候时**，**非常谨慎地**使用这个实践。因为如果使用不当，可能会导致 DOM 的不一致状态。还要注意，上面的代码不会在 WebWorkers 中运行。为了使其与 WebWorker 兼容，您需要使用 Angular 的渲染器来设置标签的值。

### 使用纯管道

`@Pipe` 装饰器接受具有以下格式的对象作为参数：

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

`pure` 标识表示管道不依赖于任何全局状态，也不产生副作用。这意味着当使用相同的输入调用时，管道将返回相同的输出。通过这种方式，Angular可以缓存管道被调用的所有输入参数的输出，并重用它们以便不必在每次评估时重新计算它们。

`pure` 属性的默认值为 `true`。

### 为 `*ngFor` 指令使用 `trackBy` 选项

`*ngFor` 指令用于呈现集合。默认情况下，`*ngFor` 通过引用标识对象的唯一性。

这意味着开发人员在更新项目内容期间更改了对象的引用时，Angular 会将其视为删除旧对象并添加新对象。这会破坏列表中的旧 DOM 节点并在其位置添加新的 DOM 节点。

开发人员可以为 Angular 提供如何识别对象唯一性的提示：自定义跟踪函数作为 `*ngFor` 指令的 `trackBy` 选项。跟踪功能有两个参数：`index` 和 `item`。 Angular使用跟踪函数返回的值来跟踪对象的标识。使用特定记录的 ID 作为唯一 key 是很常见的。

**例子**

```typescript
@Component({
  selector: 'yt-feed',
  template: `
  <h1>Your video feed</h1>
  <yt-player *ngFor="let video of feed; trackBy: trackById" [video]="video"></yt-player>
`
})
export class YtFeedComponent {
  feed = [
    {
      id: 3849, // note "id" field, we refer to it in "trackById" function
      title: "Angular in 60 minutes",
      url: "http://youtube.com/ng2-in-60-min",
      likes: "29345"
    },
    // ...
  ];

  trackById(index, item) {
    return item.id;
  }
}
```
**资料**

- ["NgFor directive"](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) - `*ngFor` 官方文档介绍
- ["Angular — Improve performance with trackBy"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - 显示了该方法的 gif 演示

# 结论

随着时间的推移，实践清单将随着新的/更新的实践而动态演变。如果您发现某些缺失或您认为任何做法都可以改进，请不要犹豫，创建 ISSUE 和/或创建 PR 。欲了解更多信息，请看下面的“[贡献](https://github.com/mgechev/angular-performance-checklist#contributing)”部分。

# 贡献

如果您发现某些遗漏，不完整或不正确的情况，我们将非常感谢您提出申请。对于未包含在文档中的实践讨论，请[打开一个问题](https://github.com/mgechev/angular2-performance-checklist/issues)。

# License

MIT

