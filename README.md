# 问题

修改了下lua_cjson来解决空table在生成json的时候二义性的问题（无法明确保证是生成“[]”还是“{}”）。在lua_cjson中是通过table的长度来区分array还是object的，如果长度大于0，就认为是array，否则就认为是object，所以默认的情况下空table都会生成“{}”。

# 解决方法

在lua文件中定义空table的时候通过metatable来指定是否为array，然后在lua_cjson中如果长度为0的时候，将增加进一步的判断，来通过这个判断来确定是否为array还是object，然后调用相应的函数。

# 基本使用

```
local cjson = require "cjson"
 
--- 得到array的metatable：  { __index = { cjsonarray = 1  }}
local mt_arr = cjson.array and cjson.array() or {}

local test = {
        array = setmetatable({}, mt_arr), -- 这个将被认为是array，将生成“[]”
        object = {}, -- 这个将被认为是object，将生成{}
}

print(cjson.encode(test))
```

同时如果是“[]”在decode的时候，将会变为setmetatable({},{ __index = { cjsonarray = 1  }})

# 说明

这个方法是参考了lua-resty-libcjson中的实现。
