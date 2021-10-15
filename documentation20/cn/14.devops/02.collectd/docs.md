# 使用 TDengine + collectd/StatsD + Grafana 快速搭建 IT 运维监控系统

## 背景介绍
TDengine是涛思数据专为物联网、车联网、工业互联网、IT运维等设计和优化的大数据平台。自从 2019年 7 月开源以来，凭借创新的数据建模设计、快捷的安装方式、易用的编程接口和强大的数据写入查询性能博得了大量时序数据开发者的青睐。

IT 运维监测数据通常都是对时间特性比较敏感的数据，例如：
- 系统资源指标：CPU、内存、IO、带宽等。
- 软件系统指标：存活状态、连接数目、请求数目、超时数目、错误数目、响应时间、服务类型及其他与业务有关的指标。

当前主流的 IT 运维系统通常包含一个数据采集模块，一个数据存储模块，和一个可视化显示模块。collectd / statsD 作为老牌开源数据采集工具，具有广泛的用户群。但是 collectd / StatsD 自身功能有限，往往需要配合 Telegraf、Grafana 以及时序数据库组合搭建成为完整的监控系统。而 TDengine 新版本支持多种数据协议接入，可以直接接受 collectd 和 statsD 的数据写入，并提供 Grafana dashboard 进行图形化展示。

本文介绍不需要写一行代码，通过简单修改几行配置文件，就可以快速搭建一个基于 TDengine + collectd / statsD + Grafana 的 IT 运维系统。架构如下图：

![IT-DevOps-Solutions-Collectd-StatsD.png](../../images/IT-DevOps-Solutions-Collectd-StatsD.png)

## 安装步骤
安装 collectd， StatsD， Grafana 和 TDengine 请参考相关官方文档，这里仅假设使用 Ubuntu 20.04 LTS 为操作系统为例。

### 安装 collectd
```
apt-get install collectd
```
### 安装 StatsD
```
1. git clone https://github.com/etsy/statsd.git
2. cd statsd
3. cp exampleConfig.js config.js
4. node stats.js config.js
```

### 安装 Grafana
```
1. wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
2. echo "deb https://packages.grafana.com/enterprise/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
3. sudo apt-get update
4. sudo apt-get install grafana
5. sudo systemctl start grafana-server.service
```
### 安装 TDengine
从涛思数据官网下载页面 （http://taosdata.com/cn/all-downloads/）最新 TDengine-server 2.3.0.0 版本，以 deb 安装包为例。
```
1. sudo dpkg -i TDengine-server-2.3.0.0-Linux-x64.deb
2. sudo systemctl start taosd
```

## 数据链路设置
### 复制 TDengine 插件到 grafana 插件目录
```
1. sudo cp -r /usr/local/taos/connector/grafanaplugin /var/lib/grafana/plugins/tdengine
2. sudo chown grafana:grafana -R /var/lib/grafana/plugins/tdengine
3. echo -e "[plugins]\nallow_loading_unsigned_plugins = taosdata-tdengine-datasource\n" | sudo tee -a /etc/grafana/grafana.ini
4. sudo systemctl restart grafana-server.service
```

### 配置 collectd
在 /etc/collectd/collectd.conf 文件中增加如下内容后启动 collectd：
```
LoadPlugin network
<Plugin network>
  Server "192.168.17.180" "25826"
</Plugin>

sudo systemctl start collectd
```

### 配置 StatsD
在 config.js 文件中增加如下内容后启动 StatsD：
```
backends 部分添加 "./backends/repeater"
repeater 部分添加 { host:'host to blm3', port: 8126 }
```

### 导入 Dashboard

使用 Web 浏览器访问 IP:3000 登录 Grafana 界面，系统初始用户名密码为 admin/admin。
点击左侧齿轮图标并选择 Plugins，应该可以找到 TDengine data source 插件图标。

#### 导入 collectd 仪表盘

点击左侧加号图标并选择 Import，按照界面提示选择 /usr/local/taos/connector/grafanaplugin/examples/telegraf/grafana/dashboards/telegraf-dashboard-v0.1.0.json 文件，然后应该可以看到如下界面的仪表板界面：

![IT-DevOps-Solutions-collectd-dashboard.png](../../images/IT-DevOps-Solutions-collectd-dashboard.png)

#### 导入 StatsD 仪表盘

点击左侧加号图标并选择 Import，按照界面提示选择 /usr/local/taos/connector/grafanaplugin/examples/collectd/grafana/dashboards/collect-metrics-with-tdengine-v0.1.0.json 文件，然后应该可以看到如下界面的仪表板界面：
![IT-DevOps-Solutions-collectd-dashboard.png](../../images/IT-DevOps-Solutions-collectd-dashboard.png)

## 总结
TDengine 作为新兴的时序大数据平台，具备极强的高性能、高可靠、易管理、易维护的优势。得力于 TDengine 2.3.0.0 版本中新增的 schemaless 协议解析功能，以及强大的生态软件适配能力，用户可以短短数分钟就可以搭建一个高效易用的 IT 运维系统或者适配一个已存在的系统。

TDengine 强大的数据写入查询性能和其他丰富功能请参考官方文档和产品成功落地案例。