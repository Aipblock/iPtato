# iPtato
通过简单的脚本，实现控制系统的出入网络流量
## 功能简介
### 适配系统
Debian\Centos\Ubuntu
### 首次运行
- 创建空文件 用于判断脚本是否首次运行
- 部署需要的软件
- 清除所有自带防火墙/规则
- 禁止一切入网方向端口(即关闭所有连接服务器的端口)
- 默认开放SSH入网端口

### 出网模块功能
> 封禁功能(即屏蔽)
- 封禁BT、PT、SPAM
- 封禁黑名单网址关键词
- 自定义封禁关键词（支持：手动输入/本地文件/在线URL 导入）
> 解禁
- 有封就有解，可以跑脚本感受一下

### 入网模块(连接服务器)
> 放行(即开放)
- 入网端口
- 入网IP (计划中)

> 取消放行(即不开放)
- 入网端口
- 入网IP(计划中)


## 使用脚本
```ssh
wget -N --no-check-certificate https://raw.githubusercontent.com/Aipblock/iPtato/main/iPtato.sh && chmod +x iPtato.sh && bash iPtato.sh
```

### 跑过脚本后想继续用
```ssh
./iPtato.sh
```

## Function
![image](https://user-images.githubusercontent.com/113791222/191405756-4a7a98fe-6302-4299-b512-6052fa28cf19.png)



