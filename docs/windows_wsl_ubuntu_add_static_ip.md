# Windows 子系统 Wsl2（Ubuntu）中 Docker 设置静态（固定） IP 地址

> 思路：停止 `Wsl2` ，添加静态 `IP` 到 `Wsl2` 中，然后就可以了。  
> 有个问题就是，这个添加固定 `IP` 每次重启 `Wsl2` 之后就会重置，所以把命令效果做成开机自启

### 停止 `Wsl2`
- 打开命令行 Windows + r ，然后输入 services.msc ，找到 LxssManager 进行停止
- windows 有搜索框的直接搜索 “服务” ，找到 LxssManager 进行停止


### 添加静态 `IP`
在 Ubuntu 中添加IP地址 192.168.12.24，名为 eth0:1
访问 Ubuntu 时将使用 192.168.12.24，访问 Win10 时将使用 192.168.12.25。
将以下两行命令保存为 ubuntu-fixed-ip.bat 文件，双击运行。
没有报错提示信息的话，没有问题
然后正常启动 Wsl2
```shell
wsl -d Ubuntu-20.04 -u root ip addr add 192.168.12.24/24 broadcast 192.168.50.255 dev eth0 label eth0:1

netsh interface ip add address "vEthernet (WSL)" 192.168.12.25 255.255.255.0
```

### 将 ubuntu-fixed-ip.bat 文件设置为开机自启
[在 Windows 10 中添加在启动时自动运行的应用](https://support.microsoft.com/zh-cn/windows/%E5%9C%A8-windows-10-%E4%B8%AD%E6%B7%BB%E5%8A%A0%E5%9C%A8%E5%90%AF%E5%8A%A8%E6%97%B6%E8%87%AA%E5%8A%A8%E8%BF%90%E8%A1%8C%E7%9A%84%E5%BA%94%E7%94%A8-150da165-dcd9-7230-517b-cf3c295d89dd)

选择“开始”按钮 ，然后滚动查找你希望在启动时运行的应用。
右键单击该应用，选择“更多”，然后选择“打开文件位置”。此操作会打开保存应用快捷方式的位置。如果没有“打开文件位置”选项，这意味着该应用无法在启动时运行。
文件位置打开后，按 Windows 徽标键  + R，键入“shell:startup”，然后选择“确定”。这将打开“启动”文件夹。
将该应用的快捷方式从文件位置复制并粘贴到“启动”文件夹中。

将 ubuntu-fixed-ip.bat 创建快捷方式到 上面的文件夹中即可