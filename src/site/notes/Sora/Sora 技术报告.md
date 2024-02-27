---
{"dg-publish":true,"permalink":"/Sora/Sora 技术报告/"}
---

# Video generation models as world simulators

OpenAI近期推出了新的视频生成模型，该工作训练了文本条件的扩散模型来生成不同类型、长度、视角和分辨率的视频。采用Transformer架构在视频图像隐编码的时空patch上运行。Sora能够生成高保真度的一分钟视频。结果表明，Scaling video generating models构建物理世界通用模拟器的有前途途径。  

<video autoplay controls="" preload="metadata" loop="true" style="width:640px; height:430px; max-width:100%">
	 <source id="mp4" src="https://cdn.openai.com/tmp/s/title_0.mp4" type="video/mp4">
</video>  

该技术报告主要内容：
1. 将各种类型视觉数据转为同一表示的方法，从而实现生成模型的大规模训练
2. 量化评估Sora的能力和局限性
> 模型和实现细节不包含在此报告中

## Turning visual data into patches

LLM通过使用token来统一多种文本模态——代码、数学公式和多种自然语言。在sora中作者思考视觉数据的生成模型如何继承该模式。由此，LLM拥有文本token，Sora拥有视觉patches。patch在先前工作中被证明是视觉模型的有效表示，在训练生成式模型时可以用来表示不同类型的视频图像数据。

![](https://images.openai.com/blob/1d2955dd-9d05-4f33-b346-be531d2a7737/figure-patches.png?trim=0,0,0,0&width=1400)

总的来说，首先通过压缩视频到低维潜在空间，然后将该表示解压缩到时空patches。

## Video compression network

训练了一个网络来减少视觉数据的维度。该网络以原始视频数据作为输入，输出压缩有时空信息的潜在表示。Sora使用该表示进行训练，并在接下来生成视频。作者同时训练了对应的解码器模型来映射生成的潜在表示到像素空间。

## Spacetime latent patches

给定一个压缩的输入视频，首先提取时空patches作为transformer的token。该模式对于图像同样适用，因为图像作为视频序列的一帧。基于patch的表示让Sora可以在不同分辨率、长度、视角的视频和图像中进行训练。在推理过程中，可以通过在合适尺寸的网格中随机初始化patches来控制生成视频的大小。

## Scaling transformers for Video generation

Sora是一个扩散模型。将充满噪声的patches作为输入（还有条件信息，例如，文本提示），该模型训练为输出原本“干净”的patches。更重要的是，Sora是一个**diffusion transformer**。transformer已在不同领域展现了强大的Scaling能力，包括语言模型、计算机视觉和图像生成。

![](https://images.openai.com/blob/aa8b687c-bee5-4d72-a1c8-1350d33c80d3/figure-diffusion.png?trim=0,0,0,0&width=1400)

在这项工作中，作者发现diffusion transformers 在处理视频模型中同样有效。在下面的视频对比中，使用同样的种子和输入，随着训练计算的增加，样本质量显著提升。

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:30%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/scaling_0.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:30%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/scaling_1.mp4" type="video/mp4">
	</video>
	<video controls=""  autoplay="true" preload="metadata" loop="true" style="width:30%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/scaling_2.mp4" type="video/mp4">
	</video>
</div>


## Variable durations, resolutions, aspect ratios
过去生成图像或视频的方法通常重置、裁剪视频到一个标准尺寸，例如：分辨率为256x256的4秒钟视频。作者发现在其原始尺寸上进行训练有以下好处：

### Sampling flexibility
Sora可以采样宽屏1920x1080p视频，竖屏1080x1920p视频和在其中的所有其他尺寸。这让Sora可以在不同设备的原生尺寸下创造内容。也可以让我们快速生成低分辨率原型后在生成全分辨率，这全部都是用同一个模型。

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:12%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/sampling_0.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:40%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/sampling_1.mp4" type="video/mp4">
	</video>
	<video controls=""  autoplay="true" preload="metadata" loop="true" style="width:55%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/sampling_2.mp4" type="video/mp4">
	</video>
</div>

### Improved framing and composition
通过实验发现，在原生宽高比的视频数据上训练可以改善构图和布局。作者比较了Sora和另一个使用裁剪为正方形视频训练的模型两者生成视频。使用正方形裁剪训练的模型（左）生成视频的主体通常不完整，作为对比，Sora（右）改善了构图。
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/sampling_3.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/sampling_4.mp4" type="video/mp4">
	</video>
</div>


## Language understanding

训练文本到视频的生成系统需要大量带有对应文本描述的视频。作者使用DALL·E 3对视频进行再描述。首先，训练了一个高度描述性的文本描述模型，然后使用它对数据集中的全部视频生成描述文本。作者发现在高描述性视频文本中提高了视频整体质量。
类似于DALL·E3，作者也使用了GPT将用户短提示词完善为长细节的描述送入到视频模型中。这使得Sora生成符合用户描述的高质量视频。


## Prompting with images and videos
Sora同样可以使用其他形式的提示，例如：已有图像、视频。这一能力使得Sora可以胜任很多图像视频编辑任务——创建完善的循环视频、静态图片动画化、向前后扩展视频等。
### Animating DALL·E images
Sora可以将图像和提示作为输入生成视频。下面作者展示的是基于DALL·E 2和DALL·E 3图像生成的视频。

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<image style="width:50%; height:100%;" src="https://cdn.openai.com/tmp/s/prompting_0.png"> 
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/prompting_1.mp4" type="video/mp4">
	</video>
</div> 

> A Shiba lnu dog wearing a beret and black turtleneck

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<image style="width:50%; height:100%;" src="https://cdn.openai.com/tmp/s/prompting_2.png"> 
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/prompting_3.mp4" type="video/mp4">
	</video>
</div> 

> Monster Illustration in flat design style of a diverse family of monsters. The group includes a furry brown monster, a sleek black monster with antennas, a spotted green monster, and a tiny polka-dotted monster, all interacting in a playful environment.

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<image style="width:50%; height:100%;" src="https://cdn.openai.com/tmp/s/prompting_4.png"> 
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/prompting_5.mp4" type="video/mp4">
	</video>
</div> 

> An image of a realistic cloud that spells “SORA”.

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<image style="width:50%; height:100%;" src="https://cdn.openai.com/tmp/s/prompting_6.png"> 
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/prompting_7.mp4" type="video/mp4">
	</video>
</div> 

> In an ornate, historical hall, a massive tidal wave peaks and begins to crash. Two surfers, seizing the moment, skillfully navigate the face of the wave.

### 生成扩展视频
Sora也可以扩展视频，无论向前扩展还是向后扩展。下面四个视频都是由同一个生成视频片段向后扩展的。结果表现，每一个视频的开始都不相同，但都以相同的结尾结束。

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/extend_1.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/extend_2.mp4" type="video/mp4">
	</video>
	
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/extend_4.mp4" type="video/mp4">
	</video>
</div> 

我们可以使用该方法向前向后扩展视频来生成无穷的视频闭环。

<video controls="" autoplay="true" preload="metadata" loop="true" style="width:100%; height:100%;">
	<source id="mp4" src="https://cdn.openai.com/tmp/s/bike_1.mp4" type="video/mp4">
</video>


### Video-to-video editing
Diffusion models 已经实现了从文本提示中对图像视频进行编辑的大量方法。下面作者使用了其中的SDEdit在Sora中。这一技术使得Sora可以零样本转换输入视频的风格和环境。  

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<div style="width:50%; "> input video </div>
	<div style="width:50%; "> change the setting to be in a lush jungle</div>
</div> 
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/edit/base.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/edit/0.mp4" type="video/mp4">
	</video>
</div> 

### Connecting videos
我们可以使用Sora在两个输入视频中进行插值，来在两个完全不同主题和场景的视频中无缝切换。
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/interp/a0.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/a1.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/a2.mp4" type="video/mp4">
	</video>
</div> 

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/interp/b0.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/b1.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/b2.mp4" type="video/mp4">
	</video>
</div> 

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/interp/c0.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/c1.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/c2.mp4" type="video/mp4">
	</video>
</div> 

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/interp/d0.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/d0.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/d0.mp4" type="video/mp4">
	</video>
</div> 

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/interp/e0.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/e1.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/interp/e2.mp4" type="video/mp4">
	</video>
</div> 

## Image generation capabilities
Sora 也可以进行图像生成。作者通过将加入高斯噪声的patches放入一个拥有单帧时间的空间网格中。模型可以生成多个尺寸图像，分辨率可以达到2048x2048。

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<image style="width:50%; height:100%;" src="https://cdn.openai.com/tmp/s/image_0.png"> 
	<image style="width:50%; height:100%;" src="https://cdn.openai.com/tmp/s/image_1.png"> 
</div> 
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<div style="width:50%; "> Close-up portrait shot of a woman in autumn, extreme detail, shallow depth of field </div>
	<div style="width:50%; "> Vibrant coral reef teeming with colorful fish and sea creatures</div>
</div> 

<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<image style="width:50%; height:100%;" src="https://cdn.openai.com/tmp/s/image_2.png"> 
	<image style="width:50%; height:100%;" src="https://cdn.openai.com/tmp/s/image_3.png"> 
</div> 
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<div style="width:50%; "> Digital art of a young tiger under an apple tree in a matte painting style with gorgeous details </div>
	<div style="width:50%; "> A snowy mountain village with cozy cabins and a northern lights display, high detail and photorealistic dslr, 50mm f/1.2</div>
</div> 

## Emerging simulation capabilities
Sora可以一定程度上模仿现实世界中人、动物和环境。这些能力的出现对3D、物体等没有任何明确的归纳偏差，纯粹是数据中学习到的。

**3D 一致性.** Sora可以生成动态相机移动的视频。当相机转换和旋转时，人物和场景元素在三维空间中连续的移动。
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/simulation_0.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/simulation_1.mp4" type="video/mp4">
	</video>
</div> 

**长范围连贯性和目标持久性** 视频生成系统的一个重大挑战就是在长视频采样中维护时间一致性。作者发现Sora经常，尽管不是一直，可以有效的在短或长范围依赖中建模。例如，模型可以在人物、动物和物体被遮挡或离开镜头后保持。同样的，其可以生成单个角色的多个镜头，在整个视频中保持了他们的外观一致性。
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/simulation_2.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/simulation_3.mp4" type="video/mp4">
	</video>
</div> 

**与现实世界交互** Sora又是可以简单的模拟现实世界的动作行为。例如，画家在画布上作画留下的一笔可以一直保留，或者男人可以吃一个汉堡并保留一个咬过的痕迹。
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/simulation_4.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/simulation_5.mp4" type="video/mp4">
	</video>
</div> 

**模拟数字世界**  Sora同样可以模拟人工程序。一个例子是电脑游戏，Sora可以同时在Minecraft中按照基本策略控制玩家，同时渲染世界和其动态高保真度。这些能力通过给Sora关于“Minecraft”的提示词来零样本实现。
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/simulation_6.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/tmp/s/simulation_7.mp4" type="video/mp4">
	</video>
</div> 
这些能力表明视频模型是一个发展高性能物理和数字世界模拟器的可行途径。

# Discussion
<video controls="" autoplay="true" preload="metadata" loop="true" style="width:100%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/tmp/s/discussion_0.mp4" type="video/mp4">
</video>

Sora目前作为一个模拟器有许多局限性。例如，其无法精确建模许多基本物理交互，如打碎玻璃。其他的交互，例如吃食物，无法一直产生正确的物体变化。在登陆页上枚举了其他常见错误，如长视频中出现的不连贯物体和突然出现的物体。
<div style="width:100%; height:auto; max-width:100%; display:flex; justify-content:center; gap:5px">
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		  <source id="mp4" src="https://cdn.openai.com/sora/videos/puppy-cloning.mp4" type="video/mp4">
	</video>
	<video controls="" autoplay="true" preload="metadata" loop="true" style="width:50%; height:100%;">
		<source id="mp4" src="https://cdn.openai.com/sora/videos/chair-archaeology.mp4" type="video/mp4">
	</video>
</div> 


