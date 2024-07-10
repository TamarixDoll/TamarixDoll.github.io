## 角色控制



#### 	角色控制器与键盘输入



**首先为自机添加一个元件(Inspector > Add Component > New script)**

创建脚本后会自带两个初始函数，Start函数会在第一帧开始前调用一次，Update函数会在每一帧都调用一次。

关于transform，官方解释为：

> The Transform stores a GameObject's Position,Rotation,Scale and parenting state.

所以，transform实际上就是存储一个物件的位置状态信息，如果想要移动一个物件，就需要改变该物件的position，即坐标。



**找到Edit > project Settings > Input Manager**

![1](photo\1.png)

我们通过输入某个按键，可以得到对应的**Axes**值。

**Negative Button** 和 **Positive Button** 分别表示按下该按键时，得到的**Axes**值为负(或正)，我们可以通过**Input Manager**对其进行设置。

**Name**表示该按键组得到对应的Axes值的变量名，在编写脚本时会用到。



**接下来我们开始编写用于控制自机的脚本**

```c#
public class saki : MonoBehaviour
{
    void Start()
    {
    }

    void Update()
    {
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");
        Vector2 position = transform.position;
        position.x = position.x + 3.0f * horizontal * Time.deltaTime;
        position.y = position.y + 3.0f * vertical * Time.deltaTime;
        transform.position = position;
    }
}
```

**Input.GetAxis()**函数用于读取**Axes**值，这里我们分别读取了水平和竖直方向上的**Axes**值，这代表了在这一帧自机的移动方式。

两个**float**类型的变量分别用于存储水平和竖直方向的**Axes**值

**Vector2**类型的变量可以存储坐标，上文说过**transform**用来存储物件的位置状态信息，故**transform.position**就是当前物件的坐标。

**Vector2**类型的变量中包含x和y两个值，分别代表了横纵坐标，如果对应的**Axes**值为正，则在该方向会正向移动。

更改完**position**的**x**值和**y**值后，对**transform**中的坐标值进行更改，物件的坐标发生变化，因此产生移动。

**Time.deltaTime**代表渲染每一帧所需要的时间，保证若渲染每一帧所用的时间长，则每一帧移动的距离也会随之变大，反之亦然，这样无论电脑的性能如何，每秒渲染的帧数都不会对最终移动的距离产生影响。

在性能较差的电脑上，因为每秒渲染的帧数少，而移动的距离相同，所以会显得卡顿。



#### 	通过直接移动刚体来移动角色

上文我们已经能够通过**asdw**按键来移动角色了，然而，在角色与建筑物产生碰撞时会发生抖动，这肯定不是我们想要的效果。

**为什么会发生抖动呢？**

当给物件添加刚体和碰撞体后，unity引擎会自动判断并阻止一个碰撞体进入到另一个碰撞体中。通过上文的脚本，自机在移动时**先移动物件本身，随后刚体会跟随移动**，然后再判断是否进入了另一个碰撞体中，若是则再将其反弹回来，也就是**先移动再判断并阻止**，所以也就产生了“抖动”。

想要解决这一问题，我们直接移动刚体本身即可。

```c#
public class saki : MonoBehaviour
{
    float horizontal;
    float vertical;
    Rigidbody2D rigidbody2d;

    void Start()
    {
        rigidbody2d =  GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        horizontal = Input.GetAxis("Horizontal");
        vertical = Input.GetAxis("Vertical");

    }

    void FixedUpdate()
    {
        Vector2 position = rigidbody2d.position;
        position.x = position.x + 3.0f * horizontal * Time.deltaTime;
        position.y = position.y + 3.0f * vertical * Time.deltaTime;
        rigidbody2d.MovePosition(position);
    }
}
```

**GetComponent<Rigidbody2D>()** 函数用于访问当前物件的**Rigidbody2D**元件。

与上文脚本大致相同，只需要新加入一个**FixedUpdate**函数，并在该函数内将坐标移动方式改为直接移动刚体即可。

**position**变量不再存储**transform**所包含的坐标，而是直接访问刚体本身的坐标。

通过**MovePosition()**函数直接移动刚体本身。





## 刚体与碰撞体



#### 	刚体

在 **Inspector > Add component** 中，为物件添加 **Rigidbody 2D** 元件，我们的物件就会变成刚体，现在它就会受物理系统的影响了。

​		![](photo\2.png)

在**2d俯视角游戏**中，暂时需要将刚体的重力改为0(**Gravity Scale** 设置为 0)。同时我们也不希望刚体们在碰撞时进行花式旋转，所以将**Freeze Rotation**勾选，禁止旋转。



#### 碰撞体

虽然我们为物件增加了刚体元件，但是在游戏中各物件之间还是会互相穿梭，不会产生碰撞。这是因为我们暂时还不知道物体的形状和体积，所以无法判定碰撞。添加碰撞体元件即可解决这一问题。

在 **Inspector > Add component** 中，为物件添加 **Box Collider 2D** 元件

![](photo\3.png)

选中 **Edit Collider** 后，可以在**Scene**界面通过物件上的绿框来调整碰撞体的大小

![](photo\4.png)

为物件添加碰撞体元件后，各物件之间就会产生碰撞了。



#### 	瓦片地图碰撞体

我们不仅仅希望物体之间产生碰撞，同样，地图上的一些建筑或特殊地形也需要与角色产生碰撞才行。

在**Hierarchy**中选中你的瓦片地图，为其添加 **Tilemap Collider 2D** 元件。你会发现整个地图都是碰撞体。然而我们只需要某些特殊地形会阻止角色移动，大部分地形是可以行走在上面的。

​		

找到储存瓦片地图的文件夹，将不需要产生碰撞的地图块的**Collider Type**设置为**None**即可。

![](photo\5.png)

![](photo\6.png)

继续为瓦片地图添加 **Rigidbody 2D** 和 **Composite Collider 2D** 元件对其进行优化，将连接在一起的小碰撞体组合成一个大的复合碰撞体。可以减少物理系统的计算量，同时防止两个相邻的小碰撞体间因出现计算误差而发生碰撞。

在**Tilemap Collider 2D** 中勾选 **Used By Composite** 即可。

将 **Rigidbody 2D** 中 **Body Type** 更改为 **Static**，我们不希望瓦片地图移动。



> 一点教训：建筑物最好还是不要直接掺杂在瓦片地图里，不太好处理显示问题......单独做成物件比较好





#### 关于不同图层间的碰撞问题

在**Inspector**面板中，可以给游戏对象设置不同的**Layer**。

![](photo\13.png)

在 **Edit > Project Settings < Physics 2D** 中，可以调整不同图层间的碰撞关系，未被勾选的两个图层之间不会产生碰撞效果。

![](photo\12.png)





## 摄像机 - Cinemachine

到目前为止，我们的游戏视野都是固定的，如果自机走出屏幕外的话就看不见了。为了解决这个问题，我们要用到**Cinemachine**插件。

打开 **Window > Package Manager** ，搜索 **Cinemachine** 并下载。

下载后，我们可以在**Hierarchy**界面创建一个**Cinemachine > 2D camera**，暂时命名为**VirCamera**。

随后将自机拖入**VirCamera**的**Follow**中，该相机就会随着自机移动了。

![](photo\7.png)

若相机的视野过大或过小，则可以通过 **Lens > Ortho Size** 来调整视野大小，该参数表示摄像机的一半高度可容纳多少个单位。

之所以只调整高度，是因为摄像机的宽度会随着用户游戏窗口的分辨率和变化，但高度始终相同。



现在摄像机可以跟随我们的自机移动了，然而还有一个问题，当自机移动到地图边缘时，摄像机的视野范围也会随着移动到地图外。

显然，为了解决此问题，我们需要给摄像机加一个限制范围。



在**Hierarchy**界面右键选择 **Create Empty** 新建一个游戏对象，这里命名为 **confiner**，并为其添加 **Polygon Collider 2D** 组件。

编辑该几何碰撞体，让其恰好覆盖整个地图即可。



通过**Inspector**面板可以调整几何体各顶点的坐标或增删顶点。

![](photo\8.png)

调整完毕后，调整几何体所在的图层，避免其与其它刚体产生碰撞。

选择 **VirCamera** ，通过**CinemachineVirtualCamera > Extensions**，为**VirCamera**添加一个拓展组件 **Cinemachine confiner**

![](photo\9.png)

然后将刚刚调整好的几何体拖入到 **Bounding Shape 2D** 中

![](photo\10.png)

这样，摄像机的视野就不会超出刚刚设置好的范围了。



如果人物产生抖动，在Hierarchy > Main Camera > CinemachineBrain 中将如下两个参数调整为 Fixed Update 即可

![](photo\11.png)





## 触发器

如果我们想在地图中设置一个陷阱，或一个可以恢复生命的血瓶之类的东西，我们就需要用到触发器。

接下来，我们用触发器来实现 **触碰回血加速并消失的小精灵 和 触碰持续掉血的小精灵**。

在此之前，需要为自机添加一点点代码。

```c#
using System.Collections;
using System.Collections.Generic;
using Unity.VisualScripting;
using UnityEditor;
using UnityEngine;
using UnityEngine.SocialPlatforms;

public class saki : MonoBehaviour
{
    public int maxhealth = 5; //最大生命
    int CurrentHealth = 5; //当前生命
    public int health {get {return CurrentHealth;}} //只读，读取当前生命
    float horizontal;
    float vertical;
    Rigidbody2D rigidbody2d;

    public float Speed = 3.0f; //常规速度
    float currentSpeed = 3.0f; //当前速度
    float SpeedTimes = 0; //剩余加速时间

    bool BestTime = false; //是否无敌
    bool IsSpeed = false; //是否加速
    float Times = 0; //剩余无敌时间
    public float maxTimes = 2; //最长无敌时间

    // Start is called before the first frame update
    void Start()
    {
        rigidbody2d =  GetComponent<Rigidbody2D>();
    }

    // Update is called once per frame
    void Update()
    {
        horizontal = Input.GetAxis("Horizontal");
        vertical = Input.GetAxis("Vertical");
        
        if(BestTime) //如果当前处于无敌状态，则持续减少剩余无敌时间
        {
            Times -= Time.deltaTime;
            //剩余无敌时间归零，则取消无敌状态
            if(Times < 0) BestTime = false;
        }

        if(IsSpeed) //如果当前处于加速状态，则持续减少剩余加速时间
        {
            SpeedTimes -= Time.deltaTime;
            //剩余加速时间归零，则取消加速状态，并将速度恢复至常规速度
            if(SpeedTimes < 0)
            {
                currentSpeed = Speed;
                IsSpeed = false;
            }
        }

    }

    void FixedUpdate()
    {
        Vector2 position = rigidbody2d.position;
        position.x = position.x + currentSpeed * horizontal * Time.deltaTime;
        position.y = position.y + currentSpeed * vertical * Time.deltaTime;
        rigidbody2d.MovePosition(position);
    }

    public void ChangeHealth(int count)
    {
        if(count < 0) //如果是掉血，则检查是否处于无敌状态
        {
            if(BestTime) return; //处于无敌状态则不掉血

            BestTime = true; //不处于无敌状态，则掉血并进入无敌状态
            Times = maxTimes; //剩余无敌时间为当前最大无敌时间
        }
        if(count > 0) //如果是回血，则赋予加速状态
        {
            currentSpeed *= 1.5f;
            SpeedTimes = 1.5f;
            IsSpeed = true;
        }
        //Mathf.Clamp(x, min, max)函数，可以让x变量不会超出 min ~ max 范围
        CurrentHealth = Mathf.Clamp(CurrentHealth + count, 0, maxhealth);
        //产生调试信息
        Debug.Log(CurrentHealth + "/" + maxhealth);
    }
}

```

需要注意，**public int health {get {return CurrentHealth;}}** 这一行代码中，**get** 函数表示只读，这样外界可以通过公开变量 **health**来读取私有变量 **CurrentHealth** 且不能更改 **CurrentHealth** 变量。



好了，现在可以在地图中新加入一个小精灵，并为其添加 **Box Collider 2D** 组件。

![](photo\15.png)

在 **Box Collider 2D** 中，勾选 **Is Trigger**，现在这个小精灵就是一个触发器了。通过 **Edit Collider** 可以像编辑碰撞体一样编辑触发器范围

![](photo\14.png)

我们想要让角色触碰到小精灵时恢复一点生命。为此需要给小精灵加一个脚本。

```c#
public class health_collider : MonoBehaviour
{
    void OnTriggerEnter2D(Collider2D other)
    {
        //读取进入触发器的游戏对象是否含有 saki 组件，以验证进入触发器的是玩家
        saki now = other.GetComponent<saki>(); 
        //如果进入触发器的对象不是玩家，则读取不到 saki 组件，now为空
        if(now != null) 
        {
            if(now.health < now.maxhealth)//判断是否满血
            {
                now.ChangeHealth(1);
                Destroy(gameObject);//回血后小精灵消失
            }
        }
    }
}

```

**OnTriggerEnter2D**函数可以用来读取进入触发器的物体（进入后将不再判断，即只会在第一次接触时判断一次）。

与之对应的是**OnTriggerStay2D**函数，它在物体进入触发器后仍可以继续判断是否触碰。



同样在地图中添加另一只小精灵，触碰则持续造成伤害。

![](photo\18.png)

```c#
public class health_down : MonoBehaviour
{
    void OnTriggerStay2D(Collider2D other)
    {
        saki now = other.GetComponent<saki>();
        if(now != null)
        {
            now.ChangeHealth(-1);
        }
    }
}
```

上文说过 **OnTriggerStay2D** 与 **OnTriggerEnter2D** 的区别，此处我们希望在触碰小精灵时持续掉血而不是只掉血一次。

但这样还不行，因为引擎为了减少计算量，会默认在刚体不动时进入休眠状态不再计算，因此，我们需要在 **saki > Inspector > Rigidbody 2D** 中，将 **Sleeping Mode** 改为 **Never Sleep**，让刚体即使不动也会被计算。

![](photo\19.png)



至此，我们成功利用触发器实现了 **触碰回血加速并消失的小精灵 和 触碰持续掉血的小精灵**。



## 敌人

目前为止，我们已经用触发器实现了补给和陷阱效果，现在我们想要让地图上有一名敌人，不然会显得很空虚对吧。

于是我们加入一名可爱的敌人

![](photo\20.png)

然后再给小可爱加几行代码

```c#
public class enemyConsole : MonoBehaviour
{
    public bool Isvertical = true; //是否纵向行走，用于控制行走方式，可以直接更改
    float Times = 3; //当前剩余行走时间(该值归零时敌人转向)
    float maxTimes = 3; //朝某一方向行走的最长时间
    int Ispositive = 1; //是否为正向
    Rigidbody2D rigidbody2d;
    void Start()
    {
        rigidbody2d = GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        Times -= Time.deltaTime;
        if(Times < 0)
        {
            Ispositive = -Ispositive;
            Times = maxTimes;
        }
    }

    void FixedUpdate()
    {
        Vector2 position = rigidbody2d.position;
        if(Isvertical) //是否纵向移动
        {
            position.y -= 3.0f * Time.deltaTime * Ispositive;
        }
        else
        {
            position.x -= 3.0f * Time.deltaTime * Ispositive;
        }

        rigidbody2d.MovePosition(position);
    }

    void OnCollisionEnter2D(Collision2D other)
    {
        saki now = other.collider.GetComponent<saki>();
        if(now != null)
        {
            now.ChangeHealth(-1);
        }
    }
}

```

现在一个**碰撞后会让自机受伤的敌人**就做好了，很简单吧！

与上文的触发器不同，我们希望敌人可以与我们产生碰撞而不是简单的穿过去，所以用到了**OnCollisionEnter2D**函数。

与**OnTriggerEnter2D**一样，**OnTriggerEnter2D**函数在有碰撞体进入触发器的触发范围时调用，而**OnCollisionEnter2D**函数则是在刚体与某个对象产生碰撞时调用。

同时要注意，**OnCollisionEnter2D** 中的**other**的类型是**Collision2D**而不是上文中的**Collider2D**！

**Collider2D**是指碰撞体本身，我们可以通过它来访问该对象的其他组件。

而**Collision2D**指的是碰撞本身，它储存了本次碰撞所包含的信息，所以我们需要先得到本次碰撞中的碰撞体，然后再通过碰撞体得到其他组件。

这也就是**OnCollisionEnter2D**函数中使用 **other.collider.GetComponent<saki>()** 而不是 **other.GetComponent<saki>()** 的原因。

具体关于 **Collider2D** 和 **Collision2D** 的事我们以后再说。



现在，我们可爱的小敌人可以与我们碰撞并造成伤害了。



## 动画



#### 为敌人添加动画

上文中，我们布置了一个可以移动的可爱小敌人，虽然可以移动了，但毕竟只是死板的平移。所以这次，我们尝试给她加上动画效果。

在**Hierarchy**中选中敌人，打开预制件模式 (方便以后布置更多敌人) ，为其添加 **Animator** 组件。

在**project**界面新建一个**Animator Controller**，这里命名为**enemy robot**，然后拖到敌人**Animator**组件的**Controller**中。

![](photo\21.png)

这样，**enemy robot** 将会用来控制敌人的动画效果。

暂时现将**enemy robot**搁在一边，我们先来创建敌人的动画。

打开 **Window > Animation > Animation** ，选择敌人的预制件，为其创建一段新的动画。

这里我们先创建向右行走的动画

![](photo\22.png)

将一段动画的几帧图像拖到**Animation**界面中并适当调整播放频率即可。

![](photo\23.png)

将第一帧的图像在动画最后播放一次，防止动画出现 “割裂感”，这里应该还有别的解决办法，暂时还没学到，先这么弄。

同理，将其他几个行走方向的动画全都设置好。

全部弄好后，我们现在有一个**Animator Controller (enemy robot)** 和四段动画。

![](photo\24.png)

打开**enemy robot**，在**Base Layer**界面中将初始的几段动画删掉，新建一个混合树 - **Blend Tree**。

![](photo\25.png)

点开刚刚创建的混合树，我们希望用两个数值来控制敌人的动画，所以调整**Inspector > Blend Type** 为 **2D Simple Directional** 。

在**Parameters**界面中新建两个**Float**类型的变量 **X** 和 **Y**。

![](photo\27.png)

随后，在**Motion**界面中添加四个**Motion Field**，因为我们有上下左右四个方向的动画。并将**Parameters** 中的两个值更改为X和Y。

![](photo\26.png)

如图，将四段动画部署好，并修改 **Pos X** 和 **Pos Y** 的值。

实际运行中会自动播放与 **X** 和 **Y** 值最接近的一组 **Pos X** 和 **Pos Y** 所对应的动画。（看图理解，不太好说，但很好懂）

好了，相信不难理解，**X** 和 **Y** 在这里就代表敌人当前移动的方向了。那么如何得到 **X** 和 **Y** 的值呢？

当然是用代码来得到了：

```c#
	//部分代码
	Animator animator;
    void Start()
    {
        rigidbody2d = GetComponent<Rigidbody2D>();
        animator = GetComponent<Animator>();
    }

    void Update()
    {
        if(Isvertical)//判断移动方向
        {
            animator.SetFloat("Y", Ispositive);//将Ispositive的值写入animator的浮点数Y中（也就是我们设置的Y）
        }
        else
        {
            animator.SetFloat("X", Ispositive);
        }

        Times -= Time.deltaTime;
        if(Times < 0)
        {
            Ispositive = -Ispositive;
            Times = maxTimes;
        }
    }
```

我们之前用**Ispositive**来判断敌人的移动方向，**1**则正向移动，**-1**则负向移动，因此这里可以直接将当前**Ispositive**的值写入。

**SetFloat()**函数用来写入**animator**中的**float**类型变量。



现在运行游戏，你会发现可爱的小敌人可以行走了，不再像之前一样只会平移。



#### 为自机添加动画

既然可爱的小敌人已经有动画了，那我们玩家操控的角色当然也不能差。

为玩家部署动画则稍微复杂一点，因为玩家的移动方向是我们实时输入的，而不是像敌人那样有固定周期。而且玩家有待机状态，不会一直移动，这也就说明我们还要涉及到一些 “动画状态转移” 。

同样为角色创建一个**Animator**，并设置 **X，Y** 和 **speed** 三个值，因为我们需要通过 **speed** 来判断角色是行走还是待机。

然后为角色创建两个混合树，分别部署待机和行走时的动画，方式同上文，四个方向。

![](photo\28.png)

右键混合树，选择 **Make Transition**，设定两个混合树之间的转移方式。

![](photo\29.png)

将**Has Exit Time**取消掉，关于他的事以后再详细考虑，反正先取消掉。

在**Conditions**界面中设置转移的条件，我们这里以速度为分界线，速度小于**0.1**时，由行走转移到待机，速度大于**0.1**时，由待机转移到行走。

同样，需要编写角色的脚本来写入 **X，Y** 和 **speed**。



直接来看完整代码吧

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using Unity.VisualScripting;
using UnityEditor;
using UnityEngine;
using UnityEngine.SocialPlatforms;

public class saki : MonoBehaviour
{
    public int maxhealth = 5; //最大生命
    int CurrentHealth = 5; //当前生命
    public int health {get {return CurrentHealth;}} //只读，读取当前生命
    float horizontal;
    float vertical;
    Rigidbody2D rigidbody2d;

    public float Speed = 3.0f; //常规速度
    float currentSpeed = 3.0f; //当前速度
    float SpeedTimes = 0; //剩余加速时间

    bool BestTime = false; //是否无敌
    bool IsSpeed = false; //是否加速
    float Times = 0; //剩余无敌时间
    public float maxTimes = 2; //最长无敌时间

    Animator animator;
    Vector2 lookDirection = new Vector2(0, -1); //存储当前朝向
    void Start()
    {
        rigidbody2d =  GetComponent<Rigidbody2D>();
        animator = GetComponent<Animator>();
    }

    // Update is called once per frame
    void Update()
    {
        horizontal = Input.GetAxis("Horizontal");
        vertical = Input.GetAxis("Vertical");

        Vector2 move = new Vector2(horizontal, vertical);//存储当前帧的移动方向

        //如果move.x和move.y都是0，说明当前帧角色没有移动，则朝向与最后行走时的朝向保持不变
        //因为我们希望角色停止行走时，朝向会依然保持刚才行走时的朝向。
        if(!Mathf.Approximately(move.x, 0.0f) || !Mathf.Approximately(move.y, 0.0f)) //如果移动
        {
            lookDirection.Set(move.x, move.y); //改变朝向
            lookDirection.Normalize(); //归一存储，关于该函数的具体信息以后再说
        }
        
        animator.SetFloat("X", lookDirection.x); //将朝向写入animator中，以控制动画演出
        animator.SetFloat("Y", lookDirection.y);
        animator.SetFloat("speed", move.magnitude); //判断当前自机是否有速度

        if(BestTime) //如果当前处于无敌状态，则持续减少剩余无敌时间
        {
            Times -= Time.deltaTime;
            //剩余无敌时间归零，则取消无敌状态
            if(Times < 0) BestTime = false;
        }

        if(IsSpeed) //如果当前处于加速状态，则持续减少剩余加速时间
        {
            SpeedTimes -= Time.deltaTime;
            //剩余加速时间归零，则取消加速状态，并将速度恢复至常规速度
            if(SpeedTimes < 0)
            {
                currentSpeed = Speed;
                IsSpeed = false;
            }
        }

    }

    void FixedUpdate()
    {
        Vector2 position = rigidbody2d.position;
        position.x = position.x + currentSpeed * horizontal * Time.deltaTime;
        position.y = position.y + currentSpeed * vertical * Time.deltaTime;
        rigidbody2d.MovePosition(position);
    }

    public void ChangeHealth(int count)
    {
        if(count < 0) //如果是掉血，则检查是否处于无敌状态
        {
            if(BestTime) return; //处于无敌状态则不掉血

            BestTime = true; //不处于无敌状态，则掉血并进入无敌状态
            Times = maxTimes; //剩余无敌时间为当前最大无敌时间
        }
        if(count > 0) //如果是回血，则赋予加速状态
        {
            currentSpeed *= 1.5f;
            SpeedTimes = 1.5f;
            IsSpeed = true;
        }
        //Mathf.Clamp(x, min, max)函数，可以让x变量不会超出 min ~ max 范围
        CurrentHealth = Mathf.Clamp(CurrentHealth + count, 0, maxhealth);
        //产生调试信息
        Debug.Log(CurrentHealth + "/" + maxhealth);
    }
}

```



#### 动画过渡



![](photo\30.png)

拖拽如图所示的进度条可以更改动画过渡的时机，如上图，可以让过渡阶段先播放**hit**动画，而后在极短时间内完成过渡。

这里有很多细节知识，以后再补充，暂时先不做重点。



## 交互

迟到了十一天的笔记hhhhhhh

感觉unity的学习断断续续的，暑假得努力了。



#### 飞弹

（飞弹是课程里用的素材，我们这里其实发射的是魔法球）

这次来试试给自机添加一个攻击机制，自机发射魔法球，魔法球命中敌人后会使其停止运动，且无法再与其发生碰撞。

首先创建一个魔法球的预制件，先把素材拖到场景里，做成预制件再删掉就行了。

为魔法球添加脚本

```c#
public class power : MonoBehaviour
{
    Rigidbody2D rigidbody2d;
    void Awake()
    {
        rigidbody2d = GetComponent<Rigidbody2D>();
    }

    public void Launch(Vector2 direction, float force)
    {
        rigidbody2d.AddForce(direction * force); //给刚体施加一个direction方向，力度为force的力
    }

    void Update()
    {
        if(transform.position.magnitude > 30.0f) //如果与中心点的距离大于30就销毁魔法球
        {
            Destroy(gameObject);
        }
    }

    void OnCollisionEnter2D(Collision2D other)
    {
        enemyConsole now = other.collider.GetComponent<enemyConsole>(); //检验是否与敌人发生碰撞
        if(now != null)
        {
            now.Fix(); //在敌人的脚本里新加入Fix函数，让敌人停止运动并取消碰撞
        }
        Destroy(gameObject);
    }
}
```



为敌人添加受击后停止运动的脚本

```c#
 bool broken = false; //是否损坏，初始没有损坏
	public void Fix()
    {
        rigidbody2d.simulated = false; //取消碰撞
        broken = true; 
        animator.SetTrigger("ok"); //播放静止动画
    }
	void FixedUpdate()
    {
        if(broken) return; //如果损坏就直接返回，不进行移动
        Vector2 position = rigidbody2d.position;
        if(Isvertical) //是否纵向移动
        {
            position.y += 3.0f * Time.deltaTime * Ispositive;
        }
        else
        {
            position.x += 3.0f * Time.deltaTime * Ispositive;
        }

        rigidbody2d.MovePosition(position);
    }
```



好了，现在飞弹可以与敌人发生碰撞，且碰撞后敌人会停止运动了。

现在需要给自机添加可以发射飞弹的代码。

```c#
//创建一个GameObject类型的公共变量，我们保存的预制件就是这种类型，将Project中的魔法球预制件拖到Inspector内对应字段中
public GameObject powerPrefab;

//在Update末尾添加，检测是否输入C
if(Input.GetKeyDown(KeyCode.C))
        {
            Launch();
        }
//

void Launch()
    {
        GameObject MoFa = Instantiate(powerPrefab, rigidbody2d.position + Vector2.up * 0.5f, Quaternion.identity);
    	//Instantiate函数会在目标位置创建一个新的目标预制件，有三个参数，第一个参数为游戏对象，这里是我们的魔法球预制件,第二个参数是坐标，这里是自机的坐标并向上0.5个单位，第三个参数是旋转，这里不需要，暂时没进行了解，Quaternion.identity是不旋转
        power pow = MoFa.GetComponent<power>(); 
        pow.Launch(lookDirection, 300); //调用魔法球的发射函数，方向为当前自机的方向，力度为300

    }
```

![](photo\31.png)

前面说的预制件，因为定义了GameObject类型的公共变量，所以可以直接将预制件拖到这上面，Instantiate函数就可以直接调用魔法球预制件了，在这儿卡了好久才搞明白。
