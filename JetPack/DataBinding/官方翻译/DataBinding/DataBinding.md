
###DataBinding 

DataBinding 库是一个支持库，它允许你使用声明形式而不是以代码的形式来绑定UI组件到数据资源上去。

布局在activity中通常是以代码形式定义的，这个代码调用了UI框架的方法。例如，以下的代码调用了findViewById来找寻一个TextView组件并将它绑定到viewModel变量的userName属性：


      TextView textView = findViewById(R.id.sample_text);
      textView.setText(viewModel.getUserName());

以下的例子展示了如何使用DataBinding库在布局文件中直接将文本赋值给控件。这就避免调用以上所有Java代码。在赋值表达式中请留意@{}:

    <TextView
    android:text="@{viewmodel.userName}" />
    
#### 使用Data Binding库

使用以下各页来学习如何在您的应用中使用Data biding 库

##### 开始

学习在使用Data Binding 库之前开发环境要做的准备，包括在Android Studio中支持DataBinding。

数据绑定库提供了灵活性和广泛的兼容性，它是一个支持库，所以你可以在运行Android 4.0(API等级14)以及更高的设备上使用它。

推荐在你的项目中使用最新的Android Gradle插件。但是数据绑定是在1.5.0及以上版本被支持的。

######构建环境
___

要开始使用数据绑定，现在从Android Sdk管理器中下载库。

要配置你的应用来使用数据绑定，在你的app模块的build.gradle文件中添加dataBinding元素额，如下例所示：

	 android {
     ...
     dataBinding {
        enabled = true
        }
    }
    
注意：你必须为依赖于使用数据绑定库的app模块配置数据绑定，即使app模块没有直接使用数据绑定。

###### Android Studio 对于数据绑定的支持
___

Android Studio支持许多对于数据绑定代码的编辑特性。例如，它支持以下对于数据绑定表达式的特性：

* 表达式高亮
* 标记表达式语法错误
* XML代码补全
* 引用，包含导航(例如导航到声明)以及快速文档
  
布局编辑器中的预览面板展示绑定表达式的默认值，如果提供了的话。例如，在下面的例子中，预览面板在TextView控件上展示了my_default这个值：

    <TextView android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.firstName, default=my_default}"/>
    
如果你需要在你产品的设计阶段展示一个默认值，你可以使用tools属性而不是一个默认的表达式值。    

######新的用于绑定类的数据绑定编译器
___

Android Gradle插件版本3.1.0-alpha06包含了一个新的用来生成绑定类的数据绑定编译器。
新的编译器是增量创建绑定类的，这在大部分情形下加速了构建过程。想要学习更多关于绑定类的内容，查看生成的绑定类。

之前版本的数据绑定编译器是按编译托管代码的相同方式来生成绑定类的。如果您的托管代码编译失败，你可能会收到绑定类没有找到的错误报告。新的绑定编译器通过在托管编译器编译你的应用之前生成绑定类的方式来避免这些错误。

要打开新的数据绑定编译器，添加以下选项到你的gradle.properties文件中：
		
		android.databinding.enableV2=true

你也可以通过添加以下参数的方式在您的gralde命令中打开新的编译器：

      -Pandroid.databinding.enableV2=true

注意：在Android插件版本3.1中的心的数据绑定编译器不是向后兼容的。你需要打开这个特性生成所有你的绑定类之后才能使用增量编译。但是，在Android 插件版本3.2中适合之前版本生成的绑定类兼容的。新的编译器在版本3.2中是默认打开的。

当你打开新的数据绑定编译器之后以下行为改变就生效了：

* Android Gradle插件在您编译你的托管代码之前为你的布局生成绑定类。
* 如果你的布局在不止一个目标资源配置中被包含，数据绑定库使用Android.view.View作为共享相同资源id但不是相同视图类型的所有视图的视图类型。
* 库模块的绑定类是被编译打包金对应的Android 压缩文件的。依赖于这些库模块的App模块不再需要生成绑定类。
* 模块的绑定适配器不在改变模块依赖的适配器的行为。绑定适配器只影响它自己所属模块以及使用这个模块的代码。



##### 布局和绑定表达式

表达式语言允许你书写表达式来将变量和布局中的视图关联起来。DataBinding库自动生成了绑定布局中视图和你的数据对象所需的类。库提供了例如imports，variables以及includes这些你能在你的布局中使用的特性。

这些库的特性和你已有的布局无缝共存。例如，能在表达式中使用的绑定变量是在data元素中定义的，data是UI布局根元素的兄弟。两者都包含中layouttag中，如下例子所示：
    
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="viewmodel"
            type="com.myapp.data.ViewModel" />
    </data>
    <ConstraintLayout... /> <!-- UI layout's root element -->
   
    </layout>
    
    
    

##### 使用可观察的数据对象

DataBinding 库提供了类和方法使观察数据的变化变得简单。当下层的数据源改变的时候，你不再需要关心刷新UI。你可以使您的变量和它们的属性变成可观察的。这个库让你能够让对象，域，或者集合变得可观察。

##### 生成的绑定类

Data Binding 库设给你生成了用来访问布局的变量和视图的绑定类。这页向你展示了如何使用和自定义生存的绑定类。

##### 绑定适配器

对于所有的布局表达式，都有这一个对应的绑定适配器，这个适配器使框架调用需要设置对应的属性和监听器。例如，绑定适配器能接管setText（）方法的调用来设置文本属性以及调用setOnClickListener（）方法来为点击事件添加监听器。例如在这个例子中使用的android：text属性是最常见的绑定适配器，它是供你使用的android.databinding.adapter的一部分。想要看常见绑定适配器的类别，看下adapters这页。你也可以自定义的适配器，如下图所示：

    @BindingAdapter("app:goneUnless")
     public static void goneUnless(View view, Boolean visible) {
    view.visibility = visible ? View.VISIBLE : View.GONE;
    }

##### 绑定布局视图到架构组件

包含架构组件在内的Android支持库，这个库你能用来设计强壮的、可测试、可维护的应用。
你可以将架构组件与数据绑定库一同使用来进一步简化界面的开发。

##### 双向数据绑定

数据绑定库支持双向数据绑定。这个标记意味着，这个类型的绑定支持接受数据改变到某一属性的能力以及同时监听用户对这个属性的改变。


    
    