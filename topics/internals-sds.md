Hacking Strings
===

The implementation of Redis strings is contained in `sds.c` (`sds` stands for Simple Dynamic Strings).

Redis字符串的实现包含在 `sds.c` 中（`sds` 代表“简单动态字符串”）。

The C structure `sdshdr` declared in `sds.h` represents a Redis string:

`sds.h` 中声明的C结构体 `sdshdr` 代表了Redis字符串： 

    struct sdshdr {
        long len;
        long free;
        char buf[];
    };

The `buf` character array stores the actual string.

字符数组 `buf` 存储实际的字符串。

The `len` field stores the length of `buf`. This makes obtaining the length
of a Redis string an O(1) operation.

`len` 存储 `buf` 的长度，使得长度的获取操作为O(1)复杂度。

The `free` field stores the number of additional bytes available for use.

`free` 存储额外可用（释放空间时可以释放掉的）的字节数。

Together the `len` and `free` field can be thought of as holding the metadata of the `buf` character array.

`len` 和 `free` 可以一并看成 `buff` 的相关信息。 

Creating Redis Strings 创建Redis字符串
---

A new data type named `sds` is defined in `sds.h` to be a synonym for a character pointer:

`sds.h` 中定义了一个新的数据类型，叫 `sds`，它是字符指针的同义词：

    typedef char *sds;

`sdsnewlen` function defined in `sds.c` creates a new Redis String:

`sds.c` 中定义的函数 `sdsnewlen` 创建了一个新的Redis字符串：

    sds sdsnewlen(const void *init, size_t initlen) {
        struct sdshdr *sh;

        sh = zmalloc(sizeof(struct sdshdr)+initlen+1); //开辟一块内存空间
    #ifdef SDS_ABORT_ON_OOM
        if (sh == NULL) sdsOomAbort();
    #else
        if (sh == NULL) return NULL;
    #endif
        sh->len = initlen;
        sh->free = 0;
        if (initlen) {
            if (init) memcpy(sh->buf, init, initlen); //把init复制给sh.buf
            else memset(sh->buf,0,initlen); //
        }
        sh->buf[initlen] = '\0';
        return (char*)sh->buf;
    }

Remember a Redis string is a variable of type `struct sdshdr`. But `sdsnewlen` returns a character pointer!!

要记住，一个Redis字符串是一个 `结构体 sdshdr` 类型的变量。然而 `sdsnewlen` 返回的是一个字符指针！！

That's a trick and needs some explanation.

这里有一些需要解释的小技巧。

Suppose I create a Redis string using `sdsnewlen` like below:

假设我用 `sdsnewlen` 创建了一个Redis字符串：

    sdsnewlen("redis", 5);

This creates a new variable of type `struct sdshdr` allocating memory for `len` and `free`
fields as well as for the `buf` character array.

这个函数创建了一个 `结构体 sdshdr` 类型的变量，为成员 `len`、`free`和`buf` 开辟了内存空间。

    sh = zmalloc(sizeof(struct sdshdr)+initlen+1); // initlen is length of init argument.

After `sdsnewlen` successfully creates a Redis string the result is something like:

创建成功的结果长这样：

    -----------
    |5|0|redis|
    -----------
    ^   ^
    sh  sh->buf

`sdsnewlen` returns `sh->buf` to the caller.

`sdsnewlen` 返回 `sh->buf` 给调用者。

What do you do if you need to free the Redis string pointed by `sh`?

如果你想要释放 `sh` 指向的Redis字符串，你该怎么做？

You want the pointer `sh` but you only have the pointer `sh->buf`.

当然是想要拿到 `sh` 的指针啦，但你有的只有 `sh->buf` 指针。

Can you get the pointer `sh` from `sh->buf`?

可以从 `sh->buf` 拿到 `sh` 的指针吗？

Yes. Pointer arithmetic. Notice from the above ASCII art that if you subtract
the size of two longs from `sh->buf` you get the pointer `sh`.

当然了。简单的地址四则运算。`sh->buf` 的地址减掉两个long（成员free和len都是long）的长度就是 `sh` 的地址了。

The `sizeof` two longs happens to be the size of `struct sdshdr`.

两个long的 `大小` 刚好是 `结构体 sdshdr` 的大小。

Look at `sdslen` function and see this trick at work:

看看 `sdslen` 是怎么利用这个小技巧的：

    size_t sdslen(const sds s) {
        struct sdshdr *sh = (void*) (s-(sizeof(struct sdshdr)));
        return sh->len;
    }

Knowing this trick you could easily go through the rest of the functions in `sds.c`.

知道这个小技巧，`sds.c` 里的其他代码就很好理解了。

The Redis string implementation is hidden behind an interface that accepts only character pointers. The users of Redis strings need not care about how its implemented and treat Redis strings as a character pointer.

Redis暴露的接口只接收字符指针，因此Redis字符串的实现被隐藏了。用户不需关心Redis字符串是怎么实现的，只要把它当成一个字符指针就好了。
