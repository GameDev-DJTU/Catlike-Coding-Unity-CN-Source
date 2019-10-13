# 数学曲面 | Mathematical Surfaces
## 使用数字进行雕刻 | Sculpting with Numbers

支持多种数学函数  
使用委托与枚举  
使用网格(Grid)展示二维函数  
在三维空间中定义曲面  

本教程是[创建图形](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/)的后续教程。本教程中我们将支持更多更复杂的数学函数。  

本教程基于 Unity 2017.1.0。

![用数个简单波形组合出复杂的形状](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/tutorial-image.jpg)

## 1 在多个函数间进行切换 | Switching Between Functions

在[上一篇教程](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/)中，我们在游戏模式(Play Mode)里绘制了一条时时运动的正弦曲线。我们也可以用它来展示其他数学函数，只需要更改数学函数的相关代码就好了。我们甚至可以在 Unity 编辑器处于游戏模式时就进行更改，如果这样做的话游戏会暂停执行，储存当前游戏状态，然后重新编译脚本，最后重新加载游戏状态并继续游戏。并不是所有东西都能在游戏模式时的重新编译后被保存下来，但是我们的图可以。它将展示新的数学函数，就像没有进行任何更改一样。  

在游戏模式时重新编译看起来非常方便，但是如果我们想在多个数学函数间进行切换的话就比较麻烦了。我们加下来要实现的是只需要更改几个简单的配置项就能实现函数间的切换。  

### 1.1 数学函数方法 | Fucntion Method

//todo 本段需要重翻  
为了使我们的图形同时支持多个数学函数，我们需要把把它们都编写到程序里。然而，遍历图形中所有顶点的代码并不关心它们使用的是什么函数。我们不必在每个独立的方法中重复这些代码，相反，我们可以将这些数学函数提取出来放到属于它们自己的方法里。  

我们为 `Graph` 类添加一个新的方法来封装我们正弦函数的代码。就像之前创建的 `Awake` 与 `Update` 方法一样，唯一的不同就是我们需要将其命名为 `SineFunction`。

```CS
	void SineFunction () {}
```

这个方法代表了数学函数//todo。为了实现它，首先需要为其添加一个浮点类型的输出。所以我们这里将方法的返回值类型由 `void` 替换为 `float`。

```CS
	float SineFunction () {}
```

目前，方法的参数列表还是空的，它需要两个参数。首先，我们将名为 `x` 的参数添加到函数名后的括号中。就如同函数有它的类型一样，函数的参数也必须显示声明类型。这里我们将其设定为 `float` 浮点数类型。  

```CS
	float SineFunction (float x) {}
```

像刚才那样，我们在为其添加一个名为 `t` 的浮点类型的参数。两个参数间使用英文逗号进行分隔。  

```CS
	float SineFunction (float x, float t) {}
```

现在，我们可以将计算正弦函数的代码放入方法中了。我们可以在其中使用方法的参数 `x` 和 `t`。  

```CS
	float SineFunction (float x, float t) {
		Mathf.Sin(Mathf.PI * (x + t));
	}
```

最后，我们需要显示的声明方法的结果是什么。由于方法的类型是 `float`，我么们需要在方法结束时返回一个 `float` 浮点数。我们使用 `return` 语句后接一个结果是期望类型的表达式来声明它。

```CS
	float SineFunction (float x, float t) {
		return Mathf.Sin(Mathf.PI * (x + t));
	}
```

现在，我们可以在 `Update` 方法中使用 `position.x` 和 `Time.time` 作为参数调用我们刚刚写好的方法了。Y 座标的值也就是这个方法的结果。    

```CS
	void Update () {
		for (int i = 0; i < points.Length; i++) {
			Transform point = points[i];
			Vector3 position = point.localPosition;
//			position.y = Mathf.Sin(Mathf.PI * (position.x + Time.time));
			position.y = SineFunction(position.x, Time.time);
			point.localPosition = position;
		}
	}
```

需要注意的是，我们每次在循环中调用 `Time.time` 属性时得到的值都是相同的。我们可以将调用提到循环外面，然后将其储存起来。  

```CS
	void Update () {
		float t = Time.time;
		for (int i = 0; i < points.Length; i++) {
			Transform point = points[i];
			Vector3 position = point.localPosition;
			position.y = SineFunction(position.x, t);
			point.localPosition = position;
		}
	}
```

### 1.2 第二个函数 | A Second Function

现在，我们已经有了一个数学函数方法了，我们来做第二个。这回我们将创建一个略微复杂的函数，使用不止一个正弦波。首先我们将刚才的 `SineFunciton` 方法复制一份，然后将其重命名为 `MultiSineFunction`。  

```CS
	float SineFunction (float x, float t) {
		return Mathf.Sin(Mathf.PI * (x + t));
	}
	
	float MultiSineFunction (float x, float t) {
		return Mathf.Sin(Mathf.PI * (x + t));
	}
```

我们要保留原来的正弦波，然后在其基础上添加一些额外的东西。这里，为了简单起见，我们在返回结果之前先将其赋值给 `y` 变量。  

```CS
	float MultiSineFunction (float x, float t) {
		float y = Mathf.Sin(Mathf.PI * (x + t));
		return y;
	}
```

让一个正弦波变得更复杂的最简单的方法就是在其基础上叠加一个频率是其两倍的正弦波。我们将正弦函数的自变量乘以2以便让他的速度变为原来的两倍。同时，我们将正弦函数的结果除以2，也就是保持波形不变，大小变为原来的二分之一。  

```CS
		float y = Mathf.Sin(Mathf.PI * (x + t));
		y += Mathf.Sin(2f * Mathf.PI * (x + t)) / 2f;
		return y;
```

如上操作，我们就得到了数学函数//todo。由于正弦函数的正负最值是 +1 和 -1，我们这个新函数的正负最值分别是 +1.5 和 -1.5。为了保证函数值在 -1 到 +1 的区间内，我们将函数值除以 1.5 也就是 2/3。  

```CS
		float y = Mathf.Sin(Mathf.PI * (x + t));
		y += Mathf.Sin(2f * Mathf.PI * (x + t)) / 2f;
		y *= 2f / 3f;
		return y;
```

我们来使用新的函数替代 `SineFunction` 看看实际运行的效果吧！

```CS
			position.y = MultiSineFunction(position.x, t);
```

![多个正弦波的叠加]()

你可以说现在是一个小一点的正弦波紧随在一个大一点的正弦波之后。我们也可以让小一点的正弦波随着大正弦波滑动，只需要让它的时间参数乘以 2 就好了。这样新的函数不仅会随着时间的推移而滑动，同时也改变了形状。因为正弦波是重复的，所以每隔两秒会恢复至原先的形状。  

```CS
	float MultiSineFunction (float x, float t) {
		float y = Mathf.Sin(Mathf.PI * (x + t));
		y += Mathf.Sin(2f * Mathf.PI * (x + 2f * t)) / 2f;
		y *= 2f / 3f;
		return y;
	}
```

![产生形变的多个正弦波]()

### 1.3 在编辑器中选择函数 | Selecing Functions in the Editor

接下来我们要做的是添加一些代码以便能够选择图形所使用的数学函数。我们可以像调节图形分辨率时一样使用一个滑块(Slider)控件。因为我们有两个函数供选择，所以我们使用一个 `int` 类型的字段，并将其区间设置为 [0, 1]。将其命名为 function 以增强可读性。  

```CS
	[Range(0, 1)]
	public int function;
```

![Function 滑块](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/function-slider.png)

我们可以在 Update 方法中使用 `if-else` 语句来选择我们将要调用哪一个函数。如果滑块被滑动至 0 ，我们将使用 `SineFunction` 函数，反之则调用 `MultiSineFunction` 函数。

```CS
	void Update () {
		float t = Time.time;
		for (int i = 0; i < points.Length; i++) {
			Transform point = points[i];
			Vector3 position = point.localPosition;
			if (function == 0) {
				position.y = SineFunction(position.x, t);
			}
			else {
				position.y = MultiSineFunction(position.x, t);
			}
			point.localPosition = position;
		}
	}
```

这样就能在 `Graph` 组件的检视器(Inspector)中方便的选择调用的函数了，即使在游戏模式中也能进行选择。  

### 1.4 静态函数 | Static Methods

虽然 `SineFunction` 和 `MultiSineFunction` 位于类 `Graph` 中，但它们实际上是独立的。它们的实现只依赖数学类和函数的参数,除此之外他们不需要访问 `Graph` 组件的任何方法或字段。这就意味着即使我们将其放入另外的类或结构体中它们依旧能正常工作。我们可以为它们创建单独的类，但是由于只有 `Graph` 类使用他们，似乎没有必要这么做。  

在默认情况下，方法和字段是和类或结构体的特定实例对象相关联的。但不一定非得这样，我们可以方法或字段的开头使用 `static` 关键字以破坏这种联系。  

```CS
	static float SineFunction (float x, float t) {
		return Mathf.Sin(Mathf.PI * (x + t));
	}

	static float MultiSineFunction (float x, float t) {
		float y = Mathf.Sin(Mathf.PI * (x + t));
		y += Mathf.Sin(2f * Mathf.PI * (x + 2f * t)) / 2f;
		y *= 2f / 3f;
		return y;
	}
```

现在这两个方法依旧是 `Graph` 类的一部分，但是只和类本身关联而不是和类的具体对象相联系了。如果我们将其访问权限设定为 public 的，我们就可以在任何地方访问它们了。例如, 我们可以像调用数学函数 `Mathf.Sin(0f)` 一样调用 `Graph.SineFunction(0f, 0f)`。而在 `Graph` 类内部，我们不需要使用类型前缀就能进行函数调用，所以我们的老代码依旧能正常工作。  

> 使用静态函数有什么用呢？
>
> 现在的确没有设么必要，但是我们很快就会用到它。现在只不过表示这些方法是独立的。
>
> 因为静态函数没有被联系到某个对象实例，所以编译器不用跟踪是哪个对象调用了函数。这以为着静态函数的调用往往会快一些，但是一般我们不考虑这点速度上的差异。

### 1.5 委托 | Delegates

对于两个函数而言使用 `if-else` 语句就已经够用了，但是如果想要支持更多的函数这样就会很麻烦。如果能使用变量来储存我们想要调用的函数的引用就好了。我们可以使用委托(Delegates)来实现这一点。委托是一种特殊的类型，它定义了对象可以引用什么类型的方法。我们的数学函数方法没有一个标准的内置类型，但是我们可以自己定义一个。我们这里创建一个名为 `GraphFunction` 的新的 C# 脚本。

![GraphFunction 脚本资源](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/graph-function.png)

> 为什么我们要创建新的脚本？
>
> 我们的确可以在 Graph 类中定义委托类型，但是把不同的功能放到不同的脚本中有利于功能的区分。相反，在大型项目中，如果一个小的类型只在另一个类型的上下文中被使用，我们一般将其嵌入到另一个类型中。  

删除掉新脚本中的默认代码，我们只需要使用 `UnityEngine` 名字空间，然后定义一个名为 `GraphFunction` 的委托类型。与类或结构体的定义不同，后面必须跟着一个分号。  

```CS
using UnityEngine;

public delegate GraphFunction;
```

委托的类型定义了其可以引用的函数的形式，也就是函数的签名，函数的签名由返回类型和参数表构成。在我们例子中，返回类型是 `float`，参数也是两个 `float`。将此签名应用到 `GraphFunction` 委托类型上，参数的名称并不重要，只要类型是正确的就可以。  

```CS
	public delegate float GraphFunction (float x, float t);
```

在 `Graph.Update` 函数里声明一个 `GraphFunction` 类型的变量，将其放置在循环之前。然后，就可以像调用函数一样调用这个变量了。这样的话我们就可以把循环中的 `if-else` 语句干掉。  

```CS
	void Update () {
		float t = Time.time;
		GraphFunction f;
		for (int i = 0; i < points.Length; i++) {
			Transform point = points[i];
			Vector3 position = point.localPosition;
//			if (function == 0) {
//				position.y = SineFunction(position.x, t);
//			}
//			else {
//				position.y = MultiSineFunction(position.x, t);
//			}
			position.y = f(position.x, t);
			point.localPosition = position;
		}
	}
```

然后我们将 `if-else` 语句提到循环之前，为委托变量分配合适的函数引用。  

```CS
		GraphFunction f;
		if (function == 0) {
			f = SineFunction;
		}
		else {
			f = MultiSineFunction;
		}
		for (int i = 0; i < points.Length; i++) {
			…
		}
```

### 1.6 委托数组 | An Array of Delegates

刚才我们只是把 `if-else` 语句移到了循环外面，而没有彻底消灭它。我们可以用一个数组彻底替换掉它。现在我们为 `Graph` 类添加一个名为 `functions` 的委托数组。  

```CS
	GraphFunction[] functions;
```

我们不打算改变数组中元素的内容，所以我们可以直接在数组声明时为其显示初始化。可以使用花括号进行列表初始化，最简单的情况就是一个空的花括号。  

```CS
	GraphFunction[] functions = {};
```

这意味着我们将立即获得一个数组实例，只不过它是空的。我们为其添加两个函数引用，第一个是 `SineFunction` ，第二个是 `MultiSineFunction`。  

```CS
	GraphFunction[] functions = {
		SineFunction, MultiSineFunction
	};
```

由于数组在整个生命周期内并不发生改变，所以没有必要为每一个 `Graph` 实例对象创建一个单独的数组，我们只需要为 `Graph` 类创建一个公共的就好了。像上面对函数做的一样，我们可以将其声明为 static 的。  

```CS
	static GraphFunction[] functions = {
		SineFunction, MultiSineFunction
	};
```

接下来只要在 `Update` 方法中用 `function` 字段索引数组就可以替代 `if-else` 语句了。  

```CS
		GraphFunction f = functions[function];
//		if (function == 0) {
//			f = SineFunction;
//		}
//		else{
//			f = MultiSineFunction;
//		}
```

### 1.7 枚举 | Enumerations

使用整形的滑块组件能够正常的工作，但是用 0 代表 sine 函数、用 1 代表 multi-sine 函数并不利于理解。如果我们能够使用有具有语义明确的名字的下拉列表就好了。我们可以使用枚举(Enumeration)来实现这一功能。  

我们可以通过定义 `enum` 类型来创建枚举，为此我们创建一个新的名为 `GraphFunctionName` 的 C# 脚本。  

![GraphFunctionName 脚本资源](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/graph-function-name.png)

枚举类的最小定义和类的定义基本相同，只不过过把 `class` 关键字换为了 `enum`。

```CS
public enum GraphFunctionName {}
```

枚举类名字后的语句块里通常包含一个由逗号分隔的标签列表。这些标签的命名规则和类型名称相同。作为代表我们函数的标签，我们将其命名为 `Sine` 和 `MultiSine`。  

```CS
public enum GraphFunctionName {
	Sine,
	MultiSine
}
```

接下来，将名为 `function` 的整形字段替换为同名的 `GraphFunctionName` 类型的字段。  

```CS
//	[Range(0, 1)]
//	public int function;
	public GraphFunctionName function;
```

枚举类可以被当做一个语法糖。默认情况下，枚举类中的每一个标签都代表一个整数。第一个标签代表 0，第二个代表 1，依此类推。所以我们可以使用枚举作为数组的索引。然而，编译器会告诉你不能把枚举隐式转换为整数，我们需要做显式类型转换。  

```CS
		GraphFunction f = functions[(int)function];
```

现在我们可以使用枚举值来选择将要使用的数学函数了。检视面板(Inspector)会将枚举类显示为其标签组成的下拉列表。这样我们就能清楚的知道我们使用的是哪一个函数了。  

![Function 下拉列表](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/function-dropdown-list.png)

![Function 下拉列表](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/function-dropdown-list-expanded.png)

## 2 添加另一个维度 | Adding Another Dimension

到目前为止，我们创建的都是传统的曲线。它将一个一维的值映射到另一个一维的值，不过，如果你将时间考虑进去的话，我们实际上是将一个二维的数据映射到一维的数据上。所以，我们已经把高维输入映射到一维的值上了。我们可以像添加时间维度一样添加其他空间维度。  

目前，我们使用 X 座标作为函数的输入，Y 座标 作为函数的输出。这样剩下的 Z 座标也可以作为一个空间维度的输入。为了将其可视化，我们需要修改一下我们的着色器(Shader)程序，使用 Z 座标来设定蓝色通道的数值。在计算反射率(Albedo)的时候使用 rgb 与 xyz 代替 rg 与 xy。  

```CS
			o.Albedo.rgb = IN.worldPos.xyz * 0.5 + 0.5;
```

我们还需要为 `Graph` 类中的连个函数增加一个参数，否则它们没法使用新的输入。  

```CS
	static float SineFunction (float x, float z, float t) {
		return Mathf.Sin(Mathf.PI * (x + t));
	}

	static float MultiSineFunction (float x, float z, float t) {
		float y = Mathf.Sin(Mathf.PI * (x + t));
		y += Mathf.Sin(2f * Mathf.PI * (x + 2f * t)) / 2f;
		y *= 2f / 3f;
		return y;
	}
```

我们还需要在 `Update` 函数中调用函数时为其传递顶点的 Z 座标作为参数。  

```CS
			position.y = f(position.x, position.z, t);
```

### 2.1 修改函数 | Adjusting the Functions

为了支持除了时间之外的第二个维度的输入，我们需要在 GraphFunction 委托类型的 x 参数后新加一个 z 参数。  

```CS
public delegate float GraphFunction (float x, float z, float t);
```

我们需要对 Graph 类中的两个函数做相同的修改，即使它们现在没有使用这两个参数。

```CS
	static float SineFunction (float x, float z, float t) {
		return Mathf.Sin(Mathf.PI * (x + t));
	}

	static float MultiSineFunction (float x, float z, float t) {
		float y = Mathf.Sin(Mathf.PI * (x + t));
		y += Mathf.Sin(2f * Mathf.PI * (x + 2f * t)) / 2f;
		y *= 2f / 3f;
		return y;
	}
```

为了能够正常工作，我们还需要在 Update 方法中调用委托的时候提供 z 座标作为实参。

```CS
			position.y = f(position.x, position.z, t);
```

### 2.2 创建顶点网格 | Creating a Grid of Points

为了显示维度 Z，我们需要把图形由顶点曲线转换为顶点网格。我们可以创建多个直线段，每个直线段沿着 Z 轴偏移一定的步长。我们打算让 Z 的取值范围和 X 相同，所以我们需要创建和当前顶点数量相同的线段。这意味着我们需要的顶点数量是当前顶点数量的平方。修改 `Awake` 函数中 `points` 数组的创建以便容纳足够多数量的顶点。  

```CS
		points = new Transform[resolution * resolution];
```

由于我们目前创建顶点时只在 X 方向上进行迭代，随着顶点数量的增加，我们只是得到了一条比原先更长的线。我们需要修改初始化循环以便引入第二个维度。  

![过长的曲线](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/long-line.png)

首先，我们需要显式的跟踪 X 坐标的值。我们需要声明一个循环变量 `x` ，然后在 for 循环中对其进行步增，就像对变量 `i` 做的一样。循环语句中的变量定义与递增可以使用逗号进行分隔。  

```CS
		for (int i = 0, x = 0; i < points.Length; i++, x++) {
			Transform point = Instantiate(pointPrefab);
			position.x = (i + 0.5f) * step - 1f;
			point.localPosition = position;
			point.localScale = scale;
			point.SetParent(transform, false);
			points[i] = point;
		}
```

每当我们遍历完一行，我们都需要将 `x` 重新置零。每行在 `x` 等于 `resolution` 时结束，我们要在循环体头部使用 `if` 语句对其进行判断，然后在计算坐标时使用 `x` 代替 `i`。  

```CS
		for (int i = 0, x = 0; i < points.Length; i++, x++) {
			if (x == resolution) {
				x = 0;
			}
			Transform point = Instantiate(pointPrefab);
			position.x = (x + 0.5f) * step - 1f;
			point.localPosition = position;
			point.localScale = scale;
			point.SetParent(transform, false);
			points[i] = point;
		}
```

然后要给每一行在 Z 方向上做一个偏移,为此我们需要为循环添加一个循环变量 `z` ，不过我们不必在每一次迭代中都对其进行递增，相反的，我们只需要在每行结束时对其进行递增就好了，也就是在刚才的 `if` 语句中对其递增。最后，我们在循环中设置顶点 Z 方向上的坐标就好了。  

```CS
		for (int i = 0, x = 0, z = 0; i < points.Length; i++, x++) {
			if (x == resolution) {
				x = 0;
				z += 1;
			}
			Transform point = Instantiate(pointPrefab);
			position.x = (x + 0.5f) * step - 1f;
			position.z = (z + 0.5f) * step - 1f;
			point.localPosition = position;
			point.localScale = scale;
			point.SetParent(transform, false);
			points[i] = point;
		}
```

我们终于创建出顶点网格了，不过由于我们的函数依旧只以 x 坐标作为输入，现在的网格看起来就像把原先的曲线的每一个顶点拉直成一条线段一样。  

![正弦波网格](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/grid.png)

由于我们在一个很小的空间内创建了大量的网格顶点，它们之间会互相投射阴影。在默认情况下方向光(Directional Light)的的 Y 方向是 -30°，这将在我们图形的正面投射大量的阴影。你可以旋转方向光源以获得更好的色彩，或者你可以直接关闭阴影投射。  

![将光照沿 Y 轴旋转30°](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/grid-light-rotated.png)

> 为什么帧率下降了那么多？
>
> 相比于之前的曲线，顶点网格包含了更多的顶点。如果 resolution 被设置为 50，就有 2,500 个顶点。如果 resolution 被设置为 100，就有 10,000 个顶点。你的电脑应该能够很好地解决这个问题，但是如果图形在游戏视图和场景视图同时可见的话可能会有些困难，因为增加了一倍的工作量。当你在场景试图中选择了物体并且编辑器为其添加描边的时候效率问题将更加严重。  

### 2.3 双重循环 | Double Looping

虽然我们现在能够正确地创建顶点网格了，但是使用 `if` 语句的方式还是有些愚蠢。更具有可读性的方式是使用二重循环分别遍历网格的每一个方向。为此，我们需要删除旧有的 `for` 循环和 `if` 语句，然后在外层的 Z 方向的循环中添加一个沿 X 方向的循环。我们在内部嵌套的二重循环内创建顶点，这样的话我们就能沿 X 方向遍历一行之后使 Z 座标递增。  

我们不再需要循环变量 `i` 来终止循环，但是我们还要使用它作为 `points` 数组的索引。我们在外层循环中定义它，然后在内层循环对其递增，这样的话其在整个循环过程中的都是可见的，并且每访问一个顶点都能对其递增。  

```CS
//		for (int i = 0, x = 0, z = 0; i < points.Length; i++, x++) {
//			if (x == resolution) {
//				x = 0;
//				z += 1;
//			}
		for (int i = 0, z = 0; z < resolution; z++) {
			for (int x = 0; x < resolution; x++, i++) {
				Transform point = Instantiate(pointPrefab);
				position.x = (x + 0.5f) * step - 1f;
				position.z = (z + 0.5f) * step - 1f;
				point.localPosition = position;
				point.localScale = scale;
				point.SetParent(transform, false);
				points[i] = point;
			}
		}
```

需要注意的是 Z 座标只在每次外层循环的迭代中发生变化，我们可以将其由内层循环提到外层循环，以减少不必要的计算。  

```CS
		for (int i = 0, z = 0; z < resolution; z++) {
			position.z = (z + 0.5f) * step - 1f;
			for (int x = 0; x < resolution; x++, i++) {
				Transform point = Instantiate(pointPrefab);
				position.x = (x + 0.5f) * step - 1f;
//				position.z = (z + 0.5f) * step - 1f;
				point.localPosition = position;
				point.localScale = scale;
				point.SetParent(transform, false);
				points[i] = point;
			}
		}
```

> 选择在外层循环中遍历哪一个方向有什么影响么？
> 
> 我们在外层循环中遍历 Z 方向，在内层循环中遍历 X 方向，但是使用相反的方式也是可以的。如果你跟我一样在外层循环中遍历 Z 方向的话，意味着你创建的行将与 X 轴平行，然后沿着 Z 轴依次偏移这些行，否则的话则相反。这两种方式除了顶点创建的顺序之外没有什么不同。  

### 2.4 添加 Z 轴 | Incorporating Z

我们已经有了一个 2D 顶点网格作为输入了，现在我们来尝试操控这第二个维度。不过首先我们要做的事是定义一个局部的 Π 常量，因为我们会经常用到它，每次都调用 `Mathf.PI` 的话不太方便。  

```CS
	const float pi = Mathf.PI;

	static float SineFunction (float x, float z, float t) {
		return Mathf.Sin(pi * (x + t));
	}

	static float MultiSineFunction (float x, float z, float t) {
		float y = Mathf.Sin(pi * (x + t));
		y += Mathf.Sin(2f * pi * (x + 2f * t)) / 2f;
		y *= 2f / 3f;
		return y;
	}
```

我们并不打算在原有的两个函数上做修改，而是创建一个使用 X 和 Z 座标进行输入的新函数。我们创建一个名为 `Sine2DFunction` 的方法来代表函数//todo,这是一个最简单的基于 X 和 Z 座标的正弦波。  

```CS
	static float Sine2DFunction (float x, float z, float t) {
		return Mathf.Sin(pi * (x + z + t));
	}
```

把这个函数添加到 functions 数组中，将其放到 SineFunction 之后。  

```CS
	static GraphFunction[] functions = {
		SineFunction, Sine2DFunction, MultiSineFunction
	};
```

同时为 GraphFunctionName 枚举类添加一个新的标签 Sine2D。  

```CS
public enum GraphFunctionName {
	Sine, Sine2D, MultiSine
}
```

![对角正弦波]()

运行游戏时你会看到一个非常熟悉的正弦波，只不过它是沿着 XZ 轴对角线方向，而不是沿着 X 轴了。这是因为我们使用 `x + z` 而不是 `x` 作为正弦函数的输入。  

另一个使用两个维度的可行并且更有意思的方案是为每一个维度的输入使用独立的正弦函数，并将它们合并在一起。一个最简单的合并方式就是将其相加，然后结果除以二，这样最终的函数值还是会落到 [-1, 1] 的区间内。这样我么们就会得到如下的数学函数//todo。为了让代码更易读，我们将使用一个中间变量 `y` 并将代码分成 2 行。  

```CS
	static float Sine2DFunction (float x, float z, float t) {
//		return Mathf.Sin(pi * (x + z + t));
		float y = Mathf.Sin(pi * (x + t));
		y += Mathf.Sin(pi * (z + t));
		y *= 0.5f;
		return y;
	}
```

![每个维度一个正弦波]()

> 为什么使用 +=0.5f 而不是 /=2f？
>
> 这两种方式在数学意义上是等价的，但是在计算机中乘法指令要比除法指令运行的快。如果你在循环中需要进行大量的计算的话，这是一种简单的优化方法。虽然这不是必须的但是是一种良好的习惯。你也可以将前面 `MultiSineFunction` 中的除法替换为乘法。

我们再为 `MultiSineFunction` 创建一个 2D 输入的版本吧。在这里，我们依旧会使用一个主波，但是会为每一个维度的输入创建一个次波。这样，我们就会得到形如//todo的函数，//todo代表主波，//todo代表 X 方向上的次波，//todo代表 Y 方向上的次波。  

主波我们将会使用//todo，是一个缓慢移动的对角波。X 方向上的次波是//todo，它的速度居中。Y 方向上的次波是//todo，它的速度最快。  

我们还想让主波的振幅最大，达到 X 方向次波的 4 倍。由于 Z 方向上的次波频率是 X 方向的两倍，我们让它的振幅是 X 方向上的一半。最终得到的函数时//todo，将其函数值除以5.5就能把值域缩小到 [-1, 1] 区间内。我们将这个函数命名为 `MultiSine2DFunction`。  

```CS
	static float MultiSine2DFunction (float x, float z, float t) {
		float y = 4f * Mathf.Sin(pi * (x + z + t * 0.5f));
		y += Mathf.Sin(pi * (x + t));
		y += Mathf.Sin(2f * pi * (z + 2f * t)) * 0.5f;
		y *= 1f / 5.5f;
		return y;
	}
```

将其添加到 `functions` 数组中。  

```CS
	static GraphFunction[] functions = {
		SineFunction, Sine2DFunction, MultiSineFunction, MultiSine2DFunction
	};
```

最后再枚举类中为其添加一个标签。  

```CS
public enum GraphFunctionName {
	Sine, Sine2D, MultiSine, MultiSine2D
}
```

![混合了三个正弦波的波形]()

### 2.5 创建波纹 | Creating a Ripple

我们将再创建一个 2D 函数来展示曲面波纹动画。我们打算让波纹沿着四面八方传开形成一些列的水圈。为了实现这一点，我们需要创建一个基于距中心点距离的正弦波。据中心点距离可以使用勾股定理(//todo)计算出来。  

对于 XZ 平面上的 2D 顶点，三角形的斜边长对应着原点到该顶点的距离，其 X 和 Z 座标对应着直角三角形的两条直角边。因此，顶点据原点的距离公式是//todo。

![勾股定理](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/pythagorean-theorem.png)

创建一个名为 `Ripple` 的函数，我们先只在其中计算距离。其中，使用 `Mathf.Sqrt` 来计算平方根。现在,我们直接将距离作为输出。  

```CS
	static float Ripple (float x, float z, float t) {
		float d = Mathf.Sqrt(x * x + z * z);
		float y = d;
		return y;
	}
```

将其添加到 `functions` 数组中。  

```CS
	static GraphFunction[] functions = {
		SineFunction, Sine2DFunction, MultiSineFunction, MultiSine2DFunction,
		Ripple
	};
```

然后把它的名字添加到枚举类里。  

```CS
public enum GraphFunctionName {
	Sine, Sine2D, MultiSine, MultiSine2D,
	Ripple
}
```

![距离原点的距离](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/distance.png)

我们得到了一个圆锥面，这个圆锥面中心最低，其他的点随着据中心距离的增大高度线性递增。最高的点是网格的四个角点，因为这四个点离中心点最远。准确的来说，这四个角距离中心的距离是//todo,约等于1.4142。  

为了得到波面,我们要使用公式//todo,其中 D 是距中心点距离。  

![基于距中心距离的正弦波](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/sin-d.png)

波纹看起来有点少，我们把频率提升为原来的 4 倍试试看。   

```CS
		float y = Mathf.Sin(4f * pi * d);
```

![更高的频率](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/sin-d-higher-frequency.png)

距我们想要的形状越来越近了，但是现在波纹的起伏实在是太大了。我们需要降低波纹的振幅，为了看起来更自然，我们让振幅的大小取决与据中心点距离。例如，我们用//todo作为振幅，这样，随着据中心点距离的增大波动会越来越弱，我们不需要使用严谨的物理公式就得到了近似的效果。然而这样会导致在中心点除以0的问题，我们可以使用//todo替代刚才的那个公式。

```CS
		float y = Mathf.Sin(4f * pi * d);
		y /= 1f + 10f * d;
		return y;
```

![按距离缩放振幅](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/sin-d-scaled.png)

最后，添加时间以便让波纹产生动画效果。由于波纹需要向外运动，我们减去 `t` 而不是加上它。  
```CS
		float y = Mathf.Sin(pi * (4f * d - t));
```

![动态波纹]()

## 3 脱离网格 | Leaving the Grid

通过使用 X 和 Z 座标来计算 Y，我们已经创建了大量的曲面，但是这些曲面还是依附在 XZ 平面上。没有两个点能拥有相同的 X 和 Z 座标而拥有不同的 Y 座标。这意味着我们的曲面的曲率是受限制的，它不能垂直，也不能向内弯曲。如果要这样的话，我们的函数不止应输出 Y 座标，也要输出 X 和 Z 座标。  

### 3.1 三维函数 | Three-dimensional Functions

如果我们的函数能够输出三维座标而不是一维的数值，我们就能过创建任意的曲面了。例如，函数//todo描述了 XZ 平面，而函数//todo描述了 XY 平面。由于它们的输入参数不在对应着最终的某个 X 或 Z 座标，那么再将其命名为 `x` 或 `z` 就不太合适了。事实上，我们习惯上将其命名为 `u` 和 `v`。所以，我们得到了类似//todo的函数。  

修改我们的 `GraphFunction` 委托以支持这个最新的方法。只需要将其返回值由 `float` 变为 `Vector3`，然后更改参数的名称。  

```CS
public delegate Vector3 GraphFunction (float u, float v, float t);
```

我们最初的 `SineFunction` 现在变成了/todo。由于我们在函数中并没有改变 `x` 和 `z` 座标，所以我们并没有更改参数的名字。它现在返回一个 `Vector`，我们直接使用参数的 `x` 和 `z` 作为返回值的 `x` 和 `z`，然后单独计算 Y 座标。  

```CS
	static Vector3 SineFunction (float x, float z, float t) {
//		return Mathf.Sin(pi * (x + t));
		
		Vector3 p;
		p.x = x;
		p.y = Mathf.Sin(pi * (x + t));
		p.z = z;
		return p;
	}
```

我们对 `Sine2DFunction` 也做相同的操作。  

```CS
	static Vector3 Sine2DFunction (float x, float z, float t) {
//		float y = Mathf.Sin(pi * (x + t));
//		y += Mathf.Sin(pi * (z + t));
//		y *= 0.5f;
//		return y;
		
		Vector3 p;
		p.x = x;
		p.y = Mathf.Sin(pi * (x + t));
		p.y += Mathf.Sin(pi * (z + t));
		p.y *= 0.5f;
		p.z = z;
		return p;
	}
```

其余的三个函数也是一样。  

```CS
	static Vector3 MultiSineFunction (float x, float z, float t) {
		Vector3 p;
		p.x = x;
		p.y = Mathf.Sin(pi * (x + t));
		p.y += Mathf.Sin(2f * pi * (x + 2f * t)) / 2f;
		p.y *= 2f / 3f;
		p.z = z;
		return p;
	}
```

```CS
	static Vector3 MultiSine2DFunction (float x, float z, float t) {
		Vector3 p;
		p.x = x;
		p.y = 4f * Mathf.Sin(pi * (x + z + t / 2f));
		p.y += Mathf.Sin(pi * (x + t));
		p.y += Mathf.Sin(2f * pi * (z + 2f * t)) * 0.5f;
		p.y *= 1f / 5.5f;
		p.z = z;
		return p;
	}
```

```CS
	static Vector3 Ripple (float x, float z, float t) {
		Vector3 p;
		float d = Mathf.Sqrt(x * x + z * z);
		p.x = x;
		p.y = Mathf.Sin(pi * (4f * d - t));
		p.y /= 1f + 10f * d;
		p.z = z;
		return p;
	}
```

因为现在顶点的 X 和 Z 座标不再保持不变了，我们不能在 `Update` 函数中直接使用它们的初始值了。相反，我们要更新 UV 座标作为输入，我们需要使用一个二重循环来实现。然后直接把计算所得的返回值作为顶点的座标。  

```CS
	void Update () {
		float t = Time.time;
		GraphFunction f = functions[(int)function];
//		for (int i = 0; i < points.Length; i++) {
//			Transform point = points[i];
//			Vector3 position = point.localPosition;
//			position.y = f(position.x, position.z, t);
//			point.localPosition = position;
//		}
		float step = 2f / resolution;
		for (int i = 0, z = 0; z < resolution; z++) {
			float v = (z + 0.5f) * step - 1f;
			for (int x = 0; x < resolution; x++, i++) {
				float u = (x + 0.5f) * step - 1f;
				points[i].localPosition = f(u, v, t);
			}
		}
	}
```

由于新的方法不再使用顶点的初始值了，我们不需要在 `Awake` 函数中计算他们了，我们可以将其直接删除。我们使用一个循环初始化所有顶点就不用再管它了。  

```CS
	void Awake () {
		float step = 2f / resolution;
		Vector3 scale = Vector3.one * step;
//		Vector3 position;
//		position.y = 0f;
//		position.z = 0f;
		points = new Transform[resolution * resolution];
//		for (int i = 0, z = 0; z < resolution; z++) {
//			position.z = (z + 0.5f) * step - 1f;
//			for (int x = 0; x < resolution; x++, i++) {
//				Transform point = Instantiate(pointPrefab);
//				position.x = (x + 0.5f) * step - 1f;
//				point.localPosition = position;
//				point.localScale = scale;
//				point.SetParent(transform, false);
//				points[i] = point;
//			}
//		}
		for (int i = 0; i < points.Length; i++) {
			Transform point = Instantiate(pointPrefab);
			point.localScale = scale;
			point.SetParent(transform, false);
			points[i] = point;
		}
	}
```

### 3.2 创建圆柱体 | Creating a Cylinder

为了证明我们不再局限于每一个 (X, Z) 数对对应一个顶点，我们来创建一个圆柱面。首先我们创建一个名为 `Cylinder` 的函数，让它直接返回原点。  

像往常一样，我们需要将其添加到 `functions` 数组中，并为其在 `GraphFunctionName` 枚举类中添加一个标签，以后我不会再重复说明。  

```CS
	static Vector3 Cylinder (float u, float v, float t) {
		Vector3 p;
		p.x = 0f;
		p.y = 0f;
		p.z = 0f;
		return p;
	}
```

圆柱面是由圆形挤压出来的，故我们首先创建一个圆形。在[上一篇教程](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/)中我们提到过。可以通过公式//todo定义二维圆环上的所有顶点，其中自变量//todo的作用域是 [0, 2π]。我们可以使用//todo来代表//todo，这里我们的//todo的作用域是 [-1, 1]。为了让圆环定义在 XZ 平面上，我们使用公式//todo。 

```CS
	static Vector3 Cylinder (float u, float v, float t) {
		Vector3 p;
		p.x = Mathf.Sin(pi * u);
		p.y = 0f;
		p.z = Mathf.Cos(pi * u);
		return p;
	}
```

![圆环](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/circle.png)

// todo 这他妈怎么翻  
Because the function doesn't use v, all points that use the same v input end up at the exact same position. So we're effectively reduced to a single line. To see how this line wraps around to form a circle, make Y equal to u.

```CS
		p.x = Mathf.Sin(pi * u);
		p.y = u;
		p.z = Mathf.Cos(pi * u);
```

![Y 座标递增的圆环](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/rising-circle.png)

这样，我们得到了一条起始于点(0, -1, -1)并且随着输入绕原点顺时针方向转动的曲线。我们使用参数 `v` 为 Y 坐标赋值就能得到一个圆柱面了。实际上这相当于把一个个圆环竖向罗放在一起。  

```CS
		p.x = Mathf.Sin(pi * u);
		p.y = v;
		p.z = Mathf.Cos(pi * u);
```

![圆柱面](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/cylinder.png)

我们现在使用一个单位圆作为圆柱面的基底，但是这不是必须的。我们可以通过修改正弦与余弦函数的振幅来调节圆环的半径。一般来说，在公式//todo中，R 就代表了圆环的半径。我们修改我们的函数将半径显式的设置为 1。  

```CS
		float r = 1f;
		p.x = r * Mathf.Sin(pi * u);
		p.y = v;
		p.z = r * Mathf.Cos(pi * u);
```

> 如果使用不一样的振幅会发生什么？
>
> 如果正弦函数和余弦函数的模式号不同的话，我们将得到一个椭圆。  

我们也可以使用其他半径，甚至可以不是一个常数。例如，我们可以让半径随着 `u` 发生变化。我们用另一个正弦函数驱动半径，例如//todo。  

```CS
		float r = 1f + Mathf.Sin(6f * pi * u) * 0.2f;
```

![波动的柱面](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/wobbly-u-cylinder.png)

这样柱面就发生了变化,由圆环变成了圆角星形，曲面沿着圆周方向发生六次波动。  

我们也可以让半径随变量 `v` 发生变化，例如//todo。在这种情况下，柱面的每层圆环的半径都是一个定值，但是半径的大小随着柱面的高度发生变化。  

```CS
		float r = 1f + Mathf.Sin(2f * pi * v) * 0.2f;
```

![使用参数 v 替代参数 u](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/wobbly-v-cylinder.png)

我们还可以让半径随着 `uv` 两者的改变而发生改变，这将产生一个对角线方向的波纹，最终使柱面发生扭曲。我们现在为其添加时间参数好让它动起来，将半径技术缩减到 4/5，以确保最终的半径不会大于 1。  

```CS
		float r = 0.8f + Mathf.Sin(pi * (6f * u + 2f * v + t)) * 0.2f;
```

![扭曲的柱面]()

### 3.3 创建球体 | Creating a Sphere

我们已经拥有了一个圆柱面，现在让我们把它变成一个球吧。复制 `Cylinder` 函数然后将其重命名为 `Sphere`。为了让圆柱面变成球面，我们需要用另外一个波函数让圆环的半径在顶部和底部缩减为 0。我们先来试试//todo。  

```CS
	static Vector3 Sphere (float u, float v, float t) {
		Vector3 p;
		float r = Mathf.Cos(pi * 0.5f * v);
		p.x = r * Mathf.Sin(pi * u);
		p.y = v;
		p.z = r * Mathf.Cos(pi * u);
		return p;
	}
```

![几乎是一个球了](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/sphere-almost.png)

我们已经很接近了，但是还是不够圆。这是因为圆形时使用正弦波和余弦波拟合而成的，但是我们这里只使用了余弦波。另外一个需要处理的 Y 坐标，现在它直接等于 `v`，我们需要让它等于//todo。  

```CS
		p.y = Mathf.Sin(pi * 0.5f * v);
```

![球面](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/sphere.png)

我们终于得到了一个球体，这个生成球体的方案一般被称为 UV-Sphere。虽然我们生成了正确的球体，但是需要注意的是球体表面顶点的分布并不均匀，这是因为球时使用不同半径大小的圆环叠加而成的。在球的两极，半径趋近于零。
为了能够控制球的半径，我们需要对公式做一些微小的调整。我们将使用公式//todo，其中//todo，R 代表半径。  

这种方法能够使球面的半径发生动态变化。我们这回使用不同的正弦波分别处理 `u` 和 `v`，//todo。  

```CS
		float r = 0.8f + Mathf.Sin(pi * (6f * u + t)) * 0.1f;
		r += Mathf.Sin(pi * (4f * v + t)) * 0.1f;
		float s = r * Mathf.Cos(pi * 0.5f * v);
		p.x = s * Mathf.Sin(pi * u);
		p.y = r * Mathf.Sin(pi * 0.5f * v);
		p.z = s * Mathf.Cos(pi * u);
```

![跳动的球面]()

### 3.4 创建环面 | Creating a Torus

我们就以将球面转变为环面作为本教程的终点。复制 `Sphere` 函数并将其重命名为 `Torus`，然后删除计算球面半径的代码。  

```CS
	static Vector3 Torus (float u, float v, float t) {
		Vector3 p;
		float s = Mathf.Cos(pi * 0.5f * v);
		p.x = s * Mathf.Sin(pi * u);
		p.y = Mathf.Sin(pi * 0.5f * v);
		p.z = s * Mathf.Cos(pi * u);
		return p;
	}
```

我们可以通过把球面拉开的方式创建环面，保持住球体的南北两极高度不变，然后在 XZ 平面沿着径向拉动球面。我们可以通过为 S 添加一个常量来做到这一点，比如说 1/2。  

```CS
		float s = Mathf.Cos(pi * 0.5f * v) + 0.5f;
```

![扯开一个球体](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/pulling-apart-sphere.png)

这样，我们就得到了半个环面了，这有朝外的那部分。为了得到完整的环面，我们需要使用参数 v 来描述整个环面而不是半个。我们可以使用 πv 来取代 πv/2。  

```CS
		float s = Mathf.Cos(pi * v) + 0.5f;
		p.x = s * Mathf.Sin(pi * u);
		p.y = Mathf.Sin(pi * v);
		p.z = s * Mathf.Cos(pi * u);
```

![自交的 Spindle Torus](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/torus-spindle.png)

因为我们把我们的球面向径向方向拉动了半个单位，这产生了一个自相交的形状，这个形状被称为 Spindle Torus。我们可以继续向外拉动半个单位，这样我们会得到一个既不会发生自相交，也没有在中央形成空洞的形状，这个形状被称为 Horn Torus。所以我们把球面向外拉动的距离将决定环面的形状。具体地来说，这定义了环面的主半径，我们使用//todo来表示。所以我们的公式变成了//todo，其中//todo。  

```CS
		float r1 = 1f;
		float s = Mathf.Cos(pi * v) + r1;
```

![A horn torus](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/torus-horn.png)

如果//todo大于 1 的话将会在环面中心形成一个空洞，这样就形成了一个圆环面(Ring Rorus)。这里我们假定包围中央圆环的外面的圆的半径一直是 1，它被称为环面的副半径，我们一样可以改变它。我们的函数变为了//todo，其中//todo。  

这里，我们将主半径设为 1，副半径设为 1/2。  

```CS
		float r1 = 1f;
		float r2 = 0.5f;
		float s = r2 * Mathf.Cos(pi * v) + r1;
		p.x = s * Mathf.Sin(pi * u);
		p.y = r2 * Mathf.Sin(pi * v);
		p.z = s * Mathf.Cos(pi * u);
```

![A ring torus](https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/3_mathematical_surfaces/torus-ring.png)

我们现在同时改变两个半径来让圆环显得有趣些。一个简单并且有趣的方式是为//todo添加一个 `u` 的正弦波，为//todo添加一个 `v` 的正弦波，然后让他动起来。同时要注意的是确保环面在[-1, +1]的范围内。  

```CS
		float r1 = 0.65f + Mathf.Sin(pi * (6f * u + t)) * 0.1f;
		float r2 = 0.2f + Mathf.Sin(pi * (4f * v + t)) * 0.05f;
```

![一个有趣的环面]()

现在我们已经有了一些关于使用非一般的函数(Nontrivial Functions)来描述三维曲面的经验，并且知道了如何将其可视化。你可以使用你自己的函数来更好的了解它的工作原理。有很多看起来非常复杂的形状使用几个正弦波就能搞定。在你完成以后，请移步到[创建分形体](https://catlikecoding.com/unity/tutorials/constructing-a-fractal/)。