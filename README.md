# dsh-plugin-index

个人自用的 DeepSeek Harness（dsh）插件目录，一份**插件市场索引格式 1** 的 JSON：给 dsh 0.2.1+ 的 **插件 → 发现 → 索引源** 用，加进来就能在「发现」里搜到、点装。

> 这里的插件都是我自己写的、按现状发布。**不声明版本号、不承诺更新**；条目钉在具体 commit 上，只有我手动改这个文件时才会变。

## 怎么用

打开 **插件 → 发现（Discover）→ 索引源（Index sources）→ 添加**，填：

```
https://cdn.jsdelivr.net/gh/apples1949/dsh-plugin-index@main/index.json
```

添加后它成为当前选中的插件市场，条目会列出来。宿主给索引源的读取时间是 **15 秒**，所以这里用 jsDelivr 而不是 raw.githubusercontent：在本机实测 raw 的域名 3 次全超时（25s 无响应），jsDelivr 2 秒返回、内容一致。

代价是 jsDelivr 会缓存 `@main`（可能滞后一段时间）。想立刻看到本仓库的最新改动，可以：

- 用提交/标签固定：`https://cdn.jsdelivr.net/gh/apples1949/dsh-plugin-index@<sha>/index.json`（或 `@v1` 这种 tag）；
- 或者换成 raw 地址自己试一次（网络允许的话）：

```
https://raw.githubusercontent.com/apples1949/dsh-plugin-index/main/index.json
```

不想用索引源也可以直接按包名装，见下面每个插件的命令。

## 收录的插件

| 插件 | 分类 | 装什么 | 安装命令 |
| --- | --- | --- | --- |
| [`@apples1949/dsh-thinking-tag-guard`](https://github.com/apples1949/dsh-thinking-tag-guard) | 会话与消息 | Host 插件：把漏进正文的思考搬回思考通道、拦截静默结束、清洗 `<|DSML|…>` | `dsh plugin --profile <profile> add github:apples1949/dsh-thinking-tag-guard` |
| [`@apples1949/dsh-speed-meter`](https://github.com/apples1949/dsh-speed-meter) | UI 增强 | 客户端插件：输入框下方的实时 tok/s | `dsh plugin --profile <profile> add github:apples1949/dsh-speed-meter` |

两个都不在 npm 上，所以只能从 GitHub 装（`git+https://…` 这种 spec）。

## `index.json` 的结构要点

```json
{
  "format": 1,
  "title": { "en": "…", "zh": "…" },
  "description": { "en": "…", "zh": "…" },
  "metadata": { "categories": { "ui": { "title": { "en": "…", "zh": "…" } } }, "labels": {}, "sortKeys": {} },
  "plugins": [
    {
      "name": "@scope/name",
      "title": "…",
      "description": { "zh": "…", "en": "…" },
      "homepage": "https://github.com/…",
      "repository": { "url": "https://github.com/…" },
      "metadata": { "category": "ui", "keywords": [], "labels": [], "sortKeys": {} },
      "versions": [
        {
          "version": "0.1.0",
          "publishTimestamp": 1793031779,
          "source": { "type": "git", "url": "https://github.com/…", "commit": "<40 位 sha>" },
          "engines": { "dsh": ">=0.2.1-rc.1" }
        }
      ]
    }
  ]
}
```

几个容易踩的点（都是拿真实校验器试出来的）：

- `format` 必须是**数字** `1`（写成字符串 `"1"` 会被拒）。
- `title` / `description` 必须**同时**有 `en` 和 `zh`。
- 每个 version 的 `version` 是**必填字符串**——所以这里写的版本号只是校验器的要求，不是发版承诺。
- git 源用 `source.url` + `source.commit`（不是 `repo`），`commit` 必须是完整的 40 位 sha。
- 校验方式：用 dsh 自带的 `@deepseek-ai/dsh-plugin-market-index` 跑 `parseMarketIndex(doc, indexUrl)`，再对每条取 `marketInstallTarget()` 看装的是什么 spec。

## 维护

- 改插件 → 改 `index.json` 里对应的 `commit` → 推送。没改就是没更新。
- **没有发版节奏**，也不会因为 dsh 上游更新就跟着动。想要稳定请自己钉 commit。
- 索引里的条目有效性由校验器保证，插件本身能否加载取决于它自己的 `dsh` 清单与入口产物。

## 许可证

MIT
