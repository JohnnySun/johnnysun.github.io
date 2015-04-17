---
layout: post
status: publish
published: true
title: Linux more命令的实现
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 175
wordpress_url: http://libdll.so/?p=175
date: !binary |-
  MjAxNS0wMy0xMyAxODoxNDowMCArMDgwMA==
date_gmt: !binary |-
  MjAxNS0wMy0xMyAxMDoxNDowMCArMDgwMA==
categories:
- C&#47;CPP
- Linux
tags: []
comments: []
---
<p>最近在写toolbox，增加了more命令，发现网上很多人写的more都有bug，我来吧我的发上来，可以通过管道，< ，以及标准文件进行读取，回头还会加入getopt的支持。</p>
<pre class="">#include <stdio.h><br />
#include <unistd.h><br />
#include <string.h><br />
#include <stdlib.h><br />
#include <sys&#47;ioctl.h><br />
#include <sys&#47;stat.h><br />
#include <sys&#47;types.h><br />
#include <termios.h><br />
#include <errno.h></p>
<p>static struct winsize winsz;<br />
static struct termios old, new;</p>
<p>static int use_tty, use_pipe;<br />
static int page_col, page_row;<br />
static char filename[1024];</p>
<p>&#47;* Initialize new terminal i&#47;o settings *&#47;<br />
static void initTermios(int echo)<br />
{<br />
	tcgetattr(0, &old); &#47;* grab old terminal i&#47;o settings *&#47;<br />
	new = old; &#47;* make new settings same as old settings *&#47;<br />
	new.c_iflag &= ~ICRNL; &#47;* Translate carriage return to newline on input (unless IGNCR is set) *&#47;<br />
	new.c_lflag &= ~ICANON; &#47;* disable buffered i&#47;o *&#47;<br />
	new.c_lflag &= echo ? ECHO : ~ECHO; &#47;* set echo mode *&#47;<br />
	tcsetattr(0, TCSANOW, &new); &#47;* use these new terminal i&#47;o settings now *&#47;<br />
}</p>
<p>&#47;* Restore old terminal i&#47;o settings *&#47;<br />
static void resetTermios(void)<br />
{<br />
	tcsetattr(0, TCSANOW, &old);<br />
}</p>
<p>&#47;* Read 1 character - echo defines echo mode *&#47;<br />
static char getch_(int echo)<br />
{<br />
	char ch;<br />
	initTermios(echo);<br />
	ch = getchar();<br />
	resetTermios();<br />
	return ch;<br />
}</p>
<p>&#47;* Read 1 character without echo *&#47;<br />
static char getch(void)<br />
{<br />
	return getch_(0);<br />
}</p>
<p>&#47;* Read 1 character with echo *&#47;<br />
static char getche(void)<br />
{<br />
	return getch_(1);<br />
}</p>
<p>static int usage(char *name) {<br />
	fprintf(stdout, "Usage:\n"<br />
				"%s [options] <file>...\n", name);<br />
	return 1;<br />
}</p>
<p>static int read_more() {<br />
	switch(getch()) {<br />
		case 13:<br />
			page_row++;<br />
			&#47;* Clean the current line *&#47;<br />
			printf("\x1b[1K");<br />
			&#47;* Set the cursor to the start of the line *&#47;<br />
			printf("\x1b[%d;0H",winsz.ws_row);<br />
			break;<br />
		case 32:<br />
			page_row = winsz.ws_row;<br />
			&#47;* Clean the current line *&#47;<br />
			printf("\x1b[1K");<br />
			&#47;* Set the cursor to the start of the line *&#47;<br />
			printf("\x1b[%d;0H",winsz.ws_row);<br />
			break;<br />
		default:<br />
			&#47;* Clean the current line *&#47;<br />
			printf("\x1b[1K");<br />
			&#47;* Set the cursor to the start of the line *&#47;<br />
			printf("\x1b[%d;0H",winsz.ws_row);<br />
			break;</p>
<p>	}<br />
	return 0;<br />
}</p>
<p>static int read_file(FILE *fp, int use_pipe) {<br />
	char line[page_col];<br />
	int seek;<br />
	if(!use_pipe) {<br />
		struct stat filestat;<br />
		lstat(filename, &filestat);<br />
		while(fgets(line, page_col, fp) != NULL) {<br />
			if(page_row != 1) {<br />
				fprintf(stdout,"%s",line);<br />
				&#47;&#47;fflush(stdout);<br />
				page_row--;<br />
			} else {<br />
				seek=ftell(fp);<br />
				int percent = ((float)seek&#47;(float)filestat.st_size) * 100;<br />
				fprintf(stdout, "--More-- (%%%d)",percent);<br />
				read_more();<br />
			}</p>
<p>		}<br />
	} else {<br />
		char **buff;<br />
		int i = 1;<br />
		char line[page_col];<br />
		&#47;* First alloc *&#47;<br />
		buff = (char **)malloc(sizeof(char *));<br />
		while(fgets(line, page_col, fp) != NULL) {<br />
			&#47;* Alloc buff[i-1] *&#47;<br />
			buff[i-1] = (char *)malloc(sizeof(char)*page_col+1);<br />
			strcpy(buff[i-1], line);<br />
			i++;<br />
			&#47;* realloc *&#47;<br />
			buff = (char **)realloc(buff, sizeof(char *)*i);<br />
			&#47;&#47;printf("%s",buff[i-1]);<br />
		}<br />
		if(close(STDIN_FILENO)) {<br />
			perror("STDIN_FILENO close faild");<br />
			return errno;<br />
		}<br />
		if((fp = fopen("&#47;dev&#47;tty","r")) == NULL) {<br />
			perror("Error open TTY");<br />
			return errno;<br />
		}</p>
<p>		int line_num = i -1;<br />
		int l = 0;<br />
		for(i-- ;i > 0; i--) {<br />
			if(page_row != 1) {<br />
				fprintf(stdout,"%s",buff[l]);<br />
				&#47;&#47;fflush(stdout);<br />
				page_row--;<br />
			} else {<br />
				int percent = ((float)l&#47;(float)line_num) * 100;<br />
				fprintf(stdout, "--More-- (%%%d)",percent);<br />
				read_more();<br />
			}<br />
			l++;<br />
		}<br />
		&#47;* Free memory *&#47;<br />
		for(i=0;i<=line_num;i++) {<br />
			free(buff[i]);<br />
		}<br />
		free(buff);<br />
	}<br />
	if(fclose(fp) != 0) {<br />
		perror("Error to close");<br />
		return errno;<br />
	}<br />
	return 0;<br />
}</p>
<p>int more_main(int argc, char *argv[]) {<br />
	use_tty = isatty(STDOUT_FILENO);<br />
	int use_pipe = !isatty(STDIN_FILENO);</p>
<p>	if(use_tty) {<br />
		&#47;*<br />
		 * Get windows size<br />
		 *<br />
		 * struct winsize {<br />
		 *	unsigned short ws_row;<br />
		 *	unsigned short ws_col;<br />
		 *	unsigned short ws_xpixel;<br />
		 *	unsigned short ws_ypixel;<br />
		 *};<br />
		 *&#47;<br />
		if(ioctl(STDOUT_FILENO, TIOCGWINSZ, &winsz) == -1) {<br />
			return errno;<br />
		}</p>
<p>		page_row = winsz.ws_row;<br />
		page_col = winsz.ws_col;</p>
<p>	} else {<br />
		usage(argv[0]);<br />
		return 1;<br />
	}</p>
<p>	&#47;*<br />
	 * Now only support open a file or pipe<br />
	 * So I dont use get opt now.<br />
	 *&#47;<br />
	FILE *fp;</p>
<p>	if(argc != 1) {<br />
		strcpy(filename, argv[1]);</p>
<p>		if((fp = fopen(filename,"r")) == NULL) {<br />
			perror("Error open file");<br />
			return errno;<br />
		}<br />
	}else if(argc == 1 && !use_pipe) {<br />
		usage(argv[0]);<br />
		return 1;<br />
	} else if(use_pipe) {<br />
		if((fp = fdopen(STDIN_FILENO,"r")) == NULL) {<br />
			perror("Error open STDIN_FILENO");<br />
			return errno;<br />
		}<br />
	}</p>
<p>	read_file(fp, use_pipe);</p>
<p>	return 0;<br />
}<br />
<&#47;pre></p>
