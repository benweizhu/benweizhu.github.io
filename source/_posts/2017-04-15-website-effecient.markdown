---
layout: post
title: "前端不止系列 - 时间都去哪了？"
date: 2017-04-15 18:58:04 +0800
comments: true
categories:
---
只有10%~20%的最终用户响应时间花在了下载HTML文档上，其余的80%~90%时间花在了下载页面中的所有组件上。   - 性能黄金法则，Steve Souders

![Alt text](/images/performance/golden-top10.png =400x "前十名网站")     

性能黄金法则由《高性能网站建站指南》的作者Steve Souders在2007年提出。在2012年，他重新发表了一篇博客[《The Performance Golden Rule》][0369e540]，分析并统计排名前10，10个在10000排名左右网站的加载时间，并计算了在[HTTP Archive][5375bd2a]上被抓取到的50000个网站的前后端耗时占比，而最终验证了自2007年提出的这个理念的准确性。

  [0369e540]: https://www.stevesouders.com/blog/2012/02/10/the-performance-golden-rule/ "《The Performance Golden Rule》"
  [5375bd2a]: http://httparchive.org/index.php "HTTP Archive"
