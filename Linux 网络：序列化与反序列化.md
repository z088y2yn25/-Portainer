# Linux 网络



## Getting started

To make it easy for you to get started with GitLab, here's a list of recommended next steps.

Already a pro? Just edit this README.md and make it your own. Want to make it easy? [Use the template at the bottom](#editing-this-readme)!

## Add your files

* [Create](https://docs.gitlab.com/user/project/repository/web_editor/#create-a-file) or [upload](https://docs.gitlab.com/user/project/repository/web_editor/#upload-a-file) files
* [Add files using the command line](https://docs.gitlab.com/topics/git/add_files/#add-files-to-a-git-repository) or push an existing Git repository with the following command:

```
cd existing_repo
git remote add origin https://gitlab.com/bronwyn-white-group/linux.git
git branch -M main
git push -uf origin main
```

## Integrate with your tools

* [Set up project integrations](https://gitlab.com/bronwyn-white-group/linux/-/settings/integrations)

## Collaborate with your team

* [Invite team members and collaborators](https://docs.gitlab.com/user/project/members/)
* [Create a new merge request](https://docs.gitlab.com/user/project/merge_requests/creating_merge_requests/)
* [Automatically close issues from merge requests](https://docs.gitlab.com/user/project/issues/managing_issues/#closing-issues-automatically)
* [Enable merge request approvals](https://docs.gitlab.com/user/project/merge_requests/approvals/)
* [Set auto-merge](https://docs.gitlab.com/user/project/merge_requests/auto_merge/)

## Test and Deploy

Use the built-in continuous integration in GitLab.

* [Get started with GitLab CI/CD](https://docs.gitlab.com/ci/quick_start/)
* [Analyze your code for known vulnerabilities with Static Application Security Testing (SAST)](https://docs.gitlab.com/user/application_security/sast/)
* [Deploy to Kubernetes, Amazon EC2, or Amazon ECS using Auto Deploy](https://docs.gitlab.com/topics/autodevops/requirements/)
* [Use pull-based deployments for improved Kubernetes management](https://docs.gitlab.com/user/clusters/agent/)
* [Set up protected environments](https://docs.gitlab.com/ci/environments/protected_environments/)

***

# Editing this README

When you're ready to make this README your own, just edit this file and use the handy template below (or feel free to structure it however you want - this is just a starting point!). Thanks to [makeareadme.com](https://www.makeareadme.com/) for this template.

## Suggestions for a good README

Every project is different, so consider which of these sections apply to yours. The sections used in the template are suggestions for most open source projects. Also keep in mind that while a README can be too long and detailed, too long is better than too short. If you think your README is too long, consider utilizing another form of documentation rather than cutting out information.

## Name
Choose a self-explaining name for your project.

## Description
Let people know what your project can do specifically. Provide context and add a link to any reference visitors might be unfamiliar with. A list of Features or a Background subsection can also be added here. If there are alternatives to your project, this is a good place to list differentiating factors.

## Badges
On some READMEs, you may see small images that convey metadata, such as whether or not all the tests are passing for the project. You can use Shields to add some to your README. Many services also have instructions for adding a badge.

## Visuals
Depending on what you are making, it can be a good idea to include screenshots or even a video (you'll frequently see GIFs rather than actual videos). Tools like ttygif can help, but check out Asciinema for a more sophisticated method.

## Installation
Within a particular ecosystem, there may be a common way of installing things, such as using Yarn, NuGet, or Homebrew. However, consider the possibility that whoever is reading your README is a novice and would like more guidance. Listing specific steps helps remove ambiguity and gets people to using your project as quickly as possible. If it only runs in a specific context like a particular programming language version or operating system or has dependencies that have to be installed manually, also add a Requirements subsection.

## Usage
Use examples liberally, and show the expected output if you can. It's helpful to have inline the smallest example of usage that you can demonstrate, while providing links to more sophisticated examples if they are too long to reasonably include in the README.

## Support
Tell people where they can go to for help. It can be any combination of an issue tracker, a chat room, an email address, etc.

## Roadmap
If you have ideas for releases in the future, it is a good idea to list them in the README.

## Contributing
State if you are open to contributions and what your requirements are for accepting them.

For people who want to make changes to your project, it's helpful to have some documentation on how to get started. Perhaps there is a script that they should run or some environment variables that they need to set. Make these steps explicit. These instructions could also be useful to your future self.

You can also document commands to lint the code or run tests. These steps help to ensure high code quality and reduce the likelihood that the changes inadvertently break something. Having instructions for running tests is especially helpful if it requires external setup, such as starting a Selenium server for testing in a browser.

## Authors and acknowledgment
Show your appreciation to those who have contributed to the project.

## License
For open source projects, say how it is licensed.

## Project status
If you have run out of energy or time for your project, put a note at the top of the README saying that development has slowed down or stopped completely. Someone may choose to fork your project or volunteer to step in as a maintainer or owner, allowing your project to keep going. You can also make an explicit request for maintainers.
# Linux 网络：序列化与反序列化.md
## 前言
在 Linux 网络编程中，进程之间通过网络传输数据时，内存里的结构体、变量是**内存二进制格式**，不同CPU架构（大端/小端）、不同编译器、内存对齐规则，会导致接收方无法直接解析原始内存数据。

**序列化（Serialization）**：把内存中的对象/结构体，转换成一段连续的二进制字节流，方便在网络上传输或者写入文件。
**反序列化（Deserialization）**：接收端收到二进制字节流后，按照约定规则，还原成内存中的结构体、变量。

> 一句话：序列化 = 对象 → 字节流；反序列化 = 字节流 → 对象。Linux网络通信本质就是两端按协议完成这一转换。

## 一、为什么网络通信必须做序列化？
1. **内存对齐问题**
C语言结构体存在内存对齐填充，结构体内部会有空白padding字节。如果直接`send`结构体指针，填充字节也会一起发送，浪费带宽，且不同编译选项对齐大小不一样，解析直接错乱。

示例：
```c
struct Test {
    char a;
    int b;
};
```
32/64位系统下，结构体占用字节大小不同，直接发送内存原始数据，接收端解析失败。

2. **大小端（字节序）问题**
x86 CPU是**小端序**；网络协议规定使用**大端序（网络字节序）**。int类型数值在不同架构下字节排布相反，不做转换解析出来数值完全错误。

3. **跨语言、跨平台**
发送端C语言，接收端Python/Go；或者x86发送，ARM接收。原生内存结构体格式不通用，必须序列化得到标准字节流。

> 简单区分：htonl/htons这类只是**字节序转换**，属于序列化里的一小部分，不等于完整序列化。

## 二、Linux下常见序列化方案分类
### 1. 手动序列化（C原生，传统网络编程）
自己定义协议，逐个取出字段，转网络字节序，打包进buffer；接收端按顺序从buffer读取、转本机字节序。
优点：无依赖、体积小、性能极高；
缺点：手写繁琐，结构体修改后两端代码都要同步修改，容易出bug。

示例伪代码：
```c
// 序列化：结构体转buffer
char buf[1024];
uint32_t id_net = htonl(st.id);
uint16_t len_net = htons(st.len);
memcpy(buf, &id_net, 4);
memcpy(buf+4, &len_net, 2);

// 反序列化：buffer还原结构体
uint32_t id;
memcpy(&id_net, buf,4);
id = ntohl(id_net);
```

### 2. 文本格式：JSON / XML
将对象转为字符串文本。
- JSON：最常用，易读、跨语言；缺点：体积大，解析开销高，不适合高性能低延迟网络。
Linux常用库：cJSON、json-c。

### 3. 二进制序列化库（高性能推荐）
- Protobuf（Google Protocol Buffers）：跨语言，预编译schema，二进制紧凑，高性能，Linux网络服务广泛使用。
- FlatBuffers：零拷贝序列化，适合低延迟场景。
- MessagePack：类似JSON的二进制版，不需要预编译IDL。
- Thrift：Apache，自带RPC框架。

> 核心特点：提前定义数据结构（schema），序列化库自动处理字节序、对齐，不用手动处理padding。

### 4. 系统自带：XDR（RPC使用）
早期Sun RPC使用的XDR序列化，处理跨平台数据表示，现在使用较少。

## 三、序列化关键注意点（Linux网络开发重点）
1. **固定长度字段 vs 变长字段**
固定长度：int、short；
变长：字符串、数组。序列化时必须携带**长度字段**，否则接收端不知道字符串到哪里结束。
> 经典坑：字符串不携带长度，遇到`\0`截断问题。

2. **字节序处理**
网络一律使用**大端序**。所有多字节整型（uint16/uint32）序列化时转为网络字节序；反序列化转回主机字节序。
> 注意：Protobuf等库内部已经自动处理字节序，不需要手动调用`htonl`。

3. **校验机制**
序列化后的字节流建议增加校验和（CRC32），网络传输存在丢包、比特错误，反序列化前校验数据完整性，防止脏数据导致程序崩溃。

4. **版本兼容**
业务迭代时，结构体新增字段。序列化协议要支持向后兼容，Protobuf天然支持；手写二进制协议很容易忽略版本号，升级后通信异常。

## 四、序列化与粘包问题（Linux Socket常见关联坑）
很多新手会混淆：序列化解决**数据表示格式**问题；TCP粘包解决**数据边界**问题。
- TCP是字节流，没有消息边界；一次recv可能读到半条消息、多条消息。
- 解决方案：序列化数据包头部增加**消息长度字段**。接收端先读取固定长度头部，拿到包体长度，再读取完整包体，之后再做反序列化。

数据包结构示例：
`[4字节包体长度][序列化后的业务数据]`

## 五、简单对比
|方案|数据类型|优点|缺点|适用场景|
| ---- | ---- | ---- | ---- | ---- |
|手动二进制|二进制|性能最高，无依赖|开发维护成本高，易出错|嵌入式、高性能轻量服务|
|JSON|文本|可读性强，调试方便|包大，解析慢|后台管理接口，低并发业务|
|Protobuf|二进制|紧凑、跨语言、版本兼容|需要写proto文件，编译|Linux后台服务间RPC通信|
|MessagePack|二进制|无需IDL，使用简单|生态弱于Protobuf|小型跨语言服务|

## 六、总结
Linux网络编程中，序列化负责把内存对象转换成可以跨主机传输的字节流，反序列化完成字节流还原。
核心需要解决三大问题：**内存对齐填充、大小端字节序、变长字段边界**。
- 追求开发效率、跨语言：优先Protobuf；
- 追求极致性能、简单结构：手动二进制序列化；
- 调试方便、非高性能场景：JSON。

序列化只是数据编码，**还需要配合消息长度头，解决TCP粘包，才能保证网络通信稳定**。

https://github.com/z088y2yn25/-Portainer/commit/0be8ae7e764c45ae4ece502e2efe258f46a8c534
https://gitlab.com/bronwyn-white-group/portainer/-/commit/53fe5d2e87a4b6280d4b9c0ce64f7bf3d1cbadf6
https://gitlink.org.cn/hao5200/Portainer/tree/master/README.md
https://jihulab.com/blank6314143/portainer/-/blob/main/%E6%9E%81%E7%A9%BA%E9%97%B4%E9%83%A8%E7%BD%B2%20Portainer%20%E5%AE%9E%E6%88%98%EF%BC%9A%E6%8C%82%E8%BD%BD%20Docker%20Socket%E3%80%81%E5%BC%80%E6%94%BE%209000%20%E7%AB%AF%E5%8F%A3%EF%BC%8C%E5%AE%9E%E7%8E%B0%E5%AE%B9%E5%99%A8%E5%8F%AF%E8%A7%86%E5%8C%96%E7%AE%A1%E7%90%86%E4%B8%8E%E5%9B%BA%E5%AE%9A%E5%9C%B0%E5%9D%80%E8%AE%BF%E9%97%AE.md
https://gitcode.com/qweas520/Portainer/blob/main/%E6%9E%81%E7%A9%BA%E9%97%B4%E9%83%A8%E7%BD%B2%20Portainer%20%E5%AE%9E%E6%88%98%EF%BC%9A%E6%8C%82%E8%BD%BD%20Docker%20Socket%E3%80%81%E5%BC%80%E6%94%BE%209000%20%E7%AB%AF%E5%8F%A3%EF%BC%8C%E5%AE%9E%E7%8E%B0%E5%AE%B9%E5%99%A8%E5%8F%AF%E8%A7%86%E5%8C%96%E7%AE%A1%E7%90%86%E4%B8%8E%E5%9B%BA%E5%AE%9A%E5%9C%B0%E5%9D%80%E8%AE%BF%E9%97%AE.md?init=initTree
https://gitlink.org.cn/hao5200/Portainer/commits/94825facc8
https://gitlab.com/bronwyn-white-group/portainer/-/commit/a950768b529ff13666fca23b0e08d45e66057cc4
https://github.com/z088y2yn25/-Portainer/commit/7b8753583c37bd34d6fd88b5918f3a426e4cf5b8
https://gitee.com/interest-in-calculation/jikongjianbushuportainerr/blob/master/NAS%20%E5%AE%B9%E5%99%A8%E7%AE%A1%E7%90%86%E8%BF%9B%E9%98%B6%EF%BD%9C%E6%9E%81%E7%A9%BA%E9%97%B4%E5%AE%89%E8%A3%85%20Portainer%EF%BC%8CDocker%20Socket%20%E6%9D%83%E9%99%90%E9%85%8D%E7%BD%AE%E3%80%819000%20%E7%AB%AF%E5%8F%A3%E6%98%A0%E5%B0%84%E4%B8%8E%E5%9B%BA%E5%AE%9A%20IP%20%E8%AE%BF%E9%97%AE%E6%95%99%E7%A8%8B.md
https://jihulab.com/blank6314143/portainer/-/commit/d6e87b61c9ea573cbc04862636ee83696a71f607
https://jihulab.com/blank6314143/portainer/-/commit/22db945547a26857c384b0c9f24c0ec164d3f05b
https://github.com/z088y2yn25/-Portainer/commit/8ebec48ba6a7e21d3896aa47debd38a349568b4e
https://gitlab.com/bronwyn-white-group/portainer/-/commit/b2c3f250e6f470833828b64777180ab0ab75bf23
https://gitee.com/interest-in-calculation/jikongjianbushuportainerr/blob/master/%E6%9E%81%E7%A9%BA%E9%97%B4%20Portainer%20%E9%83%A8%E7%BD%B2%E5%AE%9E%E6%88%98%EF%BC%8C%E8%AF%A6%E8%A7%A3%20Docker%20Socket%E3%80%819000%20%E7%AB%AF%E5%8F%A3%E4%B8%8E%E5%AE%B9%E5%99%A8%E5%9B%BA%E5%AE%9A%E5%9C%B0%E5%9D%80%E8%AE%BF%E9%97%AE.md
