# 进阶配置

以下是一些非常规场景下的配置。

## behavior <Badge type="tip" text="0.7.0+" vertical="middle" />

- 可选项。
- 类型：
  ```ts
  type Behavior = {
    /*
     * 是否启用点击跳转 IDE 定位代码的功能(默认为 true)
     */
    locate?: boolean;
    /*
     * 是否启动点击复制源代码位置信息(默认为 true)
     * 也可以设置字符串并通过 {file}、{line}、{column} 模版去指定要复制的信息格式
     * 默认值 true 相当于 "{file}:{line}:{column}" 的字符串格式
     */
    copy?: boolean | string;
    /**
     * 点击元素时触发跳转指定 url。
     * 字符串类型，可通过 {file}、{line}、{column} 模版代替源码位置信息，跳转前会将模板替换为对应的值
     */
    target?: string;
  };
  ```
- 说明：在某些场景下，如果你在点击元素时不需要定位代码，仅需要复制元素的源码位置信息，则可以设置 `locate: false` 和 `copy: true`，此时点击元素仅会复制源码位置信息。

除了上述的行为之外，`code-inspector-plugin` 在点击元素时会触发一个 `code-inspector:trackCode` 的自定义事件，你可以通过此事件来自定义想要的功能（该功能在 `1.2.0+` 的版本中支持）。例如，在点击元素时，你想统计日志，则可以通过如下方式实现：

```ts
window.addEventListener('code-inspector:trackCode', () => {
  sendLog('trackCode');
});
```

## ip <Badge type="tip" text="0.13.0+" vertical="middle" />

- 可选项
- 类型：`boolean | string`。默认值 `false`
- 说明：是否通过 ip 的方式向 node server 发送请求。默认通过 `localhost` 发送请求；设置为 `true` 时，会自动检测本地的 ip，通过 ip 发送请求；指定为 `string` 类型时，会向指定的值发送请求

## exclude <Badge type="tip" text="0.19.1+" vertical="middle" />

- 可选项
- 类型：`string | RegExp | (string | RegExp)[]`
- 说明：指定不参与编译的文件，默认为 `/node_modules/`，配置后为 `/node_modules/` 和 `exclude` 的并集。

## include <Badge type="tip" text="0.18.0+" vertical="middle" />

- 可选项
- 类型：`string | RegExp | (string | RegExp)[]`
- 说明：`code-inspector-plugin` 默认不会去编译 `node_modules` 中文件，在某些 monorepo 架构的项目中，你主项目引用的本地的 pkg 可能也是通过 `node_modules` 中链接引入的，此时你需要通过 `include` 声明这些包以让其中的代码能够参与定位。
- 示例：假设你有下述的目录结构：
  ```shell
  my-project
    - pkg-a
    - pkg-b
    - main-pkg # 通过 package.json 的 `dependencies` 引入了 pkg-a 和 pkg-b
      - node_modules
        - pkg-a
        - pkg-b
  ```
  如果你想让 `pkg-a` 和 `pkg-b` 中的源码能进行定位，则可以通过如下配置：
  ```ts
  codeInspectorPlugin({
    bundler: 'vite',
    include: ['pkg-a', 'pkg-b'],
  });
  ```

## mappings <Badge type="tip" text="0.18.1+" vertical="middle" />

- 可选项
- 类型：`Record<string, string> | Array<{ find: string | RegExp, replacement: string }>`
- 说明：配合 `include` 使用，用于将 `node_modules` 中的文件路径映射为项目中真实的文件路径。
- 示例：假设你有下述的目录结构：
  ```shell
  my-project
    - pkg-a
    - pkg-b
    - main-pkg # 通过 package.json 的 `dependencies` 引入了 pkg-a 和 pkg-b
      - node_modules
        - pkg-a
        - pkg-b
  ```
  当你通过 `include` 声明了 `pkg-a` 和 `pkg-b` 后，此时源码定位的位置是 `node_modules` 中的文件，而不是你项目中的真实文件路径，此时你可以通过 `mappings` 配置将 `node_modules` 中的文件路径映射为项目中的文件路径：
  ```ts
  import path from 'path';

  codeInspectorPlugin({
    bundler: 'vite',
    include: ['pkg-a', 'pkg-b'],
    mappings: {
      'pkg-a': path.resolve(__dirname, '../pkg-a'),
      'pkg-b': path.resolve(__dirname, '../pkg-b'),
    },
  });
  ```


## hooks <Badge type="tip" text="0.10.0+" vertical="middle" />

- 可选项
- 类型如下：
  ```ts
  type SourceInfo = {
    file: string;
    line: number;
    column: number;
  };
  type Hooks = {
    /*
     * server 端接收到 DOM 源代码定位请求后的钩子函数
     */
    afterInspectRequest?: (
      options: CodeInspectorOptions,
      source: SourceInfo
    ) => void;
  };
  // 例如
  codeInspectorPlugin({
    bundler: 'vite',
    hooks: {
      afterInspectRequest: (options, source) => {
        sendLog(source);
      },
    },
  });
  ```
- 说明：设置 `code-inspector-plugin` 某些生命周期的 hooks 回调，例如你想统计团队使用了多少次代码定位功能，则可以通过此配置实现。

## match <Badge type="tip" text="0.5.0+" vertical="middle" />

- 可选项
- 类型：`RegExp`，默认值为 `/\.(vue|jsx|tsx|js|ts|mjs|mts)$/`
- 说明：`code-inspector-plugin` 仅会对符合 `match` 正则表达式的文件会进行源码定位编译，你可以通过此项配置减少无效文件参与编译，提升编译性能。

## injectTo <Badge type="tip" text="0.5.0+" vertical="middle" />

- 可选项
- 类型：`string | string[]` (仅在 `0.17.5` 及以上版本支持 `string[]` 类型)
- 说明：用于注入 DOM 筛选和点击跳转 vscode 的相关的客户端代码的文件(必须为绝对路径且以 `.js/.ts/.mjs/.mts/.jsx/.tsx` 为结尾的文件)。`code-inspector-plugin` 默认会在首个符合 `match` 正则表达式的文件中注入客户端代码，在某些自研的 SSR 框架项目中，首个注入的文件可能只在 server 侧运行而导致客户端逻辑不生效，此时你可以通过此项配置指定一个客户端文件以保证客户端逻辑生效。

## openIn <Badge type="tip" text="0.8.0+" vertical="middle" />

- 可选项
- 类型：`'reuse' | 'new' | 'auto'`，默认值为 `'auto'`
- 说明：指定使用 vscode 或者 cursor 为编辑器时打开 IDE 窗口的方式。传 `reuse` 则指定复用当前窗口；传 `new` 则指定打开新窗口；传 `auto` 则根据当前 IDE 的安装情况自动选择。跟推荐在 IDE 设置中去配置你的习惯：

  <img width="978" alt="image" src="https://github.com/user-attachments/assets/b98b819b-363c-4b3b-98bf-8c1606821942">

## pathFormat <Badge type="tip" text="0.8.0+" vertical="middle" />

- 可选项
- 类型：`string | string[]`，默认值为 `{file}:{line}:{column}`
- 说明：指定打开 IDE 文件时的命令格式，主要是配合非内置的 IDE 一起使用的。其中， `{file}`、`{line}`、`{column}` 会作为模版被动态替换。例如你的代码位置为 `/root/my-project/index.ts` 的第 `5` 行第 `11` 列，使用的 IDE 打开文件的执行命令为 `yourIDE /root/my-project/index.ts --line 5 --column 11`，则你应该设置此项值为 `["{file}", "--line", "{line}", "--column", "{column}"]`。


## hideDomPathAttr <Badge type="tip" text="0.12.0+" vertical="middle" />

- 可选项
- 类型：`boolean`。默认值 `false`
- 说明：是否在浏览器控制台中隐藏 DOM 元素上的 `data-insp-path` 属性


## hideConsole

- 可选项
- 类型：`boolean`，默认值为 `false`
- 说明：是否隐藏在浏览器控制台打印的关于 `code-inspector-plugin` 组合键的提示

## escapeTags <Badge type="tip" text="0.11.0+" vertical="middle" />

- 可选项
- 类型：`(string | RegExp)[]`
- 说明：对于满足上述条件的标签，不会在编译时注入 `data-insp-path` 属性


## importClient <Badge type="tip" text="0.14.1+" vertical="middle" />

- 可选项
- 类型：`string`
- 说明：引入客户端交互代码的方式: `file` 为引入交互代码所在的文件; `code` 为直接将交互代码注入到入口文件中去。

## needEnvInspector <Badge type="danger" text="已废弃" vertical="middle" />

- 可选项
- 类型：`boolean`，默认值为 `false`
- 说明：设置为 `true` 时，仅当 `.env.local` 文件存在且其包含 `CODE_INSPECTOR=true` 时插件功能才生效。（主要是解决团队内有部分成员不想使用该插件功能的需求）

## port <Badge type="tip" text="0.19.0+" vertical="middle" />

- 可选项
- 类型：`number`，默认值为 `5678`
- 说明：指定 `code-inspector-plugin` 的 server 开始寻找的启动端口。

## printServer <Badge type="tip" text="0.19.0+" vertical="middle" />

- 可选项
- 类型：`boolean`，默认值为 `false`
- 说明：是否在控制台中打印 server 的启动信息。

## pathType <Badge type="tip" text="0.20.0+" vertical="middle" />

- 可选项
- 类型：`'absolute' | 'relative'`，默认值为 `'relative'`
- 说明：指定 `data-insp-path` 属性的路径类型，默认使用相对路径，可选使用绝对路径(微前端场景下如果多个项目不在同一 git 仓库下，则需要使用绝对路径)。

## cache <Badge type="tip" text="0.20.2+" vertical="middle" />

- 可选项
- 类型：`boolean`，默认为 `false`
- 说明：此选项仅对 `webpack/rspack` 且缓存类型为 `filesystem` 的项目生效，主要是为了防止页面和 IDE 通信时请求的端口号不一致而导致通信失败的问题。默认为 `false`，每次冷启动都不使用缓存；设置为 `true` 则使用缓存(设置为 `true` 时建议同时设置 `port` 为某一个固定的端口号)。

## skipSnippets <Badge type="tip" text="1.2.3+" vertical="middle" />

- 可选项
- 类型：`('console' | 'htmlScript')[]`，默认值为 `[]`
- 说明：跳过注入某些代码片段：
  - `console`：跳过注入 `console.error` 和 `console.warn` 的代码片段，nextjs 和 nuxt 项目不建议跳过此项
  - `htmlScript`：跳过在 html 中注入 script 标签的代码片段，MPA 项目不建议跳过此项