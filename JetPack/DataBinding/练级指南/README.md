# DataBinding 练级指南
___
## DataBinding是什么？

DataBinding是一个自2015 Google IO 非C位出道的的一个框架。它的设计目的是把数据绑定到图形界面上去，而这里的界面指的正是你的布局文件。从Dan Galpin 在Android Dev Sumit 2018的rap般超快语速分享中获得灵感，让我们一起在DataBinidng世界中打怪升级。

## 初级阶段

### 挥别findViewById

首些你需要打开DataBinding这个功能，方法也很简单，在Module的Gradle文件的android元素下添加以下三行:
		
		dataBinding{
		  enabled = true
		}

然后进入你的布局文件，添加layout元素来它包裹你原有的根视图。在Android Studio中，当你把光标停留在根视图时，你会看到一个小灯泡，下拉选择Convert to data binding layout，就会帮你自动生成。

![小灯泡](https://raw.githubusercontent.com/jinyulei0710/kaixue-docs/master/JetPack/%E5%9B%BE%E7%89%87%E5%BA%93/light_bulb.png)

![生成结果](https://raw.githubusercontent.com/jinyulei0710/kaixue-docs/master/JetPack/%E5%9B%BE%E7%89%87%E5%BA%93/layout_element.png)

而绑定存在于经由DataBindingUtil inflate而来的这个绑定类中：

	@Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    ActivityLoginBinding binding=DataBindingUtil.setContentView(this, R.layout.activity_login);
    }

然后你就可以通过绑定类设置你的属性和监听器了：
    
    binding.name.setText("Ada");
    binding.lastname.setText("Lovelace");
    
但这是我们想要的绑定方式吗，显然不是，所以我们要迈入下一阶段。

### 拥抱绑定表达式

为了能写出绑定表达式，我们首先要在data区域中声明变量，从而让我们的布局能找到数据。
     <?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="loginPresenter"
            type="com.cjs.br.presenter.me.LoginPresenter" />

       ...
    </data>
       ...
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>



然后，我们就能在xml的属性中使用布局表达式语言了。表达式由大括号包围着，并且有着一个@符号作为前缀：
   
	<Element
	     attr="@{expression}/>

然后我们在代码中将我们引入布局的变量进行赋值：

    binding.setLoginPresenter(mPresenter);
    
    
经历了上述过程我们已经跨越了初级阶段，这一切就像变魔法一样。但是建国之后不许成精，所以请看本期走进科学，DataBinding魔法背后的故事。  

## 中级阶段

### 内置的绑定适配器(Binding Adapters)   

魔法的背后是DataBinding框架内置的绑定适配器，处理了几乎所有的事情。框架中的所有调用，实现上都是在BindingAdaper中完成的。

### 自定义的绑定适配器

### 可观察的数据

## 高级阶段

### 双向绑定  
