---
title: UOS 原生微信多开
date: 2024-08-22
categories:
    - Misc
---


适用目前最新UOS原生微信版本，不适用于其他版本。

### 验证微信版本是否适用本方案

首先打开终端，输入命令，如果找不到终端在哪里，有件点击桌面空白处，应该能看到在终端中打开的选项。输入如下命令

```
ls /opt/apps/
```

如果结果里有
`com.tencent.wechat`，那么大概率可以适用。

### 多开脚本

将以下脚本保存到任意目录

```bash
#!/bin/bash

SANDBOX_HOME=wechat_home_1
WECHAT_PATH=/opt/apps/com.tencent.wechat/files/wechat
mkdir -p ~/$ISOLATED_HOME
bwrap --dev-bind / / \
--bind ~/$ISOLATED_HOME ~/ \
$WECHAT_PATH $@ &
```

保存为`wechat.sh`。
在文件所在文件夹空白处点击右键，“在终端中打开”，打开终端。
输入如下命令

```
chmod +x wechat.sh
```

完成之后，运行脚本

```
./wechat.sh
```

应该就可以看到第二个微信的登录画面了。