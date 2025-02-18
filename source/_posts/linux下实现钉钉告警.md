---
title: linux下实现钉钉告警
date: 2024-07-23 11:45:38
categories: linux
tag: 报警
---

在Linux系统中，可以使用Shell脚本和定时任务（cron）来监控CPU、内存和硬盘使用情况，当超过指定阈值时发送告警。为了发送钉钉告警，需要用到钉钉的自定义机器人接口。
需要注意的是，本教程的机器是12核心。因此需要得到12核心的最高cpu使用率后取最大值。

以下是实现此功能的步骤：

1. **创建钉钉机器人并获取Webhook URL：**
    - 登录钉钉，创建自定义机器人并获取Webhook URL，用于发送告警信息。
    
2. **编写Shell脚本：**
    - 编写一个Shell脚本来检查系统资源的使用情况，并在超过阈值时发送钉钉告警。
    
3. **配置定时任务（cron）：**
    - 将脚本配置为定时任务，定期检查系统资源的使用情况。

### 1. 创建钉钉机器人并获取Webhook URL

在钉钉中创建自定义机器人，记录下Webhook URL，这将在脚本中用于发送告警。
- 例如：https://oapi.dingtalk.com/robot/send?access_token=073d84ae5a2f83494c9271e0e1683603130b19e77620a836e3682a62ffd1a8f5

### 2. 编写Shell脚本

下面是一个示例Shell脚本`monitor.sh`：

```sh
#!/bin/bash

# 获取cpu使用率
function get_cpu_usage(){
    echo $(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}' | awk '{printf "%.2f", $1}')
}

# 获取磁盘使用率
data_name="/" 
diskUsage=$(df -h | grep -w $data_name | awk '{print $5}' | sed 's/%//')

# 获取内存情况
mem_total=$(free -m | awk 'NR==2 {print $2}')
mem_used=$(free -m | awk 'NR==2 {print $3}')

# 统计内存使用率
mem_used_percent=$(echo "scale=2; ($mem_used / $mem_total) * 100" | bc)

# 获取报警时间
now_time=$(date '+%F %T')

user="18857415467"

# 主机信息
Date_time=$(date "+%Y-%m-%d--%H:%M:%S")
IP_addr=$(hostname -I | awk '{print $1}')

# webhook url
Dingding_Url="https://oapi.dingtalk.com/robot/send?access_token=073d84ae5a2f83494c9271e0e1683603130b19e77620a836e3682a62ffd1a8f5"

function SendDownMessageToDingding(){
    # 发送钉钉消息
    curl -s "${Dingding_Url}" -H 'Content-Type: application/json' -d "
    {
     'msgtype': 'text',
     'text': {'content': '资源耗尽警告！\n巡查时间：${Date_time}\nIP地址：${IP_addr}\n资源状况如下:\n【CPU使用率：${cpuUsage}%】\n【磁盘使用率：${diskUsage}%】\n【内存使用率：${mem_used_percent}%】'},
     'at': {'atMobiles': ['${user}'], 'isAtAll': true}
      }"
}

function check(){
    cpuUsage=$(get_cpu_usage)
    if (( $(echo "$cpuUsage > 90" | bc -l) )); then
        echo "检测到CPU使用率高于90%，开始1分钟监控..."
        high_cpu_duration=0
        for ((i=0; i<60; i++)); do
            sleep 1
            cpuUsage=$(get_cpu_usage)
            if (( $(echo "$cpuUsage > 90" | bc -l) )); then
                ((high_cpu_duration++))
            else
                high_cpu_duration=0
            fi

            if (( high_cpu_duration >= 60 )); then
                echo "CPU使用率持续高于90%超过1分钟，发送警报..."
                SendDownMessageToDingding
                break
            fi
        done
    fi

    if (( $(echo "$diskUsage > 80" | bc -l) )) || (( $(echo "$mem_used_percent > 80" | bc -l) )); then
        SendDownMessageToDingding
    fi
}

check

```

将`073d84ae5a2f83494c9271e0e1683603130b19e77620a836e3682a62ffd1a8f5`替换为你从钉钉获取的Webhook URL的token。

### 3. 配置定时任务（cron）

通过cron定期运行这个脚本。首先，编辑cron配置：

```sh
crontab -e
```

然后添加以下内容以每分钟运行一次脚本：

```sh
* * * * * /path/to/monitor.sh
```

确保脚本`monitor.sh`有执行权限：

```sh
chmod +x /path/to/monitor.sh
```

### 总结

上述脚本和cron配置将监控CPU、内存和硬盘使用情况，并在超过设定的阈值时通过钉钉发送告警消息。你可以根据实际需求调整脚本的执行频率和告警阈值。