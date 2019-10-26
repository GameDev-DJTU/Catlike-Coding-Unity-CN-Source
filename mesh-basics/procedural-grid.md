# 程式化生成网格 | Procedural Grid
## 通过编程的方式 | Programming Form

创建网格的顶点  
使用协程来展现创建过程  
自动地生成顶点法线  
添加纹理坐标与切线  

本篇教程中我们会创建一个 Mesh 网格的顶点以及三角形片面。  

本篇教学假定您已经熟悉了基本的 Unity 脚本编程内容。如果这对您有一定的困难，请
阅读 [钟表]() 教程以了解 Unity 脚本编程的基础内容。[构造分形几何体]() 教程提供了坐标系统的入门指导。本篇教程的 Unity 版本要求为 5.0.1 及以上。  

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/tutorial-image.jpg" /></center>
<center>复杂表面下隐藏的简单几何网格</center>

## 1 渲染物体 | Rendering Things

如果你想使物体在 Unity 中可见，你需要使用 Mesh 网格。它可以是从其他软件中导出的 3D 模型，也可以是程序自动生成的网格，它还可以是精灵 ( Sprite )，UI 组件 ( UI element )，质点系统 ( Particle System ) 以及一切 Unity 需要使用网格的地方。甚至是屏幕特效 ( Screen effects ) 也是使用网格来渲染的。  

那么网格是什么呢？网格的概念与计算机图形硬件绘制复杂图形有关。它至少包含一个顶点 ( Vertices ) 的集合（在三维空间内被定义的一系列的点）再加上一个三角形片面 ( Triangles ) 的集合（最基本的二维图形，连接了点与点）。无论一个网格描绘了什么，这些三角形片面都是组成其表面的基本元素。  

因为三角形片面是平的并且拥有直线边，它很适合被用来表现那些具有平面与直线的几何体，例如正方体的表面。弯曲或者圆滑的表面只能用这些三角形片面来无限逼近。如果细分的三角形片面足够小的话，比如说比一个像素还小，那么你将不会注意到其是由三角形逼近的。然而实时渲染达不到这一点，所以曲面经常是参差不齐的。  

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/01-shaded.png" /></center>

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/01-wireframe.png" /></center>

<center>Unity自带的胶囊体，正方体，球体的着色渲染与线框渲染</center>

> 如何显示线框渲染呢？
>
> 你可以在 Scene 窗口工具栏的左上角选择它的显示模式。头三个选项分别是着色模式，线框模式，着色与线框模式

如果你想要使一个游戏对象的三维模型被显示出来的话，它需要拥有两个组件。第一个组件是 Mesh Filter，这个组件持有你想要显示的 Mesh 网格的引用。第二个组件是 Mesh Renderer，这个组件决定了这个 Mesh 网格是如何被渲染的，例如应当使用哪个材质 ( Material)、是否应当投射或接受阴影等等。  

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/01-cube-object.png" /></center>

<center>Unity自带的Cube游戏对象</center>

> 为什么这里有材质数组？
>
> 一个 Mesh Renderer 可以拥有多组材质。这主要是用于渲染具有多组独立的三角形集合的网格，例如 Submeshes。常用于外部导入的 3D 模型，本教程并不会涉及此内容。

你通过调整材质完全地改变Mesh网格的表现形式。Unity的默认材质是简单的白色固体。你可以通过 Assets / Create / Material 路径创建一个新的材质，并将其拖拽到游戏对象上。新的材质将默认使用 Unity 标准着色器，它提供了一些能够让你轻微调整多边形表面视觉效果的方法。  

为你的 Mesh 网格添加足够多的细节的一个快速方式是为其添加一个反射率贴图 ( Albedo Map )，它是一个表现了材质基本颜色的纹理 ( Texture )。当然，我们需要直到如何讲这个纹理映射到 Mesh 网格的三角形片面上，这是通过向顶点添加 2D 纹理座标 ( 2D Texture Coordinates ) 来实现的。纹理空间 ( Texture Space ) 的两个维度被称为 U 和 V，所以我们也将其称为 UV 坐标 ( UV coordinates )。纹理坐标的范围通常是从 (0, 0) 到 (1, 1)，并且覆盖了整张材质。却决于纹理的设置，超出范围的座标要么循环回去产生瓦片 ( Tile  ) 效果，要么被束缚为边界值。  

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/01-uv-texture.png" /></center>

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/01-material.png" /></center>

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/01-textured.png" /></center>

<center>一个应用于Unity 自带 网格的 UV 纹理</center>

## 2 创建顶点阵列 | Creating a Grid of Vertices

然而如何才能拥有属于你自己的 Mesh 网格呢？让我们通过创造一个简单的矩形网格来找寻答案。这个网格将由一种被成为 Quads 的单位长度的矩形瓦片构成。创建一个 C# 脚本并把它变成一个拥有水平和竖直大小的网格组件。  

```CS
using UnityEngine;
using System.Collections;

public class Grid : MonoBehaviour {

	public int xSize, ySize;
}
```

> 我们需要使用 System.Collections 名字空间么？
>
> 我们不需要它便可创建我们的 Mesh 网格。在这里使用它是因为接下来我们将要使用协程。

当我们将这一组件添加到游戏对象时，我们同样也需要为游戏对象添加 Mesh Renderer 与 Mesh Filter组件。我们可以为我们的类添加一组特性 ( Attribute )，以便 Unity 能自动地帮我们添加它们。  

```CS
[RequireComponent(typeof(MeshFilter), typeof(MeshRenderer))]
public class Grid : MonoBehaviour {

	public int xSize, ySize;
}
```

现在你需要做的事创建一个空的游戏对象，并向其添加我们自己些的组件，一旦你添加了这个组件，我们声明为特性的那两个组件就会自动添加。设置  Mesh Renderer 的材质后，将 Mesh Filter 的 Mesh 空下，然后将网格大小设置为 10 * 5。

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/02-grid.png" /></center>

<center>一个网格对象</center>

一旦该对象进入唤醒状态时我们就创建网格，唤醒状态指当我们刚进入游戏模式时。

```CS
	private void Awake () {
		Generate();
	}
```

让我们先关注顶点的位置，过一会再去看三角形相关的内容。我们需要持有一个 3D 向量的数组来储存这些点。顶点的数量取决于网格的大小。我们需要在四边形的每一个角上有一个顶点，但是相邻的四边形上共享顶点。所以在每一个维度上我们所需要的顶点比瓦片多 1。

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/02-grid-indices.png" /></center>

<center>网格的顶点与四边形</center>

```CS
	private Vector3[] vertices;

	private void Generate () {
		vertices = new Vector3[(xSize + 1) * (ySize + 1)];
	}
```

让我们将顶点可视化以便能够正确的检查它们的位置。我们可以通过添加 OnDrawGizmos方法来在Scene视图中的每个顶点位置上绘制一些球体。

```CS
	private void OnDrawGizmos () {
		Gizmos.color = Color.black;
		for (int i = 0; i < vertices.Length; i++) {
			Gizmos.DrawSphere(vertices[i], 0.1f);
		}
	}
```

> 什么是 Gizmos？
>
> Gizmos 是你可以在编辑器中看到的可视化提示。在默认情况下他们在 Scene 视图中可见而在 Game 视图中不可见，但是你可以通过工具栏来修改它。 Gizmos 类允许你绘制图标，线条或者一些其他东西。  
> Gizmos可以在OnDrawGizmos方法中被绘制，该方法可以被编辑器自动调用。还有一个可供选择的方法是 OnDrawGizmosSelected，该方法只会显示被选择的对象。

这将使我们在不处于游戏模式的时候引发一个错误，因为 OnDrawGizmos 方法会在Unity处于编辑模式的时候被调用，而这时我们还不拥有任何顶点。为了预防这个错误，我们可以在绘制之前检查顶点数组是否存在。  

```CS
	private void OnDrawGizmos () {
		if (vertices == null) {
			return;
		}
		…
	}
```

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/02-gizmo.png" /></center>

<center>A Gizmo</center>

进入游戏模式之后，我们会发现只在原点有一个球体。这是因为我们还没有为顶点指定坐标，所以他们都重叠在一个位置。我们通过一个双重循环迭代出所有的坐标。

```CS
	private void Generate () {
		vertices = new Vector3[(xSize + 1) * (ySize + 1)];
		for (int i = 0, y = 0; y <= ySize; y++) {
			for (int x = 0; x <= xSize; x++, i++) {
				vertices[i] = new Vector3(x, y);
			}
		}
	}
```

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/02-vertices.png" /></center>

<center>一个顶点阵列</center>

> 为什么 Gizmos 不会随着物体移动？
>
> Gizmos 是在世界坐标系下被绘制的，不是在物体的局部坐标系。如果你想让其座标随着游戏对象一起发生坐标变换，你需要显式地指明。例如用 `transform.TransformPoint(vertices[i])` 取代 `vertices[i]`。

我们现在已经能够看到顶点，但是顶点创建的顺序依旧是不可见的。我们可以使用颜色来表现它，但是我们也可以使用协程使 ( Coroutine ) 程序放缓。这就是我们要在脚本中包含 `using System.Collections` 的原因。

```CS
	private void Awake () {
		StartCoroutine(Generate());
	}

	private IEnumerator Generate () {
		WaitForSeconds wait = new WaitForSeconds(0.05f);
		vertices = new Vector3[(xSize + 1) * (ySize + 1)];
		for (int i = 0, y = 0; y <= ySize; y++) {
			for (int x = 0; x <= xSize; x++, i++) {
				vertices[i] = new Vector3(x, y);
				yield return wait;
			}
		}
	}
```

<center>
<video id="video" controls="" preload="none" poster="https://thumbs.gfycat.com/IgnorantEminentDragon-mobile.jpg" tabindex="-1">
	<source src="https://thumbs.gfycat.com/IgnorantEminentDragon-mobile.mp4" type="video/mp4">
	<source src="https://zippy.gfycat.com/IgnorantEminentDragon.webm" type="video/webm">
	<source src="https://zippy.gfycat.com/IgnorantEminentDragon.mp4" type="video/mp4">
	<source src="https://thumbs.gfycat.com/IgnorantEminentDragon-mobile.mp4" type="video/mp4">
</video>
</center>

<center>观察顶点生成地顺序</center>

## 3 创建 Mesh 网格 | Creating the Mesh

现在我们的顶点已经被正确归位，可以着手解决真正的 Mesh 网格问题了。我们不光要在脚本中持有自己创建 Mesh 网格的引用，还要将其指定给 Mesh Filter。这样一旦我们处理完成顶点了就可以将其赋值给 Mesh 网格。  

```CS
	private Mesh mesh;

	private IEnumerator Generate () {
		WaitForSeconds wait = new WaitForSeconds(0.05f);
		
		GetComponent<MeshFilter>().mesh = mesh = new Mesh();
		mesh.name = "Procedural Grid";

		vertices = new Vector3[(xSize + 1) * (ySize + 1)];
		…
		mesh.vertices = vertices;
	}
```

> 我们真的需要在脚本中持有 Mesh 网格地引用么？
>
> 我们目前确实只需要在 `Generate` 方法中持有其引用。因为 Mesh Filter 持有的引用将会永久保存下来。我们依旧在此脚本中持有其引用是因为在后面的教程中将会用到。

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/03-mesh.png" /></center>

<center>游戏模式中出现的Mesh网格</center>

现在在游戏模式中我们已经拥有一个 Mesh 网格了，现在它依旧不可见是因为我们没有赋予它三角形片面。三角形片面通过一个顶点数组索引定义。因为一个三角形拥有三个点，三个连贯的点描述了一个三角形。让我们从一个三角形开始吧。  

```CS
	private IEnumerator Generate () {
		…

		int[] triangles = new int[3];
		triangles[0] = 0;
		triangles[1] = 1;
		triangles[2] = 2;
		mesh.triangles = triangles;
	}
```

我们现在拥有一个三角形了，但是这三个点全部座落在一条直线上。这产生了一个退化了的不可见的三角形。前两个顶点是没有问题的，但是第三个顶点应该调到第一个顶点的下一行。  

```CS
		triangles[0] = 0;
		triangles[1] = 1;
		triangles[2] = xSize + 1;
```

这样我们构造出了一个三角形，但是他只有一个面是可见的。现在只有沿着 Z 轴的负方向能看到它，所以你可能需要旋转一下视角才可以看到它。  

三角形的那一个面可见取决于顶点的绘制顺序。默认情况下，顺时针顺序被绘制那个面被认为是正面并且可见。逆时针方向被绘制的面会被舍弃，因为我们不必浪费时间去渲染物体的内部我们无论如何都看不到的面。  

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/03-triangle-sides.png" /></center>

<center>三角形的两个面</center>

所以为了确保我们在顺着 Z 轴观察时能看到三角形，我们需要改变三角形顶点的绘制顺序。可以通过改变最后两个顶点的绘制顺序来实现。 

```CS
		triangles[0] = 0;
		triangles[1] = xSize + 1;
		triangles[2] = 1;
```

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/03-first-triangle.png" /></center>

<center>第一个三角形</center>

我们现在有一个三角形能覆盖网格中头一个瓦片的一半。为了覆盖整个瓦片，我们需要第二个三角形。

```CS
		int[] triangles = new int[6];
		triangles[0] = 0;
		triangles[1] = xSize + 1;
		triangles[2] = 1;
		triangles[3] = 1;
		triangles[4] = xSize + 1;
		triangles[5] = xSize + 2;
```

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/03-quad.png" /></center>

<center>由两个三角形组成的四边形</center>

因为这两个三角形共享了两个顶点，我们可以减少到4行代码，这样保证每个顶点只在代码中出现一次。  

```CS
		triangles[0] = 0;
		triangles[3] = triangles[2] = 1;
		triangles[4] = triangles[1] = xSize + 1;
		triangles[5] = xSize + 2;
```

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/03-first-quad.png" /></center>

<center>第一个四边形</center>

我们可以通过将上面的代码放入循环中来绘制第一行所有的三角形。`ti` 是三角形索引 `vi` 是顶点索引。让我们把之前的 `yield` 语句也移进循环中，这样我们就不用等待顶点依次显现了。

```CS
		int[] triangles = new int[xSize * 6];
		for (int ti = 0, vi = 0, x = 0; x < xSize; x++, ti += 6, vi++) {
			triangles[ti] = vi;
			triangles[ti + 3] = triangles[ti + 2] = vi + 1;
			triangles[ti + 4] = triangles[ti + 1] = vi + xSize + 1;
			triangles[ti + 5] = vi + xSize + 2;
			yield return wait;
		}
```

现在顶点将会立即显现，而三角形将会过一会一同出现。为了看到瓦片一个接一个的出现，我们需要在每一个迭代更新 Mesh 网格而不是在在循环之后。

```CS
			mesh.triangles = triangles;
			yield return wait;
```

现在将以上代码再放入一层循环中可以绘制所有的全部网格了。需要注意的是换行的时候需要将纵向坐标加一。  

```CS
		int[] triangles = new int[xSize * ySize * 6];
		for (int ti = 0, vi = 0, y = 0; y < ySize; y++, vi++) {
			for (int x = 0; x < xSize; x++, ti += 6, vi++) {
				…
			}
		}
```

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/03-full-grid.png" /></center>

<center>
<video id="video" controls="" preload="none" poster="https://thumbs.gfycat.com/InsignificantBlackandwhiteChihuahua-mobile.jpg" tabindex="-1">
	<source src="https://thumbs.gfycat.com/InsignificantBlackandwhiteChihuahua-mobile.mp4" type="video/mp4">
	<source src="https://zippy.gfycat.com/InsignificantBlackandwhiteChihuahua.webm" type="video/webm">
	<source src="https://zippy.gfycat.com/InsignificantBlackandwhiteChihuahua.mp4" type="video/mp4">
	<source src="https://thumbs.gfycat.com/InsignificantBlackandwhiteChihuahua-mobile.mp4" type="video/mp4">
</video>
</center>

<center>逐步填充整个网格</center>

正如你所看到的，三角形一行接一行地填充了整个网格。如果你对结果满意了，你可以将所有的协程代码移除以便能立即看到结果。  

```CS
	private void Awake () {
		Generate();
	}

	private void Generate () {
		GetComponent<MeshFilter>().mesh = mesh = new Mesh();
		mesh.name = "Procedural Grid";

		vertices = new Vector3[(xSize + 1) * (ySize + 1)];
		for (int i = 0, y = 0; y <= ySize; y++) {
			for (int x = 0; x <= xSize; x++, i++) {
				vertices[i] = new Vector3(x, y);
			}
		}
		mesh.vertices = vertices;

		int[] triangles = new int[xSize * ySize * 6];
		for (int ti = 0, vi = 0, y = 0; y < ySize; y++, vi++) {
			for (int x = 0; x < xSize; x++, ti += 6, vi++) {
				triangles[ti] = vi;
				triangles[ti + 3] = triangles[ti + 2] = vi + 1;
				triangles[ti + 4] = triangles[ti + 1] = vi + xSize + 1;
				triangles[ti + 5] = vi + xSize + 2;
			}
		}
		mesh.triangles = triangles;
	}
```

> 为什么不只使用一个四边形呢？
>
> 的确，因为我们只创建了一个平整的矩形平面，所以我们只需要两个三角形就可以构造它。而问题的关键点在于更复杂的结构能够让我们对平面进行更复杂的控制、实现更复杂的效果。值得进行尝试！

## 4 生成顶点附加数据 |  Generating Additonal Vertex Data

我们的网格目前在以一种特殊的方式被照明。这是因为我们还没有给MESH网格任何法线 ( Normals ) 信息。默认的法线方向是 (0, 0, 1),这与我们需要的方向是相反的。

> 法线是如何工作的？
>
> 法线是垂直于物体表面的一条向量。正常情况下，我们使用的法线是单位长度的并且指向物体表面外部而不是内部。  
> 法线可以被用来确定光线射入平面的角度，如果光线真的能够照到平面上的话(●’◡’●)。它将具体被如何使用将取决于着色器程序 ( Shader )。
> 因为三角形永远是一个平面，所以本不需要提供单独的法线信息。但是通过这种方式我们可以作弊。事实上三角形有一个法线而其顶点没有法线，但是通过对其顶点添加自定义的法线并且对顶点法线与三角形固有的法线进行插值，我们可以模拟出一个更光滑的表面而不是一个由平面三角形逼近的曲面。这样图像本身更加真实，如果你不去在意图形的边缘的话。

由于每一个顶点都需要一条发现，所以我们需要另外一个向量数组。我们可以让网格根据其三角形自动计算出其法线。我们先偷个懒就这样实现它吧。  

```CS
	private void Generate () {
		…
		mesh.triangles = triangles;
		mesh.RecalculateNormals();
	}
```

> 法线是如何被重新计算的？
>
> `Mesh.RecalculateNormals` 方法通过计算与顶点连接的三角形法线的平均数来计算顶点的法线，最后将结果标准化。

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-no-normals.png" /></center>

<center>无法线的网格</center>

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-normals.png" /></center>

<center>有法线的网格</center>

接下来是 UV 坐标。你可能注意到了虽然网格使用了 Albedo Texture但是还是已有一个统一的颜色。这因为我们并未给网格提供 UV 坐标所以其现在都是零。  

为了让纹理能够正确映射到我们的网格上，先由网格的维度简单的划分顶点坐标。  

```CS
		vertices = new Vector3[(xSize + 1) * (ySize + 1)];
		Vector2[] uv = new Vector2[vertices.Length];
		for (int i = 0, y = 0; y <= ySize; y++) {
			for (int x = 0; x <= xSize; x++, i++) {
				vertices[i] = new Vector3(x, y);
				uv[i] = new Vector2(x / xSize, y / ySize);
			}
		}
		mesh.vertices = vertices;
		mesh.uv = uv;
```

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-uv-wrong-clamp.png" /></center>

<center>错误的 UV 座标，Clamping</center>

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-uv-wrong-wrap.png" /></center>

<center>有法线的网格，Wrapping Texture</center>

这张纹理现在出现了，但是它并没有覆盖整张网格。纹理的正确显示取决于纹理的循环模式 ( Wrap Mode ) 是被设置成固定的 ( Clamp ) 还是重复的 ( repeat )。发生这种情况是因为我们用整数除整数得到的还是整数。为了得到覆盖网格的 0 ~ 1 之间的正确的坐标，我们需要确保我们使用的是浮点数。  

```CS
				uv[i] = new Vector2((float)x / xSize, (float)y / ySize);
```

现在这张纹理覆盖了整张网格了。因为我们将网格设置成5*10的，纹理将会沿水平方向拉伸。可以通过调整材质纹理的瓦片设置来改变它。通过将其设置为(2, 1)，可以将U坐标将会加倍。如果将纹理设置为重复的话，我们将会在网格上看到两个瓦片。  

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-tiling.png" /></center>

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-uv-correct-1-1.png" /></center>

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-uv-correct-2-1.png" /></center>

<center>Correct UV coordinates, tiling 1,1 vs. 2,1.</center>

另外一种增加更多表面细节的方式是使用法线贴图 ( Normal map )。这些贴图包含了编码为颜色信息的法线向量。将其应用于物体表面将产生比只在顶点添加法线更多的富有细节的光线效果。  

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-normal-map.png" /></center>

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-material.png" /></center>

<center>Correct UV coordinates, tiling 1,1 vs. 2,1.</center>

将这个材质应用到我们的网格上并没有形成崎岖不平的效果。我们应该先为我们的网格添加切向量 ( Tangent Vectors )。

> 切向量是干啥的？
>
> 法线贴图是在切线空间 ( Tangent Space )下被定义的。这个 3D 空间流动在物体表面。这种方法允许我们在不同的位置和方向应用相同的法线贴图。  
> 法线指向了切线空间的上方，但是哪个方向是右方呢？这是由切线定义的。理论上，这两个向量的夹角是 90°。它们的向量积产生了一个在 3D 空间中的第三个方向向量。实际应用中，它们的夹角通常不是 90° 但是产生的结果已经足够好了。  
> 所以切向量是一个三维向量，但是实际上 Unity 中使用的是一个四维向量。第四个维度的数值处在 -1 ~ 1 之间，用来控制第三个切向空间维度的方向——向前或是向后。这将有利于反应那些双面对称物体的法线贴图，例如人。Unity的 Shader 程序执行这个计算的过程需要我们使用 -1。

因为我们有一个平整的表面，所以所有的切线都指向了同行一个方向，都是右方。

```CS
		vertices = new Vector3[(xSize + 1) * (ySize + 1)];
		Vector2[] uv = new Vector2[vertices.Length];
		Vector4[] tangents = new Vector4[vertices.Length];
		Vector4 tangent = new Vector4(1f, 0f, 0f, -1f);
		for (int i = 0, y = 0; y <= ySize; y++) {
			for (int x = 0; x <= xSize; x++, i++) {
				vertices[i] = new Vector3(x, y);
				uv[i] = new Vector2((float)x / xSize, (float)y / ySize);
				tangents[i] = tangent;
			}
		}
		mesh.vertices = vertices;
		mesh.uv = uv;
		mesh.tangents = tangents;
```

<center><img src="https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/mesh-basics/1_procedural_grid/04-normal-mapped.png" /></center>

<center>一个伪装成粗糙的表面</center>

现在你已经知道如何创建一个简单的 Mesh 网格，并且使用材质使它看起来有更多的细节。Mesh 网格需要顶点坐标与三角形片面，一般也需要 UV 坐标和切线。你也可以添加顶点颜色，尽管 Unity 的标准着色器程序并不使用它们。你可以创建你自己的着色器程序来使用这些颜色，但是那是另外一篇教程的事情了。  

如果对你的网格感到满意了的话，你就可以移步到[圆角立方体]()教程。