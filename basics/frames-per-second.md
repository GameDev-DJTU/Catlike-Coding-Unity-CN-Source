# 帧率 | Frames Per Second
## 性能度量 | Measuring Perforance

使用物理系统创建一个不断增长的原子核模型
使用性能分析器分析性能
计算并展示帧率
防止创建临时字符串变量
计算平均帧率
为帧率标签着色

本篇教程中，我们将创建一个简单的测试场景并用它来进行性能测试。我们首先将使用 Unity 自带的 profiler 分析工具，然后我们将创建属于自己的帧率计数器。  

阅读本篇教程需你有基本的 Unity 脚本编写基础。本篇教程需要 Unity 5.0.1 及以上版本。如果你是个新手的话，请先阅读[创建分形体](https://catlikecoding.com/unity/tutorials/constructing-a-fractal/)。  

![球体会互相撞击直到帧率崩溃](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/tutorial-image.jpg)

## 1. 创建一个原子核模型 | Building an Atomic  Nucleus

我们需要一个测试场景，一个理论上能覆盖高性能与低性能情况的场景。我建议我们通过把越来愈多的核子融合在一起来创建一个原子核。这样，随着原子核的增大性能会变得越来越低。  

核子是一个简单的球体，它们能被吸引到场景中心，在那里他们将会聚集在一起形成一个球体。真正的原子核实际上并不是这个样子的，不过这不是我们要关注的重点。  

我们可以为默认的球体添加一个 `Nucleon` 脚本来创建一个核子。这个脚本确保了一定有一个 `Rigidbody` 刚体组件被添加在游戏对象上，然后对对象施加一个朝向场景中心点的力。这个力的大小是可配置的，具体的来说由距离中心点的距离决定。  

```CS
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class Nucleon : MonoBehaviour {

	public float attractionForce;

	Rigidbody body;

	void Awake () {
		body = GetComponent<Rigidbody>();
	}
	
	void FixedUpdate () {
		body.AddForce(transform.localPosition * -attractionForce);
	}
}
```

> 缺少了访问修饰符么？
>
> 是的，我现在在字段与函数声明中省略了 private 修饰符，因为它们默认就是 private 的。直接跑一下程序，看看程序能否正常运行~

使用默认的球体来创建两个核子预制体，一个代表质子另一个代表中子。给他们添加不同的材质以便分辨它们。其实，我们只需要创建一个核子类型就够了，不过这样子看起来不好看。  

> 什么是预制体(Prefab)？
>
> 一个预制体是一个 Unity 物体，或者一系列具有层次结构的 Untiy 物体，如果我们不激活它们，它们将不会在场景中出现。你可以将其视为一个模板，可以创建一个他们的拷贝并把它们添加到场景中。为了创建一个预制体，你需要想往常一样在场景中创建一个物件，然后将其拖入到 project 工程面板中。这个场景物体将自动变为一个预制体实例，如果你不在需要这个场景物体你可以将其删除。

![核子预制体](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/01-proton-prefab.png)

![核子预制体](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/01-neutron-prefab.png)

为了生成这些核子我们需要创建另一个脚本 `NucleonSpawner`。这个脚本需要知道两次生成的时间间隔、生成物据场景中心的距离、生成哪种核子。  

```CS
using UnityEngine;

public class NucleonSpawner : MonoBehaviour {

	public float timeBetweenSpawns;

	public float spawnDistance;

	public Nucleon[] nucleonPrefabs;
}
```

创建一个空的游戏物体，并将 `NucleonSpawner` 组件附加在上面，然后按你的喜好进行配置。  

![NucleonSpawner](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/01-nucleon-spawner.png)

为了定期生成核子，我们需要纪录距离上一次生成所经过的时间。我们可以使用 `FixedUpdate` 方法实现这个功能。  

```CS
	float timeSinceLastSpawn;

	void FixedUpdate () {
		timeSinceLastSpawn += Time.deltaTime;
		if (timeSinceLastSpawn >= timeBetweenSpawns) {
			timeSinceLastSpawn -= timeBetweenSpawns;
			SpawnNucleon();
		}
	}
```

>为啥使用 FixedUpdate 而不是 Update 方法？
>
>使用 FixedUpdate 能使生成速率独立于游戏帧率。如果配置的生成时间间隔短于每帧时长，使用 Update 函数会导致生成延迟。因为我们这个场景的目的是造成帧率崩溃，这将很可能发生。
>
> 你可以使用 `while` 循环代替 `if` 来追回错过的生成，但是如国过 `timeSinceLastSpawn` 一旦被错误的设置为零的话，这将导致死循环。在每个 `FixedUpdate` 中值进行一次生成是一个合理的选择。  

实际的生成包括三个步骤。随机选择一个预制体，将其实例化，然后依照设定的距离给它一个随机的位置坐标。  

```CS
	void SpawnNucleon () {
		Nucleon prefab = nucleonPrefabs[Random.Range(0, nucleonPrefabs.Length)];
		Nucleon spawn = Instantiate<Nucleon>(prefab);
		spawn.transform.localPosition = Random.onUnitSphere * spawnDistance;
	}
```

![通过轰炸形成一个原子核](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/01-nucleus.png)

![通过轰炸形成一个原子核]()

播放这个场景将会看到球体从场景四周向中央发射。它们有的时候会打过头，但最终会在场景中央碰撞在一起并形成一个球体。随着这个球体的持续增长，物理运算将会越来越复杂，最终你将会注意到帧率的下降。  

如果你花费很长时间也没有实现性能的下降，请加快你的生成速率。通过调节 Time Scale 来加速时间也是可以的，你可以通过 Edit/Project Setting/Time 路径找到这项设置。你也可以所见 Fixed TimeStep, 这些都导致每秒钟进行更多的物理运算。  

![Unity 的时间设置](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/01-time-manager.png)

> 为什么在 TimeScale 数值较低时运动不连贯了？
>
> 当 Time Scale 被设置为像 0.1 这样很低的值的时候，时间会推进的非常慢。因为固定时间步长(Fixed Timestep)是固定不变的，所以物理系统的更新速率会很低。所以依靠物理系统驱动的物体会保持不动直到 Fixed Update 更新，而这可能好几帧才更新一次。 
>
> 你可以通过在降低 Time Scale 的同时增大 Fixed Timestep 来解决这一问题。或者，你可以更改刚体组件的插值模式为在每个物理帧之间进行插值，这样就能掩盖住较慢的更新速率。

## 2. 使用 Profiler 分析工具 | Using the Profiler

现在我们有了一个最终会降低任何机器帧率的场景，是时候测试实际的运行性能了。最快捷的方式就是激活游戏视图(Game View)上的统计信息图。  

![统计信息视图](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/02-statistics.png)

然而，这里显示的的帧率数据并不精准，这更像是一个粗略的估计数值。我们可以通过路径 Window/Profiler 打开 Unity 的性能分析工具 Profiler 获取更加精确的数据。我们可以使用 Profiler 获取很多数据，例如 CPU 和 内存的利用率。  

![被垂直同步信息霸占性能分析器](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/02-profiler-vsync.png)

如果开启了垂直同步(vsync),它可能会霸占我们的 CPU 性能图像。为了获取能够更好的反应我们场景对于 CPU 资源占有情况的图像，我们需要关闭垂直同步。你可以通过路径 Edit / Project Settings / Quality 来关闭它。它在底部的其他栏目内。

![关闭垂直同步](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/02-vsync.png)

> 我现在的帧率非常高！
>
> 在简单的场景中关闭了垂直同步是可能得到非常高的帧率的，有时可能会超过100。这会给硬件施加没有必要的压力。你可以通过限制 `Application.targetFrameRate` 属性来限制最大帧率。需要注意的是，即使在退出播放模式后，该项设置依旧会被保留。你可以将其设置为 -1 来取消这项设置。

现在你可以得到一个更易观察的 CPU 使用率图像了。以我的电脑为例，物理运算占据了最多的 CPU 时间，然后是渲染，最后是执行脚本。即使是随着球体数量的增加所有运算都变慢了，也依旧如此。

![关闭垂直同步的性能分析器](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/02-profiler.png)

我们还观察到了两个意想不到的现象。第一，CPU 利用率会偶尔的出现一些峰值。第二，内存图示显示垃圾收集机制被频繁调用，这意味着我们的内存被分配之后随即就被释放。由于我们只是一直不断的创建新的对象而并没有删除它们，所以这很奇怪。

这两种现象都是由于 Unity 编辑器导致的。CPU 利用率的峰值发生在你在编辑器中选择了某些东西的时候。内存分配的分支是由于编辑器调用了 `GameView.GetMainGameViewRenderRect`。还有一些其他的额外开销，特别是当你同时打开了游戏视图与场景视图的时候。简言之，Unity 编辑器本身会干扰我们对于游戏性能的测量。  

在 Unity 编辑器内置的性能分析器中你还能发现很多有用的信息，但是如果你希望消除编辑器对性能分析的影响，你需要发布一个独立的版本。即使发布独立的版本你依旧可以使用 Unity 的性能分析器，甚至你还可以在打开应用的时候自动连接到性能分析器。你可以通过以下路径进行设置：File / Build Settings。

![将性能分析器附加到独立的应用](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/02-build.png)

![将性能分析器附加到独立的应用](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/02-profiler-attached.png)

当使用性能分析器分析独立的发布版本是，显示的数据与在编辑器内运行游戏有些不同。现在只有在创建核子的时候出现内存分配了，并且没有出现频繁的垃圾收集。在我这里渲染花费的时间更多了，因为我使用全屏模式运行游戏而不是跑在游戏视图的小框框中了。脚本运行的时间依旧是无关紧要的，你甚至在图像中都看不到它。

## 3. 计算帧率 | Measuring Frames Per Second

内置的性能分析器能够为我们提供很多有用的数据，但是依旧不能帮助我们很好的测量帧率。上面显示的时间仅仅使用 1 除以 CPU 时间，并不是我们想要的时时帧率。所以我们来自己计算它。  

我们需要一个简单的脚本来计算我们运行的应用的时时帧率。使用一个公有属性就足够了，我们将其设定为 int，因为我们不需要小数部分的精度。  

```CS
using UnityEngine;

public class FPSCounter : MonoBehaviour {

	public int FPS { get; private set; }
}
```

> 属性(Property)是如何工作的？
>
> 属性是被伪装成字段(Field)的方法(Method)。我们需要提供一个叫做 FPS 的能够被公共访问的信息，但是只有在这个组件内部能够修改它的值。上面我们使用的语法是对下面的缩写。
>
> ```CS
>	int fps;
>	public int FPS {
>		get { return fps; }
>		private set { fps = value; }
>	}
> ```
>
> 这种简写方式不支持 Unity 的序列化，但是这没有关系，因为我们不需要储存 FPS 的数值。

我们在每个 Update 中用 1 除以当前帧所花费的时间来计算帧率。我们可以将结果向下取整为 int 类型。

```CS
	void Update () {
		FPS = (int)(1f / Time.deltaTime);
	}
```

然而，这种方法存在一些问题。deltaTime 并不是真实的时间，是被当当前的时间缩放所影响的。这意味着除非时间缩放被设置为1，帧率的计算就是错误的。幸运的是，Unity 为我们提供了未被缩放的 deltaTime。

```CS
	void Update () {
		FPS = (int)(1f / Time.unscaledDeltaTime);
	}
```

我们需要使用一些 UI 控件来显示帧率。创建一个 Canvas 画布，其中要包含一个含有 Text 文本对象的 Panel 面板。你可以通过 GameObject / UI 子菜单添加它们。当你添加一个 Canvas 画布的时候 Unity 会自动创建一个 EventSystem 对象来处理输入，但是我们不需要它，你可以将其删除。

![UI 物体的层次结构](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/03-hierarchy.png)

除了 Pixel Perfect 属性之外，我都是用了默认的画布属性。  

![Pixel-Perfect](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/03-canvas.png)

我们用 Panel 面板为 FPS 标签创建半透明的黑色背景，这能保障帧率数值一直是可读的。我将其放置在窗口的左上角。将其锚点(Anchor)设置在左上角以便适应所有的窗口大小，然后将其轴心(Pivot) 设置为 (0, 1)。

Label 标签也是用相同的方式进行设置。将字体设置为白色加粗、中心对齐，调整大小以容纳两位数字。  

![创建 UI](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/03-panel.png)

![创建 UI](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/03-label.png)

![创建 UI](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/03-panel-scene.png)

![创建 UI](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/03-label-scene.png)

现在，我们需要将帧率数值绑定到 Label 对象上。我们创建一个脚本来做这件事。需要一个 `FPSCounter` 脚本来获取帧率的数值，并且需要一个 `UnityEngine.UI` 名字空间下的 `Text` 类型的引用来设置数值。  

```CS
using UnityEngine;
using UnityEngine.UI;

[RequireComponent(typeof(FPSCounter))]
public class FPSDisplay : MonoBehaviour {

	public Text fpsLabel;
}
```

将脚本绑定到 Panel 面板上，并设置好引用。我们将其绑定在 Panel 面板上而不是 Label 上，这是因为我们用它来管理所有的 FPS 显示，稍后我们会添加更多地 Label 来显示更多类型的 FPS。  

![绑定好 Label 的引用](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/03-display.png)

显示脚本需要在每帧中更新 Label 中的文字。我们将计数器脚本的引用储存起来，这样就不用在每帧中调用 GetComponent 方法。  

```CS
	FPSCounter fpsCounter;

	void Awake () {
		fpsCounter = GetComponent<FPSCounter>();
	}

	void Update () {
		fpsLabel.text = fpsCounter.FPS.ToString();
	}
```

现在 FPS Label 已经更新了！但是我们将其设计用来显示两位数字，一旦帧率超过两位，它显示的就是错误的数值。所以我们需要限制实际显示的数值范围，反正高于99的帧率都已经非常好了，即使不显示也不关紧要。  

```CS
	void Update () {
		fpsLabel.text = Mathf.Clamp(fpsCounter.FPS, 0, 99).ToString();
	}
```

![正确显示的帧率](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/03-game.png)

现在似乎一切都工作正常，但是还有一个微小的问题。我们现在每一个 Update 都创建一个字符串对象，而这些字符串对象在下一个 Update 都会被销毁。这干扰了内存管理，将会频繁触发垃圾收集机制。这对桌面应用来说并没有什么问题，但是对于没有足够内存的设备来说却是十分糟糕的。它也干扰到了性能分析器的数据，当你追踪内存分配时这是个很烦人的问题。  

![每帧中创建的临时字符串对象](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/03-string-allocations.png)

我们可以摆脱这些临时的字符串对象么？所有我们需要显示的数字都在0到99之间，只有100个不同的字符串。为什么我们不将它们储存起来并且复用它们呢？

```CS
	static string[] stringsFrom00To99 = {
		"00", "01", "02", "03", "04", "05", "06", "07", "08", "09",
		"10", "11", "12", "13", "14", "15", "16", "17", "18", "19",
		"20", "21", "22", "23", "24", "25", "26", "27", "28", "29",
		"30", "31", "32", "33", "34", "35", "36", "37", "38", "39",
		"40", "41", "42", "43", "44", "45", "46", "47", "48", "49",
		"50", "51", "52", "53", "54", "55", "56", "57", "58", "59",
		"60", "61", "62", "63", "64", "65", "66", "67", "68", "69",
		"70", "71", "72", "73", "74", "75", "76", "77", "78", "79",
		"80", "81", "82", "83", "84", "85", "86", "87", "88", "89",
		"90", "91", "92", "93", "94", "95", "96", "97", "98", "99"
	};
	
	void Update () {
		fpsLabel.text = stringsFrom00To99[Mathf.Clamp(fpsCounter.FPS, 0, 99)];
	}
```

使用定长字符串数组来代表将要使用的每一个数字，这样就能避免临时字符串对象的分配！

## 4. 平均帧率 | Averaging Frames Per Second

每帧都更新 FPS 数值有一个糟糕的副作用，如果帧率不稳定的话数值会不断波动，以至于难以阅读。我们可以每隔一段时间更新一下数值，但是这样我们就不能对帧率如何变化产生清晰的印象。  

一个可行的解决方案是单独计算平均帧率。我们修改 `FPSCounter` 脚本以便他能计算一个时间段内 FPS 的平均值，这个时间段需要是可配置的。当其被设置为 1 时就相当于不计算平均值。  

```CS
	public int frameRange = 60;
```

![配置计算帧数](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/04-frame-range.png)

我们将属性 FPS 重命名为更合适的 `AverageFPS`。使用你的 IDE 进行重命名，否则你需要手动更改所有的脚本中的名字。

```CS
	public int AverageFPS { get; private set; }
```

现在我们需要一个缓存来储存每帧的 FPS 数值，再加上一个 index 变量用来储存下一帧的位置。

```CS
	int[] fpsBuffer;
	int fpsBufferIndex;
```

初始化缓存的手确保 `frameRange` 至少是1，然后将 index 设置为0。

```CS
	void InitializeBuffer () {
		if (frameRange <= 0) {
			frameRange = 1;
		}
		fpsBuffer = new int[frameRange];
		fpsBufferIndex = 0;
	}
```

`Update` 方法变得更加复杂了，首先我们在首次运行时或者 `frameRange` 被重置之后初始化缓冲，然后跟新缓冲，最后计算帧率。  

```CS
	void Update () {
		if (fpsBuffer == null || fpsBuffer.Length != frameRange) {
			InitializeBuffer();
		}
		UpdateBuffer();
		CalculateFPS();
	}
```

我们将当前的 FPS 储存在当前 index 所指的位置里，然后再递增 index。

```CS
	void UpdateBuffer () {
		fpsBuffer[fpsBufferIndex++] = (int)(1f / Time.unscaledDeltaTime);
	}
```

当我们把整个数组都填满之后，我们只需让索引归零，这样最新的值总会覆盖最老的值。

```CS
	void UpdateBuffer () {
		fpsBuffer[fpsBufferIndex++] = (int)(1f / Time.unscaledDeltaTime);
		if (fpsBufferIndex >= frameRange) {
			fpsBufferIndex = 0;
		}
	}
```

计算平均值就很简单啦，就是把所有的值都加到一块然后再除以他们的个数。   

```CS
	void CalculateFPS () {
		int sum = 0;
		for (int i = 0; i < frameRange; i++) {
			sum += fpsBuffer[i];
		}
		AverageFPS = sum / frameRange;
	}
```

我们的平均帧率标签现在可以正常工作了，我们还它设置了一个合理的范围以便不会出现阅读上的歧义。但是我们还可以做的更多。因为我们现在又从很多帧里或得到的数据，我们可以得到这些帧中的最高 FPS 和最低 FPS。这比只有平均数据能够让我们获取道更多有用的信息。  

```CS
	public int HighestFPS { get; private set; }
	public int LowestFPS { get; private set; }
```

我们可以在计算帧率综合的时候获得这两个数据。  

```CS
	void CalculateFPS () {
		int sum = 0;
		int highest = 0;
		int lowest = int.MaxValue;
		for (int i = 0; i < frameRange; i++) {
			int fps = fpsBuffer[i];
			sum += fps;
			if (fps > highest) {
				highest = fps;
			}
			if (fps < lowest) {
				lowest = fps;
			}
		}
		AverageFPS = sum / frameRange;
		HighestFPS = highest;
		LowestFPS = lowest;
	}
```

我们为 `FPSDisplay` 脚本绑定两个额外的标签。  

```CS
	public Text highestFPSLabel, averageFPSLabel, lowestFPSLabel;

	void Update () {
		highestFPSLabel.text =
			stringsFrom00To99[Mathf.Clamp(fpsCounter.HighestFPS, 0, 99)];
		averageFPSLabel.text =
			stringsFrom00To99[Mathf.Clamp(fpsCounter.AverageFPS, 0, 99)];
		lowestFPSLabel.text =
			stringsFrom00To99[Mathf.Clamp(fpsCounter.LowestFPS, 0, 99)];
	}
```

为 UI 添加连个额外的 Label 标签控件然后正确设置它们的引用。我把最高 FPS 放在最上面，最低的放在最下面，平均 FPS 放在最中央。  

![更多的信息，更少的抖动](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/04-three-labels-hierarchy.png)

![更多的信息，更少的抖动](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/04-three-labels.png)

![更多的信息，更少的抖动](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/04-three-labels-scene.png)

![更多的信息，更少的抖动](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/04-game.png)

## 5. 为 Label 标签控件着色 | Coloring the Labels

最后我们需要做的是为 FPS 标签设置着色。我们可以让不同大小的 FPS 数值拥有不同的颜色。我们可以使用一个自定义的结构体来实现这种关系。  

```CS
	[System.Serializable]
	private struct FPSColor {
		public Color color;
		public int minimumFPS;
	}
```

因为 `FPSDisplay` 是唯一使用这个结构体的类，所以我们直接将结构体定义到这个类中，然后将其设置为 private 的，这样结构体在类外部的名字空间作用域就无法被访问了。然后将其设置为可序列化的(Serializable)，这样就能将它暴露给 Unity 编辑器。  

然后添加一个这个结构体的数组，这样我们就能编辑 FPS 标签控件的颜色了。我们通常会将这样的字段设置为 public 的，但是我们这回没法这么做，因为这个结构体本身是 private 的。所以我们还是将其设置为 private 的，但是需要为其添加一个 `SerializeField` 属性，这样就能将其暴露给 Unity 编辑器，并且可以保存它。  

```CS
	[SerializeField]
	private FPSColor[] coloring;
```

然后我们来设置颜色吧！确保数组至少含有一个元素，将他们由 FPS 的数值从大到小排列，以 0 FPS 作为结束。  

![着色](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/05-coloring.png)

在为标签实际添加颜色之前，我们需要从 `Update` 函数中抽出一个 `Display` 函数用以单独处理每一个标签。  

```CS
	void Update () {
		Display(highestFPSLabel, fpsCounter.HighestFPS);
		Display(averageFPSLabel, fpsCounter.AverageFPS);
		Display(lowestFPSLabel, fpsCounter.LowestFPS);
	}

	void Display (Text label, int fps) {
		label.text = stringsFrom00To99[Mathf.Clamp(fps, 0, 99)];
	}
```

我们通过依次比较数组中每个颜色对应的最小 FPS 数值来确定最终所显示的颜色，找到需要的颜色之后直接跳出循环。  

```CS
	void Display (Text label, int fps) {
		label.text = stringsFrom00To99[Mathf.Clamp(fps, 0, 99)];
		for (int i = 0; i < coloring.Length; i++) {
			if (fps >= coloring[i].minimumFPS) {
				label.color = coloring[i].color;
				break;
			}
		}
	}
```

> 为什么我的标签消失了？
>
> 默认的颜色四个通道的值都被设置为零，其中包含了 alpha 通道，它代表了颜色的透明度。如果你没有改变 alpha 通道的值，你会得到一个透明的标签。  

![彩色的 FPS 标签](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/5_frames_per_second/05-game.png)

// todo 这个咋翻  
一切都搞定啦！好好欣赏你彩色的帧率大崩溃吧~~~