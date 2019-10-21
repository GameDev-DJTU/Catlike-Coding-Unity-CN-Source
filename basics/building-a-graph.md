# 绘制函数图象 | Building a Graph

## 数学可视化 | Visualizing Math

> 本教程译自Jasper Flick编写的[Catlike Coding Unity系列教程](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/)；由[GameDev@DJTU](https://github.com/gamedev-djtu)翻译并发布于[Catlike-Coding-Unity-CN](https://gamedev-djtu.github.io/Catlike-Coding-Unity-CN/)。</br>
转载时请保留该文本、原文链接和本文作者及链接等信息。

- 创建**预制体 _(prefab)_**
- 实例化一条立方体组成的线
- 展示一个数学函数
- 创建自定义着色器
- 让图象动起来

本次教程中，我们将使用**游戏对象 _(game object)_** 来绘制一个图象并展示数学公式。我们还会使函数与时间相关，最终得到一个动态图象。

在尝试本次教程前，请先完成教程[游戏对象和脚本](https://gamedev-djtu.github.io/Catlike-Coding-Unity-CN/basics/game-objects-and-scripts.html)，并确保使用2017.1.0以上版本的Unity。

![使用立方体展示正弦波](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/tutorial-image.jpg)

## 1. 绘制一条立方体组成的线 | Creating a Line of Cubes

对数学有良好的理解是编程的基本。根本上讲，算数是对代表数字的符号进行操作。解等式的过程可以归结于将一组符号重写称另一个——通常相对更短的——一组符号。数学法则决定了如何进行重写。

我们用$f(x) = x + 1$这个函数来举例。我们可以把$x$替换为一个数字，比如3。这样会得出$f(3)= 3 + 1 = 4$。我们输入3，函数最终输出4。我们可以说这个函数把3映射到4。形如$(3, 4)$的输入-输出数对是写出映射关系的一种简便方式。我们可以以$(x, f(x))$的方式构建很多数对，比如$(5, 6)$、$(8, 9)$，还有$(1, 2)$和$(6, 7)$。然而，根据数对的输入数进行排序有助于我们理解函数，就像$(1, 2)$、$(2, 3)$、$(3, 4)$这样。

函数$f(x) = x + 1$很好理解，而$f(x) = (x-1)^4 + 5x^3 - 8x^2 + 3x$这样的函数则困难一些。我们同样可以写下一些输入-输出数对，但这样不太会让我们清晰地理解它的映射关系。我们需要用到很多足够近的点，但这样也只是一大堆难以转化的数字。我们可以用$\begin{bmatrix} x \\ f(x)\end{bmatrix}$形式的二维坐标代替数对来表达这一关系。二维坐标可以表示一个二维向量，第一个数表示沿X轴的横坐标，第二个数表示沿Y轴的纵坐标，也就是$y = f(x)$。我们可以把这些点放在一个平面上。如果我们使用足够多的点，最后会形成一条线，这条线就是一个图象。

![一个$x$在-2到2之间的图象](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-a-line-of-cubes/graph.png)

看图象可以让我们快速了解一个函数的行为。这是一个有用的工具，我们可以在Unity中实现出来。首先我们需要通过 **_文件 / 新建场景 (File / New Scene)_** 创建一个新的场景，或者使用新项目默认的场景。

### 1.1 **预制体 _(prefab)_** | Prefabs

图象是通过把点放在正确的坐标上绘制出来的。我们需要一种对点进行三维可视化的方式。在这里，我们直接使用Unity的默认立方体物件。在场景中加入一个立方体，移除我们不需要的碰撞体 _(colider)_ **组件 _(component)_**。

> 立方体是可视化图象的最佳方式吗？
>
> 您也可以使用粒子系统或线段，只是使用独立的立方体是最简单的。

我们需要一个脚本来创建很多个这样的立方体并放置在正确的位置上。为了达到这个目标，我们把这个立方体当作模板。从**层级 _(Hierarchy)_** 窗口将立方体拖拽至**项目 _(Project)_** 窗口就可以创建一个新的资源，它的图标是一个蓝色的立方体，表示它是一个**预制体 _(prefab)_**。**预制体 _(prefab)_** 是一种存在于项目而非场景的预先制作好的物件。

![一个立方体**预制体 _(prefab)_**](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-a-line-of-cubes/prefab.png)

**预制体 _(prefab)_** 是配置游戏对象的有效方式。当一个**预制体 _(prefab)_** 资源被修改，所有它的实体都会同样被修改。例如，修改这个**预制体 _(prefab)_** 的**缩放 _(scale)_** 也会修改还在场景中的立方体。然而，每个实体都有自己的**位置 _(position)_**和**旋转 _(rotation)_**。另外，游戏对象的值可以被修改，这些修改会覆盖**预制体 _(prefab)_** 的值。如果实体的修改太大，比如添加或移除**组件 _(component)_**，**预制体 _(prefab)_** 和实体之间的关系会断开。<sup>[1]</sup>

---

_[1]：最后一句话仅对一些旧版本有效。在2018.3以上版本的Unity中，场景编辑和**预制体 _(prefab)_** 编辑已经分离。添加或移除**组件 _(component)_**不会断开**预制体 _(prefab)_** 和实体的关系，但在场景中修改**预制体 _(prefab)_** 的实体时有一定的限制。参考[在**预制体 _(prefab)_** 模式中编辑(暂仅英文)](https://docs.unity3d.com/Manual/EditingInPrefabMode.html)。_

---

我们会用一个脚本来创建**预制体 _(prefab)_** 实体，所以我们不需要现在场景中的立方体，于是删掉它。

### 1.2 图象**组件 _(component)_** | Graph Component

我们需要一个C#脚本生成图象，于是创建新脚本并命名为 _Graph_。一开始我们会得到一个简单的类，因为它继承自 _MonoBehaviour_，它可以用作游戏对象的一个**组件 _(component)_**。首先添加一个公有字段来保存**预制体 _(prefab)_** 的引用，用来创建图象的点，我们给它命名为`pointPrefab`。因为我们需要**预制体 _(prefab)_** 的 _Transform_ **组件 _(component)_** 来摆放这些点，我们使用 _Transform_ 作为字段的类型。

```csharp
using UnityEngine;

public class Graph : MonoBehaviour {
    public Transform pointPrefab;
}
```

在场景中通过 _游戏对象 / 创建空对象(GameObject / Create Empty)_ 添加一个游戏对象，并把它放在场景的原点，命名为 `Graph`。通过拖拽或 **_添加组件(Add Component)_** 按钮，把之前创建的`Graph`脚本添加到这个对象上，然后把我们的立方体**预制体 _(prefab)_** 拖到 `Graph`脚本的 `Point Prefab` 字段上，这样它就得到了**预制体 _(prefab)_** 的`Transform`**组件 _(component)_**。

![引用预制体的`Graph`对象](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-a-line-of-cubes/graph-with-prefab.png)

### 1.3 实例化**预制体 _(prefab)_** | Instantiating Prefabs

我们需要使用`Instantiate`方法来实例化一个游戏对象。`Instantiate`是Unity的`Object`类中的一个公有方法，我们的`Graph`通过继承`MonoBehaviour`间接获得了它。<sup>[2]</sup>`Instantiate`方法会将参数传入的Unity对象复制出来。当使用**预制体 _(prefab)_** 时，它会返回一个已被加入当前场景的实体。我们可以在`Graph`脚本唤醒时实例化一个对象。

---

_[2]：Unity的`MonoBehaviour`类继承自`Object`，因此创建`Graph`脚本（继承自`MonoBehaviour`）就已经间接获得了`Object`中的公有方法。在Visual Studio中对`MonoBehaviour`和`Object`查看定义，就可以看到继承获得的公有属性、字段和方法。也可以在在线文档中查看这些信息。_

---

```csharp
public class Graph : MonoBehaviour {
    public Transform pointPrefab;

    void Awake() {
        Instantiate(pointPrefab);
    }
}
```

![对预制体进行实例化](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-a-line-of-cubes/instantiated-prefab.png)

到现在为止，进入运行模式<sup>[3]</sup>就会在远点处创造一个简单的立方体，我们也可以看到相对于**预制体 _(prefab)_** ，它的位置被设置在原点上。为了把实例放置到其他位置，我们需要调整它的位置。`Instantiate`方法会将它创建出来的实例的引用返回给我们。因为我们调用时提供的是一个`Transform`**组件 _(component)_** 的引用，我们通过返回获得的也是一个`Transform`**组件 _(component)_**。我们可以用变量来保存它。

---

_[3]：截至目前的Unity版本，运行中对脚本和资源进行修改，可能都不会立刻在运行时生效。而在使用更高级的工具，如JetBrains® Rider，运行中修改并保存脚本，可能会造成其他的错误。在对脚本和资源操作前，我们应该保证Unity编辑器已经结束运行模式。_

---

```csharp
    void Awake() {
        Transform point = Instantiate(pointPrefab);
    }
```

现在我们可以通过赋值的方式，把一个三维向量作为这个点的位置。就像我们在[游戏对象和脚本](https://gamedev-djtu.github.io/Catlike-Coding-Unity-CN/basics/game-objects-and-scripts.html)那样，在这里我们通过`localPosition`属性，而不是`position`，来调整它的本地坐标，而非世界坐标。

三维向量通过`Vector3`结构体来创建。作为一个结构体，它相对于对象来说，更像是一个数字这样的值。例如，我们希望把X坐标设置为1，Y和Z坐标保持为0，`Vector3`中为这样的需求提供了一个`right`属性。

```csharp
        Transform point = Instantiate(pointPrefab);
        point.localPosition = Vector3.right;
```

> 属性不是应该首字母大写的吗？
>
> 没错，属性首字母大写确实是约定俗成的<sup>[4]</sup>，但Unity经常不这么做。

---

_[4]：在编程领域，我们称类似的约定俗成的规则为“编码规范”或“代码风格”或类似的名字。尽管编译和运行过程中，代码是否遵守编码规范**并不会**影响其执行结果和行为过程，但编码规范仍然是低成本而有效的减少编码错误的基本工具。因此，尽管只有部分专门的课程提及这一点，也尽管不同的团队可能遵循不同的编码规范，是否遵循一套编码规范仍然是考察一位程序同学编码水平的**直观评判标准之一**。</br>
需要额外注意的是，对于编码规范这一点，合格的程序同学应该**保持在团队中遵循完全相同的编码规范**，而非独立遵循不同的规范，即使每个人的规范都十分合理时，也应如是。因为编码规范中的一个共识是，整个项目都应该遵循统一的编码规范。实际操作时，这样对整个团队和团队的每个人来说，都是最好的。_

---

现在再次进入运行模式，我们仍然会看到一个立方体，但它在不同的位置。让我们再实例化一个立方体，并把它放到更右侧的位置。我们可以通过对`right`向量乘2来实现这个效果。这里我们重复实例化和修改位置的代码，将乘法运算加入新的代码中。

```csharp
    void Awake() {
        Transform point = Instantiate(pointPrefab);
        point.localPosition = Vector3.right;

        Transform point = Instantiate(pointPrefab);
        point.localPosition = Vector3.right * 2f;
    }
```

> 可以用数字乘结构体吗？
>
> 通常来说不能，但结构体可以通过定义乘法过程来做到这一点。使用特殊的语法来创造一个方法之后，这个方法便可以在尝试进行乘法时调用。在这里，进行乘法实际上进行了一次方法调用，大概就像`Vector3.Multiply(Vector3.right, 2f);`这样。
> 允许类似简单运算的方式使用方法会令代码编写更快，更易读懂。这不是必须的，但就像允许隐式使用命名空间一样，拥有这样的特性会更好。类似的比较方便的语法被称作语法糖。
> 尽管这样，编码者应该只提供符合运算符原意的简便方式。对于向量来说，有些数学操作符已经被完整定义，所以这些语句没有问题。<sup>[5]</sup>

---

_[5]：规定使用某个操作符时调用一个方法的技术，在语法上称作“运算符重载”，被认为是对整体语法进行扩展的一种方式。</br>
但在有些场合和讨论中，类似的做法被认为是弊大于利的，甚至被通过各种方式阻止，包括将之纳入编码规范。其原因之一便是担心文中提到的“符合原意”。</br>
比如说，语法上来说，我们可以规定一个表示超大数字的结构体，令其乘法运算符实际进行加法运算，而非乘法。对于类似的反面典型，这尽管达到了快速进行加法操作的目的，但全局地说，这样的定义混淆甚至改写了语法和常识，是被深恶痛绝并应该被坚决抵制的。</br>
类似运算符重载这样的语法糖的威力十分强大，所以对待这样的特性应该慎之又慎。在编写时如果有所犹豫，不使用这一特性也是一个正确的选择，因为对于调用者来说，调用一个类或结构体提供的方法，也不是十分麻烦的事情。反过来说，对代码的理解造成负面影响，是得不偿失并应尽力避免的。_

---

这段代码会导致编译错误，因为我们尝试多次定义`point`变量。如果我们想要用其他的变量，我们需要给它不同的名字。或者，我们可以重用已经存在的变量。无论如何，我们都不需要再持有第一个点的引用。这里我们简单地将新点的引用赋于同一个变量。

```csharp
        Transform point = Instantiate(pointPrefab);
        point.localPosition = Vector3.right;

        // Transform point = Instantiate(pointPrefab);
        point = Instantiate(pointPrefab);
        point.localPosition = Vector3.right * 2f;
```

![X坐标分别为1和2的两个实例](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-a-line-of-cubes/two-instances.png)

### 1.4 代码循环 | Code Loops

接下来让我们连续创建10个点。我们可以再重复8次刚才的代码，但这样是低效的编码行为。最好的方式是，我们只为一个点编写代码，并指示程序多次但略微不同地执行它。

`while`语句可以被用来重复执行一段代码。在这里我们将`Awake`方法的前两行放入`while`语句的代码块，并移除其他代码。

```csharp
    void Awake() {
        while {
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right;
        }
        // point = Instantiate(pointPrefab);
        // point.localPosition = Vector3.right * 2f;
    }
```

就像`if`语句一样，`while`语句要求用括号包裹一个表达式。与`if`相同，`while`语句的代码块只会在条件表达式结果为真时执行。代码块执行之后，程序会往回循环到`while`语句。这时，如果表达式的结果仍然为真，代码块就会被再次执行。整个过程直到表达式的值为假时结束。

因此我们需要在`while`后面加入表达式。我们必须小心确保循环不会永远循环下去。无限循环会造成程序卡住并需要用户手动终止。`false`是最安全且符合语法的表达式。

```csharp
        while (false) {
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right;
        }
```

> 可以在循环内定义`point`吗？
> 可以。尽管代码被重复执行，而我们只定义了一次变量也是可以的。就像我们之前写的那样，变量会在每次循环迭代时复用。
> 在循环之前定义`point`也是可以的，这样允许我们在循环之外也使用这个变量。不这样做的话，它的作用域被限制在`while`循环的代码块内。

可以通过追踪代码块执行的次数来限制循环。我们使用一个整型变量就可以实现这一点。因为它用来记录循环的迭代数，在这里我们给它命名为`i`。它必须在循环之前就被定义，这样才可以用在`while`表达式中。

```csharp
    void Awake () {
        int i;
        while (false) {
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right;
        }
    }
```

接下来，每次迭代，我们都让`i`加1。

```csharp
    void Awake () {
        int i;
        while (false) {
            i = i + 1;
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right;
        }
    }
```

这样会导致编译错误，因为我们在给`i`赋初始值之前就使用了它。为了让整个循环按我们的需要执行，`i`的初始值需要赋为0。

```csharp
        int i = 0;
```

现在，第一次迭代开始时，`i`的值就会变为1，第二次时则会变为2，以此类推。但`while`表达式在每次迭代之前就会被判断。所以在第一次迭代之前，`i`的值为0，第二次之前则为1，以此类推，知道第10次迭代之后，`i`的值为10，这个时候我们希望循环结束，因此此时的条件表达式的值需要为假。数学上讲，这个条件为i < 10。在代码里也是同样的写法，运算符也是一样的。

```csharp
        int i = 0;
        while (i < 10) {
            i = i + 1;
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right;
        }
```

现在，我们可以在运行时得到10个立方体，但它们都在同一个位置。通过对`right`向量乘`i`，我们可以让它们沿X轴排成一行。

```csharp
            point.localPosition = Vector3.right * i;
```

![排成一行的十个立方体](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-a-line-of-cubes/ten-cubes.png)

现在我们的第一个立方体的X坐标为1，最后一个的X坐标为10。但我们期望它们从X坐标的0，也就是原点开始。我们可以通过对`right`乘`(i - 1)`而不是`i`来做到这一点，但我们可以通过把`i`的增加运算从代码块的起始移动到结束来省去额外的减法运算。

```csharp
        while (i < 10) {
            // i = i + 1;
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right;
            i = i + 1;
        }
```

### 1.5 简便语法 | Concise Syntax

令代码循环多次是非常普遍的，尽可能地使用简便的语法会让我们更方便。有一些语法糖会帮到我们。

首先，我们再来想一下迭代次数的增长过程。形如`x = x * y`的表达式可以缩写为`x *= y`，类似的缩写对所有使用两个同类型操作数的运算都是有效的。

```csharp
            // i = i + 1;
            i += 1;
```

我们可以再缩写一下。当一个数加1可以缩写为`++x`，减1时可以缩写为`--x`。

```csharp
            // i += 1;
            ++i;
```

赋值表达式的一个属性是，它可以被同时当作一个表达式来使用。这意味着`y = (x += 3)`是有效的。<sup>[6]</sup>这样写将会让`x`加3，同时将结果赋值给```y```。这就是说，我们可以在while表达式内让i增长来缩短代码块。

---

_[6]：在同一行语句中同时进行运算和修改变量的值的行为需要小心，尤其是一个变量既作为其他运算的输入，又同时进行了自身的修改的情况。例如，有些C语言或者C++语言的试题上会有类似```i=i+++1```这样的问题。对于大多数的编程语言，未被禁止的操作都是可以被编译和运行的，尽管这一行为**并不在**语法规范的明确规则之内。在规则不明确的情况下，不同的编译器可能会导致不同的过程和结果，也可能不会。这样就会导致我们的程序可能，也可能不能稳定得到我们期望的结果。我们称这样的情况为“未定义行为”。</br>
当然，这并不意味着我们应该完全禁止这样的写法。在更为现代的编程语言或更为先进的语法规范下（例如C++17添加了对这种行为的正式规定），我们总会得到稳定的结果。但聪明的开发者都应该从根本上避免可能导致误解的写法，毕竟将类似的操作拆分成不同的语句并不麻烦，但会降低合作者的理解成本和犯错几率。这也是我们说“小心某些语法”的另一个案例。_

---

```csharp
        while (++i < 10) {
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right;
            // ++i;
        }
```

然而，现在我们在比较之前就增加了`i`的值，而不是比较之后，这样会导致少一次迭代。专对于这样的情况，加法和减法运算符可以放在变量之后而不是之前，从而让条件表达式使用它的原值，在增长之前就进行判断。

```csharp
        while (i++ < 10) {
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right;
            // ++i;
        }
```

虽然`while`语句可以用在任何种类的循环中，我们可以使用一个专为在一定范围内进行迭代的另一个循环语法，这就是```for```循环。```for```循环类似`while`，只是迭代变量的声明和比较都在括号之内，由分号隔开。

```csharp
//      int i = 0;
//      while (i++ < 10) {
        for (int i = 0; i++ < 10) {
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right * i;
        }
```

这样会导致编译错误，因为for语句的括号内实际上有三个部分。第三部分用于增长迭代器<sup>[7]</sup>，将这个过程与比较分离。

---

_[7]：最粗略地讲，迭代器是用来遍历一个容器的部分或全部元素的对象。所有可以达到这个目的的东西（如变量）都可以算作迭代器，不一定是某些定义中“修改地址”的东西（如迭代函数）。_

---

```csharp
        // for (int i = 0; i++ < 10) {
        for (int i = 0; i < 10; i++) {
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right * i;
        }
```

> 为什么在`for`循环中使用`i++`而不是`++i`？
> 当自增表达式不用做增加以外的事情时，使用哪个写法都是一样的。在上面的`for`表达式中，我们甚至也可以使用`i += 1`或`i = i + 1`。
> `for`循环的经典形式为`for (int i = 0; i < someLimit; i++) `。这样的代码段在很多程序和脚本中比比皆是。

### 1.6 修改定义域 | Changing the Domain

现在，我们创建的立方体的X坐标时从0到9的。这对于处理函数来说，并不是一个很好的数值范围。通常来说，我们会让X坐标在[0, 1]内。或者，对于值围绕0的函数，我们会令范围为[-1, 1]。现在我们就来这样调整这些立方体。

沿线段摆放我们的10个立方体会造成重叠。我们要缩小它们的**缩放 _(scale)_**。默认地，每个立方体的每个维度的长度均为1单位，所以这里我们将缩放减少为$\frac{2}{10}$也就是$\frac{1}{5}$。我们可以通过将每个点的本地**缩放 _(scale)_**赋值为`Vector3.one`属性除以5的结果。

```csharp
        for (int i = 0; i < 10; i++) {
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right * i;
            point.localScale = Vector3.one / 5f;
        }
```

![小立方体](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-a-line-of-cubes/small-cubes.png)

为了让立方体靠近一些，它们的位置也要除以5。

```csharp
            point.localPosition = Vector3.right * i / 5f;
```

这样会让结果落在[0, 2]的范围内。我们需要在对向量缩放之前，先减去1。

```csharp
            point.localPosition = Vector3.right * (i / 5f - 1f);
```

现在，我们的第一个立方体X坐标为-1，最后一个为0.8，但立方体的大小为0.2。由于立方体的位置是基于中心的，第一个立方体的最左边处的X坐标为-1.1，最后一个则为0.9。为了让所有的立方体都整齐地在[-1, 1]范围内，我们必须让它们向右侧移动半个立方体。在i被除之前加0.5就可以了。

```csharp
            point.localPosition = Vector3.right * ((i + 0.5f) / 5f - 1f);
```

### 1.7 在循环之外计算向量 | Hoisting the Vectors out of the Loop

尽管所有的立方体都使用相同的缩放，我们仍然在每次迭代中都计算了一遍，这是没有必要的。我们可以在循环前就完成计算，保存在一个`Vector3`类型的变量中，并在循环中使用它。

```csharp
    void Awake () {
        Vector3 scale = Vector3.one / 5f;
        for (int i = 0; i < 10; i++) {
            Transform point = Instantiate(pointPrefab);
            point.localPosition = Vector3.right * ((i + 0.5f) / 5f - 1f);
            point.locdalScale = scale;
        }
    }
```

我们也可以在循环之前为点的位置定义一个变量。因为我们想要做出一条沿着X轴的线，我们只需要在循环内调整X坐标就可以了。因此，我们不再需要对`Vector3.right`做乘法。

```csharp
        Vector3 scale = Vector3.one / 5f;
        Vector3 position;
        for (int i = 0; i < 10; i++) {
            Transform point = Instantiate(pointPrefab);
            // point.localPosition = Vector3.right * ((i + 0.5f) / 5f - 1f);
            position.x = (i + 0.5f) / 5f - 1f;
            point.localPosition = position;
            point.locdalScale = scale;
        }
```

> 能否单独修改向量的某个成员？
>
> `Vector3`结构体有三个浮点值，分别为x、y和z。因为它们是公有字段，我们可以修改他们。
> 由于结构体的行为接近于基本值，从思路上说结构体应该是不变的。一旦构造完成，它们就不应该被改变。如果需要一个不同的值，我们就应该对该字段或变量赋予一个新的结构体，就像数字一样。假设有`x = 3`，然后执行`x = 5`，过程中，我们就已经为x赋予了一个新的数字，而不是将数字3本身改变为5。然而，Unity的向量类型是可变的。这一设计出于便捷和性能的双重考量，因为向量的成员变量经常被独立修改。
> 为了理解可变向量是如何工作的，我们可以把`Vector3`的使用当作是使用三个独立浮点值的简便替代，既可以独立访问某个浮点值，也可以作为一组数据来复制并赋值。

这段代码将导致编译错误，因为我们使用了未初始化的变量。这是因为我们在没有设置Y轴和Z轴坐标的情况下，将`position`赋予其他变量。这里我们在循环之前，将它的Y轴和Z轴坐标设置为0。

```csharp
        Vector3 position;
        position.y = 0f;
        position.z = 0f;
        for (int i = 0; i < 10; i++) {
            …
        }
```

### 1.8 用X值定义Y值 | Using X to Define Y

我们的思路是，立方体的位置被定义为$\begin{bmatrix} x \\ f(x) \\ 0 \end{bmatrix}$，这样它们就可以用来展示一个函数。到目前为止，所有立方体的Y坐标恒为0，表示了平凡函数$f(x) = 0$。我们需要在循环内部而非外部确定每个点的Y坐标，这样才能展示一个不同的函数。在这里，我们让Y坐标与X坐标相同，以此表示函数$f(x) = x$。

```csharp
        Vector3 position;
        // position.y = 0f;
        position.z = 0f;
        for (int i = 0; i < 10; i++) {
            Transform point = Instantiate(pointPrefab);
            position.x = (i + 0.5f) / 5f - 1f;
            position.y = position.x;
            point.localPosition = position;
            point.localScale = scale;
        }
```

![$y=x$](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-a-line-of-cubes/y-equals-x.png)

我们再来看一个相对没那么明显的函数，比如$f(x)=x^2$。它定义了一个最小值为0的抛物线。

```csharp
            position.y = position.x * position.x;
```

![$y=x^2$](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-a-line-of-cubes/y-equals-x-squared.png)

## 2 创造更多的立方体 | Creating More Cubes

尽管目前为止，我们已经有了一个函数图像，但看起来有点糙。因为我们只用了10个立方体，整条曲线看起来非常不均匀，也非常不连续。如果我们使用更多更小的立方体，看起来会好一些。

### 2.1 可变解析度 | Varible Resolution

我们可以让立方体的数量从固定编程可配置的。在`Graph`脚本中为解析度添加一个公有整型字段就可以了。将它的默认值设置为我们目前使用的10。

```csharp
    public int resolution = 10;
```

![解析度数值](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-more-cubes/resolution-number.png)

我们想要通过在编辑器中修改它的值，就可以调整图象的解析度。然而，不是所有的整数都是有效值，至少它必须是正数。我们可以让 **_检查器(inspector)_** 强制解析度在一个范围内。在字段的定义之前用方括号包裹`Range`就可以了。

```csharp
    [Range] public int resolution = 10;
```

`Range`是Unity提供的一个特性类。特性是一种将元数据附加在代码结构当中的方式。Unity的 **_检查器(inspector)_** 会检查一个字段是否有`Range`特性。如果有， **_检查器(inspector)_** 会用一个滑动条而非默认的输入框来表示它。然而，我们需要知道允许的范围，这才能起效。因此`Range`有两个参数分别用于最小值和最大值。这里我们分别使用10和100。另外，通常来说，特性都被写在代码结构的上面而非前面。

```csharp
    [Range(10, 100)]
    public int resolution = 10;
```

![解析度滑动条](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-more-cubes/resolution-slider.png)

> 这样能否保证`resolution`的值被限制在10-100以内？
>
> 不能。`Range`只能用来指示 **_检查器(inspector)_** 显示规定范围的滑动条，没有其他的作用。因此，我们可以通过代码来赋予任何整型。在本次教程中，我们假设只在 **_检查器(inspector)_** 中调整解析度。

### 2.2 可变实例化 | Variable Instantiation

我们需要修改实例化立方体的个数来真正地使解析度生效。`Awake`方法内的循环次数需要由固定次数改为由解析度决定。假设解析度设置为50，在进入运行模式之后，我们会床架50个立方体。

```csharp
            for (int i = 0; i < resolution; i++) {
                ...
            }
```

我们还需要调整立方体的缩放和位置来让它们保持在[-1, 1]的定义域内。每次迭代之间的步长要从固定的1/5调整为2/*resolution*。这里我们将步长保存为变量并用它来计算立方体的缩放和X坐标。

```csharp
    void Awake () {
        float step = 2f / resolution;
        Vector3 scale = Vector3.one * step;
        Vector3 position;
        position.z = 0f;
        for (int i = 0; i < resolution; i++) {
            Transform point = Instantiate(pointPrefab);
            position.x = (i + 0.5f) * step - 1f;
            position.y = position.x * position.x;
            point.localPosition = position;
            point.localScale = scale;
        }
    }
```

![解析度为50时](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-more-cubes/resolution-50.png)

### 2.3 设置父对象 | Setting the Parent

将解析度设置为50并运行，我们可以在场景和Hierarchy的场景视图中看到很多个立方体实例。

![根对象太多了](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-more-cubes/many-root-objects.png)

这些立方体现在都是根对象，但把它们放在graph对象下才合理。我们可以通过调用立方体`Transform`**组件 _(component)_** 的`SetParent`方法，来为实例化完成的立方体设置对象关系。我们需要提供它的新父对象的`Transform`**组件 _(component)_**。我们可以通过`Graph`继承得到的`transform`属性直接获得graph对象的`Transform`**组件 _(component)_**。

```csharp
        for (int i = 0; i < resolution; i++) {
            Transform point = Instantiate(pointPrefab);
            position.x = (i + 0.5f) * step - 1f;
            position.y = position.x * position.x;
            point.localPosition = position;
            point.localScale = scale;
            point.SetParent(transform);
        }
```

![Graph的子对象](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/creating-more-cubes/child-objects.png)

当设置新的父对象时，Unity会尝试保持对象本身的原世界坐标、旋转和缩放。在这个例子中，我们不需要这么做。我们为`SetParent`传入第二个参数为`false`就可以了。

```csharp
            point.SetParent(transform, false);
```

## 3 给图象上色 | Coloring the Graph

一个纯白色的图象不是特别漂亮。我们的确可以用其他的纯色，但这样也没什么意思。我们可以通过一个点的位置来决定它的颜色。

调整每个立方体颜色的一种直接手段是设置立方体材质的颜色属性。我们可以在循环中这么做。但因为每个立方体都会有不同的颜色，这意味着我们需要为每个对象创建独立的材质。尽管这样做可行，但效率不高。如果我们能用一个材质来直接使用它的位置作为颜色，这样会比较简单。不幸的是，Unity本身并没有这样的材质。我们需要自己做一个出来。

### 3.1 创建自定义着色器 | Creating a Custom Shader

GPU通过运行着色器程序来渲染三维对象。Unity的材质资源决定使用哪种着色器，也允许配置着色器的属性。我们需要创建一个自定义着色器来实现我们需要的功能。点击 **_资源(Assets)/创建(Create)/着色器(Shader)/标准表面着色器(Standard Surface Shader)_** 并将新的着色器命名为 _ColoredPoint_。

![自定义着色器资源](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/coloring-the-graph/custom-shader-asset.png)

现在我们获得了一个着色器资源，我们可以像打开脚本一样打开它。我们的着色器文件包含一个用与C#不同的语法定义的表面着色器。下面是文件中的内容，为了简洁，这里删除了所有的注释。

```glsl
Shader "Custom/ColoredPoint"
{
    Properties
    {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 200

        CGPROGRAM
        #pragma surface surf Standard fullforwardshadows

        #pragma target 3.0

        sampler2D _MainTex;

        struct Input
        {
            float2 uv_MainTex;
        };

        half _Glossiness;
        half _Metallic;
        fixed4 _Color;

        UNITY_INSTANCING_BUFFER_START(Props)
        UNITY_INSTANCING_BUFFER_END(Props)

        void surf (Input IN, inout SurfaceOutputStandard o)
        {
            fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
            o.Albedo = c.rgb;
            o.Metallic = _Metallic;
            o.Smoothness = _Glossiness;
            o.Alpha = c.a;
        }
        ENDCG
    }
    FallBack "Diffuse"
}

```

> 表面着色器是怎么工作的？
>
> Unity提供了一个框架来快速生成负责默认光照计算的着色器，我们只要调整特定的值就可以影响计算过程。这样的着色器被称作表面着色器。浏览[渲染]()<sup>[施工中]</sup>来详细了解着色器。

我们的新着色器有纯色、贴图、表面有多光滑、多接近金属之类的属性，正如其他的表面着色器一样。由于我们是由点的位置决定颜色，我们不需要纯色和贴图。下面的代码移除了所有没必要的东西，将albedo设置为纯黑，alpha设置为1。

```glsl
Shader "Custom/ColoredPoint" {
    Properties {
//        _Color ("Color", Color) = (1,1,1,1)
//        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _Glossiness ("Smoothness", Range(0,1)) = 0.5
        _Metallic ("Metallic", Range(0,1)) = 0.0
    }
    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 200

        CGPROGRAM
        #pragma surface surf Standard fullforwardshadows

        #pragma target 3.0

//        sampler2D _MainTex;

        struct Input {
//            float2 uv_MainTex;
        };

        half _Glossiness;
        half _Metallic;
//        fixed4 _Color;

        UNITY_INSTANCING_CBUFFER_START(Props)
        UNITY_INSTANCING_CBUFFER_END

        void surf (Input IN, inout SurfaceOutputStandard o) {
//            fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
//            o.Albedo = c.rgb;
            o.Metallic = _Metallic;
            o.Smoothness = _Glossiness;
//            o.Alpha = c.a;
            o.Alpha = 1;
        }
        ENDCG
    }
    FallBack "Diffuse"
}
```

> 什么是albedo和alpha？
>
> 一个材质的漫反射映出的颜色称作albedo。Albedo的本意是拉丁语中的白色。它用来描述漫反射出来的红色、绿色和蓝色通道的值。其余的值都被吸收了。
> Alpha被用作透明度的一种衡量方式。一个表面在alpha为0时完全透明，为1时完全不透明。

目前这个着色器不会被编译，因为表面着色器在输入结构为空时是无法工作的。输入结构是我们用来选定为像素上色所需的自定义数据。对我们来说，我们需要这个点的位置。我们可以为输入结构添加`float3 worldPos;`来访问世界位置。

```glsl
        struct Input {
            float3 worldPos;
        }
```

> 这样做的话，移动图象会影响到颜色吗？
>
> 是的。这么做的话，图象只有放在原点时，颜色才会正常。</br>
另外，位置是由每个顶点决定的。在本例中，顶点由立方体的每个点决定。颜色会插值在每个立方体的面上。立方体越大，色彩的过渡就会越明显。

既然我们有了有效的着色器，我们要为它创建一个材质，名字叫做Colored Point。在Shader下拉列表中选择Custom/Colored Point将着色器设置在材质上。

![Colored Point的材质](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/coloring-the-graph/colored-point-material.png)

接下来让我们的立方体**预制体 _(prefab)_** 使用这个材质而非默认材质。将材质拖拽到**预制体 _(prefab)_** 资源上就可以了。

### 3.2 基于世界位置的上色 | Coloring Based on World Position

现在进入运行模式后，我们的图象会实例化为黑色立方体。我们需要修改`o.Albedo`来为他们指定颜色。首先让`albedo`的红色值等于位置的X坐标。

```glsl
            o.Albedo.r = IN.worldPos.x;
            o.Metallic = _Metallic;
```

![根据X值上色的图象](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/coloring-the-graph/colored-graph-x.png)

> 我们不应该完全初始化`o.Albedo`吗？
>
> 我们不需要设定它的绿色或蓝色值，因为它们在`surf`参与之前就已经设置为0了。

X坐标为正数的立方体现在会逐渐变红了。X坐标为负数的立方体还保持为黑色，因为颜色值不能为负数。为了得到-1到1的红色渐变，我们需要让X坐标减半并加上0.5。

```glsl
            o.Albedo.r = IN.worldPos.x * 0.5 + 0.5;
            o.Metallic = _Metallic;
```

![完整的渐变](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/coloring-the-graph/colored-graph-x-normalized.png)

接下来我们用同样的方法，让黄色至等于坐标的Y坐标。在着色器中，我们可以通过将`o.Albedo.rg`赋值为`IN.worldPos.xy`，只用一行代码就能实现了。

```glsl
            o.Albedo.rg = IN.worldPos.xy * 0.5 + 0.5;
            o.Metallic = _Metallic;
```

![使用X和Y的值上色](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/coloring-the-graph/colored-graph-xy.png)

红色加上绿色会变为黄色，所以我们的图象从绿色变为黄色。Y坐标也是从-1开始的，因此我们也能得到深绿色。修改`Graph.Awake`让图象显示为函数$f(x) = x^3$就能看到效果了。

```csharp
            position.y = position.x * position.x * position.x;
```

![从-1到1的Y值](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/coloring-the-graph/x-cubed.png)

## 4 让图象动起来 | Animating the Graph

显示一个静止的图象的确有用，但动起来的图象看起来更有趣。所以接下来，让我们为函数动画化添加支持。我们可以将时间作为额外的函数变量，使用形如$f(x, t)$，其中t表示时间，而不是单纯的$f(x)$来实现。

### 4.1 保持点的引用 | Keeping Track of the Points

为了让图象动起来，我们需要随时间调整图象中的点。虽然我们可以在每次`Update`中删除所有的点并重新创建，但那样的运行效率太低了。重复使用相同的点并在每次`Update`中调整它们的位置会相对好很多。为了便于调整位置，我们需要一个字段来保持点的引用。首先为`Graph`添加一个`Transform`类型的字段，取名为points。

```csharp
    Transform points;
```

这个字段允许我们引用单独一个点，但我们需要访问所有的点。为此，我们需要使用一个点的数组。我们可以在其类型的后面加上一对方括号，将我们的字段编程一个数组。

```csharp
    Transform[] points;
```

`points`字段现在可以代表一个数组，其中的元素都是`Transform`类型的。数组是对象而不是简单的值。我们需要显式创建一个对象并让我们的字段引用它。在数组类型的前面加上`new`，也就是`new Transform[]`，就可以了。我们需要在Awake方法中，我们的循环之前，创建并赋予`points`字段。

```csharp
    points = new Transform[];
    for (int i = 0; i < resolution; i++) {
        ...
    }
```

当我们要创建数组时，我们需要指定数组的大小。数组的大小用来定义数组包含多少个元素，这一点从它被创建出来之后便无法修改。数组的长度要写在构造数组时的方括号内。对我们来说，它的长度与解析度相等。

```csharp
    points = new Transform[resolution];
```

现在我们可以将创建出来的点的引用填入数组了。在数组字段或变量后面的方括号中填写下表即可访问数组中的指定元素。数组的下表从0开始，0表示第一个元素，就像我们的循环中的迭代计数器一样。我们可以用迭代计数器访问对应的元素。

```csharp
    points = new Transform[resolution];
    for (int i = 0; i < resolution; i++) {
        ...
        points[i] = point;
    }
```

这样我们就可以在点的数组中循环了。因为数组的长度与解析度相等，我们也可以用数组的长度来限制我们的循环。每个数组都有一个`Length`属性来获取它的长度，在这里我们就利用上这个属性。

```csharp
    points = new Transform[resolution;
    for (int i = 0; i < points.Length; i++) {
        ...
        points[i] = point;
    }
```

### 4.2 更新所有点 | Updating the Points

为了让图像动起来，我们需要在`Graph`**组件 _(component)_** 的`Update`方法中设置图象中点的Y坐标。这样我们就无需在`Awake`方法中计算它们了。尽管如此，我们还是需要显式地将它们的Y坐标值设置为0。

```csharp
        position.y = 0f;
        position.z = 0f;
        points = new Transform[resolution];
        for (int i = 0; i < points.Length; i++) {
            Transform point = Instantiate(pointPrefab);
            position.x = (i + 0.5f) * step - 1f;
            // position.y = position.x * position.x *position.x;
            point.localPosition = position;
            point.localScale = scale;
            point.SetParent(transform, false);
            points[i] = point;
        }
```

接下来，添加一个`Update`方法，其中写一个和`Awake`中相同，但代码块里没有任何代码的`for`循环。

```csharp
    void Update() {
        for (int i = 0; i < points.Length; i++) {}
    }
```

每次迭代，我们都从获取当前数组元素的引用做起，随后获取对应点的位置。

```csharp
    for (int i = 0; i < points.Length; i++) {
        Transform point = points[i];
        Vector3 position = point.localPosition;
    }
```

接下来我们可以像之前那样得到位置的Y坐标值。

```csharp
    for (int i = 0; i < points.Length; i++) {
        Transform point = points[i];
        Vector3 position = point.localPosition;
        position.y = position.x * position.x * position.x;
    }
```

因为向量并不是对象，方才我们只是修改了一个本地变量。我们需要重新设置点的位置使它生效。

```csharp
    for (int i = 0; i < points.Length; i++) {
        Transform point = points[i];
        Vector3 position = point.localPosition;
        position.y = position.x * position.x * position.x;
        point.localPosition = position;
    }
```

> 我们不能直接向`point.localPosition.y`赋值吗？
>
> 如果`localPosition`是一个字段的话，这样做是可以的。我们可以直接设置点的位置的Y坐标。然而，`localPosition`是一个属性。它向我们传递一个向量，或从我们这里接受一个向量。因此，当我们直接赋值时，我们最终只是修改了一个本地向量的值，这根本不能影响点的位置。由于我们没有首先显式地将本地向量保存为一个变量，赋值行为将毫无意义，并产生一个编译错误。

### 4.3 展示正弦波形 | Showing a Sine Wave

从现在起，在运行模式下，我们图象中的点每帧都会更新位置，只是我们没有意识到，因为每个点都最终设置在相同的位置上。我们需要将时间连入函数来使图象产生变化。然而，单纯地加入时间会造成函数的整体增长，并导致图像很快从视野中消失。为避免这样的效果，我们需要使用一个值在指定范围内变化的函数。正弦函数正是如此。于是我们接下来会使用函数$f(x)=sin(x)$。我们可以使用Unity的`Mathf`类型中的`Sin`函数来计算它的值。

```csharp
        position.y = Mathf.Sin(position.x);
```

![正弦的X值](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/animating-the-graph/sin-x.png)

> Mathf是什么？
>
> Mathf是一个包含一些数学函数和常量，用来计算数字和向量的结构体。由于它通常使用浮点数，它被赋予`f`这么一个词缀。

```csharp
        position.y = Mathf.Sin(Mathf.PI * position.x);
```

![$\pi$X的正弦](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/animating-the-graph/sin-pix.png)

> 正弦波和$\pi$是什么？
正弦是一个对角度运算的三角函数。对本例而言，最有用的例子就是一个半径为1的圆，也就是单位圆。圆上的每个点的二维坐标都与一个角度$\theta$(theta)相关。圆上各点坐标的定义之一即为$\begin{bmatrix} sin(\theta) \\ sin(\theta + \frac{\pi}{2}) \end{bmatrix}$。它表现为自圆的顶部起沿顺时针方向的角度。除$sin(\theta + \frac{\pi}{2})$之外，我们也可以用余弦函数，即$\begin{bmatrix} sin(\theta) \\ cos(\theta) \end{bmatrix}$。</br>
角度$\theta$表示为弧度，也就是一个角度在单位圆上两个交点的弧线距离。当走过180度，也就是半圆时，其弧线距离即为$\pi$，其粗略值为3.14。因此整个圆的周长长度为$2\pi$。换句话说，$\pi$是周长与直径的比率。

为了让这个函数动起来，我们要在计算正弦函数之前，将当前的运行时间加入X。如果我们将时间扩大$\pi$倍，整个函数会每个秒重复一次。所以我们使用函数$f(x,t)=sin(\pi(x+t))$，其中$t$为经过的游戏时间。这样，正弦波就会随着时间前进，沿着X轴的负方向变化。

```csharp
        position.y = Mathf.Sin(Mathf.PI * (position.x + Time.time));
```

<video id="video" controls="" preload="none" poster="https://thumbs.gfycat.com/FlamboyantCheapHoki-mobile.jpg" tabindex="-1">
    <source src="https://thumbs.gfycat.com/FlamboyantCheapHoki-mobile.mp4" type="video/mp4"/>
    <source src="https://giant.gfycat.com/FlamboyantCheapHoki.webm" type="video/webm"/>
    <source src="https://giant.gfycat.com/FlamboyantCheapHoki.mp4" type="video/mp4"/>
    <source src="https://thumbs.gfycat.com/FlamboyantCheapHoki-mobile.mp4" type="video/mp4"/>
</video>

接下来请移步[数学曲面](https://gamedev-djtu.github.io/Catlike-Coding-Unity-CN/basics/mathematical-surfaces.html)
