2. Edward realizes his previous solution was insufficient, so he decide to implement a slightly more com
plicated protocol.
The client will perform the following.
(a) Send an identifier for the function it wants as an integer (0 for ith_prime, 1 for is_coprime).
(b) Send all bytes for all the arguments.
The server will then perform the following.
(a) Read identifier.
(b) Use identifier to allocate memory and set read size.
(c) Read arguments.
/* Converts NETLONG from network byte order to host byte order. */
uint32_t ntohl(uint32_t netlong);
/* Converts NETLONG from host byte order to network byte order. */
uint32_t htonl(uint32_t hostlong);
void receive_rpc (int sock_fd) {
/* Read in procedure identifier. */
uint32_t id;
int bytes_read = 0, cur_read = 0;
while (_______________) {
bytes_read += cur_read;
}
id = _______________;
/* Get sizes and allocate space for arguments and return values. */
char *args, *rets;
size_t arg_bytes, ret_bytes;
get_sizes(id, &args, &arg_bytes, &rets, &ret_bytes);
/* Read in arguments. */
bytes_read = 0;
while (_______________) {
bytes_read += cur_read;
}
/* Call appropriate server stub stub function based on id. */
call_server_stub(id, args, arg_bytes, rets, ret_bytes);
/* Write return values. */
int bytes_written = 0, cur_written = 0;
while (_______________) {
bytes_written += cur_written;
}
/* Clean up socket. */
_______________;
}
void receive_rpc (int sock_fd) {
/* Read in procedure identifier. */
uint32_t id;
int bytes_read = 0, cur_read = 0;
while ((cur_read = read(sock_fd, ((char *) &id) + bytes_read,
sizeof(uint32_t)- bytes_read)) > 0) {
bytes_read += cur_read;
}
id = ntohl(id);
/* Get sizes and allocate space for arguments and return values. */
char *args, *rets;
size_t arg_bytes, ret_bytes;
get_sizes(id, &args, &arg_bytes, &rets, &ret_bytes);
/* Read in arguments. */
bytes_read = 0;
while ((cur_read = read(sock_fd, &args[bytes_read],
arg_bytes- bytes_read)) > 0) {
bytes_read += cur_read;
}
/* Call appropriate server stub stub function based on id. */
call_server_stub(id, args, arg_bytes, rets, ret_bytes);
/* Write return values. */
int bytes_written = 0, cur_written = 0;
while ((cur_written = write(sock_fd, &rets[bytes_written],
ret_bytes- bytes_written)) > 0) {
bytes_written += cur_written;
}
/* Clean up socket. */
close(socket_fd);
}



一、 解题思路
在网络编程中，基于 TCP 套接字 (Socket) 实现远程过程调用（RPC）时，必须克服 TCP 协议自身的两个核心特性：流式传输特性（半包/粘包） 与 网络字节序差异（Endianness）。

为此，自定义 RPC 协议采用了 “固定长度 Header（标示符） + 动态长度 Payload（变长参数）” 的设计范式。服务端处理请求的整体逻辑遵循严格的递进关系：

Plaintext
[接收固定 Header (4B)] ➔ [网络字节序转换] ➔ [查找/分配 Payload 内存] ➔ [接收完整 Payload] ➔ [执行本地存根] ➔ [写回结果] ➔ [关闭连接]
整体步骤拆解
读取标示符 (id)：客户端首先发送 4 字节的整数 id，指定要调用的函数。服务端需通过循环读取确保 4 字节完全接收。

转换网络字节序：网络传输采用统一的大端序，接收后需立即转换为本地主机字节序，保证 id 值正确。

元数据解析与内存分配 (get_sizes)：依据 id 检索参数量与返回值所需空间，并在堆区动态申请缓冲区内存。

读取变长参数 (args)：根据解出的参数长度，再次通过循环读取，确保所有参数字节完整入缓冲区。

执行本地存根 (call_server_stub)：参数就绪后，调用目标函数处理请求，并将输出结果写入返回缓冲区。

循环写回结果 (rets)：将返回数据写回客户端，同样需采用循环写入防范 partial write（部分写入）。

资源回收 (close)：关闭套接字文件描述符，释放系统网络资源。

二、 代码详解
1. 标示符读取与字节序转换
C
uint32_t id;
int bytes_read = 0, cur_read = 0;

// 1. 循环读取 4 字节 Header
while ((cur_read = read(sock_fd, ((char *) &id) + bytes_read, sizeof(uint32_t) - bytes_read)) > 0) {
    bytes_read += cur_read;
}

// 2. 将网络大端序转换为主机字节序
id = ntohl(id);
((char *) &id) + bytes_read：&id 获取 id 的首地址；强转为 char * 可确保指针算术运算以单字节为单位偏移；加上 bytes_read 确保后续数据精准写入尚未填充的内存空位。

sizeof(uint32_t) - bytes_read：动态计算剩余待接收字节数，防止缓冲区溢出。

ntohl(id)：Network to Host Long，将网络标准的 Big-Endian 序转换为本地 CPU 的 Endian 序。

2. 动态内存分配与参数接收
C
char *args, *rets;
size_t arg_bytes, ret_bytes;

// 1. 根据 id 获取参数/返回值尺寸并分配内存
get_sizes(id, &args, &arg_bytes, &rets, &ret_bytes);

// 2. 循环读取 Payload 参数
bytes_read = 0;
while ((cur_read = read(sock_fd, &args[bytes_read], arg_bytes - bytes_read)) > 0) {
    bytes_read += cur_read;
}
get_sizes(...)：依据解析正确的 id 查找输入/输出大小，并初始化 args 与 rets 指针。

&args[bytes_read]：由于 args 本身为 char * 类型，采用数组下标取地址法 &args[bytes_read] 等价于指针偏移算法，使得代码更为清晰直观。

3. 本地函数执行与结果写回
C
// 1. 调用本地 Stub 执行目标逻辑
call_server_stub(id, args, arg_bytes, rets, ret_bytes);

// 2. 循环写回计算结果
int bytes_written = 0, cur_written = 0;
while ((cur_written = write(sock_fd, &rets[bytes_written], ret_bytes - bytes_written)) > 0) {
    bytes_written += cur_written;
}

// 3. 清理连接套接字
close(sock_fd);
write(...) 循环：TCP 发送缓冲区满或网络拥堵时，write 同样可能发生部分写入（Partial Write），必须依靠 while 循环推动 &rets[bytes_written] 指针，直到发送完所有的 ret_bytes。

close(sock_fd)：正确关闭通信套接字。（注：原题伪代码中 close(socket_fd) 存在变量名拼写笔误，应为 sock_fd）。

三、 代码间强关联与链条关系
各个步骤构成一个紧密耦合的逻辑链条，前一步骤的正确性直接决定后续步骤的执行：

Plaintext
┌────────────────┐      转换正确 id      ┌─────────────┐      得到 Payload 边界      ┌─────────────────┐
│ read id 4 字节 │ ───────────────────> │  ntohl(id)  │ ───────────────────────> │ get_sizes(...)  │
└────────────────┘                      └─────────────┘                          └─────────┬───────┘
                                                                                           │
┌────────────────┐       清空缓冲区      ┌─────────────┐      填充 rets 完毕      ┌────────┴────────┐
│ close(sock_fd) │ <─────────────────── │ write rets  │ <──────────────────────── │ call_server_stub│
└────────────────┘                      └─────────────┘                           └─────────────────┘
read id 决定 ntohl 的有效性：只有在读满 4 字节后，ntohl 转换出来的数字才具有业务意义，否则会将垃圾数据进行字节序转换。

ntohl 决定 get_sizes 的正确性：若遗漏字节序转换，大端序下的数值 1 在小端序机器上会被解析为 16,777,216，导致 get_sizes 尝试申请内存巨幅超载，引发崩溃。

get_sizes 决定后续读写循环的终止条件：其算出的 arg_bytes 与 ret_bytes 构成了 read args 与 write rets 两个 while 循环退出的基准边界。

四、 常见 Bug 与易错点
1. 错误的指针算术运算 (Pointer Arithmetic)
错误写法：read(sock_fd, &id + bytes_read, ...)

原因分析：&id 的类型为 uint32_t *。在 C 语言中，对 uint32_t * 执行 + 1 操作，指针在内存中实际偏移 sizeof(uint32_t)（即 4 字节）。若 bytes_read 为 1，该写法将导致指针偏移 4 字节，造成严重的内存越界覆盖（Segmentation Fault）。

纠正方案：必须强转为字节指针 (char *)&id + bytes_read。

2. 忽视 TCP 流式特征导致的单次 read() 假定
错误写法：read(sock_fd, &id, sizeof(id));（不使用 while 循环）

原因分析：TCP 是字节流协议，底层分包或网络延迟可能导致 read 单次仅返回部分数据（如 1~3 字节）。若无循环校验，后续的参数读取逻辑会将 Header 的剩余字节误当作 Payload 解析，造成协议脱节。

3. 遗漏网络字节序转换 (Endianness)
错误写法：未调用 ntohl() 直接使用接收到的 id。

原因分析：网络传输标准为大端序（Big-Endian），而 x86/x64 架构的主机为小端序（Little-Endian）。忽略 ntohl() 转换会导致数值高低位翻转，解析出错误的函数编号。

4. 忽略 write() 的部分写入现象 (Partial Write)
错误写法：单次调用 write(sock_fd, rets, ret_bytes); 假定完整发送。

原因分析：当发送缓冲区满时，write 系统调用仅会写入当前能容纳的字节数并立即返回。忽略写循环会导致客户端接收到截断的返回值。

五、 总结与考核知识点
这道经典考题的核心在于评估开发者在系统级网络编程中对底层协议细节与内存控制的理解：

网络字节序 (Endianness Consistency)：掌握 ntohl() / htonl() 在跨平台、跨 CPU 架构通信时的数据对齐与转换作用。

TCP 流式传输与粘包/半包处理 (Stream-oriented I/O)：理解 TCP 无数据边界（Boundaryless）特性，掌握使用 while 循环配合指针偏移量计算和剩余字节数校验来实现可靠的拆包/组包操作。

自定义应用层协议设计 (Application-Layer Protocol Design)：掌握通过“固定 Header + 动态 Payload”的方式显式划分数据帧边界的基本范式。
