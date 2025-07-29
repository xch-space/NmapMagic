# NmapMagic
nmap魔改小计

## 目录结构

```text
NmapMagic
├── nmap
└── nmap-mswin32-aux
```

## 环境配置

- windows 11

- visual studio 2019 C++ 桌面开发环境
- nmap-mswin32-aux
  - svn checkout https://svn.nmap.org/nmap-mswin32-aux
- nmap
  - git clone https://github.com/nmap/nmap.git 

## Nmap魔改

1. ./nmap/tcpip.cc
   tcp->th_win = htons(1024); /* Who cares */

   --> 特征 1024

2. ./nmap/osscan2.cc

   static u8 patternbyte = 0x43; /* character 'C' / 

   memset(data, patternbyte, datalen);
   --> 特征data，多个C，改patternbyte或自定义字符串生成函数

3. ./nmap/nselib/http.lua
   USER_AGENT = stdnse.get_script_args('http.useragent') or "Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)"
   --> 特征Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)，改正常UA即可

4. ./nmap/nmap-service-probes
   nmap，nm@nm，nm2@nm2，nm@p，nm，0PT10NS sip
   --> 特征：nmap，nm@nm，nm2@nm2，nm@p，nm，整体替换为任意字符串即可

   --> 特征：0PT10NS sip 删除该头

5. ./nmap/nselib/* ./nmap/script/*
   --> 特征：Nmap NSE，整体替换为任意字符串即可
   --> 特征：mstshash=nmap，整体替换为任意字符串即可

6. ./nmap/scripts/ssl-heartbleed.nse
   --> 特征：sqlspider，整体替换为任意字符串即可

7. ./nmap/scripts/ssl-heartbleed.nse
   --> 特征：local payload = "Nmap ssl-heartbleed" ，"Nmap ssl-heartbleed"改为随机字符串即可

## ZNmap魔改（存疑）

- src/probe_modules/packet.c （未知目录）
  tcp_header->th_win = htons(65535);
  iph->ip_id = htons(54321);

## 编译

cd NmapMagic\nmap\mswin32

Build.bat build Release

> nmap.exe生成在NmapMagic\nmap\mswin32\Release文件夹中

### bug

- Assertion failed: pinfo->used < pinfo->max_idx, file NmapMagic\nmap\nsock\src\engine_poll.c 380

  ```
  # 临时修复，暂未排查原因
  注释NmapMagic\nmap\nsock\src\engine_poll.c 380行，删除release重新编译即可
  // assert(pinfo->used < pinfo->max_idx);
  ```

## 使用

默认生成的nmap.exe缺失dll，将nmap.exe复制替换已安装的原nmap即可正常使用。

将修改的nselib、script文件夹和nmap-service-probes复制替换到已安装的原nmap。

## 参考文章

[nmap流量特征修改](https://mp.weixin.qq.com/s/qa5gbVU3ydHbBVixx30j6Q)

[al0ne/Nmap_Bypass_IDS: Nmap&Zmap特征识别，绕过IDS探测](https://github.com/al0ne/Nmap_Bypass_IDS)

[如何修改nmap， 重新编译，bypass emergingthreats 的公开ids规则-先知社区](https://xz.aliyun.com/news/5615)

[Windows下完整NMAP源码编译指南](https://mp.weixin.qq.com/s/hh2I-OWpvVji1e_9CjdKNA)
