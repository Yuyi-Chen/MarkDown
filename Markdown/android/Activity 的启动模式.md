Android 是通过任务栈进行 Activity 的管理的，栈是一种先进后出的数据结构，窗口显示的 Activity 是处于栈顶的 Activity，只有栈顶的 Activity 是可见的，当我们进入一个新的 Activity，这个 Activity 便会进入栈中并处于栈顶，当我们通过 back 键退出这个Activity，它便会出栈。

任务栈并不是唯一的，一般情况下（standard 模式），Activity 会进入启动它的那个 Activity 所在的栈中。

### Activity 的启动模式

Activity 目前有四种启动模式，分别是 standard，singleTop，singleTask 和 singleInstance，下面是各种启动模式的含义。

##### standard 标准模式

标准模式是系统的默认启动模式，即每次启动 Activity 时都创建一个新的实例，无论是否已经存在这个实例，它们都会走典型情况下 Activity 的生命周期，即 onCreate，onStart 和 onResume （**无论它是否已经存在**）。

##### singleTop 栈顶复用模式

在栈顶复用模式下，如果即将启动的 Activity 位于任务栈的栈顶，那么这个  Activity 不会被重新创建，也不会去走正常的生命周期，onCreate 和 onStart 不会被调用，但是会回调 onNewIntent 方法，这个方法的参数使我们新的请求信息，即 Intent。如果即将启动的 Activity 已经存在但是不在栈顶，那么这个Activity 还是会被重新创建。

##### singleTask 栈内复用模式

栈内复用模式是一种单例模式，在这种模式下，只要即将启动的 Activity 在栈内存在（**这个栈是它即将进入的栈**），这个 Activity 就不会被创建，系统会把这个 Activity （**已经存在的 Activity**）调到栈顶，并和 singleTop 一样回调 onNewIntent 方法，反之如果栈内不存在，这个 Activity 将会被创建并进入栈内。

需要注意的是，如果栈内存在这个 Activity，系统将它调到栈顶的方式是把它上面的所有 Activity 都出栈，即 clearTop。

例如当前任务栈的情况为 ADBC，D即将进入这个栈，由于这个栈内存在 D，所以它会被调到栈顶，此时栈内的情况为 AD。

##### singleInstance 单实例模式

单实例模式可以理解为是 singleTask （栈内复用模式） 的加强版，它除了拥有 singleTask 模式的所有特性外，还有另外一种特性，即 **具有此种模式的 Activity 只能单独位于一个任务栈中**，可能这样说有点不好理解。通俗的讲，系统在创建 singleTask 模式的 Activity 时，会为他单独创建一个任务栈，如果这个 Activity 被销毁，那么它的任务栈也会被随之销毁。

### 如何设置 Activity 的启动模式

合理的使用 Activity 的启动模式可以提升产品的性能，那么如何设置 Activity 的设计模式呢？

##### 通过 AndroidMainfest 为 Activity 设置启动模式

```Kotlin
<activity
    android:name=".activity.SplashActivity"          
    android:configChanges="screenLayout"
    android:launchMode="singleTask"
    android:label="@string/app_name" />
```

##### 通过 Intent 设置标志位为 Activity 指定启动模式

```kotlin
val intent = Intent()
intent.setClass(this, MainActivity::class.java)       
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)       
startActivity(intent)
```

虽然上述两种方法都可以为 Activity 设置启动模式，但是两种方法的优先级不同，第二种方法的优先级高于第一种方法，即当一个 Activity 同时被这两种方法设置了启动模式时，以第二种方式设置的为准。

