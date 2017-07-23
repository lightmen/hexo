---
title: linux kernel中读写文件
categories: kernel
tags:
- 文件系统
- Linux
- kernel
---


# 头文件
```
#include <linux/fs.h>
#include <asm/segment.h>
#include <asm/uaccess.h>
#include <linux/buffer_head.h>
```

# 打开文件
```
struct file* file_open(const char* path, int flags, int rights) {
    struct file* filp = NULL;
    mm_segment_t oldfs;
    int err = 0;

    oldfs = get_fs();
    set_fs(get_ds());
    filp = filp_open(path, flags, rights);
    set_fs(oldfs);
    if(IS_ERR(filp)) {
        err = PTR_ERR(filp);
        return NULL;
    }
    return filp;
}
```

# 关闭文件
```
void file_close(struct file* file) {
    filp_close(file, NULL);
}
```

# 读文件
```
int file_read(struct file* file, unsigned long long offset, unsigned char* data, unsigned int size) {
    mm_segment_t oldfs;
    int ret;

    oldfs = get_fs();
    set_fs(get_ds());

    ret = vfs_read(file, data, size, &offset);

    set_fs(oldfs);
    return ret;
}
```

# 写文件
```
int file_write(struct file* file,vunsigned long long offset, unsigned char* data, unsigned int size) {
    mm_segment_t oldfs;
    int ret;

    oldfs = get_fs(); 
    set_fs(get_ds());

    ret = vfs_write(file, data, size, &offset);

    set_fs(oldfs);
    return ret;
}
```

# 同步文件
```
int file_sync(struct file* file) {
    vfs_fsync(file, 0);
    return 0;
}
```

备注：
get_ds():  获得kernel的内存访问地址范围
get_fs():  取得当前的地址访问限制值
set_fs():  设置当前的地址访问值

