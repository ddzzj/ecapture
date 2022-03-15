# ssldump 介绍
eBPF HOOK uprobe实现的SSL/HTTPS数据导出功能，针对HTTPS的数据包抓取，不需要导入CA证书。

# 原理

## eBPF技术
参考[ebpf](https://ebpf.io)官网的介绍
![](https://ebpf.io/static/overview-bf463455a5666fc3fb841b9240d588ff.png)

## uprobe HOOK
本项目hook了`/lib/x86_64-linux-gnu/libssl.so.1.1`的`SSL_write`、`SSL_read`函数的返回值，拿到明文信息，通过ebpf map传递给用户进程。

```go
Probes: []*manager.Probe{
    {
        Section:          "uprobe/SSL_write",
        EbpfFuncName:     "probe_entry_SSL_write",
        AttachToFuncName: "SSL_write",
        //UprobeOffset:     0x386B0,
        BinaryPath: "/lib/x86_64-linux-gnu/libssl.so.1.1",
    },
    {
        Section:          "uretprobe/SSL_write",
        EbpfFuncName:     "probe_ret_SSL_write",
        AttachToFuncName: "SSL_write",
        //UprobeOffset:     0x386B0,
        BinaryPath: "/lib/x86_64-linux-gnu/libssl.so.1.1",
    },
    {
        Section:          "uprobe/SSL_read",
        EbpfFuncName:     "probe_entry_SSL_read",
        AttachToFuncName: "SSL_read",
        //UprobeOffset:     0x38380,
        BinaryPath: "/lib/x86_64-linux-gnu/libssl.so.1.1",
    },
    {
        Section:          "uretprobe/SSL_read",
        EbpfFuncName:     "probe_ret_SSL_read",
        AttachToFuncName: "SSL_read",
        //UprobeOffset:     0x38380,
        BinaryPath: "/lib/x86_64-linux-gnu/libssl.so.1.1",
    },
    /**/
},
```

# 使用方法
安装使用，可以选择编译，也可以直接下载二进制包。
笔者环境`ubuntu 21.04`， linux kernel 5.10以上通用。
## 编译
```shell
git clone git@github.com:ehids/ssldump.git
cd ssldump
make
bin/ssldump
```
## 使用
执行任意https网络请求即可使用。
```shell
wget https://www.qq.com
```