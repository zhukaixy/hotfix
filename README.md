# hotfix
Lua 5.2/5.3 hotfix. Hot update functions and keep old data.

Local variable which is not referenced by _G is not updated.
```
-- test.lua: return { function func() return "old" end }
local test = require("test")  -- referenced by _G.package.loaded["test"]
local func = test.func        -- is not upvalue or is referenced by _G
-- test.lua: return { function func() return "new" end }
require("hotfix").hotfix_module("test")
test.func()  -- "new"  
func()       -- "old"
```

Todo: Replace functions of local variables in all threads using:

    debug.getlocal ([thread,] f, local)
    debug.setlocal ([thread,] level, local, value)

hotfix do have side-effect. Global variables may be changed.
In the following example, t is OK but math.sin is changed.

<pre>
Lua 5.3.2  Copyright (C) 1994-2015 Lua.org, PUC-Rio
> math.sin(123)
-0.45990349068959
> do
>> local _ENV = setmetatable({}, {__index = _G})
>> t = 123
>> math.sin = print
>> end
> t
nil
> math.sin(123)
123
</pre>

Reference:
* hotfix by tickbh
  <br>https://github.com/tickbh/td_rlua/blob/11523931b0dd271ad4c5e9c532a9d3bae252a264/td_rlua/src/hotfix.rs
  <br>http://www.cnblogs.com/tickbh/articles/5459120.html (In Chinese)
  <br>Lua 5.2/5.3.
  
  Can only update global functions.
  
```
local M = {}
+ function M.foo() end  -- Can not add M.foo().
return M
```  
  
* lua_hotupdate
  <br>https://github.com/asqbtcupid/lua_hotupdate
  <br>Lua 5.1.
  
  Can not init new local variables.
  
```
local M = {}
+ local log = require("log")  -- Can not require!
function M.foo()
+    log.info("test")
end
return M
```

How to run test
------------------
Run main.lua in test dir.
main.lua will write a test.lua file and hotfix it.
main.lua will write log to log.txt.
<pre>
D:\Jinq\Git\hotfix\test>..\..\..\tools\lua-5.3.2_Win64_bin\lua53
Lua 5.3.2  Copyright (C) 1994-2015 Lua.org, PUC-Rio
> require("main").run()
main.lua:80: assertion failed!
</pre>

Unexpected update
-------------------
log function is changed from print() to an empty function.
The hotfix will replace all print() to an empty function which is totally unexpected.
```
local M = {}
local log = print
function M.foo() log("Old") end
return M
```
```
local M = {}
local log = function() end
function M.foo() log("Old") end
return M
```
Todo: replace module returned table or function but do not replace global.
