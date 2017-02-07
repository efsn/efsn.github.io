---
layout: post
title: "shell脚本实现的网站日志分析统计"
category: Transfer
tags: [Shell, Log]
---
* TOC
{:toc}
## shell脚本实现的网站日志分析统计
> 可以统计9种数据

1. 总访问量
2. 总带宽
3. 独立访客量
4. 访问IP统计
5. 访问url统计
6. 来源统计
7. 404统计
8. 搜索引擎访问统计(谷歌，百度)
9. 搜索引擎来源统计(谷歌，百度)

> 需要修改的三个变量log_path,domain和email，然后把此脚本添加到计划任务，就可以每天接收到统计的数据了

```shell
#!/bin/bash
    log_path=/home/www.jb51.net/log/access.log.1
    domain="jb51.net"
    email="log@jb51.net"
    maketime=`date +%Y-%m-%d" "%H":"%M`
    logdate=`date -d "yesterday" +%Y-%m-%d`
    total_visit=`wc -l ${log_path} | awk '{print $1}'`
    total_bandwidth=`awk -v total=0 '{total+=$10}END{print total/1024/1024}' ${log_path}`
    total_unique=`awk '{ip[$1]++}END{print asort(ip)}' ${log_path}`
    ip_pv=`awk '{ip[$1]++}END{for (k in ip){print ip[k],k}}' ${log_path} | sort -rn | head -20`
    url_num=`awk '{url[$7]++}END{for (k in url){print url[k],k}}' ${log_path} | sort -rn | head -20`
    referer=`awk -v domain=$domain '$11 !~ /http:\/\/[^/]*'"$domain"'/{url[$11]++}END{for (k in url){print url[k],k}}' ${log_path} | sort -rn | head -20`
    notfound=`awk '$9 == 404 {url[$7]++}END{for (k in url){print url[k],k}}' ${log_path} | sort -rn | head -20`
    spider=`awk -F'"' '$6 ~ /Baiduspider/ {spider["baiduspider"]++} $6 ~ /Googlebot/ {spider["googlebot"]++}END{for (k in spider){print k,spider[k]}}' ${log_path}`
    search=`awk -F'"' '$4 ~ /http:\/\/www\.baidu\.com/ {search["baidu_search"]++} $4 ~ /http:\/\/www\.google\.com/ {search["google_search"]++}END{for (k in search){print k,search[k]}}' ${log_path}`
    echo -e "概况\n报告生成时间：${maketime}\n总访问量:${total_visit}\n总带宽:${total_bandwidth}M\n独立访客:${total_unique}\n\n访问IP统计\n${ip_pv}\n\n访问url统计\n${url_num}\n\n来源页面统计\n${referer}\n\n404统计\n${notfound}\n\n蜘蛛统计\n${spider}\n\n搜索引擎来源统计\n${search}" | mail -s "$domain $logdate log statistics" ${email}
```
