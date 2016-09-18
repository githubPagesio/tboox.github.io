---
layout: post.cn
title:  "哈希容器的使用"
tags: tbox 哈希 容器
categories: tbox
---

stl的容器库非常强大，但是为了要兼容各种元素类型，采用了模板进行泛化，这样的好处就是使用非常的方便，但是编译器会对使用到的每种类型都进行一遍实例化，用的类型太多的话不仅影响编译速度而且生成的可执行文件也很冗余。

因此，TBOX在设计容器架构的时候，引入tb_item_func_t类型，来设置容器使用的成员类型，这样在实现容器通用性的同时，也不会产生过的冗余，而且容器接口操作上，同样相当的便利。

可以先看个简单使用哈希的例子：


```c
    /* 初始化hash, 哈希桶大小8
     * 键：大小写敏感字符串
     * 值：long整型
     */
    tb_hash_ref_t hash = tb_hash_init(8, tb_item_func_str(tb_true), tb_item_func_long());
    if (hash)
    {
        // 设置键值对："key" => 123
        tb_hash_set(hash, "key", (tb_pointer_t)123);

        // 获取值
        tb_long_t value = (tb_long_t)tb_hash_get(hash, "key");

        // 退出hash
        tb_hash_exit(hash);
    }

    /* 初始化hash, 哈希桶大小: TB_HASH_BULK_SIZE_MICRO
     * 键：tb_struct_xxxx_t 结构体类型，内存数据由hash内部自己维护, 后面两个参数设置成员的释放回调函数
     * 值：true类型，永远是tb_true, 这种hash相当于stl的set<tb_struct_xxxx_t>，内部会去优化掉value占用的内存
     */
    tb_hash_ref_t hash = tb_hash_init(TB_HASH_BULK_SIZE_MICRO, tb_item_func_mem(sizeof(tb_struct_xxxx_t), tb_null, tb_null), tb_item_func_true());
    if (hash)
    {
        // 初始化tb_struct_xxxx_t
        tb_struct_xxxx_t xxxx = {0};

        // 设置键值对：xxxx => tb_true
        tb_hash_set(hash, &xxxx, (tb_pointer_t)tb_true);

        // 判断键是否存在
        if (tb_hash_get(hash, &xxxx)) 
        {
            // ...
        }

        // 退出hash
        tb_hash_exit(hash);
    }

    /* 初始化hash, 哈希桶大小使用默认大小: 0
     * 键：大小写不敏感字符串
     * 值：uint8整型
     */
    tb_hash_ref_t hash = tb_hash_init(0, tb_item_func_str(tb_false), tb_item_func_uint8());
    if (hash)
    {
        // 设置键值对："key" => 123
        tb_hash_set(hash, "key", (tb_pointer_t)123);

        // 获取u位整型键值
        tb_uint8_t value = (tb_uint8_t)tb_hash_get(hash, "key");

        // 退出hash
        tb_hash_exit(hash);
    }
```



怎么样，简单吧。各种类型项都是可以在键值上互用的，而且会去适配`tb_hash_get`和`tb_hash_set`等容器接口参数。

你也可以很方便的在初始化容器的时候，自定义成员释放函数、成员比较函数、哈希计算函数等，例如：

```c
    // 指针成员释放函数
    static tb_void_t tb_hash_item_ptr_free(tb_item_func_t* func, tb_pointer_t buff)
    {
        // 断言检测
        tb_assert_and_check_return(func && buff);

        // 获取用户私有数据
        tb_pointer_t priv = func->priv;

        /* 获取成员数据，这里为tb_pointer_t类型，buff是指向成员数据的指针
         *
         * 如果是tb_item_func_str()项类型，数据就是：
         * tb_char_t* data = *((tb_char_t**)buff);
         *    
         * 如果是tb_item_func_mem()项类型，数据就是：
         * tb_byte_t* data = (tb_byte_t*)buff;
         *
         * 因为tb_item_func_mem是吧成员数据的内存都放到了容器里面维护，所以
         * buff指向的就是成员数据本身的地址
         *
         * 而str、ptr只是把指针存到了容器中，所以item指向的是指针的地址，这个需要
         * 注意的，不然很容易出问题
         */
        tb_pointer_t data = *((tb_pointer_t*)buff);
        
        // 释放它
        if (data) tb_free(data);

        // 清空成员数据
        *((tb_pointer_t*)buff) = tb_null;
    }

    // long 比较函数，改成反序比较
    static tb_long_t tb_hash_item_long_comp(tb_item_func_t* func, tb_cpointer_t ldata, tb_cpointer_t rdata)
    {
        return ((tb_long_t)ldata < (tb_long_t)rdata? 1 : ((tb_long_t)ldata > (tb_long_t)rdata? -1 : 0));
    }

    // 初始化long整型比较函数
    tb_item_func_t func = tb_item_func_long();

    // 替换比较函数, ptr有快捷的传入方式，当然也可以这样传
    func.comp = tb_hash_item_long_comp;

    // 初始化hash
    tb_hash_ref_t hash = tb_hash_init(0, func, tb_item_func_ptr(tb_hash_item_ptr_free, "private data"));
```
