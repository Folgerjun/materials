---
title: Kong 自定义插件编写详解
date: 2023-06-10 21:10:55
categories: [开发,总结]
tags: [Kong]
---



最近在研究 Kong，如果你跟我一样之前对 Kong 不了解可以看他们的[官网](https://docs.konghq.com/gateway/3.3.x/)，我就不在这多说废话了。

调研了几天，Kong 的功能蛮多的，对我们也有很大的帮助，引入也很有意义，能减少我们很多操作。Kong 也比较成熟，只是网上的资料都比较零散，很多也已经随着改版而变得过时。

官网的 [quickstart](https://docs.konghq.com/gateway/3.3.x/get-started/) 也算是详细了，跟着走一遍基本就能知道个大概，有哪些功能。

我想要尝试通过自定义编写插件来更好得对数据做一些处理，就各种找资料。终于经过一段时间的冲浪后，我开始掌握了一些门道。

首先，Apache 的 [skywalking-kong](https://github.com/apache/skywalking-kong) 帮我理清了绝大部分道路，只是不知道是因为 Kong 版本的更迭还是因为我本地环境的原因，跟 README 上描述的步骤还是有一点点的差别：

- `luarocks install kong-skywalking --local` 用 `luarocks` 安装 kong-plugin-skywalking，你也可以用 `--tree` 指定目录位置
- 然后是 `kong.conf`，一般在 `/etc/kong`下，如果没有就 `cp kong.conf.default kong.conf` 然后对应配置加上，`lua_package_path` 注意一下写上你实际的位置，可能与 README 上的会有偏差，我的就是
- 关键的还有一步没有写上，`/usr/local/share/lua/5.1/kong/constants.lua` 里记录了所有插件的名字，如果不加上的话你新加的插件也不会显示出来，需要特别注意。最后最好 kong restart

---

这一个走通后，就会对自定义的插件有个概念了。下面我们来自己手写一个试试，很简单的一个 Demo，主要是对 Response 的修改（只要走通了一个 其他的也就差不多了 看看官方 [API](https://docs.konghq.com/gateway/latest/plugin-development/pdk/)）。

其实别看这里我只是轻描淡写，实际却是踩了很多坑，网上太多都是不负责任的文章，也不清楚是纯粹的 copy 还是因为版本更迭导致的失效。

我先说下网上找资料然后最终发现的几个问题：

- `local BasePlugin = require "kong.plugins.base_plugin"`  这个 BasePlugin 在 [2.7.x](https://docs.konghq.com/gateway/2.7.x/plugin-development/custom-logic/#migrating-from-baseplugin-module) 已经废弃了，害得我还在那抓头，ChatGPT 资料库不够新，问了半天跟个傻子一样
- `kong.service.response` 和`kong.response` ，网上找到的资料都是用 `kong.service.response` 来改变响应体的内容，也害得我只抓头，纳闷了怎么也不行，直到我在官网 [kong.response](https://docs.konghq.com/gateway/latest/plugin-development/pdk/kong.response/) 找到了这么一句 `Unlike kong.service.response, this module allows mutating the response before sending it back to the client.` 我丢

然后其他的就是 Lua 脚本的编写了。

1. 先在之前 `skywalking` 同级新建一个 `response-handler` 目录（当然也可以在 Kong 自己的插件目录中添加自定义的插件）
2. 再进入 `response-handler` 目录，新建两个 Lua 脚本，插件主要也就是这两个文件 `handler.lua` 和 `schema.lua`。至于你说这两个是什么作用，我只能说业务逻辑在 handler 中，所用到的一些可变参数在 schema 中，具体的可以自行去冲浪哈，在这我就不赘述了。下面是我写的 Demo 脚本内容：

> 官网资料：[Plugin Configuration](https://docs.konghq.com/gateway/latest/plugin-development/configuration/)

**handler.lua** 

```
local cjson = require("cjson")

local MyPluginHandler = {
	VERSION = "0.1.0", -- 版本
	PRIORITY = 10 -- 脚本执行优先值
}

function MyPluginHandler:header_filter(config)
  -- 修改响应头
  kong.response.set_header("Handle-Response-Header", "Response-Handler")
end

function MyPluginHandler:access(conf)
end

function MyPluginHandler:body_filter(config)
  -- 修改响应体
  local response_body = kong.response.get_raw_body()
  if response_body then
     local response_json = cjson.decode(response_body)
     response_json.data.num = config.num -- 我这里修改了 response.data.num 的值 （config.num 对应 schema.lua 中的 config.num）
     local modified_body = cjson.encode(response_json)
     kong.response.set_raw_body(modified_body)
  end
end

return MyPluginHandler
```



**schema.lua**

```
local typedefs = require "kong.db.schema.typedefs"

local PLUGIN_NAME = "myplugin"

local schema = {
  name = PLUGIN_NAME,
  fields = {
    { consumer = typedefs.no_consumer }, -- 该插件不能设置为 consumer
    { protocols = typedefs.protocols_http },
    { config = {
        type = "record",
        fields = { -- 可配置参数
          { num = {
              type = "integer",
              default = 10,
              required = false,
              gt = 0, }}, 
        },
        entity_checks = {
          -- 可以写一些校验 at_least_one_of distinct
        },
      },
    },
  },
}

return schema
```



3. 脚本写好后在 `kong.conf`  中的 `plugins = bundled, skywalking, response-handler` 加上
4. 在 `/usr/local/share/lua/5.1/kong/constants.lua` 中的 `local plugins = {....}` 加上新写的 `response-handler`
5. kong restart



到这里，自定义的插件算是可以了。要还不行，那就可能是版本更迭了哈哈