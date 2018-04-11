# DataBinding 事件处理

DataBinding 也可以处理 view 的一些事件属性 (e.g. onClick). 

> 事件属性的名称通常是通过相关监听器的名称确定的, 比如, View.OnLongClickListener 有方法 onLongClick(), 所以事件属性的名称就是 android:onLongClick.

事件处理有2中方式

* 方法引用 (Method References)
* 监听绑定 (Listener Bindings)

#### 方法引用

类似于绑定一个 Activity 的方法给 android:onClick. 相比于 View#onClick 属性, 一个主要的优点就是在编译时就绑定了, 不用担心运行时报错. Activity 那种是运行时报错, 如果方法不存在或者方法名错误, 应用在运行的时候才会崩溃.

方法引用和监听绑定主要的不同就是, 

The major difference between Method References and Listener Bindings is that the actual listener implementation is created when the data is bound, not when the event is triggered. If you prefer to evaluate the expression when the event happens, you should use listener binding.

To assign an event to its handler, use a normal binding expression, with the value being the method name to call. For example, if your data object has two methods:

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

监听绑定就是绑定事件发生时要运行的公式. They are similar to method references, but they let you run arbitrary data binding expressions.

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