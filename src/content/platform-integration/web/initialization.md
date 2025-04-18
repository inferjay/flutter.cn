---
# title: Flutter web app initialization
title: Flutter web 应用的初始化
# description: Customize how Flutter apps are initialized on the web.
description: 自定义 Flutter 应用在 web 上的初始化方式。
---

This page details the initialization process for Flutter web apps and
how it can be customized.

## Bootstrapping

The `flutter build web` command produces
a script called `flutter_bootstrap.js` in
the build output directory (`build/web`).
This file contains the JavaScript code needed to initialize and
run your Flutter app.
You can use this script by placing an async-script tag for it in
your `index.html` file in the `web` subdirectory of your Flutter app:

```html highlightLines=3
<html>
  <body>
    <script src="flutter_bootstrap.js" async></script>
  </body>
</html>
```

Alternatively, you can inline the entire contents of
the `flutter_bootstrap.js` file by inserting the
template token `{% raw %}{{flutter_bootstrap_js}}{% endraw %}` in
your `index.html` file:

```html highlightLines=4
<html>
  <body>
    <script>
      {% raw %}{{flutter_bootstrap_js}}{% endraw %}
    </script>
  </body>
</html>
```

The `{% raw %}{{flutter_bootstrap_js}}{% endraw %}` token is
replaced with the contents of the `flutter_bootstrap.js` file when
the `index.html` file is copied to the
output directory (`build/web`) during the build step.

<a id="customizing-initialization" aria-hidden="true"></a>

## Customize initialization

## 自定义初始化

By default, `flutter build web` generates a `flutter_bootstrap.js` file that
does a simple initialization of your Flutter app.
However, in some scenarios, you might have a reason to
customize this initialization process, such as:

* Setting a custom Flutter configuration for your app.
* Changing the settings for the Flutter service worker.
* Writing custom JavaScript code to
  run at different stages of the startup process.

To write your own custom bootstrapping logic instead of
using the default script produced by the build step, you can
place a `flutter_bootstrap.js` file in the `web` subdirectory of your project,
which is copied over and used instead of
the default script produced by the build.
This file is also templated, and you can insert several special tokens that
the build step substitutes at build time when copying
the `flutter_bootstrap.js` file to the output directory.
The following table lists the tokens that the build step will
substitute in either the `flutter_bootstrap.js` or `index.html` files:

| Token | Replaced with |
|---|---|
| `{% raw %}{{flutter_js}}{% endraw %}` | The JavaScript code that makes the `FlutterLoader` object available in the `_flutter.loader` global variable. (See the `_flutter.loader.load() API` section below for more details.) |
| `{% raw %}{{flutter_build_config}}{% endraw %}` | A JavaScript statement that sets metadata produced by the build process which gives the `FlutterLoader` information needed to properly bootstrap your application. |
| `{% raw %}{{flutter_service_worker_version}}{% endraw %}` | A unique number representing the build version of the service worker, which can be passed as part of the service worker configuration (see the "Service Worker Settings" table below). |
| `{% raw %}{{flutter_bootstrap_js}}{% endraw %}` | As mentioned above, this inlines the contents of the `flutter_bootstrap.js` file directly into the `index.html` file. Note that this token can only be used in the `index.html` and not the `flutter_bootstrap.js` file itself. |

{:.table}

<a id="write-a-custom-flutter_bootstrap-js" aria-hidden="true"></a>

## Write a custom bootstrap script {:#custom-bootstrap-js}

## 编写自定义的启动脚本

Any custom `flutter_bootstrap.js` script needs to have three components in
order to successfully start your Flutter app:

* A `{% raw %}{{flutter_js}}{% endraw %}` token,
  to make `_flutter.loader` available.
* A `{% raw %}{{flutter_build_config}}{% endraw %}` token,
  which provides information about the build to the
  `FlutterLoader` needed to start your app.
* A call to `_flutter.loader.load()`, which actually starts the app.

The most basic `flutter_bootstrap.js` file would look something like this:

```js
{% raw %}{{flutter_js}}{% endraw %}
{% raw %}{{flutter_build_config}}{% endraw %}

_flutter.loader.load();
```

## Customize the Flutter Loader

## 自定义 Flutter Loader

The `_flutter.loader.load()` JavaScript API can be invoked with optional
arguments to customize initialization behavior:

可以使用以下可选参数调用 `_flutter.loader.load()` JavaScript API，
来自定义初始化行为：

| <t>Name</t><t>参数名称</t> | <t>Description</t><t>描述</t> | <t>JS&nbsp;type</t><t>JS&nbsp;类型</t> |
|-------------------------|-------------------------------------------------------------------------------------------------------------------------------|--------------|
| `config`                | The Flutter configuration of your app.                                                                                        | `Object`     |
| `config`                | 应用程序的 Flutter 配置 | `Object` |
| `onEntrypointLoaded`    | The function called when the engine is ready to be initialized. Receives an `engineInitializer` object as its only parameter. | `Function`   |
| `onEntrypointLoaded` | 当引擎准备初始化时调用的函数。该函数的唯一参数是一个 `engineInitializer` 对象。  | `Function` |

{:.table}

The `config` argument is an object that can have the following optional fields:

`config` 是一个对象，你可以添加以下任何可选参数：

| <t>Name</t><t>参数名称</t> | <t>Description</t><t>描述</t> | <t>Dart&nbsp;type</t><t>Dart&nbsp;类型</t> |
|---|---|---|
|`assetBase`| The base URL of the `assets` directory of the app. Add this when Flutter loads from a different domain or subdirectory than the actual web app. You might need this when you embed Flutter web into another app, or when you deploy its assets to a CDN. |`String`|
|`assetBase`| 应用程序 `assets` 目录的根 URL。当 Flutter 从实际 web 应用不同的域或子目录加载时，请添加此 URL。当你将 Flutter web 嵌入另一个应用程序，或将其静态资源部署到 CDN 时，可能需要使用此 URL。 |`String`|
|`canvasKitBaseUrl`| The base URL from where `canvaskit.wasm` is downloaded. |`String`|
|`canvasKitBaseUrl`| 下载 `canvaskit.wasm` 的根 URL。 |`String`|
|`canvasKitVariant`| The CanvasKit variant to download. Your options cover:<br><br>1. `auto`: Downloads the optimal variant for the browser. The option defaults to this value.<br>2. `full`: Downloads the full variant of CanvasKit that works in all browsers.<br>3. `chromium`: Downloads a smaller variant of CanvasKit that uses Chromium compatible APIs. **_Warning_**: Don't use the `chromium` option unless you plan on only using Chromium-based browsers. |`String`|
|`canvasKitVariant`| 下载 CanvasKit。你的可选值：<br><br>1. `auto`：为浏览器下载最佳版本。（默认值）<br>2. `full`：下载适用于所有浏览器的 CanvasKit 完整版。<br>3. `chromium`：下载较小的 CanvasKit，该版本使用与 Chromium 兼容的 API。**_警告_**：除非你只打算使用基于 Chromium 的浏览器，否则不要使用 `chromium` 值。 |`String`|
|`canvasKitForceCpuOnly`| When `true`, forces CPU-only rendering in CanvasKit (the engine won't use WebGL). |`bool`|
|`canvasKitForceCpuOnly`| 为 `true` 时，强制在 CanvasKit 中只使用 CPU 进行渲染（引擎不会使用 WebGL）。 |`bool`|
|`canvasKitMaximumSurfaces`| The maximum number of overlay surfaces that the CanvasKit renderer can use. |`double`|
|`canvasKitMaximumSurfaces`| CanvasKit 渲染器可使用的最大覆盖层数。 |`double`|
|`debugShowSemanticNodes`| If `true`, Flutter visibly renders the semantics tree onscreen (for debugging).  |`bool`|
|`debugShowSemanticNodes`| 如果为 `true`，Flutter 会在屏幕上明显呈现 semantics 语义树（用于调试）。 |`bool`|
|`entryPointBaseUrl`| The base URL of your Flutter app's entrypoint. Defaults to "/".  |`String`|
|`entryPointBaseUrl`| Flutter 应用入口的根 URL。默认为 "/"。  |`String`|
|`hostElement`| HTML Element into which Flutter renders the app. When not set, Flutter web takes over the whole page. |`HtmlElement`|
|`hostElement`| 用于 Flutter 渲染应用程序的 HTML 元素。未设置时，Flutter web 会占据整个页面。 |`HtmlElement`|
|`renderer`| Specifies the [web renderer][web-renderers] for the current Flutter application, either `"canvaskit"` or `"skwasm"`. |`String`|
|`renderer`| 指定当前 Flutter 应用程序的 [web 渲染器][web-renderers]，可选 `"canvaskit"` 或 `"skwasm"`。 |`String`|

{:.table}

[web-renderers]: /platform-integration/web/renderers

## Example: Customizing Flutter configuration based on URL query parameters

## 示例：根据 URL 查询参数自定义 Flutter 配置

The following example shows a custom `flutter_bootstrap.js` that allows
the user to select a renderer by providing a `renderer` query parameter,
e.g. `?renderer=skwasm`, in the URL of their website:

下面的示例演示了一个自定义的 `flutter_bootstrap.js`，
他允许用户在网站 URL 中提供一个 `?force_canvaskit=true` 的查询参数，
从而强制应用使用 `CanvasKit` 渲染器：

```js
{% raw %}{{flutter_js}}{% endraw %}
{% raw %}{{flutter_build_config}}{% endraw %}

const searchParams = new URLSearchParams(window.location.search);
const renderer = searchParams.get('renderer');
const userConfig = renderer ? {'renderer': renderer} : {};
_flutter.loader.load({
  config: userConfig,
});
```

This script evaluates the `URLSearchParams` of the page to determine whether
the user passed a `renderer` query parameter and then
changes the user configuration of the Flutter app.

## The onEntrypointLoaded callback

You can also pass an `onEntrypointLoaded` callback into the `load` API in order
to perform custom logic at different parts of the initialization process.
The initialization process is split into the following stages:

**Loading the entrypoint script**
: The `load` function calls the `onEntrypointLoaded` callback once the
  Service Worker is initialized, and the `main.dart.js` entrypoint has
  been downloaded and run by the browser.
  Flutter also calls `onEntrypointLoaded` on
  every hot restart during development.

**Initializing the Flutter engine**
: The `onEntrypointLoaded` callback receives an
  **engine initializer** object as its only parameter.
  Use the engine initializer `initializeEngine()` function to
  set the run-time configuration, like `multiViewEnabled: true`,
  and start the Flutter web engine.

**Running the app**
: The `initializeEngine()` function returns a [`Promise`][js-promise]
  that resolves with an **app runner** object. The app runner has a
  single method, `runApp()`, that runs the Flutter app.

**Adding views to (or removing views from) an app**
: The `runApp()` method returns a **flutter app** object.
  In multi-view mode, the `addView` and `removeView`
  methods can be used to manage app views from the host app.
  To learn more, check out [Embedded mode][embedded-mode].

[embedded-mode]: {{site.docs}}/platform-integration/web/embedding-flutter-web/#embedded-mode
[js-promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise

## Example: Display a progress indicator

## 示例：显示进度指示器

To give the user of your application feedback
during the initialization process,
use the hooks provided for each stage to update the DOM:

为了在应用程序初始化过程中向用户提供反馈，
请使用各个阶段提供的钩子来更新 DOM：

```js
{% raw %}{{flutter_js}}{% endraw %}
{% raw %}{{flutter_build_config}}{% endraw %}

const loading = document.createElement('div');
document.body.appendChild(loading);
loading.textContent = "Loading Entrypoint...";
_flutter.loader.load({
  onEntrypointLoaded: async function(engineInitializer) {
    loading.textContent = "Initializing engine...";
    const appRunner = await engineInitializer.initializeEngine();

    loading.textContent = "Running app...";
    await appRunner.runApp();
  }
});
```
