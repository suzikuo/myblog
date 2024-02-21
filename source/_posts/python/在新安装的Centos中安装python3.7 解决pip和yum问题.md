---
title: 在新安装的Centos中安装python3.7 解决pip和yum问题
tags: 
- python
categories:
- python
date: 2021-03-29 08:45:04
---

<div id="article_content" class="article_content clearfix">
        <link rel="stylesheet" href="https://csdnimg.cn/release/blogv2/dist/mdeditor/css/editerView/ck_htmledit_views-b5506197d8.css">
                <div id="content_views" class="htmledit_views">
                    <p>首先要先安装依赖包：</p> 
<pre class="has" name="code"><code class="hljs sql">yum <span class="hljs-keyword">install</span> zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
</code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p>cd到一个你想放在的地方，哪里都可以。接着找到python3.7的安装包：</p> 
<pre class="has" name="code"><code class="hljs apache"><span class="hljs-attribute">wget</span> https://www.python.org/ftp/python/<span class="hljs-number">3</span>.<span class="hljs-number">7</span>.<span class="hljs-number">0</span>/Python-<span class="hljs-number">3</span>.<span class="hljs-number">7</span>.<span class="hljs-number">0</span>.tgz</code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p>现在这个.tgz文件就下载到了你现在目录中，接着解压：</p> 
<pre class="has" name="code"><code class="hljs apache"><span class="hljs-attribute">tar</span> -zxvf Python-<span class="hljs-number">3</span>.<span class="hljs-number">7</span>.<span class="hljs-number">0</span>.tgz</code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p>然后此目录下就多出了一个解压后的文件Python-3.7.0，下面进入文件夹中：</p> 
<pre class="has" name="code"><code class="hljs go"><ol class="hljs-ln"><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="1"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">cd&nbsp;Python<span class="hljs-number">-3.7</span><span class="hljs-number">.0</span></div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="2"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">./configure</div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="3"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line"><span class="hljs-built_in">make</span>&amp;&amp;<span class="hljs-built_in">make</span> install</div></div></li></ol></code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p>执行完make&amp;&amp;make install之后，可能会出现这种报错：</p> 
<pre class="has" name="code"><code class="hljs crystal">“ModuleNotFound：No <span class="hljs-class"><span class="hljs-keyword">module</span> <span class="hljs-title">named</span> '<span class="hljs-title">_ctypes</span>'”</span></code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p>这里需要执行：</p> 
<pre class="has" name="code"><code class="hljs sql">yum <span class="hljs-keyword">install</span> libffi-devel -y</code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p>执行完继续make&amp;&amp;make install</p> 
<p>这样，基本上python3.7我们就安装完成了，默认情况下，python3.7安装在/usr/local/bin/，这里为了使默认python变成python3，需要加一条软链接,并把之前的python命令改成python.bak：</p> 
<pre class="has" name="code"><code class="hljs groovy"><ol class="hljs-ln"><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="1"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">mv <span class="hljs-regexp">/usr/</span>bin<span class="hljs-regexp">/python /</span>usr<span class="hljs-regexp">/bin/</span>python.bak</div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="2"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">ln -s <span class="hljs-regexp">/usr/</span>local<span class="hljs-regexp">/bin/</span>python3 <span class="hljs-regexp">/usr/</span>bin/python</div></div></li></ol></code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p>接着默认pip也是需要修改的，可以通过find / -name 'pip3'找到pip3的位置，同样的，加一条软链到bin里面：</p> 
<pre class="has" name="code"><code class="hljs groovy"><ol class="hljs-ln"><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="1"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">mv <span class="hljs-regexp">/usr/</span>bin<span class="hljs-regexp">/pip /</span>usr<span class="hljs-regexp">/bin/</span>pip.bak</div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="2"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">ln -s <span class="hljs-regexp">/usr/</span>local<span class="hljs-regexp">/bin/</span>pip3 <span class="hljs-regexp">/usr/</span>bin/pip</div></div></li></ol></code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p>既然把默认python改成了python3的版本，那么这时候yum就出问题了，因为yum貌似不支持python3，开发了这个命令的老哥也不打算继续写支持python3的版本了，所以，如果和python版本相关的文件就不要通过yum下载了，这里我们需要把yum默认的指向改为python2.7的版本，分别是两个文件，使用vi打开，输入i进行修改，修改完之后按esc键，然后输入":wq"，这就完成了修改并保存：</p> 
<pre class="has" name="code"><code class="hljs groovy">vi <span class="hljs-regexp">/usr/</span>libexec/urlgrabber-ext-down
</code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p><img alt="" class="has" height="520" src="https://img-blog.csdn.net/20180809175937778?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyMTQyMTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" width="554"></p> 
<pre class="has" name="code"><code class="hljs groovy">vi <span class="hljs-regexp">/usr/</span>bin/yum</code><div class="hljs-button {2}" data-title="复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4259&quot;}" onclick="hljs.copyCode(event)"></div></pre> 
<p><img alt="" class="has" height="381" src="https://img-blog.csdn.net/20180809180044293?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMyMTQyMTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" width="488"></p> 
<p>自此，我们就完成了新安装的centos系统中的两个python版本的全部流程。</p> 
<p>在小黑框中输入python2则调起python2，输入python，则默认调起python3，pip2调起python2下的pip，pip调起python下的pip。</p>
                </div><div><div></div></div>
        </div>
