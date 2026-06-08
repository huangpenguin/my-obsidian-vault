
除了去网络设置里面把ipv6的地址改回自动之外：
- `route -f`_(警告：敲完这行你会瞬间彻底断网，这是正常的，因为它清空了所有路由规则)_
- `netsh winsock reset`_(重置被污染的网络接口)_
- `netsh int ip reset`_(重置 TCP/IP 底层协议栈)_
- `ipconfig /flushdns`