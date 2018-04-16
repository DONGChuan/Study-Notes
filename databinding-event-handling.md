# DataBinding 事件处理

Google 官方文档该部分有点扯淡, 英文表达十分模糊, 尤其对于我这样的初学者. 所以这里归纳总结一下.

DataBinding 可以指定绑定处理 view 的一些事件属性, 比如 onClick, onLongClick 之类.

没有 Databinding 时候, 在XML布局里给View设置点击事件:

```
android:onClick="onClickView"
```

这里的 onClickView 就是该布局所在的 Activity 里的一个公共方法:

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

通过这种方式引用的方法, 它必须是 public 并且有且只能有 View view 作为唯一参数, 否则必报 did not match signature 异常. 在 Google 官方文档里提到的 "Note that the signature of the method in the expression must exactly match the signature of the method in the Listener objec" 就是指方法引用的方法必须和 onClickListener (如果是用 android:onClick) 的 onClick 方法有一个样的 signature.

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        binding.setHandler(this);
    }

    public void clickView(View view){
        Toast.makeText(view.getContext(),"hello",Toast.LENGTH_SHORT).show();
    }
}
```

The binding expression can assign the click listener for a View:

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handler" type="com.example.MainActivity"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">

       <TextView 
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:onClick="@{handler::clickView}"/>

   </LinearLayout>
</layout>
```

> 这里既可以写 handler::clickView 也可以写 handler.clickView

#### 监听绑定

在方法引用里, 方法的参数必须符合事件监听器的参数. 在监听绑定中, 只需要你的方法的返回值符合事件监听器即可. 举个例子:

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

这里只需要保证 onSaveClick 方法的返回值为 void 即可. 不再需要强制写 View view, 且能自定义参数.

### 两种方法的比较

* 方法引用要求响应方法的签名和事件监听器中的签名完全一致，监听绑定只要求返回值一致。
* 方法引用在编译时绑定, 监听绑定在运行时才会执行绑定的 lambd a表达式。
* 方法引用不可以自定义参数, 监听绑定可以自定义参数.
* 监听绑定可以编写简单的代码逻辑

### 参考

* [Google Databinding](https://developer.android.google.cn/topic/libraries/data-binding/index.html)
* [DataBinding学习使用进阶之路](https://www.jianshu.com/p/5d6132e6dc14)