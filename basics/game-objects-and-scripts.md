#**游戏对象和脚本** 创建一个时钟   
*用简单的对象构建一个时钟*  
*写一个C#脚本*  
*旋转时钟的指针来显示时间*  
*动画化指针*  
在这个教程中，我们将要创建一个简单的时钟并且通过编写一个组件来使它显示当前的时间。你只需要简单的了解Unity编辑器，知道如何使用*Scene*窗口的导航控制就可以了。  
本教程默认你使用的至少是Unity 2017.1.0。  

![图片1][图片1]  
<center>*做一个时钟吧*</center>

1**创建一个时钟**  
打开Unity然后新建一个3D工程，你不需要添加额外的包和analytics。如果你没有自定义调整过你的Unity编辑器，那么你会打开如下窗口布局。  

![图片2][图片2]  
<center>*默认窗口布局*</center>  
我使用了不同的布局，这种2乘3的布局你可以在编辑器右上角的下拉列表里选择。我更深一步的调整了我的布局，我把*Project*窗口转换成了单列布局，这样更适合纵向使用。你可以通过窗口右上角工具栏上方的锁定图标旁边的下拉列表进行更改。我还在*Gizmos*的下拉菜单中禁用了*Scene*窗口中的*Show Grid*（显示网格）功能。  
  
![图片3][图片3]  
<center>*自定义2乘3布局*</center> 


**为什么我的***Game***窗口缩小后有黑边框？**  
使用高分辨率显示的情况下可能会发生这种情况。要使其扩展并填满整个*Game*窗口，请打开游戏窗口的*aspect-ratio*（纵横比）下拉菜单，并禁用*Low Resolution Aspect Ratios*选项。  
  
![图片4][图片4]  
<center>*禁用Low Resolution Aspect Ratios*</center> 
  
1.1**创建一个游戏对象**  
在默认场景中有两个游戏对象。他们被列在*Hierarchy*窗口中，你也可以在*Scene*窗口中看到他们的图标。第一个是*Main Camera*（主相机），用来渲染场景。*Game*窗口就是通过这个相机来渲染的。第二个是*Directional Light*（方向灯），用来照亮场景。
  
要创建你自己的游戏对象，使用*GameObject/Create Empty*选项。你也可以通过*Hierarchy*窗口中的内容菜单来创建。在向场景中添加新对象的时候可以给它命名。既然我们要创建一个时钟，就命名为*Clock*。
  
![图片5][图片5]  
<center>*Hierarchy中的Clock对象*</center> 
  
*Inspector*（监视器）窗口用来显示游戏对象的细节。当我们选中我们的时钟对象时，它会显示对象的名字和一些配置选项。对象默认是禁用的，不是静态的，没有标记的，并且属于默认层。我们不需要更改这些设置。在这些下面显示了关于游戏对象拥有的所有组件，其中必有一个*Transform*组件，也是我们目前Clock对象拥有的唯一一个组件。
  
![图片6][图片6]  
<center>*选中clock对象后的Inspector窗口*</center> 
  
*Transform*组件中包含对象在3D空间中的position（位置），rotation（旋转）和scale（放缩）。请确保clock的position和rotation都是0，scale为1。
  
**2D对象怎么办？**  
当你使用的是2D而不是3D对象时，你可以忽略三个坐标中的一个。特殊的2D对象比如UI界面之类的对象通常有一个特殊的*Transform* 组件叫*Rect Transform*。  
  
1.2**制作钟面**  
  
虽然我们已经创建了clock对象，但是我们现在还看不见它。我们需要向这个对象里添加3D模型来渲染才能显示出来。Unity有一些基础的对象，我们可以用它们来做一个简单的时钟。我们先通过*GameObject/3D object/Cylinder*来向场景中添加一个圆柱体，确认它的*Transform*的值和我们的钟的值一样。  
  
![图片7][图片7]![图片8][图片8]
<center>*GameObject圆柱体*</center>  
这个新的对象比空的游戏对象多三个组件。第一个是*Mesh Filter*，它是圆柱对象的内置圆柱网格的引用。第二个是*Capsule Collider*，用来掌管对象的3D物理属性。第三个是*Mesh Renderer*。这个组件是用来保证对象的网格模型被渲染的。它也控制用来渲染的材质，默认的材质是内置的*Default-Material*。材质也被列在了*inspector*中，在组件列表的下面。  
  
虽然我们的对象是圆柱体，但是它的碰撞是方形碰撞，因为Unity默认是没有圆柱形碰撞的。不过我们不需要用到碰撞，所以我们可以把这个组件给移除。如果你想要你的时钟拥有物理效果，你最好使用*Mesh Collider*（网格碰撞）组件。组件的右边有一个齿轮图标的下拉列表，你可以通过它来移除组件。
  
![图片9][图片9]

<center>*移除Capsule Collider*</center>  
  
我们要把圆柱体压扁来做成我们的钟面。我们通过更改它的*sacle*里的**Y** 的值来压扁它，把y的值减少为0.1。因为圆柱体的网格是2个单位高，所以它压扁后的有效高度为0.2个单位。我们来做一个大的钟，把sacle里的**X**和**Z**的值增加到10.  
  
![图片10][图片10]![图片11][图片11]

<center>*压扁后的圆柱体*</center>  
  
把圆柱体对象的名称改为*Face*，表示钟面。因为它是我们钟的一部分，所以我们把它归属到*Clock*对象下变成它的子对象。只需要在*Hierarchy*窗口下把*face*拖到*Clock*上就可以了。  
  
![图片12][图片12]

<center>*子对象*</center>  
  
子对象跟随其父对象的变化而变化，当*Clock*改变*Position*（位置）的时候，*Face*也跟着一起改变，就如同它们是一体的一样，*Rotation*（旋转）和*scale*（放缩）同理。你可以利用这点来制作复杂的几何体对象。  
  
1.3**制作钟面上的时间**  
  
钟面的外沿通常有一些记号来帮助指示时间。我们用方块来代表12个小时。
  
通过*GameObject/3D object/Cube*向场景中添加一个立方体对象。把它的*Scale*的值改为（0.5，0.2，1），让它变成一个小长方体。现在，我们把它放到钟面上。设置它的*Position*的值为（0，0.2，4）。这样它就被放到了钟面上并且可以用来表示时间。我们把它命名为*Hour Indicator*（时间标志物）。
  
![图片13][图片13]![图片14][图片14]

<center>*时间标志物*</center>  
  
现在的这个立方体的颜色跟钟面的颜色一样，很难分辨，我们来通过*Assets/Create/Material*或者*Project*窗口中的内容菜单来给它新建一个材质。这个材质跟默认的材质是一样的，我们把它的*Albedo*（反射率）改的黑一些，比如把red，green，blue的值改为73。它就变成了一个灰黑色的材质。我们给它命名为*Clock Dark*。
  
![图片15][图片15]  
![图片16][图片16]

<center>*Clock Dark材质设置*</center>  
  
**什么是albedo（反射率）？**  
albedo是一个拉丁词语，意思是光度。简单的将就是材质的颜色。  
  
我们的时间标志物需要用到这个材质，你可以直接把它拖拽到场景中的对象上或者拖到*Hierarchy*窗口中的对象上。你也可以把它拖到*Inspector*窗口的下面，或者把它更改为*mesh renderer*（网格渲染器）中的*Materials*（材质）数组中的*Element 0*。  
  
![图片17][图片17]

<center>*黑色的时间标志物*</center>
  
我们的时间标志物现在放在12点的位置上，要是我们想把它放到1点的位置上该怎么办呢？我们都知道12个小时有12个位置，钟面有360°，我们只需要将它绕着Y轴旋转30°就可以了。我们试一下吧。  
  
![图片18][图片18]![图片19][图片19]

<center>*旋转后的时间标志物不在正确的位置上*</center>  
  
我们可以看到，我们的时间标志物还在12点的位置上，只不过是绕着自己的y轴旋转了30°，因为对象的*rotation*（旋转）是在对象自己的位置上进行的旋转。  
我们需要把我们的时间标志物沿着钟面的边缘旋转到1点的位置上。我们不需要自己算出1点的确切位置，*Object Hierarchy*可以帮我们算。首先，我们先把我们的时间标志物的*rotation*（旋转）改回0。然后我们新建一个空对象，position和rotation都是0，scale是1。把时间标志物变成它的子对象。  
  
![图片20][图片20]

<center>*暂时的父对象*</center>  
  
现在，把父对象的*rotation*的y值改为30°。这样我们的时间标志物就会绕着它的父对象的位置旋转，并停到我们想要的位置上。  
  
![图片21][图片21]

<center>*正确位置上的时间标志物*</center>  
  
我们可以通过按ctrl或者command+D键或者*Hierarchy*窗口中的内容菜单里复制这个暂时的父对象。每次复制时将*rotation*的y值增加30°，直到我们做完每个小时的时间标志物。  
  
![图片22][图片22]

<center>*12个小时*</center>  
  
我们不需要这个暂时的父对象了。在*Hierarchy*窗口中选中一个时间标志物，把它拖到*Clock*中，现在它就成了*Clock*的子对象了。进行这步操作时，Unity会更改时间标志物的*Transformation*，这样它在这个世界空间里的*position*（位置）和*rotation*（旋转）不会被改变。对剩下的11个时间标志物也进行同样的操作，然后删除那个暂时的父对象。你可以按住ctrl或者command加鼠标左键来一次性选中多个对象来快速完成这一操作。  
  
![图片23][图片23]

<center>*时间标志物子对象*</center>  
  
**我能看到90.00001这类的值，有什么问题吗？**  
这是因为position，rotation，和scale这类组件是浮点数类型的值。这些数字的精密度是有限的，会跟你预期的值有非常小的偏差。所以你不必为这微不足道的0.00001的偏差而感到担心。  
  
1.4**制作指针**  
我们可以用相同的方式做时钟的指针。创建一个新的立方体，命名为*Arm*并给它赋予刚才用的灰黑色的材质。把它的*scale*的值设为（0.3，0.2，2.5），让它比时间标志物更窄，更长。将它的*position*的设为（0，0.2，0.75），它会出现在钟面上并且指着12点的位置，而且在相反的方向也露出一点长度，这样在它旋转的时候会显得有一些平衡在里面，更加好看。  
  
![图片24][图片24]![图片25][图片25]

<center>*指针*</center>  
  
**光照图标哪去了？**  
我把光照去掉了，让它不会再在场景中捣乱了。它变成了平行光后，它的位置就不重要了。  
  
为了让指针绕着时钟的中心旋转，我们需要给它创建一个父对象，就像我们之前给我们的时间标志物做的那样。让这个父对象的position和rotation的值为0，sacle的值为1，方便我们之后旋转它。将这个父对象命名为*Hours Arm*（时针）并把它变为*Clock*的子对象。这样我们的*Arm*就变成了*Clock*的子对象的子对象。  
  
![图片26][图片26]

<center>*带Hours Arm的Clock几何体*</center>  
  
将*Hours Arm*（时针）复制两次来制作*Minutes Arm*（分针）和*Seconds Arm*（秒针）。分针应该比时针更窄更长，所以把它的*Arm*子对象的scale值设为（0.2，0.15，4），把它的position的值设为（0，0.375，1），分钟就出现在了钟面上。  
设置*Seconds Arm*（秒针）的子对象的scale为（0.1，0.1，5），position为（0，0.5，1.25）。我为了让它更与众不同，我制作了一个*Clock Red*材质，将材质的*albedo*的RGB的值为（197，0，0），并把它赋给秒针的子对象。  
  
![图片27][图片27]![图片28][图片28]

<center>*3个指针*</center>  
  
这样，我们的时钟就搭建好了。如果你以前没做过这个，现在最好保存一下场景，它会作为一个资源保存在项目中。  
  
![图片29][图片29]

<center>*保存场景*</center>  
  
如果你有问题卡住了，想要跟源文件对比一下，或者你想跳过这个创建钟表的过程，你可以下载这个[源项目包](https://catlikecoding.com/unity/tutorials/basics/game-objects-and-scripts/building-a-simple-clock/building-a-simple-clock.unitypackage)。这个源项目包包含本章到目前所做的全部工作，你可以通过*Assets/Import Package/Custom Package......*来向Unity项目中导入这些包，把它拖入Unity窗口，或者直接双击打开。  
  
2**动画化时钟**  
  
我们的时钟还不能正确的指示时间。它只是一个几何体对象，是Unity渲染出来的一系列的网格模型，仅此而已。如果你有默认的*clock*组件，我们可以直接用它来显示时间。但是我们没有啊T^T！！我们只能自己做一个。组件是通过脚本来定义的。通过*Assets/Create/C# Script*来在项目里添加一个新的脚本资源，命名为*Clock*。  
  
![图片30][图片30]![图片31][图片31]

<center>*clock脚本*</center>  
  
当选中这个脚本的时候，*Inspector*窗口会显示它的内容和一个按钮，按按钮可以在代码编辑器中打开文件。你也可以双击脚本直接打开编辑器。脚本文件默认自带一些代码如下。  
  
	using System.Collections;
	using System.Collections.Generic;
	using UnityEngine;

	public class Clock : MonoBehaviour {

	// Use this for initialization
	void Start () {
		
	}
	
	// Update is called once per frame
	void Update () {
		
	}
	}  
  
这是C#代码，是Unity用来编写脚本的编程语言。为了理解代码是如何工作的，我们要把代码全部删除从头开始。 
  
**JavaScript怎么用？**  
Unity还支持另一种编程语言，通常是指JavaScript，但其实它真正的名字是UnityScript。Unity 2017.1.0 依旧支持这种语言。但是菜单项中*create a JavaScript asset*功能将会在Unity 2017.2.0中被移除。
之后可能会完全不支持JavaScript。  
  
2.1**定义组件类型**  
  
空文件是不可用的脚本，它必须包含我们时钟组件的定义。我们不会定义一个单一的组件的实例，我们要定义一个类或者类型为clock。这样以后，我们可以在Unity中创建很多这样的组件。  
  
在C#中，我们要定义一个clock类，要在class 后面写上要定义的类的名称，这里是clock，代码如下所示。我们目前是空文件，所以只有下面这一行代码。  
  
	class Clock  
  
**什么是class（类）？**  
你可以把class（类）当做是一个蓝图，它能在计算机内存中创建对象。这个蓝图定义了这些对象所拥有的的数据和功能。  
class（类）也可以定义那些不属于对象的数据和功能，只能用于类本身。这经常被用于提供全局函数。  
  
因为我们不想限制访问Clock类的代码权限，所以我们用*public*来修饰我们的类。  
  
	public class Clock  
  
**class（类）的默认访问权限是什么？**  
  
如果我们不更改访问权限，那么系统会默认修改代码为*internal class Clock*。这样就限制了只允许内部的代码访问该类，这样当你使用多个DLL文件包的时候就会有问题，为了保证它随时都是可用的，我们把访问权限改为public。  
  
现在我们这个还不是C#语句，要完整的定义一个class（类），需要定义它究竟是什么样的，是用{}圈起来的一大块代码，目前我们还没有内容，所以只写一对{}即可。  
  
	public class Clock {}  
  
现在我们的代码是可用的了，保存文件并返回Unity。Unity编辑器会检测到脚本资源被修改了并且触发重编译。重编译结束后，选择我们的脚本，*Inspector*会通知我们资源不包含*Monobehaviour script*。  
  
![图片32][图片32]

<center>*没有组件的脚本*</center>  
  
这个意思是告诉我们，我们不能在Unity中通过这个脚本来创建组件，我们定义的clock是一个泛型C#对象类。Unity只能*Monobehaviour*的子类来创建组件。  
  
**什么是mono-behavior**  
  
它的意思是我们可以编写自己的组件来给游戏对象添加自定义行为。这就是这个词的behavior的意思。只不过是要用英语拼写，也就是behaviour。mono指的是Unity支持添加自定义代码的方法，使用的是Mono项目，是.NET框架的多平台实现。因此有了这个名字*MonoBehaviour*  
  
为了将*Clock*转换为*MonoBehaviour*的子类，我们要修改我们的类的声明，在Clock和MonoBehaviour中加一个冒号，代表Clock是MonoBehaviour的扩展，Clock继承了MonoBehaviour的所有功能。  
  
	public class Clock : MonoBehaviour {}  
  
然而，这样编译后会产生一个错误。编译器会提示我们找不到*MonoBehaviour*类。因为这个类是包含在一个*namespace*（名空间）里的，这个名空间为*UnityEngine*。要使用它，我们需要使用它的全称，*UnityEngine、MonoBehaviour*。  
  
	public class Clock : UnityEngine.MonoBehaviour {}  
  
**什么是namespace（名空间）**  
  
一个*namespace*就类似于一个代码的网站域名。域名可以有子域名，namespace（名空间）也可以有*subnamespaces*（子名空间）。它们最大的区别就是写法顺序不一样。一个是*forum.unity3d.com*，另一个是*com.unity3d.forum*。这些代码是Unity自带的，不需要自寻下载。namespace（名空间）是为了组织代码和预防名字冲突的。  
  
因为每次访问Unity类的时候都要加上UnityEngine很不方便，所以我们可以告诉编译器到哪里去寻找我们没有明确指出的namespace（名空间）。我们只需要在文件的顶部添加一行代码*using UnityEngine;*即可。千万别忘了结尾的分号‘;’。  
  
	using UnityEngine;

	public class Clock : MonoBehaviour {}  
  
现在我们可以在Unity中向我们的clock对象添加我们的组件了。你可以直接将脚本资源拖拽到对象上，也可以通过对象的*Inspector*窗口中的*Add Component*按钮来添加。  
  
![图片33][图片33]

<center>*带有我们自己的组件的Clock对象*</center>  
  
一个以我们的Clock类作为模板的C#对象实例被创建并添加到了Clock对象的组件列表中。  
  
2.2**控制指针**  
  
要旋转指针，Clock对象需要知道如何旋转它们。让我们先从时针开始。就像所有的游戏对象一样，可以通过修改*transform*组件中的*rotation*来旋转它。所以我们向Clock对象中添加一些组件来让它知道如何旋转指针。示例代码如下。  
*Hours transform*是一个不错的名字，但是命名必须为一个单词，所以根据国际惯例，首字母小写，其余的单词的首字母大写，然后组合在一起，就变成了*hoursTransform*  
  
	public class Clock : MonoBehaviour {

		hoursTransform;
	}  
  
**using语句哪去了？**  
using语句还在，我没展示而已。  
  
我们还有定义这个类的域，也就是*UnityEngine.Transform*。把它放在类名前。  
  
	Transform hoursTransform;  
  
现在我们的类定义了一个字段拥有另一个对象的引用，这个字段的类型是*Transform*。我们要确保它包含时针的transform组件的引用。  
  
字段的默认访问权限是private，意思是它只能被属于Clock的代码访问。但是类并不认识Unity场景，所以我们把访问权限改为public，这样谁都可以访问它了。  
  
	public Transform hoursTransform;  
  
**public域不是一种不好的形式吗？**  
一般来说，在编程的时候要避免public域来保证一致性。然而，在Unity编辑器中，需要public域来把一切组织起来。这样你就可以围绕着它来工作，使代码变得更简单。  
  
一旦字段的访问权限变为public，它就会显示在*Inspector*窗口中。这是因为*Inspector*自动使所有的public字段组件可编辑化。  
  
![图片34][图片34]

<center>*时钟的transform域*</center>  
  
为了让它们联系在一起，我们把*Hours Arm*从几何体中拖拽到*Hours Transform*字段中。或者使用字段右边的圆形按钮来搜索*Hours Arm*也可以。  
  
![图片35][图片35]

<center>*Hours transform关联*</center>  
  
在拖拽或者选中*Hours Arm*（时针）对象后，Unity编辑器会抓取它的*transform*组件并把它的引用放入我们的字段中。  
  
2.3**控制所有指针**  
  
我们也要对我们的分针和秒针做相同的操作，所以我们再向*Clock*中添加两个命名好的字段。  
  
	public Transform hoursTransform;
	public Transform minutesTransform;
	public Transform secondsTransform;  
  

我们可以把它们的声明变得更简洁一些。因为它们的类型和访问权限都是一样的，我们可以将它们放在一起，用逗号分割开来。  
  
	public Transform hoursTransform, minutesTransform, secondsTransform;
	//	public Transform minutesTransform;
	//	public Transform secondsTransform;
  
**//是用来干嘛的？**  
  
//用来代表注释，在//后面的全部算是注释部分，也可以用来删除代码。  
  
在Unity编辑器中把剩下两个指针链接起来。  
  
![图片36][图片36]

<center>*链接所有的指针*</center>   
  
2.4**获取时间**  
  
现在我们能控制Clock上的指针了，接下来就要知道当前的时间。要做到这一点，我们需要让我们的Clock来执行一些代码。我们需要想代码块中加入一些方法。方法名首字母大写，我们把这个方法命名为*Awake*，意思是当组件激活的时候就执行这些代码。  
  
	public class Clock : MonoBehaviour {

		public Transform hoursTransform, minutesTransform, secondsTransform;

		Awake {}
	}  
  
方法就像是数学方程一样，例如**f（x）=2x+3**。这个方程将一个数先乘以2然后再加3。这个方程有一个未知数，对应一个解。在此基础上，方法就类似于**f（p）=c**。p代表输入参数，c代表执行代码。和泛型不同，这个函数的结果是什么呢？这是需要明确声明出来的。在我们这个例子中，我们只想执行一些代码，并不需要返回运算结果，所以这个方法的返回值是void。我们用void来修饰它。  
  
	void Awake {}  
  
我们也不需要任何的输入参数，所以只需要在方法后面加一对（）就可以了，括号内不需要填参数。  
  
	void Awake () {}  
  
我们现在有了一个可用的方法，尽管它现在什么都干不了。就像Unity检测我们的域一样，它也会检测到这个*Awake*方法。当一个组件拥有*Awake*方法时，Unity会在这个组件创建或者加载时执行这个方法。  
  
**Awake方法需要用public修饰吗？**  
  
Awake和一些其他的方法在Unity中是特殊的方法。不论我们怎么修饰这些方法，引擎都会自动找到它们并在适当的时候激活它们。我们也不应该用public来修饰它们，因为它们不应该被除了Unity引擎外的其他东西激活。  
  
为了测试它是否能够正常执行，我们让*Awake*创造一些测试信息。*UnityEngine.Debug*类中有一个公共方法*Log*。我们让它打印一行简单的文本。代码如下。  
  
	void Awake () {
		Debug.Log("Test");
	}  
  
进入Unity引擎的play模式。你可以在Unity编辑器的底部的状态栏中看到一行字符串。你也可以在*Window/Console*控制台窗口中看到这个字符串。控制台还提供一些额外的信息，当你选中那些字符串的时候会显示哪些代码发出了这些信息。  
  
现在我们了解了方法是如何工作的，现在让我们显示激活时的时间吧。*UnityEngine*名空间中包含一个*Time*类，类中包含*time*属性。我们要用这个，把它打印出来。  
  
	void Awake () {
		Debug.Log(Time.time);
	}  
  
**什么是属性？**  
属性是封装在方法中的字段。它可以是只读或者只写的。但是Unity通常不这么做。  
  
然而结果一直是0.这是因为*Time.time*是告诉我们进入play模式后过了多少秒。而我们的Awake方法是瞬间被激活的，并没有经过任何时间。所以它并不告诉我们真实的时间。  
  
为了得到我们计算机上的系统时间，我们可以用结构体*DateTime*。这不是Unity类型。它可以在*System*名空间中被找到。它是Unity用来支持脚本的.NET框架的核心功能的一部分。
  
**什么是结构体？**  
结构体是一个蓝图，类似于类。区别在于它创建的是一个类似于integer或者color的值，而不是一个对象。它没有对象的特性。自定义结构体和自定义类是一样的，只不过关键字是*struct*而不是*class*。  
  
*DateTime*有一个元素是*Now*。它能提供一个*DateTime*值包含系统的当前时间和日期。让我们测试它一下。  
  
	using System;
	using UnityEngine;

	public class Clock : MonoBehaviour {

		public Transform hoursTransform, minutesTransform, secondsTransform;

		void Awake () {
			Debug.Log(DateTime.Now);
		}
	}  
  
现在我们每次进入play模式就会得到时间戳。  
  
2.6**旋转指针**  
  
下一步就是要让指针跟着当前时间来旋转。我们再先从时针开始。*DateTime*有*Hour*元素能够告诉我们*DateTime*值中小时的值。在当前的时间戳上激活它会给我们提供现在的小时。  
  
	void Awake () {
			Debug.Log(DateTime.Now.Hour);
		}  
  
我们可以用这个来进行旋转。旋转在Unity中被存在一个四元数中。我们可以通过*Quaternion.Euler*方法来创建一个公共的四元数。它的参数是X，Y，Z轴的旋转角并产生一个恰当的四元数。  
  
	void Awake () {
	//		Debug.Log(DateTime.Now.Hour);
		Quaternion.Euler(0, DateTime.Now.Hour, 0);
	}  
  
**什么是四元数？**  
四元数的基础是复杂的数字，用来表示3D旋转。它比简单的3D向量难理解，但是也有好处，比如它不用担心万向节死锁。  
  
*UnityEngine.Quaternion*是一个简单的值，是一个结构体，不是类。  
  
三个参数都是真实的数字，在C#中是浮点数。为了更准确的表达，我们在这些数字后面加上f表示浮点数。  
  
		Quaternion.Euler(0f, DateTime.Now.Hour, 0f);  
  
因为我们每隔30°一个时间标志物，一共十二个。为了让旋转能够符合这一标准，我们要让小时乘以30。用*代表乘。  
  
		Quaternion.Euler(0f, DateTime.Now.Hour * 30f, 0f);
  
我们可以声明一个中间值来表示我们把时间转换为了角度。类型为float，值为30f，命名为*degreesPerHour*。  
  
	float degreesPerHour = 30f;

	public Transform hoursTransform, minutesTransform, secondsTransform;

	void Awake () {
		Quaternion.Euler(0f, DateTime.Now.Hour * degreesPerHour, 0f);
	}   
  
因为这个中间值是不应该被更改的，所以我们可以在声明时用*const*来修饰它。这样我们的*degreesPerHour*就变成了一个常量。  
  
	const float degreesPerHour = 30f;  
  
**常量有什么特殊的？**  
关键字*const*标志某个量不会被改变。在编译中会被计算并且在之后也会作为常数一直被使用。只有像数字这种基本数据类型可以被修饰。  
  
现在我们得到了旋转，但是我们还没有应用它，它就直接被丢弃了。我们要把它应用到时针上，我们要把它作为值赋给时针的transform组件的*localRotation*方法。  
  
	void Awake () {
		hoursTransform.localRotation =
			Quaternion.Euler(0f, DateTime.Now.Hour * degreesPerHour, 0f);
	}  
  
![图片37][图片37]

<center>*时针指向4点*</center>  
  
**什么是localRotation**  
*localRotation*指的是transform组件中的实际的旋转，独立于其父类的旋转。换句话说，是在对象内部的旋转，是在*Inspector*中transform组件显示的值。所以如果我们把clock的根对象旋转了，那么它的指针也会跟着时钟一起旋转，正如我们所期待的那样。  
还有一个方法是*rotation*。它指的是对象在这个世界里的最终的旋，会把父类的transformations的值也进行计算。如果我们用这个，那么我们的指针就不会旋转，因为钟也会跟着指针一起旋转，二者抵消了。  
  
现在当我们进入play模式的时候，时针可以指向正确的位置了。我们对剩下的两个指针进行相同的操作，分针和秒针每次变6°。  
  
	const float
		degreesPerHour = 30f,
		degreesPerMinute = 6f,
		degreesPerSecond = 6f;

	public Transform hoursTransform, minutesTransform, secondsTransform;

	void Awake () {
		hoursTransform.localRotation =
			Quaternion.Euler(0f, DateTime.Now.Hour * degreesPerHour, 0f);
		minutesTransform.localRotation =
			Quaternion.Euler(0f, DateTime.Now.Minute * degreesPerMinute, 0f);
		secondsTransform.localRotation =
			Quaternion.Euler(0f, DateTime.Now.Second * degreesPerSecond, 0f);
	}  
  
![图片38][图片38]

<center>*时钟显示16：29:06*</center>  
  
因为我们用了三次*DateTime.Now*来搜索小时，分钟，秒。但是每次我们进行搜索的时候会消耗一定量的时间，所以我们实际上找到的：小时，分钟，秒的时间是属于三个不同的时间，而我们想显示的是一个时间的小时，分钟，秒。我们可以在方法内部声明一个变量，把时间存在变量里，这样就只需要搜索一次时间然后显示出来就可以了。  
  
**什么是变量？**  
变量类似于一个字段，它只存在于方法被执行的时间里。它属于方法，而不属于类。  
  
	void Awake () {
		DateTime time = DateTime.Now;
		hoursTransform.localRotation =
			Quaternion.Euler(0f, time.Hour * degreesPerHour, 0f);
		minutesTransform.localRotation =
			Quaternion.Euler(0f, time.Minute * degreesPerMinute, 0f);
		secondsTransform.localRotation =
			Quaternion.Euler(0f, time.Second * degreesPerSecond, 0f);
	}  
  
2.6**动画化指针**  
  
现在，我们进入play模式后就会得到当前的时间，但是之后我们的钟就会停在那不动了。为了让我们的钟和时间同步下去，我们要把我们的*Awake*方法改为*Update*方法。这个方法在我们进入play模式后会每一帧激活一次，而不是像*Awake*那样只激活一次。  
  
	void Update () {
		DateTime time = DateTime.Now;
		hoursTransform.localRotation =
			Quaternion.Euler(0f, time.Hour * degreesPerHour, 0f);
		minutesTransform.localRotation =
			Quaternion.Euler(0f, time.Minute * degreesPerMinute, 0f);
		secondsTransform.localRotation =
			Quaternion.Euler(0f, time.Second * degreesPerSecond, 0f);
	}  
  
   
  
  
可以注意到在*Inspector*中我们的组件的名字前得到了一个开关。这可以让我们禁用这些组件，可以阻止Unity激活它们的*Update*方法。  
  
![图片39][图片39]

<center>*禁用开关*</center>  
  
2.7**持续旋转**  
  
现在我们的时钟的指针可以按照当前的小时，分钟，秒来进行正确的旋转了，但是它看上去就像电子表一样，一下一下的蹦，而正常的钟表应该是跟随时间慢慢旋转的。让我们通过向组件的*inspector*中添加开关来实现这个效果吧。  
  
向clock中添加一个public字段，命名为*continuous*。它可以是开也可以是关，所以它的变量类型为bool型。  
  
	public Transform hoursTransform, minutesTransform, secondsTransform;

	public bool continuous;  
  
布尔型的值只有true和false，在我们这里表示开和关。它的默认值是false，所以我们要在*inspector*中把它打开。  
  
![图片40][图片40]

<center>*打开continuous*</center>  
  
现在我们有了两条路线了，我们将我们的*Update*方法复制一份并把它们重命名为*UpdateContinuous*和*UpdateDiscrete*。  
  
    void UpdateContinuous () {}
        DateTime time = DateTime.Now;
        hoursTransform.localRotation =
            Quaternion.Euler(0f, time.Hour * degreesPerHour, 0f);
        minutesTransform.localRotation =
            Quaternion.Euler(0f, time.Minute * degreesPerMinute, 0f);
        secondsTransform.localRotation =
            Quaternion.Euler(0f, time.Second * degreesPerSecond, 0f);
    }

    void UpdateDiscrete () {
        DateTime time = DateTime.Now;
        hoursTransform.localRotation =
            Quaternion.Euler(0f, time.Hour * degreesPerHour, 0f);
        minutesTransform.localRotation =
            Quaternion.Euler(0f, time.Minute * degreesPerMinute, 0f);
        secondsTransform.localRotation =
            Quaternion.Euler(0f, time.Second * degreesPerSecond, 0f);
    }    
  
新建一个*Update*方法，如果*continuous*是true，那么久激活*UpdateContinuous*。可以用if语句实现这个。if后面有一对括号，括号内为判断条件，如果为true，则执行代码块内的代码。如果判断条件为false，这跳过执行。  
  
    void Update () {
        if (continuous) {
            UpdateContinuous();
        }
    }  
  
**新的*Update*方法需要在哪里定义？**  
  
在Clock类的内部即可。  
  
也可以在if语句中加入其它的代码块，在判断条件为false时执行这部分代码块。关键字为*else*，我们可以用它来激活我们的*UpdateDiscrete*方法。  
  
    void Update () {
        if (continuous) {
            UpdateContinuous();
        }
        else {
            UpdateDiscrete();
        }
    }  
  
现在，我们可以在两条路线进行转换，但是两条路线的内容是一样的。我们需要调整*UpdateContinuous*让它来显示非整点的小时，分钟，秒。不幸的是，*DateTime*不包含现成的非整点的数据。幸运的是，它有一个*TimeOfDay*属性。这个属性有*TimeSpan*值是我们需要的那种形式的值。特别是*TotalHours*，*TotalMinutes*，和*TotalSeconds*。  
  
    void UpdateContinuous () {
        TimeSpan time = DateTime.Now.TimeOfDay;
        hoursTransform.localRotation =
            Quaternion.Euler(0f, time.TotalHours * degreesPerHour, 0f);
        minutesTransform.localRotation =
            Quaternion.Euler(0f, time.TotalMinutes * degreesPerMinute, 0f);
        secondsTransform.localRotation =
            Quaternion.Euler(0f, time.TotalSeconds * degreesPerSecond, 0f);
    }  
  
这会在编译时产生一个错误，这是因为这些新的值的类型是错的。它们是双精度浮点数（double）。这种数据类型的精度要比单精度浮点数（float）要高，但是Unity代码只能运行float类型的值。  
  
**单精度够吗？**  
  
对于大部分的游戏来说是够用的，但是如果遇到非常大的场景或者精度特别高的放缩的话就会出现问题。这时候就要用一些小技巧来解决这些问题了，比如让当前游玩场景接近世界的边缘。然而双精度浮点数就可以解决这个问题，但是它也会使数据的大小也翻倍，这样会产生别的问题，所以，大多数的游戏引擎都会使用单精度浮点数。  
  
我们可以通过强制类型转换，把double变成float来解决这个问题。只需要在需要转换的数据前加上一对括号，并在括号内填上需要的类型就可以了。  
  
        hoursTransform.localRotation =
            Quaternion.Euler(0f, (float)time.TotalHours * degreesPerHour, 0f);
        minutesTransform.localRotation =
            Quaternion.Euler(0f, (float)time.TotalMinutes * degreesPerMinute, 0f);
        secondsTransform.localRotation =
            Quaternion.Euler(0f, (float)time.TotalSeconds * degreesPerSecond, 0f);  
  

  

  
现在你学会了Unity基本的对象的创建和脚本的编写。下一个教程是[创建一个图片](链接)  
  
[动画化时钟源文件](https://catlikecoding.com/unity/tutorials/basics/game-objects-and-scripts/animating-the-clock/animating-the-clock.unitypackage)



[图片1]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/tutorial-image.jpg

[图片2]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/default-layout.png

[图片3]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/2-by-3.png

[图片4]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/low-resolution-aspect-ratios.png

[图片5]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/hierarchy.png

[图片6]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/inspector.png

[图片7]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/cylinder.png

[图片8]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/cylinder-inspector.png

[图片9]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/cylinder-no-collider.png

[图片10]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/cylinder-scaled-scene.png

[图片11]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/cylinder-scaled-inspector.png

[图片12]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/child-object.png

[图片13]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/indicator-scene.png

[图片14]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/indicator-inspector.png

[图片15]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/dark-material.png

[图片16]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/color-popup.png

[图片17]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/dark-indicator.png

[图片18]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/rotated-scene.png

[图片19]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/rotated-inspector.png

[图片20]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/temporary-parent.png

[图片21]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/rotated-correct.png

[图片22]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/12-hour-indicators.png

[图片23]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/periphery-children.png

[图片24]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/hours-arm-scene.png

[图片25]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/hours-arm-inspector.png

[图片26]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/hours-arm-hierarchy.png

[图片27]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/all-arms-scene.png

[图片28]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/all-arms-hierarchy.png

[图片29]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/saved-scene.png

[图片30]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/clock-project.png

[图片31]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/clock-inspector.png

[图片32]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/non-component.png

[图片33]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/clock-with-component.png

[图片34]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/hours-transform-field.png

[图片35]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/hours-transform-connected.png

[图片36]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/arms-connected.png

[图片37]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/hours-arm-rotated.png

[图片38]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/all-arms-rotated.png      

[图片39]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/enabled-toggle.png

[图片40]:https://raw.githubusercontent.com/GameDev-DJTU/Catlike-Coding-Unity-CN-Picture/master/basic/1_game_objects_and_scripts/continuous-option.png  

