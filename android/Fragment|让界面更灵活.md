# Fragment | 让界面更灵活

Fragment 从 Android 3.0 版本开始引入，它可以称为“迷你版”的 Activity，在 Android 开发中可以让我们的界面更加灵活。

Fragment 允许将界面划分为零散的区块，它可以让你的界面模块化，并且它可以重复使用。但是 Fragment 不能独立存在，它必须由 Activity 或 另一个 Fragment 托管，Fragment 的视图层次结构会成为宿主的视图层次结构的一部分，或附加到宿主的视图层次结构。

## 如何创建一个 Fragment

当我们想要创建一个 Fragment，我们只需要创建一个类并让这个类继承 Fragment，然后重写它的 onCreateView 方法加载它的布局即可。

```kotlin
class MyFirstFragment : Fragment() {
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_my_first, container, false)
    }
}
```

或者使用更简单的方法：

Kotlin：

```kotlin
class MyFirstFragment : Fragment(R.layout.fragment_my_first) {
    
}
```

Java：

```java
class MyFirstFragment extends Fragment {
    public MyFirstFragment() {
        super(R.layout.fragment_my_first);
    }
}
```

这样就可以创建出一个 Fragment 了，你还可以在类中定义其它方法去处理事务。

## 添加 Fragment

Fragment 是不能单独存在的，必须依附于 Activity 或另一个 Fragment，下面介绍一下添加 Fragment 的两种方法。

### 静态添加

静态添加需要在宿主的布局中添加一个 fragment 标签并指定 android:name 为你要添加的 Fragment 的类名(**包括类的包名**)。

```xml
<fragment
        android:id="@+id/my_first_fragment"    //id 是必需的
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:name="com.cpw.learningapplication.MyFirstFragment"/>
```

这样这个 Fragment 就成功的添加进去了。

### 动态添加

动态添加即通过代码进行添加，而不是像静态添加那样写死，实现 Fragment 的动态添加需要以下五步：

> 1. 创建待添加的 Fragment 的实例。
>
> 2. 获取 FragmentManager，在 Activity 中可以直接调用 getSupportFragmentManager() 方法获取。
>
> 3. 开启一个事务，通过调用 beginTranscation() 方法开启。
>
> 4. 向容器内添加或替换 Fragment，一般使用 replace() 方法实现，需传入容器的 id 和待添加 Fragment 的实例。
>
> 5. 提交事务，调用 commit() 方法来完成。

代码如下:

```kotlin
val myFirstFragment = MyFirstFragment() //第一步
val manager = supportFragmentManager //第二步
val transaction = manager.beginTransaction() //第三步
transaction.replace(R.id.my_first_fragment, myFirstFragment) //第四步
transaction.commit() //第五步
```

当 Activity 处于 `STARTED` 生命周期状态或更高的状态时，都可以通过上述方法添加( **add( )**)、替换( **replace( )**)或移除(  **remove( )**) Fragment。

### 实现返回栈

通常情况下，我们向 Activity 中添加多个Fragment 后，按返回键会退出这个 Activity，那如果我们想实现按返回键返回到上一个 Fragment 的效果，我们该怎么做呢？

FragmentTransaction 中提供了一个 addToBackStack( ) 方法，用于将一个事务添加到返回栈中，我们可以将上面的代码改写为：

```kotlin
val myFirstFragment = MyFirstFragment()
val manager = supportFragmentManager        
val transaction = manager.beginTransaction()
transaction.replace(R.id.my_first_fragment, myFirstFragment)
transaction.addToBackStack(null) //参数接收一个名字用于描述返回栈，一般传null
transaction.commit()
```

## Fragment 与 Activity 的交互

既然 Fragment 是依附于 Activity 存在的，那么 Activity 和 Fragment 的交互就是必需的，但是 Fragmemt 和 Activity 是两个独立的类，它们要怎么进行交互呢？

当我们使用动态添加的方法添加 Fragment 时，我们创建了添加的 Fragment 的实例，我们可以通过这个实例直接调用 Fragment 中的方法，那使用静态方法添加时怎么获取这个 Fragment 的实例呢？为了方便 Fragment 和 Activity 进行交互，FragmentMananger 提供了一个类似于 findViewById( ) 的方法，专门从布局文件中获取 Activity 的实例，代码如下：

```kotlin
val fragment = supportFragmentManager.findFragmentById(R.id.my_first_fragment) as MyFirstFragment
```

上面的方法可以让我们在 Activity 中轻松调用 Fragment 中的方法，那么我们在 Fragment 中又该如何去调用 Activity 的方法呢？

在 Fragment 中可以通过调用 getActivity( ) 方法得到和当前 Fragment 相关联的 Activity。

```kotlin
if (activity != null) {
    val mainActivity = activity as MainActivity
}
```

## Fragment 的生命周期

Fragement 的生命周期与 Activity 极为相似，也会经历以下几种状态：

> - 运行状态
>
>   当一个 Fragment 所关联的 Activity 正处于运行状态时，该 Fragment 也处于运行状态。
>
> - 暂停状态
>
>   当一个 Activity 进入暂停状态时（由于另一个未占满屏幕的 Activity 被添加到了栈顶），与它相关联的 Fragment 就会进入暂停状态。
>
> - 停止状态
>
>   当一个 Activity 进入停止状态时，与它相关联的 Fragment 就会进入停止状态，或者通过调用 FragmentTransaction 的 remove()、replace() 方法将 Fragment 从 Activity 中移除，但在事务提交之前调用了 addToBackStack() 方法，这时的 Fragment 也会进入停止状态。总的来说，进入停止状态的 Fragment 对用户来说是完全不可见的，有可能会被系统回收。
>
> - 销毁状态
>
>   Fragment 总是依附于 Activity 而存在，因此当 Activity 被销毁时，与它相关联的 Fragment 就会进入销毁状态。或者通过调用 FragmentTransaction 的 remove()、replace() 方法将 Fragment 从 Activity 中移除，但在事务提交之前并没有调用 addToBackStack() 方法，这时的 Fragment 也会进入销毁状态。

Fragment 完整的生命周期如下图：

<img src="https://developer.android.google.cn/images/guide/fragments/fragment-view-lifecycle.png" alt="Fragment 的生命周期（来自Android开发者文档）" style="zoom:67%;" />

在 onCreate( ) 之前和 onDestroy( ) 之后分别会调用  onAttach( ) 和 onDetach( ) 方法，表示和 Activy 建立\解除关联。

## Fragment 的数据传递

Activity 向 Fragment 传递数据是有很多种方法的，因为 Activity 拥有 Fragment 的实例，它可以调用 Fragment 中所有的公有函数来达到数据传递的目的。除此之外，在将 Fragment 添加到 Activity 之前，可以调用 Fragment 的 setArguments( ) 方法向 Fragment 传递数据，然后在 Fragment 中通过调用  getArguments( ) 解析数据，它们传递的是一个 Bundle 对象。一般推荐使用 arguments 来传递数据，这是因为**当 Fragment 销毁重建时，使用 arguments 传递的数据不会丢失，而其他方法传递的数据在没有经过存储时是会丢失的**。

Fragment 向 Activity 传递数据通常使用接口，在 Fragment 中定义接口，然后在 Activity 中实现接口，从而 Fragment 在需要传出数据时可以调用接口方法将数据传出。

具体代码如下：

```kotlin
// 在Fragment中定义接口
interface dataListener{
  fun onDataReceive(data: String)
}


//在Activity中实现接口
class SecondActivity : AppCompatActivity(), MyFirstFragment.dataListener {
  
    override fun onDataReceive(data: String) {
        Toast.makeText(this, data, Toast.LENGTH_SHORT).show()
    }
  
}


// 在 Fragment 中调用接口方法
fun setData(data: String) {
  if (activity != null) {
    val myDataListener = activity as dataListener
    myDataListener.onDataReceive(data)
  }
}
```

## 实践，用 Fragment 实现 Android 底部导航栏

在 Android 应用中，底部导航栏是随处可见的，例如我们经常使用的微信、QQ、支付宝等，都有底部导航栏的身影，实现底部导航栏的方式也有很多种，这里我使用 Fragment 和其它基础的 View 来实现一个简单的底部导航栏。

Activity:

```kotlin
package com.cpw.learningapplication

import android.os.Bundle
import android.view.MotionEvent
import android.view.MotionEvent.ACTION_DOWN
import android.view.MotionEvent.ACTION_MOVE
import android.widget.LinearLayout
import androidx.appcompat.app.AppCompatActivity
import kotlinx.android.synthetic.main.activity_second.*
import kotlin.math.abs

class SecondActivity : AppCompatActivity() {
    private lateinit var myFirstFragment: MyFirstFragment
    private lateinit var mySecondFragment: MySecondFragment
    private lateinit var myThirdFragment: MyThirdFragment
    private var currentFragmentIndex: Int = 0


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)
        myFirstFragment = MyFirstFragment()
        mySecondFragment = MySecondFragment()
        myThirdFragment = MyThirdFragment()
        selectButton(button_1)
        button_1.setOnClickListener {
            selectButton(button_1)
        }

        button_2.setOnClickListener {
            selectButton(button_2)
        }

        button_3.setOnClickListener {
            selectButton(button_3)
        }

    }

    private fun selectButton(button: LinearLayout) {
        val currentFragment = when (button.tag.toString()) {
            "button_2" -> {
                currentFragmentIndex = 2
                mySecondFragment
            }
            "button_3" -> {
                currentFragmentIndex = 3
                myThirdFragment
            }
            else -> {
                currentFragmentIndex = 1
                myFirstFragment
            }
        }
        button_1.isSelected = currentFragmentIndex == 1
        button_2.isSelected = currentFragmentIndex == 2
        button_3.isSelected = currentFragmentIndex == 3
        supportFragmentManager.beginTransaction()
                .replace(R.id.fragment_container, currentFragment)
                .commit()
    }
}
```

三个 Fragment 除了背景颜色和文字不同，其他的都一样。效果如下：

![20210819_161922](/Users/administrator/Desktop/20210819_161922.gif)

我们甚至可以通过处理滑动事件来实现左滑和右滑切换页面的功能，只需向 Activity 添加以下方法：

```kotlin
private fun toLeft() {
        if (!isMove) {
            when (currentFragmentIndex) {
                1 -> selectButton(button_2)
                2 -> selectButton(button_3)
            }
            isMove = true
        }
    }

    private fun toRight() {
        if (!isMove) {
            when (currentFragmentIndex) {
                2 -> selectButton(button_1)
                3 -> selectButton(button_2)
            }
            isMove = true
        }
    }

    private var oldX = 0
    private var currentX = 0
    private var isMove = false

    override fun dispatchTouchEvent(ev: MotionEvent?): Boolean {

        when(ev?.action) {
            ACTION_DOWN -> {
                isMove = false
                oldX = ev.x.toInt()
            }
            ACTION_MOVE -> {
                currentX = ev.x.toInt()
                var delta = currentX - oldX
                if (abs(delta) > 300 && delta > 0) {
                    toRight()
                } else if (abs(delta) > 300 && delta < 0) {
                    toLeft()
                }
            }
        }
        return super.dispatchTouchEvent(ev)
    }
```

需要注意的是，如果你的页面内含有其他可以滑动的部件，你可能需要处理滑动冲突。

滑动效果：

![20210819_162511](/Users/administrator/Desktop/20210819_162511.gif)

>欢迎微信搜索关注公众号：**Android没有烦恼**
>
>一起学习更多Android知识

