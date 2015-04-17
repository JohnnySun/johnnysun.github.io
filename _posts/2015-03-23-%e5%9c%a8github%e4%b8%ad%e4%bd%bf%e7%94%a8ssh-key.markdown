---
layout: post
status: publish
published: true
title: 在github中使用ssh key
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 215
wordpress_url: http://libdll.so/?p=215
date: !binary |-
  MjAxNS0wMy0yMyAxMjoyMDo1NyArMDgwMA==
date_gmt: !binary |-
  MjAxNS0wMy0yMyAwNDoyMDo1NyArMDgwMA==
categories:
- 未分类
tags: []
comments:
- id: 81
  author: 科技爱好者
  author_email: admin@lxx1.com
  author_url: http://blog.lxx1.com
  date: !binary |-
    MjAxNS0wNC0xMCAxMDowMDo1NyArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wNC0xMCAwMjowMDo1NyArMDgwMA==
  content: 这样不仅方便，而且也很安全，不用担心密码被非法截取。
---
<h2 style="text-align: left;"><&#47;h2></p>
<h2 style="text-align: left;">在github中，默认我们使用的是http来进行git，这样很方便。但是也有很大的缺点，那就是每次都要输入用户名和密码。这真的很折腾，那么用ssh key管理github就成了我们不二的选择。<&#47;h2><br />
下面介绍下如何使用sshkey：</p>
<p>创建SSH key的方法很简单，执行如下命令就可以：</p>
<p>ssh-keygen</p>
<p>然后系统提示输入文件保存位置等信息，连续敲三次回车即可，生成的SSH key文件保存在中～&#47;.ssh&#47;id_rsa.pub</p>
<p>这时使用文本编辑器打开该文件，复制里面的所有内容</p>
<p>打开github帐号管理中的添加SSH key界面,步骤如下：</p>
<p>1. 登录github</p>
<p>2. 点击右上方的Accounting settings图标</p>
<p>3. 选择 SSH key</p>
<p>4. 点击 Add SSH key</p>
<p>名字自己起一个喜欢的就好，把刚刚的内容粘贴到key一栏，保存。 大功告成。</p>
<p>之后呢，用ssh方式克隆工程，就可以使用sshkey管理拉。</p>
