lua-libmodbus
=============

Lua bindings to [libmodbus](http://www.libmodbus.org/)

Parameters are mostly as per libmodbus documentation, but values are returned
directly, in tables, rather than in pointers and return codes.  Instead of
a return code, you would get a lua style "nil, error_msg" return pair.

Of particular relevance to modbus, is that tables are addressed with lua
style 1 based counting, but converted to zero based internally.  ie:

```
	res = dev:read_registers(0x2000, 2)
	print(#res) -- prints 2
	print(res[1]) - prints register address 0x2000, _not_ 0x2001
```

Status
------

* Client bindings for RTU/TCP and almost all operations.
* No helpers for integer32/float/signed/bits.
  needs thoughts on what would be nicest to use
* Server side limited to receive and reply exception.
  Needs thoughts on how to handle the mapping objects.
* Compatible with both 3.0.x and 3.1.x but you must run with the version you compiled with.

Compile
-------
You need Lua and libmodbus development packages (headers and libs) to
build lua-libmodbus.

Compile with

    make

You can override the pkg-config package name to set a specific Lua version.
For example:

    make LUAPKG=lua-5.1

Example usage
-------------

See demo.lua and other sample files.

Here is a simple example printing out 10 registers at 0x2000,
then writing 5 registers back to that address

```Lua
mb = require("libmodbus")
local dev = mb.new_tcp_pi("192.168.255.74", 1502)
dev:connect()
local regs, err = dev:read_registers(0x2000, 10)
if not regs then error("read failed: " .. err) end
for r,v in ipairs(regs) do
        print(string.format("register (offset %d) %d: %d (%#x): %#x (%d)",
                r, r, r + base_address - 1, r + base_address -1, v, v))
end
```

Another example writing registers instead

```Lua
mb = require("libmodbus")
local dev = mb.new_tcp_pi("192.168.255.74", 1502)
dev:connect()

-- will autoround 32.98 to 32.  no magic support for writing floats yet :|
-- numbers are cast to uint16_t internally.
local res, err = dev:write_registers(0x2000, { 0xabcd, 32.98, 0xfffe, 0xabcd, -1 })
if not res then print(err) end
```

