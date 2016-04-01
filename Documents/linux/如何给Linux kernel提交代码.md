#如何给Linux Kernel提交代码

**[英文][1]**
## <span id="前言">前言</span>
个人或者公司如果不熟悉提交代码流程，代码很难被接受。利用本文给出一些建议将会大大的增加代码被接受的概率。

在提交代码之前，你需要阅读一些注意事项`Documentation\SubmitChecklist`。如果是提交的驱动，同时也请阅读`Documentation\SubmittingDrivers`。

## 创造并提交改动
###diff -up

利用`diff -up`或者`diff -uprN`创造patch文件。

linux所有的更改都以patche方式存在。patch命令中`-u`代表`unified diff`,`-p`会显示哪个函数发生了改变。patche都是以linux源码的根目录为其实点。
比如你更改了一个文件。你需要进行下面的操作。
```
SRCTREE= linux-2.6
MYFILE= drivers/net/mydriver.c

cd $SRCTREE
cp $MYFILE $MYFILE.orig
vi $MYFILE       # make your change
cd ..
diff -up $SRCTREE/$MYFILE{.orig,} > /tmp/patch
```

如果更改了多个文件，你需要解压一个`vanilla`或者原版linux kernel。
```
MYSRC= /devel/linux-2.6

tar xvfz linux-2.6.12.tar.gz
mv linux-2.6.12 linux-2.6.12-vanilla
diff -uprN -X linux-2.6.12-vanilla/Documentation/dontdiff \
    linux-2.6.12-vanilla $MYSRC > /tmp/patch
```
其中`dontdiff`是diff需要忽略的文件列表。

确保patche中不包含其他无关文件。

如果你的更改比较巨大，可以按照逻辑分成几个部分。这方便其他内核开发者审查。

patch管理工具[Quilt][quilt].一些脚本工具[Scripts][scripts]


###描述更改

描述更改过程中的技术细节，做到尽可能清晰。如果描述过长，可能你需要分成几个patch。

###分割更改

按照逻辑分割patch文件。
比如，你填补了一个驱动的bug同时还提高了它的性能，那么你需要将他分成多个patche。如果你更改了一个API,并提供一个新的驱动利用了该API，这需要分成2个patch。另一方面，如果你只是更改了很多不同文件的同一个地方，你需要将更改放在同一个patch中。如果你的patch比如依赖另外一个patch，你需要在描述patch时说明`this patch depends on patch X`。

###检查代码风格

按照`Documentation/Codingstyle`中的描述，检查更改的编码风格。没有进行这一步的代码可能在甚至没有被阅读情况下被拒绝。最基本的，你可以采用`scripts/checkpatch.pl`检查patch的编码风格，并且修改所有不恰当的地方。

###选择发送者

linux主线的每一部分都由一个维护者负责。更改属于哪一部分就发给对应的维护者。如果维护者没有列出，或者不回应邮件，你可以将patch发给`linux-kernel@vger.kernel.org`。

Linus Torvalds是接受更改的仲裁者，但是一般不要给他发邮件。比较明显的bug修复或者不需要讨论的patch可以直接抄送给Linux Torvalds。如果Patch没有明显的优势或者需要讨论，你需要先发给linux-kernel.

###选择抄送人

如果没有特别原因需要抄送给`linux-kernel@vger.kernel.org`。这样可以让其他开发者可以检查代码和提出意见。

如果你的更改影响了用户和内核接口，请抄送给MAN-PAGES维护者，方便他们做更改。
小的更改：比如字符拼写错误等你需要抄送给`trivial@kernel.org`。
比如在文档中的拼写错误等。关于小错误的定义可以查看[trivial](http://www.kernel.org/pub/linux/kernel/people/juhl/trivial/)。

###只允许文字（没有多媒体，链接，压缩，附件）

所以patch都写在邮件中。

###email的大小

email的大小如果超过40kb，你需要放到网络上，并提供链接。

###指明kernel版本

你需要指明patch的kernel版本，如果patch不适用于最新的kernel，linus不会接受。

###别沮丧，重新提交

提交更改后，需要耐心等待。如果更改没有出现在下一个版本中，那么可能是一下原因：
- 更改不适用于最新内核版本
- 更改没有经过足够讨论
- 代码风格问题
- e-mail格式问题
- 更改存在技术问题
- 意外丢失
- 你太烦了

如果有疑问，可以在linux-kernel邮件列表上询问。

###在邮件主题上加上[PATCH]


###签名

为了方便跟踪作者和管理复杂的更改，引入了签名机制。如果确认一下4点就可以在对patch的描述后加上一句：
Signed-off-by: Random J Developer <random@developer.example.org>
+ 更改部分或者全部由我完成。我有权利开源
+ 更改基于一个开源项目，需提交到同一个开源license下。否则可以提交给别的开源license
+ 更改由别人转交给我，别人确认1,2,3条，我没有修改更改
+ 我知道这是一个开源项目，并且贡献有可能不被记录

###Acked-by

子系统的维护者需要这么做，表示patch你觉得还不错。

###Tested-by和Reviewed-by

Tested-by表示某人在某种环境下patch测试是有效的。
Reviewed-by表示这个patch被检查，说明你对patch的观点。

###patch的规范格式

主题：[PATCH 001/123] 子系统: 总结

```
from:
空行
patch的描述
Signed-off-by:
标识符---
其他一些评论
diff output
```

主题的2个示例：
```
    Subject: [patch 2/5] ext2: improve scalability of bitmap searching
    Subject: [PATCHv2 001/207] x86: fix eflags tracking
```

from行的示例：
```
        From: Original Author <author@example.com>
```

---表示改动日志的结束。
其他一些评论可以用来写上`diffstat -p 1 -w 70`。

###发送git pull请求

发送从哪能够获得更改，格式如下
```
        Please pull from
                git://jdelvare.pck.nerim.net/jdelvare-2.6 i2c-for-linus
         to get these changes:
```
同时用`git diff -M --stat --summary`生成diffstat.

## 忠告、建议、技巧
+ 阅读Documentation/CodingStyle

    可以利用scripts/checkpatch.pl来检查代码风格。这个脚本会提供3种水平的忠告
    + ERROR: 很可能有错
    + WARNING: 需要认真检查
    + CHECK：需要考虑

+ 不要在函数内使用ifdef。
    例如：
    ```
        dev = alloc_etherdev (sizeof(struct funky_private));
        if (!dev)
                return -ENODEV;
        #ifdef CONFIG_NET_FUNKINESS
        init_funky_net(dev);
        #endif
    ```
    上面的可以更改为
    ```
    (in header)
        #ifndef CONFIG_NET_FUNKINESS
        static inline void init_funky_net (struct net_device *d) {}
        #endif
    (in the code itself)
        dev = alloc_etherdev (sizeof(struct funky_private));
        if (!dev)
                return -ENODEV;
        init_funky_net(dev);
    ```

+ static inline 比宏更好

+ 不要过度设计

    Make it as simple as you can, and no simpler.

## 引用

这篇文章翻译于[英文原稿][2].

[回到顶部](#前言)
[1]:http://kernelnewbies.org/UpstreamMerge/SubmittingPatches
[quilt]:http://savannah.nongnu.org/projects/quilt
[scripts]:http://userweb.kernel.org/~akpm/stuff/patch-scripts.tar.gz
[2]:http://kernelnewbies.org/UpstreamMerge/SubmittingPatches