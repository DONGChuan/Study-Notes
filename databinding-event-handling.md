# DataBinding 事件处理

Google 官方文档该部分有点扯淡, 英文表达十分模糊, 尤其对于我这样的初学者. 所以这里归纳总结一下.

DataBinding 可以指定绑定处理 view 的一些事件属性, 比如 onClick, onLongClick 之类.

没有 Databinding 时候, 在XML布局里给View设置点击事件:

```
android:onClick="onClickView"
```

这里的 onClickView 就是该布局里的一个公共方法:

```
public void onClickView(View view){
    // do something
}
```

这里, View 是在运行时，使用反射的方式从 Activity 找到 onClickView 方法并调用，这种做法相比于实现 View.onClickListener 并没有用到任何接口. 这里至少有 2 个缺点:

1. 由于是运行时反射, 如果方法名称错误或其他问题, 是没有办法在编译的时候就发现的. 运行时发现应用将直接崩溃.
2. 这种方式只能处理最简单的事件逻辑, 如果我想传递自定义参数呢?

> XML 里事件属性的名称通常是与相关监听器的名称相关联的, 比如, View.OnLongClickListener 有方法 onLongClick(), 所以事件属性的名称就是 android:onLongClick. 同理可得 android:onClick.

Databinding 则提供了 2 种不同的方式:

* 方法引用 (Method References)
* 监听绑定 (Listener Bindings)

#### 方法引用

顾名思义, 在 XML 里引用一个方法. 有点类似于上面提到的绑定一个 Activity 里的方法给 android:onClick. 但是最大的不同是在编译时就绑定了, 不用担心运行时报错.

```
public class MyHandlers {
    public void onClickFriend(View view) { ... }
}
```

The binding expression can assign the click listener for a View:

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.MyHandlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">

       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>

   </LinearLayout>
</layout>
```

#### 监听绑定

监听绑定就是绑定事件发生时要运行的公式. 和方法引用有点像, but they let you run arbitrary data binding expressions.

In method references, the parameters of the method must match the parameters of the event listener. In Listener Bindings, only your return value must match the expected return value of the listener (unless it is expecting void). 举个例子:

```
public class Presenter {
    public void onSaveClick(Task task){}
}
```

```
<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android">
      	<data>
         	<variable name="task" type="com.android.example.Task" />
          	<variable name="presenter" type="com.android.example.Presenter" />
      	</data>
      	<LinearLayout 
      		android:layout_width="match_parent" 
      		android:layout_height="match_parent">

          	<Button 
          		android:layout_width="wrap_content" 
          		android:layout_height="wrap_content"
          		android:onClick="@{() -> presenter.onSaveClick(task)}" />

      </LinearLayout>
  </layout>
```

Listeners are represented by lambda expressions that are allowed only as root elements of your expressions. When a callback is used in an expression, Data Binding automatically creates the necessary listener and registers for the event. When the view fires the event, Data Binding evaluates the given expression. As in regular binding expressions, you still get the null and thread safety of Data Binding while these listener expressions are being evaluated.

Note that in the example above, we haven't defined the view parameter that is passed into onClick(android.view.View). Listener bindings provide two choices for listener parameters: you can either ignore all parameters to the method or name all of them. If you prefer to name the parameters, you can use them in your expression. For example, the expression above could be written as:

```
android:onClick="@{(view) -> presenter.onSaveClick(task)}"
```

Or if you wanted to use the parameter in the expression, it could work as follows:

```
public class Presenter {
    public void onSaveClick(View view, Task task){}
}
```
```
android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```

You can use a lambda expression with more than one parameter:

```
public class Presenter {
    public void onCompletedChanged(Task task, boolean completed){}
}
```
```
<CheckBox 
    android:layout_width="wrap_content" 
    android:layout_height="wrap_content"
    android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```

If the event you are listening to returns a value whose type is not void, your expressions must return the same type of value as well. For example, if you want to listen for the long click event, your expression should return boolean.

```
public class Presenter {
    public boolean onLongClick(View view, Task task){}
}
```
```
android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
```

如果因为对象为 null 的原因, 公式没办法计算, DataBinding 就会根据 Java 返回值类型返回默认值. 比如, 引用类型就是 null , int 为 0, boolean 为 false for boolean 等等.

你也可以在监听绑定中使用三元公式或其他简单的逻辑:

```
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

### 两种方法的比较

* 方法引用要求响应方法的签名和事件listener中的签名完全一致，Listener Bindings只要求返回值一致。
* 方法引用在编译时绑定, 监听绑定在运行时才会执行绑定lambda表达式。
* 方法引用不可以自定义参数, 监听绑定可以自定义参数.
* 监听绑定可以编写简单的代码逻辑

### 参考

* [Google Databinding](https://developer.android.google.cn/topic/libraries/data-binding/index.html)
* [DataBinding学习使用进阶之路](https://www.jianshu.com/p/5d6132e6dc14)