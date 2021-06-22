Cocoapods镜像制作 - Zoor

- [Home](/)
- [Github](https://github.com)
- [V2EX](https://www.v2ex.com/)

MENU![](http://callfiles.ueibo.com/hexo-theme-laughing/post_background.jpg)

# Cocoapods镜像制作1111

- caoye
- 7 Minutes
- November 18, 2017

项目开发过程中避免不了使用第三方，常用的第三方包管理工具有`Carthage`、`Package Manager`、`Cocoapods`,目前为止`Cocoapods`的使用范围比较广，由于是从远程仓库托管到了github上，导致拉取更新比较慢，尤其是用`Cocoapods`做项目插件化的项目， `pod update`、`pod install`的频率比较高，制作一套自己的`Cocoapods`镜像是比较合适的选择。先说一下我自己的方案：  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">我们都知道`Cocoapods `的repo被托管到了github，我们可以将它克隆到本地，把这个</span><br><span class="line">repo定时更新并推到自己服务器上，把项目中依赖的源修改成我们自己服务器上镜像地址。</span><br></pre></td></tr></tbody></table>

我们先安装[Cocoapods](https://cocoapods.org)，安装方法可参考其他博文的[安装方法](http://www.jianshu.com/p/f43b5964f582)

1.  我们先拉取官方库裸库到本地

    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">$ git --bare https://github.com/CocoaPods/Specs.git</span><br></pre></td></tr></tbody></table>

push镜像到我们自己git服务器

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">$ git push --mirror git@code.ds.gome.com.cn:Aeromind-iOS/Specs.git</span><br></pre></td></tr></tbody></table>

我们通过命令打开本地repo仓库  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">$ cd ~/.cocoapods/repos</span><br><span class="line">$ open .</span><br></pre></td></tr></tbody></table>

会发现存在一个`master`分支的仓库,在这个`master`这个仓库里我们能看到fetch和push的地址  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td><td class="code"><pre><span class="line">$ git remote -v</span><br><span class="line">结果:</span><br><span class="line">origin    https://github.com/CocoaPods/Specs.git (fetch)</span><br><span class="line">origin    https://github.com/CocoaPods/Specs.git (push)</span><br></pre></td></tr></tbody></table>

我们看到这个库的fetch和push的地址都是github，我们要把这个master分之作为一个官方库和我们自己服务器镜像库同步的一个桥梁，那么我们接下来需要做一些工作进入master文件夹，其中有个隐藏文件`.git`，我们可以通过显示隐藏文件或者命令找到  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">$ cd ~/.cocoapods/repos/master/.git</span><br><span class="line">$ open .</span><br></pre></td></tr></tbody></table>

在里面会发现有个`config`文件  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">$ vi config</span><br></pre></td></tr></tbody></table>

我们需要在config文件中将push的地址变为我们自己服务器的地址  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br></pre></td><td class="code"><pre><span class="line">[core]</span><br><span class="line">repositoryformatversion = 0</span><br><span class="line">filemode = true</span><br><span class="line">bare = false</span><br><span class="line">logallrefupdates = true</span><br><span class="line">ignorecase = true</span><br><span class="line">precomposeunicode = true</span><br><span class="line">[remote "origin"]</span><br><span class="line">url = https://github.com/CocoaPods/Specs.git</span><br><span class="line">fetch = +refs/heads/*:refs/remotes/origin/*</span><br><span class="line">[branch "master"]</span><br><span class="line">remote = origin</span><br><span class="line">merge = refs/heads/master</span><br><span class="line">pushurl = git@code.ds.code.gome.com.cn:Aeromin-iOS/Specs.git</span><br></pre></td></tr></tbody></table>

在master分支再次查看远端地址  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td><td class="code"><pre><span class="line">$ git remote -v</span><br><span class="line">结果：</span><br><span class="line">origin    https://github.com/CocoaPods/Specs.git (fetch)</span><br><span class="line">origin    git@code.ds.code.gome.com.cn:Aeromin-iOS/Specs.git (push)</span><br></pre></td></tr></tbody></table>

OK，这要我们修改成功了，我们接下来需要做的事就定时去拉取github上的更新，并push到我们自己的服务器上。我是在github上配置了ssh，建议自己git服务器和github都配上ssh，这样git不会受密码变动影响。接下来开始写个简单定时任务脚本  
创建一个`mirror_sync.sh`的文件，然后将这个文件变成可执行文件，文件内容为  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br></pre></td><td class="code"><pre><span class="line"># !/bin/sh</span><br><span class="line"></span><br><span class="line">cd ~/.cocoapods/repos/master</span><br><span class="line">git pull origin master</span><br><span class="line">echo '\033[31m ********* pull finish ^_^ ******** \033[0m'</span><br><span class="line">git push --mirror</span><br><span class="line">echo '\033[31m ********* push finish ^_^ ******** \033[0m'</span><br></pre></td></tr></tbody></table>

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">$ chmod +x mirror_sync.sh</span><br></pre></td></tr></tbody></table>

创建一个log文件`mirror_sync.log`存储log添加crontab定时任务  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">$ crontab -e</span><br><span class="line">添加内容：</span><br><span class="line">* */2 * * * ~/mirror_shell/mirror_sync.sh &gt; ~/mirror_shell/mirror_sync.log 2&gt;&amp;1</span><br></pre></td></tr></tbody></table>

两小时更新一次，这样镜像就基本制作完毕了。最后使用的时候我们需要在`podfile`里把源替换掉  
![屏幕快照](http://upload-images.jianshu.io/upload_images/9045592-13790b0eaa2057ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

var siteName = document.getElementById\('ds-thread'\).getAttribute\('site-name'\); var duoshuoQuery = \{short\_name: siteName\}; \(function\(\) \{ var ds = document.createElement\('script'\); ds.type = 'text/javascript';ds.async = true; ds.src = \(document.location.protocol == 'https:' \? 'https:' : 'http:'\) + '//static.duoshuo.com/embed.js'; ds.charset = 'UTF-8'; \(document.getElementsByTagName\('head'\)\[0\] || document.getElementsByTagName\('body'\)\[0\]\).appendChild\(ds\); \}\)\(\);

- [](/)
- [](</2017/11/18/Framework脚本打包并分发(针对framework依赖了其他包的基础上)/>)

- [](https://github.com/BoizZ "Github")
- [](https://weibo.com/heqibang "Weibo")
- [](https://www.segmentfault.com/u/bon "SegmentFault")

© 2017 Zoor  
POWER BY [HEXO](https://hexo.io), THEME BY [LAUGHING](https://github.com/BoizZ/hexo-theme-laughing)

var wrap = document.getElementById\('wrap'\); window.onload = function \(\) \{ wrap.className += ' done'; \}
