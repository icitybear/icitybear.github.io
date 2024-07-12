---
title: "google的ab实验" #标题
date: 2024-05-28T11:35:48+08:00 #创建时间
lastmod: 2024-05-28T11:35:48+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 基础
keywords: 
- 
description: "" #描述 每个文章内容前面的展示描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论 有自带的扩展成twikoo
showToc: true # 显示目录 文章侧边栏toc目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false

# reward: true # 打赏
mermaid: true #自己加的是否开启mermaid
---

[文章解读](https://blog.csdn.net/weixin_36894490/article/details/118733380)

通过实验的对比指标来衡量这些修改对用户的影响程度，以此来提高用户体验，提高公司收益等等。
参考google经典论文《Overlapping Experiment Infrastructure:More, Better, Faster Experimentation》第4部分，阐述谷歌等大厂都是如何高效地做ab实验的

# 想到的实验方案
1. 方案1）将用户分成两半，一半做对照组，一半做实验组

- 缺点：一次只能做一个实验，假如想要同时做两个实验，将无法满足

2. 方案2）将用户分桶，例如通过用户ID取模分成1000个桶，一些桶做基线，一些桶做实验

- 缺点：如果桶分的太少，同一时间可以做的实验还是受到限制。如果桶分的太多，每个实验桶的流量将会变少，置信度将会降低，需要更长的实验时间。

假如我们的<font color="red">实验涉及到多个系统服务，这些系统服务之间是有关联的。</font>以一个新闻推荐系统为例，我们知道，当系统接受到一个用户请求的时候，首先会去召回系统里面取出候选的新闻，然后再将这些候选的新闻发送到精排系统进行排序，假如召回系统和精排系统同时在做ab实验，**怎么消除这两个系统的相互影响？**

- 缺点是一个用户请求只能进行一个实验，那假如一个用户请求我们同时进行N个实验呢？如果一个用户请求同时进行N个实验，我们怎么排除这些实验之间相互干扰？答案就是使用正交实验

![alt text](image1.png)

# 正交实验

假如我们要做一个关于UI界面的实验。实验1为将按钮的颜色设置为红色（20%流量）和蓝色（80%流量），实验2将页面背景设置为白色（50%流量）或黑色（50%流量），正交实验如下图所示
![alt text](image2.png)
我们只要将实验1的用户像洗牌一样重新打散，均匀地分布在实验2里面，就可以消除实验1对实验2的影响。同时，每个用户请求是同时进行进行2个实验的（按钮为红/蓝，背景白色/黑色）。利用正交实验，我们的每个实验都可以利用到全部的流量。

## 怎么将实验1的用户均匀地分布到实验2呢？
答案就是在hash的时候加一个前缀。例如通过使用函数 hash(实验ID+用户ID)%1000，将（实验1的ID+用户ID）取模1000后小于200的用户的按钮颜色设置为红色，大于等于200的设置为蓝色，同时，将（实验2的ID+用户ID）取模1000后小于500的用户背景设置为白色，大于等于500的用户背景设置为黑色即可

- <font color="red">为了确保不同层的实验独立转移，谷歌使用 mod = f(cookie, layer) % 1000 代替。</font>
- AB实验+hash分流+大数定律 应该是天然的正交
# 谷歌分层重叠实验框架
在正交实验的基础上，通过对流量的切割，以及分层重叠嵌套，便可设计出更为灵活的AB实验框架。
## 概念(直接看得物的)
1) domain：域，是流量的分段。全部流量被切割之后的一段流量  

2）layer：层， 在layer里面包含一系列可以改变的参数。例如上面的实验可以分成2个layer，layer1对应实验1，layer2对应实验2。

3）experiment：<font color="red">实验， 在layer里面可以添加桶</font>，例如通过 hash(layerId + userId)%1000 , 然后把实验放入桶中。

- 1个domain可以有多个重叠的layer，1个layer反过来也可以嵌套多个domain，实验最终落入到layer里面的bucket里面。
- <font color="red">各个layer之间的实验是要独立的。</font>例如layer1中的实验是设置按钮为白色和黑色，layer3的实验是设置按钮为白色和红色，这样的两个layer之间就不是独立的了，那么正交性就会遭到破坏。需要特别注意这一点。
  - 如果layer之间相关，也该怎么办呢？合并成一个吗（比两个实验颜色对比，改成三个颜色同时对比）？答案：<font color="red">两个颜色或三个颜色对比 应该放在同一层 层里面有再分桶 桶里面放不同的颜色</font>
  
## 案例
![alt text](image3.png)
在(a)中，只有1个domain，domain里面嵌套了3个layer。当用户请求过来的时候，会依次经过UI Layer，Search results layer和Ads result layer，在各个layer里面通过 hash(layerId + userId)%1000 映射到对应的桶取出相应的experiment。<font color="red">因此1个用户请求最多会同时进行3个实验</font>

在(b)中，流量被切割成2个domain，1个domain只有1个layer，另一个domain有3个layer。当用户请求被分配到domain1的时候，最多将会进行1个实验，当用户请求被分配到domain2的时候，用户最多将会进行3个实验。

比如复杂的
![alt text](image4.png)


## 启动层
启动层始终包含在默认域中（即，它们在所有流量上运行）。启动层中的实验为参数提供了替代默认值。

# hash算法
[DEK Hash 和 Murmur Hash](https://zhuanlan.zhihu.com/p/648347825) 
DEK Hash 的缺陷

- github.com/spaolacci/murmur3 现有的包
``` go 
import (
    "fmt"
    "github.com/spaolacci/murmur3"
)

func main() {
    text := "Hello, World!"
    hashValue := murmur3.Sum32([]byte(text))
    fmt.Printf("Hash of '%s': 0x%x\n", text, hashValue)
}
``` 
- 离散性验证
``` go 
import (
	"math/rand"
	"github.com/spaolacci/murmur3"
)

func TestParse(t *testing.T) {
	scale := 50 //概率
	// 演示随机生成1w次hash的分布情况
	hit1 := 0
	hit2 := 0
	uhit1 := 0
	uhit2 := 0
	for i := 0; i < 100; i++ {
		text := randStr() // 生产的随机字符串
		hash := murmur3.Sum32([]byte(text))
		fmt.Printf("str: %s, Hash: %d\n", text, hash)
		// 模是1000的情况
		if hash%1000 < uint32(scale)*10 {
			hit1 += 1
		} else {
			uhit1 += 1
		}
		// 模是100的情况
		if hash%100 < uint32(scale) {
			hit2 += 1
		} else {
			uhit2 += 1
		}
	}
    // 不管模取多少 循环大于100后 概率都是接近scale
	fmt.Printf("hit1: %d, uhit1: %d \n hit2: %d, uhit2: %d\n", hit1, uhit1, hit2, uhit2)
}

func randStr() string {
	n := 32
	const charset = "abcdefghijklmnopqrstuvwxyz0123456789"
	rand.Seed(time.Now().UnixNano())

	b := make([]byte, n)
	for i := range b {
		b[i] = charset[rand.Intn(len(charset))]
	}
	return string(b)
}
```

- 实现MurmurHash算法在Go语言中通常涉及较复杂的位操作和数学计算
``` go 
package main
import (
    "encoding/binary"
    "fmt"
)
//  Go 语言中实现 MurmurHash3 算法（32-bit 版本） MurmurHash算法的不同变体（例如32位、64位、128位版本）需要不同的实现。
func murmur3_32(key []byte, seed uint32) uint32 {
    const (
        c1 uint32 = 0xcc9e2d51
        c2 uint32 = 0x1b873593
        r1 uint32 = 15
        r2 uint32 = 13
        m  uint32 = 5
        n  uint32 = 0xe6546b64
    )

    hash := seed
    length := len(key)
    roundedEnd := (length & 0xfffffffc) // round down to 4 byte block

    for i := 0; i < roundedEnd; i += 4 {
        k1 := binary.LittleEndian.Uint32(key[i : i+4])
        k1 *= c1
        k1 = (k1 << r1) | (k1 >> (32 - r1))
        k1 *= c2

        hash ^= k1
        hash = ((hash << r2) | (hash >> (32 - r2))) * m + n
    }

    if length > roundedEnd {
        tail := uint32(0)
        switch length & 3 {
        case 3:
            tail |= uint32(key[roundedEnd+2]) << 16
            fallthrough
        case 2:
            tail |= uint32(key[roundedEnd+1]) << 8
            fallthrough
        case 1:
            tail |= uint32(key[roundedEnd])
            tail *= c1
            tail = (tail << r1) | (tail >> (32 - r1))
            tail *= c2
            hash ^= tail
        }
    }

    hash ^= uint32(length)
    hash ^= hash >> 16
    hash *= 0x85ebca6b
    hash ^= hash >> 13
    hash *= 0xc2b2ae35
    hash ^= hash >> 16

    return hash
}

func main() {
    text := "Hello, World!"
    hashValue := murmur3_32([]byte(text), 0)
    fmt.Printf("Hash of '%s': 0x%x\n", text, hashValue)
}
```
## hash算法也用于布隆过滤器
[布隆过滤器如何实现? - 小徐先生的回答 - 知乎](https://www.zhihu.com/question/389604738/answer/3152180842)

# 案例用于投放广告系统的设计

https://o15vj1m4ie.feishu.cn/wiki/D58lwXOAoisY8TkSbr8cQvIrn4d

# 得物
[得物技术浅谈AB实验设计实现与分流算法](https://segmentfault.com/a/1190000039180775)
<font color="red">得物系列技术文章</font>