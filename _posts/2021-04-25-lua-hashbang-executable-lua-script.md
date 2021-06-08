---
layout: post
title: "Extended Lua Hashbang - Portable Lua Scripts"
---


So you have a lua file, and you want to be able to run it from the command line like `./your_script.lua`. In languages like python or ruby you would accomplish this by adding `#!/usr/bin/env ruby` or `#!/usr/bin/env python` as the first line of the file. The `#!` magic pattern, known as a [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) (sheh-bang) or hashbang, tells linux to use a specific command to execute the file. You can do this in Lua too, but it's not the most portable option.

If you use `#!/usr/bin/env lua`, it'll work just fine as long as a program named `lua` exists. However, some people use an alternative lua implementation called [LuaJIT](https://luajit.org/). LuaJIT provides an executable named `luajit`, so the simple shebang won't find it unless the person running your code has added `lua` as an alias to `luajit`. If your code is compatible with both the reference Lua and LuaJIT, it's nice to make your script work with both out of the box.

The following solution is modified from [code by William Ahern on the lua-l mailing list](http://lua-users.org/lists/lua-l/2015-01/msg00633.html). This will search `$PATH` for `lua`, `lua5*`, and `luajit*`, executing the script with first one it finds:

```
#!/bin/sh
_=[[
IFS=":"
for dir in $PATH; do
    for lua in "$dir"/lua "$dir"/lua5* "$dir"/luajit*; do
        if [ -x "$lua" ]; then
            exec "$lua" "$0" "$@"
        fi
    done
done
printf '%s: no lua found\n' "$0" >&2
exit 1
]]
_=nil

-- Now we're running lua code
print("lua code here!")
```

So there you have it. Add those 14 lines at the top of your lua script, mark it as executable with `chmod +x your_script.lua`, and you're done! You can now run `./your_script.lua` as a command on a system with `lua` or a system with `luajit`.

We'll get into this in the next section, but that's just linux shell script code at the top there. You could extend this solution even further to do fun things like auto-installing dependencies with `luarocks`.

## Wow, how does that even work?

First, notice our hashbang is `#!/bin/sh`, so linux is actually going to run this file as if it's a shell script. When running as a shell script, the code searches for an available lua interpreter on the system, and then re-executes the file as a lua script. Our code therefore means two things in two different programming languages!


Let's break this down.

When running as `/bin/sh`:

- `_=[[` sets the `_` shell variable to the string `"[["`
- the for-loop runs, looking for an available lua interpreter in `$PATH`
- if no lua is found, `sh` fails with `exit 1`
- if a lua is found, `exec` replaces the `sh` process with a `lua` process, providing the current file as the first argument


Then, when running as lua code:

- `_=[[` starts defining the lua variable `_` as a multi-line string
- the sh code is ignored because it's in the string
- `]]` closes the multi-line string
- we re-assign the `_` variable to `nil` to clean up by undefining it
- the rest of the lua file runs as normal


There's one other nuance of shell scripts that makes this work. Most scripting languages these days will try to parse an entire file before running any of it. A language like that won't run your code if there's syntax errors anywhere in the file. So, given most lua probably isn't going to be valid shell script code, why does this trick work?

Well, `sh` and derivatives like `bash` work differently. They parse the file line by line as they execute, instead of parsing the whole thing at once. That means they don't care if you have a syntax error half way down the file; if they never get to that line of code then the error doesn't exist! Tools like [makeself](https://makeself.io/) even use this behavior to create self-extracting tar files, by adding a small shell script to the beginning of the tar file that extracts the rest of it.

And that's it! That's all the magic. Now go write some lua scripts!

