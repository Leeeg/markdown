---
title: Stable Diffusion 入门（一）
date: 2023-03-31 11:37:02
top: true
cover: true
tags:
    - Stable Diffusion
    - AI
---

# Stable Diffusion

##  历史
    Stable Diffusion 算法上来自 CompVis 和 Runway 团队于 2021 年 12 月提出的 “潜在扩散模型”（LDM / Latent Diffusion Model），这个模型又是基于 2015 年提出的扩散模型（DM / Diffusion Model）。参考论文中介绍算法核心逻辑的插图，Stable Diffusion 的数据会在像素空间（Pixel Space）、潜在空间（Latent Space）、条件（Conditioning）三部分之间流转，其算法逻辑大概分这几步（可以按 ↩️ 顺序对照下图）：

## 模型 
图像编码器将图像从像素空间（Pixel Space）压缩到更小维度的潜在空间（Latent Space），捕捉图像更本质的信息；对潜在空间中的图片添加噪声，进行扩散过程（Diffusion Process）；通过 CLIP 文本编码器将输入的描述语转换为去噪过程的条件（Conditioning）；基于一些条件对图像进行去噪（Denoising）以获得生成图片的潜在表示，去噪步骤可以灵活地以文本、图像和其他形式为条件（以文本为条件即 text2img、以图像为条件即 img2img）；图像解码器通过将图像从潜在空间转换回像素空间来生成最终图像。

![](../Stable-Diffusion-入门（一）/FBB7424D-F9A4-4D16-9077-3F8A4974215D.png)

## 扩散模型
扩散模型（DM / Diffusion Model）
“扩散” 来自一个物理现象：当我们把墨汁滴入水中，墨汁会均匀散开；这个过程一般不能逆转，那 AI 可以做到么？（AI：我太难了）

![](../Stable-Diffusion-入门（一）/565341C3-37CB-47AD-A9CC-D3571341C2CF.png)
当墨汁刚滴入水中时，我们能区分哪里是墨哪里是水，信息是非常集中的；当墨汁扩散开来，墨和水就难分彼此了，信息是分散的。类比于图片，这个墨汁扩散的过程就是图片逐渐变成噪点的过程：从信息集中的图片变成信息分散、没有信息的噪点图很简单，逆转这个过程就需要 AI 的加持了。
研究人员对图片加噪点，让图片逐渐变成纯噪点图；再让 AI 学习这个过程的逆过程，也就是如何从一张噪点图得到一张有信息的高清图。这个模型就是 AI 绘画中各种算法，如 Disco Diffusion、Stable Diffusion 中的常客扩散模型（Diffusion Model）。

![](../Stable-Diffusion-入门（一）/577E8A52-5E58-47F8-AA85-9659435A51D9.png)

## 潜在扩散模型
潜在扩散模型（LDM / Latent Diffusion Model）
在计算机眼中，一张 512x512 分辨率的图片，就是一组 512 * 512 * 3 的数字，如果直接对图片进行学习，相当于 AI 要处理 786432 维的数据，这对算力、计算机性能要求很高。

![](../Stable-Diffusion-入门（一）/08FD9211-7ECC-4E64-9C68-D55F0D92B157.png)

## 潜在空间
CompVis 的研究人员提出，可以将图片映射到潜在空间（Latent Space）后进行扩散和逆扩散学习。如何理解 “潜在空间” 呢？大家都有自己的身份证号码，前 6 位代表地区、中间 8 位代表生日、后 4 位代表个人其他信息。放到空间上如图所示，这个空间就是「人类潜在空间」。

![](../Stable-Diffusion-入门（一）/9B2997A6-D191-4351-AD6A-8BF112BA458E.png)

这个空间上相近的人，可能就是生日、地区接近的人。人可以对应为这个空间的一个点，这个空间的一个点也对应一个人。如果在空间中我的附近找一个点，对应的人可能跟我非常相似，没准就是我失散多年的兄弟 hh
AI 就是通过学习找到了一个「图片潜在空间」，每张图片都可以对应到其中一个点，相近的两个点可能就是内容、风格相似的图片。

![](../Stable-Diffusion-入门（一）/5DAEE960-2697-46C7-8C51-3C5FCDEE6690.png)

同时这个 “潜在空间” 的维度（比如可能是 768）远小于 “像素维度”（786432），AI 处理起来会更加得心应手，在保持效果相同甚至更好的情况下，潜在扩散模型对算力、显卡性能的要求显著降低。这也就是为什么 Stable Diffusion 能在消费级显卡上运行，从而让 AI 绘画 “飞入寻常百姓家”。
说句题外话，我非常想知道为什么 Stable Diffusion 叫 Stable Diffusion，但没找到官方说明，这里做一个猜测：之所以这个基于 Latent Diffusion 的模型叫 Stable Diffusion，可能一方面表示这个模型效果很稳定（Stable），另一方面是致敬一下（算力 &amp; 数据上的）金主爸爸 Stability.ai。
CLIP（Contrastive Language-Image Pre-Training）
如果让你把左侧三张图和右侧三句话配对，你可以轻松完成这个连线。但对 AI 来说，图片就是一系列像素点，文本就是一串字符，要完成这个工作可不简单。

![](../Stable-Diffusion-入门（一）/76DB4BD8-EED8-409C-83CC-BE4FFEAFF6DE.png)

## 文本转换
这需要 AI 在海量「文本-图片」数据上学习图片和文本的匹配。图中绿色方块是「图片潜在空间」的 N 张图片，紫色方块是「文本潜在空间」的 N 句描述语。AI 会努力将对应的 I1 与 T1 （蓝色方块）匹配，而不是 I2 与 T3 （灰色方块）匹配。这个 AI 就是广泛被用在 AI 作画中的 CLIP（Contrastive Language-Image Pre-Training / 对比式语言-文字预训练）。

![](../Stable-Diffusion-入门（一）/D6FF864A-C9F3-4548-AAF6-4D680645D2C0.png)

当 AI 能成功完成这个连线，也就意味着 AI 建立了「文字潜在空间」到「图片潜在空间」的对应关系，这样才能通过文字控制图片的去噪过程，实现通过文字描述左右图像的生成。

![](../Stable-Diffusion-入门（一）/F5736BCA-78AA-4C98-AC15-998336542444.png)

