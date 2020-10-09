---
title: hexo 踩过的坑
date: 2019-09-21 22:11:47
categories:
- 编程
tags:
- hexo

keywords: hexo,443
---
  * 最近在使用hexo d时一直报下面的错误：
  fatal: unable to access 'https://github.com/yanisoung/xy-blog.github.io/': Failed to connect to github.com port 443: Timed out
FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
Error: Spawn failed
    at ChildProcess.<anonymous> (D:\develop\hexo-home\node_modules\hexo-util\lib\spawn.js:52:19)
    at ChildProcess.emit (events.js:198:13)
    at ChildProcess.cp.emit (D:\develop\hexo-home\node_modules\cross-spawn\lib\enoent.js:40:29)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:248:12)

  <!-- more -->

  *   网上替换.deploy_git文件夹的方法试过了，还是无果。
  *   后在此博客寻得解决方案：https://www.yuanchunrong.com/hexo-github-blog/

  *   使用命令行$ hexo g -d时，确保有网情况下一次性上传github成功，不要中途中断，否则会出现两个分支没有同时同步的情况，下次同步时就会报上面的错误。
  *  目前的粗暴的解决方法是：把github的yuanchunrong.github.io仓库删除，同时把本地隐藏文件夹.deploy_git/删除，重新使用命令行$ hexo g -d同步文件夹，方法简单有效粗暴，但是比较伤元气，所以以后操作的时候注意一下．

  *  经过此次惨痛的教训，定不再使用备份命令，毕竟科学上网还是需要时间的。






