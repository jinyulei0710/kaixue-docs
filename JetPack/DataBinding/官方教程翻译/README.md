
# 数据绑定库 

数据绑定库是一个支持库，它允许你使用声明形式而不是以代码的形式来绑定用户界面组件到数据源上去。

布局在activity中通常是以调用了用户界面框架方法的代码形式定义的。例如，下述代码，调用了findViewById()来找寻一个TextView组件,并将它与viewModel变量的userName属性绑定起来:

      TextView textView = findViewById(R.id.sample_text);
      textView.setText(viewModel.getUserName());

下述的例子中展示了如何使用数据绑定库在布局文件中直接将文本赋值给控件。这就避免了调用以上所有Java代码。在赋值表达式中请注意@{}的使用:


    <TextView
        android:text="@{viewmodel.userName}" />
        
布局中的绑定组件，让你从你的activity中移除了许多用户界面框架的调用，使它们的维护更加简易。这也能帮助你提高你的应用的性能并且对避免内存泄漏和空指针异常也有帮助。    
   
## 使用数据绑定库

通过以下的文章来学习如何在你的应用中使用数据绑定库。

### [使用入门](https://github.com/jinyulei0710/kaixue-docs/blob/master/JetPack/DataBinding/%E5%AE%98%E6%96%B9%E6%95%99%E7%A8%8B%E7%BF%BB%E8%AF%91/%E4%BD%BF%E7%94%A8%E5%85%A5%E9%97%A8.md)

学习如何准备好你的开发环境，让它能使用数据绑定库，包括在Android Studio中对数据绑定代码的支持。

### [布局和绑定表达式](https://github.com/jinyulei0710/kaixue-docs/blob/master/JetPack/DataBinding/%E5%AE%98%E6%96%B9%E6%95%99%E7%A8%8B%E7%BF%BB%E8%AF%91/%E5%B8%83%E5%B1%80%E5%92%8C%E7%BB%91%E5%AE%9A%E8%A1%A8%E8%BE%BE%E5%BC%8F.md)

表达式语言允许你写表达式来将变量和布局中的视图关联起来。数据绑定库自动生成了绑定布局中视图和你的数据对象所需的类。这个库提供了例如imports，variables以及includes这些你能在布局中使用的特征。

这些库的特征和你已有的布局无缝共存。例如，能在表达式中使用的绑定变量是在data元素中定义的，而data是UI布局根元素的兄弟节点。两者都包含中layout标签中，如下例子所示：
    
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="viewmodel"
            type="com.myapp.data.ViewModel" />
    </data>
    <ConstraintLayout... /> <!-- UI layout's root element -->
   
    </layout>
    
### [使用可观察的数据对象](https://github.com/jinyulei0710/kaixue-docs/blob/master/JetPack/DataBinding/%E5%AE%98%E6%96%B9%E6%95%99%E7%A8%8B%E7%BF%BB%E8%AF%91/%E4%BD%BF%E7%94%A8%E5%8F%AF%E8%A7%82%E5%AF%9F%E7%9A%84%E6%95%B0%E6%8D%AE%E5%AF%B9%E8%B1%A1.md)

数据绑定库提供了类和方法，从而能很简单滴使观察数据的变化。当底层的数据源改变时，你不再需要关心刷新用户界面。你可以使你的变量或者它们的属性变成可观察的。这个库能允许你让对象(objects)，域(fields)，又或是集合(collections)变得可观察。

### [生成的绑定类](https://github.com/jinyulei0710/kaixue-docs/blob/master/JetPack/DataBinding/%E5%AE%98%E6%96%B9%E6%95%99%E7%A8%8B%E7%BF%BB%E8%AF%91/%E7%94%9F%E6%88%90%E7%9A%84%E7%BB%91%E5%AE%9A%E7%B1%BB.md)

数据绑定库设给你生成了用来访问布局的变量和视图的绑定类。这篇文章向你你展示了如何使用和自定义被生成的绑定类。

### [绑定适配器](https://github.com/jinyulei0710/kaixue-docs/blob/master/JetPack/DataBinding/%E5%AE%98%E6%96%B9%E6%95%99%E7%A8%8B%E7%BF%BB%E8%AF%91/%E7%BB%91%E5%AE%9A%E9%80%82%E9%85%8D%E5%99%A8.md)

对于所有的布局表达式，都存在着一个绑定适配器，这个适配器要做出必要的框架调用，对对应的属性或监听器进行设置。例如，绑定适配器能够调用setText()方法来对文本属性进行设置以及调用setOnClickListener()方法来为点击事件添加监听器。例如在这个例子中使用的android：text属性是最常见的绑定适配器存在于android.databinding.adapter包中。想要看绑定适配器的列表，看下[adapters](https://android.googlesource.com/platform/frameworks/data-binding/+/studio-master-dev/extensions/baseAdapters/src/main/java/androidx/databinding/adapters)这个页面。你也可以创建自定义的适配器，如下例所示：

    @BindingAdapter("app:goneUnless")
     public static void goneUnless(View view, Boolean visible) {
        view.visibility = visible ? View.VISIBLE : View.GONE;
    }

### [绑定布局视图到架构组件](https://github.com/jinyulei0710/kaixue-docs/blob/master/JetPack/DataBinding/%E5%AE%98%E6%96%B9%E6%95%99%E7%A8%8B%E7%BF%BB%E8%AF%91/%E5%B0%86%E5%B8%83%E5%B1%80%E8%A7%86%E5%9B%BE%E7%BB%91%E5%AE%9A%E5%88%B0%E6%9E%B6%E6%9E%84%E7%BB%84%E4%BB%B6.md)

包含架构组件在内的Android支持库，使用这个库你能够设计强壮的、可测试、可维护的应用。
你可以将架构组件与数据绑定库一同使用来进一步简化用户界面的开发。

### [双向数据绑定]()

数据绑定库支持双向数据绑定。用于此类绑定的表示法支持接收对属性的数据更改并同时侦听对该属性的用户更新的能力。


    
    
