---
layout: post
status: publish
published: true
title: 详谈POSIX中getopt以及getopt_long的使用
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 201
wordpress_url: http://libdll.so/?p=201
date: !binary |-
  MjAxNS0wMy0xNyAxNjo1Mzo0MiArMDgwMA==
date_gmt: !binary |-
  MjAxNS0wMy0xNyAwODo1Mzo0MiArMDgwMA==
categories:
- C&#47;CPP
- Linux
tags: []
comments:
- id: 79
  author: Low-power
  author_email: 100005963380781@facebook.com
  author_url: ''
  date: !binary |-
    MjAxNS0wMy0yMiAyMjo0NzoyNCArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0yMiAxNDo0NzoyNCArMDgwMA==
  content: Available in GNU and BSD
- id: 80
  author: JohnnySun
  author_email: bmy001@gmail.com
  author_url: http://libdll.so
  date: !binary |-
    MjAxNS0wMy0yMiAyMjo0ODoyOSArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0yMiAxNDo0ODoyOSArMDgwMA==
  content: Yep~~
---
<p>getopt是在开发中很常用的函数，用来获取命令行的参数，使用getopt可以给我们很多方便，免得我们使用大量的strcmp来判断argv。</p>
<p>函数的原型如下：</p>
<pre class="">int getopt(int argc, char * const argv[],<br />
                  const char *optstring);</p>
<p>       extern char *optarg;<br />
       extern int optind, opterr, optopt;</p>
<p>       #include <getopt.h></p>
<p>       int getopt_long(int argc, char * const argv[],<br />
                  const char *optstring,<br />
                  const struct option *longopts, int *longindex);</p>
<p>       int getopt_long_only(int argc, char * const argv[],<br />
                  const char *optstring,<br />
                  const struct option *longopts, int *longindex);<br />
<&#47;pre><br />
其中getopt市只能用来获取短参数，例如-h，而getopt_long可以获取短参数也可以获取长参数（例如 ---help）。getopt_long_only 则顾名思义，只能获取长参数。</p>
<p>&nbsp;</p>
<p>这里我们重点介绍get_opt &amp; get_opt_long.</p>
<p>先来说说返回值：如果getopt执行成功，他将会返回option character，如果没有option了，则返回-1</p>
<p>这是一段基本的示例代码</p>
<pre class="">       #include<br />
       #include<br />
       #include </p>
<p>       int<br />
       main(int argc, char *argv[])<br />
       {<br />
           int flags, opt;<br />
           int nsecs, tfnd;</p>
<p>           nsecs = 0;<br />
           tfnd = 0;<br />
           flags = 0;<br />
           while ((opt = getopt(argc, argv, "nt:")) != -1) {<br />
               switch (opt) {<br />
               case 'n':<br />
                   flags = 1;<br />
                   break;<br />
               case 't':<br />
                   nsecs = atoi(optarg);<br />
                   tfnd = 1;<br />
                   break;<br />
               default: &#47;* '?' *&#47;<br />
                   fprintf(stderr, "Usage: %s [-t nsecs] [-n] name\n",<br />
                           argv[0]);<br />
                   exit(EXIT_FAILURE);<br />
               }<br />
           }</p>
<p>           printf("flags=%d; tfnd=%d; optind=%d\n", flags, tfnd, optind);</p>
<p>           if (optind >= argc) {<br />
               fprintf(stderr, "Expected argument after options\n");<br />
               exit(EXIT_FAILURE);<br />
           }</p>
<p>           printf("name argument = %s\n", argv[optind]);</p>
<p>           &#47;* Other code omitted *&#47;</p>
<p>           exit(EXIT_SUCCESS);<br />
       }<br />
<&#47;pre><br />
这里重点解释下optind。 optind的值是第一个非options的argc，这里还有一点很有趣，在执行getopt时会将argc重新排序。将options放到前面，非options放到后面。 例如： command <file> -c 会排序成为 command -c <file>,这时的optind就等于排序后file的argc == 3。这一点值得大家注意。</p>
<p class="">上面的示例代码里用了flags，可以通过判断flags的真假来判断用户是否输入了 options 。实际应用中，有一个好用的方式来判断用户是否输入了非选项参数<&#47;p></p>
<pre class="">if (optind == argc) {<br />
    &#47;* Code *&#47;<br />
}<&#47;pre><br />
再来简单的介绍下getopt_long</p>
<p>首先使用getopt_long 需要下面这个结构体</p>
<pre class="">static struct option long_options[] = {<br />
    {"add",   required_argument, 0,  0 },<br />
    {"help",  no_argument,       0,  'h' },<br />
    {"file",  required_argument, 0,  'f' },<br />
    {0,       0,                 0,  0 }<br />
};<&#47;pre><br />
这里面的required_argument意味着后面需要接参数，最后一个0意味着不与短选项相对应。让我们看下示例代码</p>
<pre class="">#include      &#47;* for printf *&#47;<br />
       #include     &#47;* for exit *&#47;<br />
       #include </p>
<p>       int<br />
       main(int argc, char **argv)<br />
       {<br />
           int c;<br />
           int digit_optind = 0;</p>
<p>           while (1) {<br />
               int this_option_optind = optind ? optind : 1;<br />
               int option_index = 0;<br />
               static struct option long_options[] = {<br />
                   {"add",     required_argument, 0,  0 },<br />
                   {"append",  no_argument,       0,  0 },<br />
                   {"delete",  required_argument, 0,  0 },<br />
                   {"verbose", no_argument,       0,  0 },<br />
                   {"create",  required_argument, 0, 'c'},<br />
                   {"file",    required_argument, 0,  0 },<br />
                   {0,         0,                 0,  0 }<br />
               };</p>
<p>               c = getopt_long(argc, argv, "abc:d:012",<br />
                        long_options, &amp;option_index);<br />
               if (c == -1)<br />
                   break;</p>
<p>               switch (c) {<br />
               case 0:<br />
                   printf("option %s", long_options[option_index].name);<br />
                   if (optarg)<br />
                       printf(" with arg %s", optarg);<br />
                   printf("\n");<br />
                   break;</p>
<p>               case '0':<br />
               case '1':<br />
               case '2':<br />
                   if (digit_optind != 0 &amp;&amp; digit_optind != this_option_optind)<br />
                     printf("digits occur in two different argv-elements.\n");<br />
                   digit_optind = this_option_optind;<br />
                   printf("option %c\n", c);<br />
                   break;</p>
<p>               case 'a':<br />
                   printf("option a\n");<br />
                   break;</p>
<p>               case 'b':<br />
                   printf("option b\n");<br />
                   break;</p>
<p>               case 'c':<br />
                   printf("option c with value '%s'\n", optarg);<br />
                   break;</p>
<p>               case 'd':<br />
                   printf("option d with value '%s'\n", optarg);<br />
                   break;</p>
<p>               case '?':<br />
                   break;</p>
<p>               default:<br />
                   printf("?? getopt returned character code 0%o ??\n", c);<br />
               }<br />
           }</p>
<p>           if (optind < argc) {<br />
               printf("non-option ARGV-elements: ");<br />
               while (optind < argc)<br />
                   printf("%s ", argv[optind++]);<br />
               printf("\n");<br />
           }</p>
<p>           exit(EXIT_SUCCESS);<br />
       }<br />
<&#47;pre><br />
与短选项对应的好办，检测到这些长选项时会返回其对应的短选项，可是不对应的呢？</p>
<p>这里就要用到option_index了，比如上面的源代码中有很多不对应短选项的，那么我们如何判断呢？ 方法就是</p>
<pre class="">case 0：<br />
    if(opt_index == 0) &#47;*code *&#47;<br />
    if(opt_index == 1) &#47;*code *&#47;<br />
    break;<&#47;pre><br />
option long_options[]结构体里的第一项是0，这样以此类推，可以轻易地确定用户输入的long_option。</p>
<p>&nbsp;</p>
