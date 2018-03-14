# Stop listen specific server with RpcMgmtStopServerListening

Windows RPC 提供了一个 `RpcMgmtStopServerListening` 函数用于停止监听，此函数原型为

```c++
RPC_STATUS RPC_ENTRY RpcMgmtStopServerListening(
   RPC_BINDING_HANDLE Binding
);
```

根据 MSDN 对此函数的描述， `Binding` 参数可以传入一个服务端的句柄来指定关闭，大多数情况下，大家只会开一个 RPC Server，使用时会向`Binding`参数传入 NULL 来停止监听，不过最近遇到了开了多个 RPC Server 时，关闭特定的 Server 的需求。

## 获取 RPC_BINDING_HANDLE

经过一番搜索，发现网上对此相关的文档比较少，最后还是从 MSDN 中找到了一个函数可以查询句柄 `RpcServerInqBindings`。此函数会返回一个 `RPC_BINDING_VECTOR` ，里面包含了多个句柄，为了识别出正确的句柄，需要使用 `RpcBindingToStringBinding` 函数转换成字符串，字符串中就有相关信息，可以根据字符串匹配识别。

```c++
RPC_BINDING_VECTOR *bindings;
RPC_BINDING_HANDLE handle;
RPC_STATUS status;
status = RpcServerInqBindings(&bindings);
assert(!status); // status error handle.
for (unsigned long i = 0; i < bindings->Count; i++) {
    RPC_CSTR str;
    status = RpcBindingToStringBindingA(bindings->BindingH[i], &str_bind);
    assert(!status); // status error handle.
    if (strstr((const char*)str, "YOUR RPC ENDPOINT NAME") != NULL) {
        // Found
        handle = bindings->BindingH[i];
        break;
    }
    RpcStringFreeA(&str);
}

// Do something with handle

RpcBindingVectorFree(&bindings);
```

## Status 5 ACCESS_DENIED

以为这样就解决问题了，将上面获取的 `RPC_BINDING_HANDLE` 对象，传入 `RpcMgmtStopServerListening` 并调用后发现，返回的 `RPC_STATUS` 一直是 5 。

继续翻阅 MSDN 后发现，此操作需要认证才可以。主要是通过 `RpcMgmtSetAuthorizationFn` 函数设置认证回调函数。

```c++
int __RPC_API RpcMgmtAuthorizationFn(
    RPC_BINDING_HANDLE ClientBinding,
    ULONG RequestedMgmtOperation,
    RPC_STATUS* Status
) {
	(void)ClientBinding;
	(void)RequestedMgmtOperation;
	*Status = RPC_S_OK;
    return TRUE
}

// ... in your setup code ...
RPC_STATUS status = RpcMgmtSetAuthorizationFn(RpcMgmtAuthorizationFn);
assert(!status); // error handling...
```





