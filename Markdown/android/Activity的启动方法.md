在 Android 中，界面的跳转通常是通过启动不同的 Activity 来实现的，下面介绍一下 Activity 的启动方法。

## 显式调用

显式调用，字面意思即”明显的调用“，我们可以在调用方法中明确的知道我们即将启动的 Activity，显式调用的具体方法如下：

```kotlin
val intent = Intent(this, SecondActivity::class.java)
// SecondActivity::class.java 相当于Java中的 SecondActivity.class
startActivity(intent)
```

我们需要构建一个 Intent 对象，第一个参数传入 this 即当前 Activity 的上下文，第二个对象传入 SecondActivity::class.java 作为目标 Activity，这样我们的意图就很明显，即我们想跳转到 SecondActivity 这个界面，我们只需要调用 startActivity 这个函数就可以达到我们的目的了。

## 隐式调用

隐式调用与显式调用相反，它没有明确的说明要跳转到哪个 Activity，而是通过 action, category **[ˈkætəɡəri] ** 和 data 这三个过滤信息由系统去匹配复合条件的 Activity。

### 为 Activity 设置过滤信息

如果我们想要通过隐式调用去启动一个 Activity，我们首先要为这个 Activity 设置过滤信息，否则是不能通过隐式调用去启动这个 Activity 的。过滤信息在 AndroidMainfest.xml 文件中注册 Activity 时设置，通过在 \<activity\> 标签下配置\<intent-filter\> 的内容，可以指定当前Activity能够响应的action，category和data。

```xml
<activity
    android:name=".activity.SplashActivity"
    android:theme="@style/SplashTheme">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:host="room.join"
            android:scheme="bjhlliveapp" />
    </intent-filter>
</activity>
```

当使用隐式调用启动 Activity 时，需要同时匹配 action，category 和 data，否则匹配失败。一个 \<intent-filter\>下的 action，category 和 data 可以有多个，一个 activity 可以有多个 \<intent-filter\> 标签，只要成功匹配其中任意一个\<intent-filter\>中的信息即可匹配成功。

下面详细说明一下各种信息的匹配规则。

#### action

action 是一个字符串**(区分大小写)**，系统已经为我们预定义了一些 action，如 android.intent.action.MAIN 等，同时我们也可以自己定义 action，一个\<intent-filter\> 下可以有多个 action。

当我们使用隐式调用时 Intent 中必须指定 action，当 action 和 activity 的 \<intent-filter\> 中任意一个 action 相同**(字符串的值相同)**时，匹配成功。

#### category

category 和 action 一样是一个字符串，同时系统中有定义的 category，我们也可以自己定义 category，但是如果想让一个 activity 支持隐式调用，那么必须在\<intent-filter\> 中指定 **“android.intent.category.DEFAULT”** 这个 category。

category 的匹配规则与 action 不同，隐式调用时 Intent 中必须指定 action，但可以没有 category，但是如果指定了 category **(可以是一个或多个)**，那么所有指定的 category 都要和\<intent-filter\> 中的 category 相同，否则匹配失败。

用通俗的话来讲，category 你可以不指定**(如果不指定一定可以匹配成功)**，但是一旦你指定了，你就要保证你指定的这些 category 都要和你即将启动的 activity 中某一个 \<intent-filter\> 中的category 相同。

为什么不指定反而可以匹配成功呢？因为前面说了如果 activity 支持隐式调用，则一定要有 **“android.intent.category.DEFAULT”** 这个 category，系统在调用 startActivity 或者 startActivityForResult 的时候会默认为 Intent 加上 “android.intent. category.DEFAULT” 这个category，所以可以匹配成功。

#### data

data 的语法结构如下：

```xml
<data android:scheme="string"
      android:host="string"
      android:port="string"
      android:path="string"
      android:pathPattern="string"
      android:pathPrefix="string"
      android:mimeType="string" />
```

data 由两部分组成，mimeType 和 URI。mimeType 指媒体类型，比如image/jpeg、audio/mpeg4-generic 和 video/* 等，可以表示图片、文本、视频等不同的媒体格式，而URI中包含的数据就比较多了，下面是URI的结构：

```XML
<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
```

例如：

```xml
content://com.example.project:200/folder/subfolder/etc
http://www.baidu.com:80/search/info
```

URI 中每个数据的含义如下：

- Scheme：URI 的模式，比如http、file、content 等，如果 URI 中没有指定scheme，那么整个 URI 的其他参数无效，这也意味着 URI 是无效的。
- Host：URI 的主机名，比如 www.baidu.com，如果 host 未指定，那么整个 URI 中的其他参数无效，这也意味着 URI 是无效的。
- Port：URI 中的端口号，比如80，仅当 URI 中指定了 scheme 和 host 参数的时候 port 参数才是有意义的。
- path、pathPattern 和 pathPrefix：这三个参数表述路径信息，其中 path 表示完整的路径信息；pathPattern 也表示完整的路径信息，但是它里面可以包含通配符 “\*”，“\*” 表示0个或多个任意字符，需要注意的是，由于正则表达式的规范，如果想表示真实的字符串，那么“\*” 要写成“\\\*”, “\”要写成“ \\\\\\”；pathPrefix 表示路径的前缀信息。

data 的匹配规则较为复杂，但总的来说就是 Intent 中的 data 必须和 \<intent-filter\> 中的一致，如果 \<intent-filter\> 中没有，则 Intent 也没有，如果 \<intent-filter\> 中有，则 Intent 中必须有切必须和 \<intent-filter\> 中相同。

### 调用方法

```kotlin
val intent = Intent("android.intent.action.CPW")
// intent 构造函数指定 action
intent.addCategory("android.intent.category.DEFAULT")
// addCategory 方法指定category
intent.setDataAndType(Uri.parse("file://abc"), "text/plain")
// setDataAndType 方法指定data，参数为 URI 和 mimeType
startActivity(intent)
```

以上就是关于 Activity 启动方法的全部内容了！
