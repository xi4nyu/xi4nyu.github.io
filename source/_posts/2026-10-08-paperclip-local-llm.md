---
title: Paperclip 接入自建模型网关详解
tags: AI PaperClip
categories: AI PaperClip
date: 2026-10-08 10:33:00
---

Paperclip 是一个 AI Agent 管理平台，可以同时管理多个 Agent。但它的模型连接步骤只支持 Anthropic 和 OpenAI 的官方接口，自建网关无法连接。

这个问题困扰了很多自建网关的用户。本文通过实例，详细介绍原因和解决办法。

## 一、现象

Paperclip 跑在本地，控制台地址是 `http://127.0.0.1:3100`。

首次进入，它会自动弹出一个引导向导，共三步：

> Create your first agent → Connect a model → Review

第二步 Connect a model，只有两个选项：Claude API 和 OpenAI API。

两个都是官方接口。填入自建网关的 API key，点击 Connect，得到一行报错：

```
The provider rejected this API key.
```

这行报错让人费解，因为同一个 key 在别处是好用的。

在 Paperclip 所在的容器里，执行下面的命令：

```bash
$ curl -H "x-api-key: sk-****" http://sub2api:5050/v1/models
```

上面代码中，直接向网关查询模型列表。返回 200，模型列表完整。

所以问题不在 key 上，也不在网络上，而在 Paperclip 的校验方式上。

## 二、原因

Paperclip 的源码里，校验 API key 的函数叫做 `validateAiApiKey`。

它位于 `server/src/routes/ai-connections.ts`：

```ts
/**
 * Fixed provider endpoints; credentials are never sent to a
 * caller-supplied URL or through a redirect.
 */
export async function validateAiApiKey(provider, key, request = fetch) {
  const endpoints = {
    anthropic: "https://api.anthropic.com/v1/models?limit=1",
    ...
  };
```

关键细节在这里：`endpoints` 对象里的地址是**写死的**，不接受配置。

函数的注释解释了原因：凭据永远不会发往调用方提供的 URL，也不会经过跳转。

这是一条安全约束，防止 API key 被诱导发送到恶意地址。

所以，它拿着自建网关的 key，去问真实的 `api.anthropic.com`，被拒绝是必然的。

这里要说清楚 Paperclip 的两套机制。

官方接口走**托管连接**（managed aiConnection），凭据由平台校验和保管。

自建接口走**适配器环境变量**（adapter environment variables），由适配器进程自己读取。

上面那行报错来自前者。而且 `POST /companies/:id/ai-connections` 调用的是同一个函数，所以托管连接这条路，对自建网关是彻底关闭的。

顺带一提，前端还有第二重限制。

Connect 步之后的 `Finish setup` 按钮，被这个条件禁用：

```js
Boolean(connectionAdapter && !connection)
```

上面代码中，`connectionAdapter` 只有 `claude_local`、`codex_local`、`grok_local` 三种适配器为真。

也就是说，用这三种适配器，不连模型就创建不了 Agent。

结论是：不要在这个按钮上停留，要换一条路。

## 三、适配器环境变量

你可能会问，既然 UI 连不上，那模型地址到底该写在哪里？

答案是适配器的环境变量，字段名叫做 `adapterConfig.env`。

它是 Agent 配置的一部分，用来给适配器进程注入环境变量。

这条路是官方支持的。证据在适配器源码 `packages/adapters/claude-local/src/server/probe-env.ts` 里：

```ts
LOCAL_PROBE_ALLOWED_CALLER_ENV_KEYS = [
  "ANTHROPIC_API_KEY",
  "ANTHROPIC_AUTH_TOKEN",
  "ANTHROPIC_BASE_URL",
  "ANTHROPIC_MODEL",
  "ANTHROPIC_SMALL_FAST_MODEL",
  "CLAUDE_CONFIG_DIR",
  ...
]
```

上面代码中，这是探测环境时允许出现的键名单。我们要用的三个键，都在名单里面。

另外，运行期的禁用键名单叫做 `FORBIDDEN_ENV_BINDING_KEYS`，只包含 `PAPERCLIP_*` 和 GitHub token，不含 `ANTHROPIC_*`。

所以写进去的环境变量，会正常进入适配器进程。

### 3.1 三个写入入口

写入这个字段有三个入口，写的是同一份数据，用哪个都行：

> - Agent 页面：Runtime 页的 Environment variables 区块
> - REST API：`PATCH /api/agents/:id`
> - 创建 Agent 时：`POST /api/companies/:companyId/agents`

### 3.2 两种绑定形态

每一个环境变量的值，有两种写法。

一种是明文，适合 base_url、模型名这类非敏感值：

```json
{ "type": "plain", "value": "http://sub2api:5050" }
```

一种是密钥引用，指向一个加密存储的密钥，适合 API key：

```json
{ "type": "secret_ref", "secretId": "d15c321f-…" }
```

如果你的实例开启了严格密钥模式（`PAPERCLIP_SECRETS_STRICT_MODE`），API key 用明文写会返回 422，必须用后一种。

## 四、完整步骤

下面依次演示。

### 4.1 确定容器里能用的网关地址

Paperclip 的适配器进程运行在 Docker 容器里。

所以，容器里的 `127.0.0.1` 指的是容器自己，不是你的电脑。

这一点最容易出错，先解决它。

宿主机上的网关，在容器里要用服务名或者特殊域名访问：

> - 网关和 Paperclip 在同一个 Docker 网络：`http://sub2api:5050`
> - 网关跑在宿主机上（Docker Desktop）：`http://host.docker.internal:5050`
> - 在宿主机终端里自己测试：`http://127.0.0.1:5050`

确认容器能访问，返回 200 就对了：

```bash
$ docker exec docker-paperclip-1 sh -c \
    'curl -sS -m 8 -o /dev/null -w "%{http_code}\n" http://sub2api:5050/'
200
```

上面代码中，在容器内执行 curl 命令，只返回 HTTP 状态码。

### 4.2 验证 key 并拿到模型名

执行下面的命令：

```bash
$ docker exec docker-paperclip-1 sh -c \
    'curl -sS -H "x-api-key: sk-****" http://sub2api:5050/v1/models'
```

返回的 `data` 数组里，每一项的 `id` 就是一个模型名，后面要填进 `ANTHROPIC_MODEL`。

本例中，网关提供了十三个模型，常用的有：

> - claude-sonnet-5
> - claude-opus-5
> - claude-haiku-4-5
> - deepseek-v4-pro
> - glm-5.3

### 4.3 创建密钥

先把 API key 存成密钥，记下返回的 `id`：

```bash
$ curl -X POST "$PAPERCLIP/api/companies/$COMPANY_ID/secrets" \
    -H "content-type: application/json" \
    -d '{"name":"sub2api-anthropic-key","value":"sk-****"}'
```

密钥是加密存储的，比把明文写在配置里更安全。

### 4.4 创建 Agent

把网关信息写进 `adapterConfig.env`：

```bash
$ curl -X POST "$PAPERCLIP/api/companies/$COMPANY_ID/agents" \
    -H "content-type: application/json" \
    -d '{
      "name": "CEO",
      "role": "general",
      "adapterType": "claude_local",
      "adapterConfig": {
        "env": {
          "ANTHROPIC_BASE_URL":         { "type": "plain", "value": "http://sub2api:5050" },
          "ANTHROPIC_MODEL":            { "type": "plain", "value": "claude-sonnet-5" },
          "ANTHROPIC_SMALL_FAST_MODEL": { "type": "plain", "value": "claude-haiku-4-5" },
          "ANTHROPIC_API_KEY":          { "type": "secret_ref", "secretId": "<上一步的 id>" }
        }
      }
    }'
```

上面代码中，四个变量的含义如下：

> - `ANTHROPIC_BASE_URL`：网关地址
> - `ANTHROPIC_API_KEY`：密钥引用
> - `ANTHROPIC_MODEL`：主模型，负责主力推理
> - `ANTHROPIC_SMALL_FAST_MODEL`：小模型，负责标题生成这类轻量任务

以后要改配置，用 `PATCH /api/agents/:id`，请求体结构一样。

创建 Agent 的接口不要求先有托管连接。UI 上那个禁用的 `Finish setup`，只是前端门禁。

### 4.5 三重验证

配置写完了，但不要只看界面上的绿灯。

第一重，绕开 Paperclip，直接测这三个环境变量：

```bash
$ docker exec docker-paperclip-1 sh -c '
    cd /tmp &&
    ANTHROPIC_BASE_URL=http://sub2api:5050 \
    ANTHROPIC_API_KEY=sk-**** \
    ANTHROPIC_MODEL=claude-sonnet-5 \
    CLAUDE_CONFIG_DIR=/tmp/claude-verify \
    claude -p "Reply with exactly: OK"'
OK
```

上面代码中，只靠 `ANTHROPIC_MODEL` 就能选模型，不必依赖 UI 的下拉框。

第二重，走 Paperclip 自己的探测接口：

```bash
$ curl -X POST \
    "$PAPERCLIP/api/companies/$COMPANY_ID/adapters/claude_local/test-environment" \
    -H "content-type: application/json" \
    -d '{
      "agentId": "<agent-id>",
      "adapterConfig": { "env": { ...和上面完全一样... } }
    }'
```

期望返回 `status` 为 `pass`，并且有一项检查会报告 `Detected in adapter config env.`

这里有一个陷阱：`adapterConfig.env` **必须一起回传**。

这个接口只会从请求里恢复被脱敏的值，只传 `agentId` 会误报 `ANTHROPIC_API_KEY is not set`。

看着像配置没生效，其实是你没给它。

第三重，回到页面上，在 Agent 的 Runtime 页点 **Run test**。

显示 Connection successful，说明整条链路都通了。

三步都过了，配置才算真的完成。

## 五、常见错误

**（1）环境变量有优先级**

routine 的环境变量会覆盖同名键，而且同时覆盖 project 和 agent 两层：

```
routine env   >   project env   >   agent env
```

`PAPERCLIP_*` 是保留前缀，写不进去。

所以遇到"我明明改了怎么没生效"，先看有没有更上层在覆盖。

**（2）Model 下拉里的默认值不是实际值**

Agent 的 Runtime 页有个 Model 下拉，默认显示 `Default (claude-opus-5)`。

那是适配器自带的展示值，**不是它实际跑的模型**。

解析逻辑是 `resolveClaudeModel(config.model, env)`——`config.model` 为空时，回退到环境变量里的 `ANTHROPIC_MODEL`。

想换模型有两条路：改环境变量，或者在下拉里显式选一个覆盖它。

**（3）这层环境变量同时作用于探测和运行**

不需要配两遍。

连接探测（Run test）和真实运行（heartbeat 任务）用的是同一套合并后的环境。

**（4）常见报错对照**

> - `The provider rejected this API key.`：还在用 UI 的 Connect 步，那条路只连官方端点
> - 连接超时或 ECONNREFUSED：base_url 用了 `127.0.0.1`，容器内不通
> - 422 `requires secret references`：严格密钥模式下传了明文 key
> - 探测报 `ANTHROPIC_API_KEY is not set`：调 `test-environment` 时没回传 `adapterConfig.env`
> - 改完没生效：被 routine 或 project 层的同名变量覆盖了
> - 模型不对：把 Model 下拉的默认值当成了实际值

## 六、小结

这件事的本质是：Paperclip 对官方端点和对自建端点，用了两套机制。

官方端点走托管连接，凭据由平台校验和保管，校验地址写死。

为了不让凭据流向任意 URL，这个设计是对的。

自建端点走适配器环境变量。

`ANTHROPIC_BASE_URL`、`ANTHROPIC_API_KEY`、`ANTHROPIC_MODEL` 都在官方白名单里，是受支持的配置面。

所以，遇到 UI 死活连不上的时候，不要跟那个按钮较劲。

直接问一句：这个适配器把配置读成环境变量，那些环境变量叫什么名字？

答案通常就在适配器源码里，grep 一个 `process.env` 就能找到。

## 七、参考链接

- [Paperclip AI Connections Routes](https://github.com/paperclipai/paperclip/blob/main/server/src/routes/ai-connections.ts)，validateAiApiKey 函数
- [Paperclip Claude Local Adapter](https://github.com/paperclipai/paperclip/blob/main/packages/adapters/claude-local/src/server/probe-env.ts)，环境变量白名单
- [Paperclip Agent Validators](https://github.com/paperclipai/paperclip/blob/main/packages/shared/src/validators/agent.ts)，adapterConfigSchema 定义
- [sub2api](https://github.com/Wei-Shaw/sub2api)，Anthropic 兼容网关

（完）
