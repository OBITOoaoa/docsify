# EPON/GPON组网

EPON 默认 AXU IP 为 192.168.8.100

GPON 默认 AXU IP 为 192.168.8.200

D4：4个PON、8个网口

D8：8个PON、16个网口

G0：4个PON、4个网口

G1：8个PON、8个网口

## EPON组网

光模块有区分：EPON 和 GPON 不能搞反。

组网时，OLT 出现 ping 不通外网的问题，OLT 没有分配上联网的网段和网关。

账号：admin 密码：admin123

访问 OLT：通过 AUX 端口 IP 访问 OLT，浏览器输入 https;\\\ip， ip 地址可以看标签，当不准时可以使用串口线连接 PC 机和 OLT，通过串口查看或修改 AUX 端口 IP。

```shell
enable			
c t				// 进入aux终端
show int aux	// 显示端口ip信息

int aux
ip add ip mask
```

上联口配置：

*   数据带 VLAN，端口用 tag
*   数据不带 VLAN，端口用 untag，PVID 要修改

SFU 和 HGU

SFU 是二级桥接，需要手动拨号，HGU 有路由模式，可以自动拨号。

### 两种环境的组网配置

*   服务器数据流带 VLAN 标签

    *   上联口要设置为 tag
*   服务器数据流不带 VLAN 标签

    *   上联口要设置为 untag，设置业务的 PVID

### ONU状态灯

*   LOS 灯

    *   闪烁：ONU 的 PON 口未收到光
    *   熄灭：ONU 的 PON 口收到光
*   REG灯/PON灯

    *   熄灭：未开始注册
    *   闪烁：正在尝试注册
    *   常亮：注册成功

## GPON组网

电脑不能识别带有 vlan 标签的报文，所以 ONU 配置为 tag，是为了将 olt 的报文到 lan 口转发出去时剥离到 vlan 标签，这样电脑才能正常上网。

GPON 中的 ONU 设置：

*   Tcont：承载业务的 Buffer，需要绑定对应的 DBA 模板来确定带宽分配大小。

*   Gemport：业务的最小承载单位，绑定相应的 Tcont 容器，绑定 Upstream Traffic 模板和 DownStream Traffic 模板，这个模板是针对某个 Gemport。

*   Service 设置 Vlan mode 和 Vlan list 下发，mode 有 tag 或 untag，list 根据 WAN 设置：WAN 带 tag，则 mode 设置为 tag，反之亦然。

*   Service Port：Cvlan、Cvlan Range Transparent：
    *   Cvlan：User Vlan 是 ONU 上来的 vlan，Translate Vlan 是进入 OLT 的 vlan。
    *   Cvlan Range Transparent：Begin Vid 和 End Vid，这个范围的 vid 进行透传。

[GPON介绍及华为OLT网关注册配置流程-CSDN博客](https://blog.csdn.net/zhouzhaoxiong1227/article/details/79244862)

## XG(S)-PON

XG(S)-PON 有模块 SN 码检验，需要提前在 OLT 写入 SN 码，重启生效。
