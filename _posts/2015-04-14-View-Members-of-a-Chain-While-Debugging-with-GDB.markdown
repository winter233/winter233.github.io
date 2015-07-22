---
layout: post
title: View Members of a Chain While Debugging with GDB
categories: gdb
tags: [linux, gdb]
---

To print a element of the whole chain, `user-defined commands` can help.

>A user-defined command is a sequence of gdb commands to which you assign a new name as a command.

An example:

```sh
define dump_node
    set $_node = (Type*)$arg0
    while $_node
        printf "value is %d\n", $_node->value
        set $_node = $_node->next
    end
end
```
*User-defined command* start with `define command-name`, the example above defines a command named dump_list. `$arg0` stands for the first argument of the command, more arguments can be accepted, and the limitation is 10 (`$arg9`). 

Save these user-defined command in a file, say `gdb.sh`. To make it come into effect, using `source` command in gdb shell:

```
source gdb.sh
```

use command:

```
dump_node node
```

You can also create a correspongding `help` info using `document`.  See the example below:

```sh
document dump_node
    usage: dump_node _node
    dump_node is used to dump all elements in _node chain.
    arg0 is the head of the chain.
end
```
Document of a command is optional. Check these info in gdb shell:

```
help dump_node
```

Ref:   
[User-defined Command](http://sourceware.org/gdb/current/onlinedocs/gdb/Define.html)

