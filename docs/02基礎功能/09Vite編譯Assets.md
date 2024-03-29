# Vite 編譯 Assets

## 介紹

[Vite](https://vitejs.dev/) 是一款現代前端建構工具，提供極快的開發環境並將你的程式碼捆綁到生產準備的資源中。在使用 Laravel 建構應用程式時，通常會使用 Vite 將你的應用程式的 CSS 和 JavaScript 檔案繫結到生產環境的資源中。

Laravel 通過提供官方外掛和 Blade 指令，與 Vite 完美整合，以載入你的資源進行開發和生產。

>注意：你正在運行 Laravel Mix 嗎？在新的 Laravel 安裝中，Vite 已經取代了 Laravel Mix 。有關 Mix 的文件，請訪問 [Laravel Mix](https://laravel-mix.com/) 網站。如果你想切換到 Vite，請參閱我們的 [遷移指南](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite)。

#### 選擇 Vite 還是 Laravel Mix

在轉向 Vite 之前，新的 Laravel 應用程式在打包資產時通常使用 [Mix](https://laravel-mix.com/)，它由 [webpack](https://webpack.js.org/) 支援。Vite 專注於在建構豐富的 JavaScript 應用程式時提供更快、更高效的開發體驗。如果你正在開發單頁面應用程式（SPA），包括使用 [Inertia](https://inertiajs.com/) 工具開發的應用程式，則 Vite 是完美選擇。

Vite 也適用於具有 JavaScript “sprinkles” 的傳統伺服器端渲染應用程式，包括使用 [Livewire](https://laravel-livewire.com/) 的應用程式。但是，它缺少 Laravel Mix 支援的某些功能，例如將沒有直接在 JavaScript 應用程式中引用的任意資產複製到建構中的能力。

#### 切換回 Mix

如果你使用我們的 Vite 腳手架建立了一個新的 Laravel 應用程式，但需要切換回 Laravel Mix 和 webpack，那麼也沒有問題。請參閱我們的[從 Vite 切換到 Mix 的官方指南](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-vite-to-laravel-mix)。

## 安裝和設定

> **注意**
> 以下文件討論如何手動安裝和組態 Laravel Vite 外掛。但是，Laravel 的[起始套件](/docs/laravel/10.x/starter-kits)已經包含了所有的腳手架，並且是使用 Laravel 和 Vite 開始最快的方式。

### 安裝 Node

在運行 Vite 和 Laravel 外掛之前，你必須確保已安裝 Node.js（16+）和 NPM：


`node -v
npm -v` 

你可以通過[官方 Node 網站](https://nodejs.org/en/download/)的簡單圖形安裝程序輕鬆安裝最新版本的 Node 和 NPM。或者，如果你使用的是 [Laravel Sail](/docs/laravel/10.x/sail)，可以通過 Sail 呼叫 Node 和 NPM：


`./vendor/bin/sail node -v
./vendor/bin/sail npm -v`

### 安裝 Vite 和 Laravel 外掛

在 Laravel 的全新安裝中，你會在應用程式目錄結構的根目錄下找到一個 package.json 檔案。默認的 package.json 檔案已經包含了你開始使用 Vite 和 Laravel 外掛所需的一切。你可以通過 NPM 安裝應用程式的前端依賴：

```sh
npm install
```

### 組態 Vite

Vite 通過項目根目錄中的 `vite.config.js` 檔案進行組態。你可以根據自己的需要自訂此檔案，也可以安裝任何其他外掛，例如 `@vitejs/plugin-vue` 或 `@vitejs/plugin-react`。

Laravel Vite 外掛需要你指定應用程式的入口點。這些入口點可以是 JavaScript 或 CSS 檔案，并包括預處理語言，例如 TypeScript、JSX、TSX 和 Sass。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

如果你正在建構一個單頁應用程式，包括使用 Inertia 建構的應用程式，則最好不要使用 CSS 入口點：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css', // [tl! remove]
            'resources/js/app.js',
        ]),
    ],
});
```

相反，你應該通過 JavaScript 匯入你的 CSS。通常，這將在應用程式的 resources/js/app.js 檔案中完成：

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

Laravel 外掛還支援多個入口點和高級組態選項，例如[SSR 入口點](#ssr)。



#### 使用安全的開發伺服器

如果你的本地開發 Web 伺服器通過 HTTPS 提供應用程式服務，則可能會遇到連接到 Vite 開發伺服器的問題。

如果你在本地開發中使用 [Laravel Valet](/docs/laravel/10.x/valet) 並已針對你的應用程式運行 [secure 命令](/docs/laravel/10.x/valetmd#securing-sites)，則可以組態 Vite 開發伺服器自動使用 Valet 生成的 TLS 證書：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            valetTls: 'my-app.test', // [tl! add]
        }),
    ],
});
```
當使用其他 Web 伺服器時，你應生成一個受信任的證書並手動組態 Vite 使用生成的證書：

```js
// ...
import fs from 'fs'; // [tl! add]

const host = 'my-app.test'; // [tl! add]

export default defineConfig({
    // ...
    server: { // [tl! add]
        host, // [tl! add]
        hmr: { host }, // [tl! add]
        https: { // [tl! add]
            key: fs.readFileSync(`/path/to/${host}.key`), // [tl! add]
            cert: fs.readFileSync(`/path/to/${host}.crt`), // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

如果你無法為系統生成可信證書，則可以安裝並組態 [`@vitejs/plugin-basic-ssl` 外掛](https://github.com/vitejs/vite-plugin-basic-ssl)。使用不受信任的證書時，你需要通過在運行 `npm run dev` 命令時按照控制台中的“Local”連結接受 Vite 開發伺服器的證書警告。


### 載入你的指令碼和樣式

組態了 Vite 入口點後，你只需要在應用程式根範本的 `<head>` 中新增一個 `@vite()` Blade 指令引用它們即可：

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

如果你通過 JavaScript 匯入你的 CSS 檔案，你只需要包含 JavaScript 的入口點：

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

`@vite` 指令會自動檢測 Vite 開發伺服器並注入 Vite 客戶端以啟用熱模組替換。在建構模式下，該指令將載入已編譯和版本化的資產，包括任何匯入的 CSS 檔案。

如果需要，在呼叫 `@vite` 指令時，你還可以指定已編譯資產的建構路徑：

```blade
<!doctype html>
<head>
    {{-- Given build path is relative to public path. --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

## 運行 Vite

你可以通過兩種方式運行 Vite。你可以通過 `dev` 命令運行開發伺服器，在本地開發時非常有用。開發伺服器會自動檢測檔案的更改，並立即在任何打開的瀏覽器窗口中反映這些更改。

或者，運行 `build` 命令將版本化並打包應用程式的資產，並準備好部署到生產環境：

Or, running the `build` command will version and bundle your application's assets and get them ready for you to deploy to production:

```shell
# Run the Vite development server...
npm run dev

# Build and version the assets for production...
npm run build
```

## 使用 JavaScript

### 別名

默認情況下，Laravel 外掛提供一個常用的別名，以幫助你快速開始並方便地匯入你的應用程式的資產：

```js
{
    '@' => '/resources/js'
}
```

你可以通過新增自己的別名到 `vite.config.js` 組態檔案中，覆蓋 `'@'` 別名：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```



### Vue

如果你想要使用 [Vue](https://vuejs.org/) 框架建構前端，那麼你還需要安裝 `@vitejs/plugin-vue` 外掛：

```sh
npm install --save-dev @vitejs/plugin-vue
```

然後你可以在 `vite.config.js` 組態檔案中包含該外掛。當使用 Laravel 和 Vue 外掛時，還需要一些附加選項：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // Vue 外掛會重新編寫資產 URL，以便在單檔案元件中引用時，指向 Laravel web 伺服器。
                    // 將其設定為 `null`，則 Laravel 外掛會將資產 URL 重新編寫為指向 Vite 伺服器。
                    base: null,

                    // Vue 外掛將解析絕對 URL 並將其視為磁碟上檔案的絕對路徑。
                    // 將其設定為 `false`，將保留絕對 URL 不變，以便可以像預期那樣引用公共目錄中的資源。
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

**注意**
Laravel 的 [起步套件](/docs/laravel/10.x/starter-kits) 已經包含了適當的 Laravel、Vue 和 Vite 組態。請查看 [Laravel Breeze](/docs/laravel/10.x/starter-kitsmd#breeze-and-inertia) 以瞭解使用 Laravel、Vue 和 Vite 快速入門的最快方法。

### React

如果你想要使用 [React](https://reactjs.org/) 框架建構前端，那麼你還需要安裝 `@vitejs/plugin-react` 外掛：

```sh
npm install --save-dev @vitejs/plugin-react
```

你可以在 `vite.config.js` 組態檔案中包含該外掛：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

當使用 Vite 和 React 時，你將需要確保任何包含 JSX 的檔案都有一個 .jsx 和 .tsx 擴展，記住更新入口檔案，如果需要 [如上所示](#configuring-vite)。你還需要在現有的 `@vite` 指令旁邊包含額外的 `@viteReactRefresh` Blade 指令。

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

`@viteReactRefresh` 指令必須在 `@vite` 指令之前呼叫 。

> **注意**  
> Laravel 的 [起步套件](/docs/laravel/10.x/starter-kits) 已經包含了適合的 Laravel、React 和 Vite 組態。查看 [Laravel Breeze](/docs/laravel/10.x/starter-kitsmd#breeze-and-inertia) 以最快的方式開始學習 Laravel、React 和 Vite。

### Inertia

Laravel Vite 外掛提供了一個方便的 `resolvePageComponent` 函數，幫助你解決 Inertia 頁面元件。以下是使用 Vue 3 的助手示例；然而，你也可以在其他框架中使用該函數，例如 React：

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    return createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

> **注意**  
> Laravel 的 [起步套件](/docs/laravel/10.x/starter-kits) 已經包含了適合的 Laravel、Inertia 和 Vite 組態。查看 [Laravel Breeze](/docs/laravel/10.x/starter-kitsmd#breeze-and-inertia) 以最快的方式開始學習 Laravel、Inertia 和 Vite。

### URL 處理

當使用 Vite 並在你的應用程式的 HTML，CSS 和 JS 中引用資源時，有幾件事情需要考慮。首先，如果你使用絕對路徑引用資源，Vite 將不會在建構中包含資源；因此，你需要確認資源在你的公共目錄中是可用的。

在引用相對路徑的資源時，你應該記住這些路徑是相對於它們被引用的檔案的路徑。通過相對路徑引用的所有資源都將被 Vite 重寫、版本化和打包。

參考以下項目結構：

```
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
``` 

以下示例演示了 Vite 如何處理相對路徑和絕對 URL：

```
<!-- 這個資源不被 Vite 處理，不會被包含在建構中 -->
<img src="/taylor.png">

<!-- 這個資源將由 Vite 重寫、版本化和打包 -->
<img src="../../images/abigail.png">` 
```

## 使用樣式表

你可以在 [Vite 文件](https://vitejs.dev/guide/features.html#css) 中瞭解有關 Vite 的 CSS 支援更多的資訊。如果你使用 PostCSS 外掛，如 [Tailwind](https://tailwindcss.com/)，你可以在項目的根目錄中建立一個 `postcss.config.js` 檔案，Vite 會自動應用它：

```
module.exports = {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```
> **注意** Laravel 的 [起始套件](/docs/laravel/10.x/starter-kits) 已經包含了正確的 Tailwind、PostCSS 和 Vite 組態。或者，如果你想在不使用我們的起始套件的情況下使用 Tailwind 和 Laravel，請查看 [Tailwind 的 Laravel 安裝指南](https://tailwindcss.com/docs/guides/laravel)。

## 使用 Blade 和 路由

### 通過 Vite 處理靜態資源

在你的 JavaScript 或 CSS 中引用資源時，Vite 會自動處理和版本化它們。此外，在建構基於 Blade 的應用程式時，Vite 還可以處理和版本化你僅在 Blade 範本中引用的靜態資源。

然而，要實現這一點，你需要通過將靜態資源匯入到應用程式的入口點來讓 Vite 瞭解你的資源。例如，如果你想處理和版本化儲存在 `resources/images` 中的所有圖像和儲存在 `resources/fonts` 中的所有字型，你應該在應用程式的 `resources/js/app.js` 入口點中新增以下內容：

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

這些資源將在運行 `npm run build` 時由 Vite 處理。然後，你可以在 Blade 範本中使用 `Vite::asset` 方法引用這些資源，該方法將返回給定資源的版本化 URL：

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

### 保存刷新

當你的應用程式使用傳統的伺服器端渲染 Blade 建構時，Vite 可以通過在你的應用程式中更改檢視檔案時自動刷新瀏覽器來提高你的開發工作流程。要開始，你可以簡單地將 `refresh` 選項指定為 `true`。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: true,
        }),
    ],
});
```

當 `refresh` 選項為 `true` 時，保存以下目錄中的檔案將在你運行 `npm run dev` 時觸發瀏覽器進行全面的頁面刷新：

- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

監聽 `routes/**` 目錄對於在應用程式前端中利用 [Ziggy](https://github.com/tighten/ziggy) 生成路由連結非常有用。

如果這些默認路徑不符合你的需求，你可以指定自己的路徑列表：

```
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: ['resources/views/**'],
        }),
    ],
});
``` 

在後台，Laravel Vite 外掛使用了 [`vite-plugin-full-reload`](https://github.com/ElMassimo/vite-plugin-full-reload) 包，該包提供了一些高級組態選項，可微調此功能的行為。如果你需要這種等級的自訂，可以提供一個 `config` 定義：

```
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: [{
                paths: ['path/to/watch/**'],
                config: { delay: 300 }
            }],
        }),
    ],
});
``` 

### 別名

在 JavaScript 應用程式中建立別名來引用常用目錄是很常見的。但是，你也可以通過在 `Illuminate\Support\Facades\Vite` 類上使用 `macro` 方法來建立在 Blade 中使用的別名。通常，“宏”應在 [服務提供商](/docs/laravel/10.x/providers) 的 `boot` 方法中定義：

```
/**
 * Bootstrap any application services.
 */
public function boot(): void {
    Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
}
``` 

定義了宏之後，可以在範本中呼叫它。例如，我們可以使用上面定義的 `image` 宏來引用位於 `resources/images/logo.png` 的資源：

```
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
``` 

### 自訂 base URL

如果你的 Vite 編譯的資產部署到與應用程式不同的域（例如通過 CDN），必須在應用程式的 `.env` 檔案中指定 `ASSET_URL` 環境變數：

```
ASSET_URL=https://cdn.example.com
```

在組態了資源 URL 之後，所有重寫的 URL 都將以組態的值為前綴：

```
https://cdn.example.com/build/assets/app.9dce8d17.js
``` 

請記住，[絕對路徑的 URL 不會被 Vite 重新編寫](/#url-processing)，因此它們不會被新增前綴。

### 環境變數

你可以在應用程式的 `.env` 檔案中以 `VITE_` 為前綴注入環境變數以在 JavaScript 中使用：

```
VITE_SENTRY_DSN_PUBLIC=http://example.com
``` 

你可以通過 `import.meta.env` 對象訪問注入的環境變數：

```
import.meta.env.VITE_SENTRY_DSN_PUBLIC
``` 

### 在測試中停用 Vite

Laravel 的 Vite 整合將在運行測試時嘗試解析你的資產，這需要你運行 Vite 開發伺服器或建構你的資產。

如果你希望在測試中模擬 Vite，你可以呼叫 `withoutVite` 方法，該方法對任何擴展 Laravel 的 `TestCase` 類的測試都可用：

```
use Tests\TestCase;

class ExampleTest extends TestCase {
    public function test_without_vite_example(): void {
        $this->withoutVite();

        // ...
    }
}
``` 

如果你想在所有測試中停用 Vite，可以在基本的 `TestCase` 類上的 `setUp` 方法中呼叫 `withoutVite` 方法：

```
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase {
    use CreatesApplication;

    protected function setUp(): void// [tl! add:start] {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
``` 

### 伺服器端渲染

Laravel Vite 外掛可以輕鬆地設定與 Vite 的伺服器端渲染。要開始使用，請在 `resources/js` 中建立一個 SSR（Server-Side Rendering）入口點，並通過將組態選項傳遞給 Laravel 外掛來指定入口點：

```
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

為確保不遺漏重建 SSR 入口點，我們建議增加應用程式的 `package.json` 中的 「build」 指令碼來建立 SSR 建構：

```
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
``` 

然後，要建構和啟動 SSR 伺服器，你可以運行以下命令：

```
npm run build
node bootstrap/ssr/ssr.mjs
``` 

> **注意** Laravel 的 [starter kits](/docs/laravel/10.x/starter-kits) 已經包含了適當的 Laravel、Inertia SSR 和 Vite 組態。查看 [Laravel Breeze](/docs/laravel/10.x/starter-kitsmd#breeze-and-inertia) ，以獲得使用 Laravel、Inertia SSR 和 Vite 的最快速的方法。

## Script & Style 標籤的屬性

### Content Security Policy (CSP) Nonce

如果你希望在你的指令碼和樣式標籤中包含 [`nonce` 屬性](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)，作為你的 [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) 的一部分，你可以使用自訂 [middleware](/docs/laravel/10.x/middleware) 中的 `useCspNonce` 方法生成或指定一個 nonce：

Copy code

```
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Vite;
use Symfony\Component\HttpFoundation\Response;

class AddContentSecurityPolicyHeaders {
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response {
        Vite::useCspNonce();

        return $next($request)->withHeaders([
            'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
        ]);
    }
}
``` 

呼叫了 `useCspNonce` 方法後，Laravel 將自動在所有生成的指令碼和樣式標籤上包含 `nonce` 屬性。

如果你需要在其他地方指定 nonce，包括 Laravel 的 starter kits 中帶有的 [Ziggy `@route` directive](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy) 指令，你可以使用 `cspNonce` 方法來檢索它：

```
@routes(nonce: Vite::cspNonce())
```
如果你已經有了一個 nonce，想要告訴 Laravel 使用它，你可以通過將 nonce 傳遞給 `useCspNonce` 方法來實現：

```
Vite::useCspNonce($nonce);
```
###子資源完整性 (SRI)
如果你的 `Vite manifest` 中包括資源的完整性雜湊，則 Laravel 將自動向其生成的任何指令碼和樣式標籤中新增 `integrity` 屬性，以執行 子資源完整性。默認情況下，Vite 不包括其清單中的 `integrity` 雜湊，但是你可以通過安裝 `vite-plugin-manifest-sri` NPM 外掛來啟用它：

```shell
npm install --save-dev vite-plugin-manifest-sri
```
然後，在你的 `vite.config.js` 檔案中啟用此外掛：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import manifestSRI from 'vite-plugin-manifest-sri';// [tl! add]

export default defineConfig({
    plugins: [
        laravel({
            // ...
        }),
        manifestSRI(),// [tl! add]
    ],
});
```
如果需要，你也可以自訂清單中的完整性雜湊鍵：

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

如果你想完全停用這個自動檢測，你可以將 `false` 傳遞給 `useIntegrityKey` 方法：

```
Vite::useIntegrityKey(false);
```
### 任意屬性
如果你需要在指令碼和樣式標籤中包含其他屬性，例如 `data-turbo-track` 屬性，你可以通過 `useScriptTagAttributes` 和 `useStyleTagAttributes` 方法指定它們。通常，這些方法應從一個服務提供程序中呼叫：

```
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // 為屬性指定一個值...
    'async' => true, // 在不使用值的情況下指定屬性...
    'integrity' => false, // 排除一個將被包含的屬性...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

如果你需要有條件地新增屬性，你可以傳遞一個回呼函數，它將接收到資產源路徑、它的URL、它的清單塊和整個清單：

```
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
``` 

> **警告**
> 在 Vite 開發伺服器執行階段，`$chunk` 和 `$manifest` 參數將為 `null`。


## 高級定製

默認情況下，Laravel 的 Vite 外掛使用合理的約定，適用於大多數應用，但是有時你可能需要自訂 Vite 的行為。為了啟用額外的自訂選項，我們提供了以下方法和選項，可以用於替代 `@vite` Blade 指令：

```
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // 自訂 "hot" 檔案...
            ->useBuildDirectory('bundle') // 自訂建構目錄...
            ->useManifestFilename('assets.json') // 自訂清單檔案名稱...
            ->withEntryPoints(['resources/js/app.js']) // 指定入口點...
    }}
</head>
``` 

然後，在 `vite.config.js` 檔案中，你應該指定相同的組態：

```
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // 自訂 "hot" 檔案...
            buildDirectory: 'bundle', // 自訂建構目錄...
            input: ['resources/js/app.js'], // 指定入口點...
        }),
    ],
    build: {
      manifest: 'assets.json', // 自訂清單檔案名稱...
    },
});
``` 


### 修正開發伺服器 URL

Vite 生態系統中的某些外掛默認假設以正斜槓開頭的 URL 始終指向 Vite 開發伺服器。然而，由於 Laravel 整合的性質，實際情況並非如此。

例如，`vite-imagetools` 外掛在 Vite 服務時，你的資產時會輸出以下類似的 URL：

```
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
``` 

`vite-imagetools` 外掛期望輸出URL將被 Vite 攔截，並且外掛可以處理所有以 `/@imagetools` 開頭的 URL。如果你正在使用期望此行為的外掛，則需要手動糾正 URL。你可以在 `vite.config.js` 檔案中使用 `transformOnServe` 選項來實現。

在這個例子中，我們將在生成的程式碼中的所有 `/@imagetools` 錢加上開發伺服器的 URL：

```
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            transformOnServe: (code, devServerUrl) => code.replaceAll('/@imagetools', devServerUrl+'/@imagetools'),
        }),
        imagetools(),
    ],
});
``` 

現在，在 Vite 提供資產服務時，它會輸出URL指向 Vite 開發伺服器：

```
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```
