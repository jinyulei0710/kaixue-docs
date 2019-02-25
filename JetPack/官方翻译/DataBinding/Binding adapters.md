绑定适配器

绑定适配器负责在设置值的时候做出合适的系统调用。一个例子是设置一个属性值，这个值会调用setText()方法。另一个是设置事件监听，这个会调用setOnClickListener()方法。

数据绑定库允许你为一个值指定方法调用，提供你自己的绑定逻辑，以及指定使用适配器时要返回的类型。

###自动方法选择
	
对于一个叫做exmaple的属性，库自动地尝试去找出方法setExample(args),这个方法接受合适的类型作为参数。属性的命名空间不是考虑的内容，在搜索方法的时候只有属性名和类型是被使用的。

例如，android:text="@{user.name}"表达式，库查找的是setText(arg）方法，这个方法接收了由user.getName()返回的类型。如果user.getName()返回类型是String,库会查找接收String变量的setText()方法。如果表达式返回的是int，库会查找接收int参数的setText()方法。表达式必须返回正确的类型，必须的时候你可以对返回值进行强制转换。

DataBinding 在给出的名称的属性不存在的时候也能工作。你可以对所有的setter方法创建属性。例如，支持库类DrawerLayout没有一个属性，但是有大量的setter。以下的布局自动使用了setScrimColor(int)以及setDrawerListener(DrawListener)方法作为app:scrimColor以及app:drawerListener属性的setter。

	<android.support.v4.widget.DrawerLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:scrimColor="@{@color/scrim}"
    app:drawerListener="@{fragment.drawerListener}">	
###指定自定义的方法名称

一些属性有着跟名称不对应的setter。在这些情形中，属性和setter之间是用BindingMethods注解相关联的。注解在类中使用并可以包含许多BindingMethod注解，每个重命名的方法都有一个。Binding methods是能被添加到你应用中任意类的注解。在下例中，android:tint属性是和setImageTintList(ColorStateList)方法相关联的，而不是setTint()方法：

		@BindingMethods({
       @BindingMethod(type = "android.widget.ImageView",
                      attribute = "android:tint",
                      method = "setImageTintList"),
       })
	
绝大数情况下，你不需要重命名Android 框架类中的setter。属性已经按名称约束实现了，会自动找到对应的方法。

###提供自定义逻辑	

一些属性需要自定义的绑定逻辑。例如，没有针对android:paddingLeft属性的setter。作为替代，被提供的是setPadding(left,top,right,bottom)。有着BindingAdapter注解的静态绑定方法允许你在属性被调用的时候自定义setter。

Android框架类的属性已有有创建好的BindingAdapter。例如，以下的例子展示了针对paddingLeft的属性的绑定适配器。

    @BindingAdapter("android:paddingLeft")
    public static void setPaddingLeft(View view, int padding) {
    view.setPadding(padding,
                  view.getPaddingTop(),
                  view.getPaddingRight(),
                  view.getPaddingBottom());
    }
    
参数类型是十分重要的。首个参数决定了属性相关的视图的类型。第二个参数决定了给定参数在绑定表达式中接收的类型。

绑定适配器对其他类型的自定义是十分有用的。例如，一个自定义的加载器可以从工作线程加载图片。

当存在冲突的时候，你自定义的适配器会重载由Android框架提供的默认适配器。

你可以有接收多个属性的适配器，如下例所示：
      
     @BindingAdapter({"imageUrl", "error"})
       public static void loadImage(ImageView view, String url, Drawable error) {
     Picasso.get().load(url).error(error).into(view);
    }
  
你可以像下例一样在你的布局中使用适配器。注意，@drawable/vennuErroe指向的是你应用中的资源。将资源包裹在@{}中，使其成为一个合法的绑定表达式。

		<ImageView app:imageUrl="@{venue.imageUrl}" app:error="@{@drawable/venueError}" />

注意：处于匹配的目的，数据绑定库忽略了自定义的命名空间。

这个适配器在imageUrl和error同时被用到ImageView对象的时候会被使用，imageUrl是一个字符串并且error是一个Drawable。如果你想要任一的属性设置之后来让适配器被调用，你可以把适配器的可选的requireAll标记设置成false,如下例所示：
   
      @BindingAdapter(value={"imageUrl", "placeholder"}, requireAll=false)
     public static void setImageUrl(ImageView imageView, String url, Drawable placeHolder) {
       if (url == null) {
    imageView.setImageDrawable(placeholder);
      } else {
    MyImageLoader.loadInto(imageView, url, placeholder);
     }
      }
		
注意当有冲突的时候，你的绑定适配器会重载默认的绑定适配器。

绑定适配器方法可能会把旧的值带到他们的handler中。一个接收旧和新的值的方法，应该先声明旧的值，然后才是新的值，如下例所示：

		@BindingAdapter("android:paddingLeft")
     public static void setPaddingLeft(View view, int oldPadding, int newPadding) {
    if (oldPadding != newPadding) {
      view.setPadding(newPadding,
                      view.getPaddingTop(),
                      view.getPaddingRight(),
                      view.getPaddingBottom());
     }
    }		

事件处理可能只能在只有一个抽象方法的接口或抽象类中使用，如下例所示：

		@BindingAdapter("android:onLayoutChange")
     public static void setOnLayoutChangeListener(View view, View.OnLayoutChangeListener oldValue,
       View.OnLayoutChangeListener newValue) {
       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
    if (oldValue != null) {
      view.removeOnLayoutChangeListener(oldValue);
    }
    if (newValue != null) {
      view.addOnLayoutChangeListener(newValue);
    }
     }
     }
按如下所示在你的布局中使用这个事件处理器：

 		<View android:onLayoutChange="@{() -> handler.layoutChanged()}"/>
当一个监听器有多个方法的时候，它必须划分为多个监听器。例如,View.OnAttachStateChangeListener有着两个方法：OnViewAttachedToWindown(View)以及onViewDetachedFromWindow(View)。这个库提供了两个接口来区分它们的属性和handler。

	@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
    public interface OnViewDetachedFromWindow {
       void onViewDetachedFromWindow(View v);
     }

     @TargetApi(VERSION_CODES.HONEYCOMB_MR1)
     public interface OnViewAttachedToWindow {
       void onViewAttachedToWindow(View v);
       } 
 
 因为改变一个监听器的同时也会影响另一个，你需要一个对单个同时两个属性都好用的适配器。你可以在注解在把requireAll设置成false来制定不是每个属性都必须在绑定表达式中被赋值，如下例所示：
 
 			@BindingAdapter({"android:onViewDetachedFromWindow", 			"android:onViewAttachedToWindow"}, requireAll=false)
		public static void setListener(View view, OnViewDetachedFromWindow detach, OnViewAttachedToWindow attach) {
    if (VERSION.SDK_INT >= VERSION_CODES.HONEYCOMB_MR1) {
        OnAttachStateChangeListener newListener;
        if (detach == null && attach == null) {
            newListener = null;
        } else {
            newListener = new OnAttachStateChangeListener() {
                @Override
                public void onViewAttachedToWindow(View v) {
                    if (attach != null) {
                        attach.onViewAttachedToWindow(v);
                    }
                }
                @Override
                public void onViewDetachedFromWindow(View v) {
                    if (detach != null) {
                        detach.onViewDetachedFromWindow(v);
                    }
                }
            };
        }

        OnAttachStateChangeListener oldListener = ListenerUtil.trackListener(view, newListener,
                R.id.onAttachStateChangeListener);
        if (oldListener != null) {
            view.removeOnAttachStateChangeListener(oldListener);
        }
        if (newListener != null) {
            view.addOnAttachStateChangeListener(newListener);
        }
    }
}
      
以上的例子跟常见的相比要稍微复杂一些，因为View类使用的是addOnAttachStateChangeListener（）以及removeOnAttachStateChangeListener()而不是OnAttachStateChangeListener的setter方法。android.databinding.adapter.ListenerUtil类帮助记录了之前监听器的记录，所以他们能在绑定适配器中被移除。

在接口OnViewDetachedFromWindow以及OnViewAttachedToWindow添加@TargetApi(VERSION_CODES.HONEYCOMB_MR1注解，数据绑定代码生成器知道监听器只在运行在Android 3.1(API level 12)以及更高版本的时候被生成，与addOnAttachStateChangeListener()方法支持的版本一致。

####自动对象转换

在一些情形中，在特有的类型之间，一个自定义的转换是需要的。例如，视图的android:background属性期待的是Drawable,但是color值指定的是一个整形。以下的例子展示的是属性期待的是Drawable但是，提供的却是integer的值。

	<View
    android:background="@{isError ? @color/red : @color/white}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
    
当期待的是Drawable，但是返回的却是integer的时候，int应该被转换成ColorDrawable。这个转换过程可以用一个带有BindingConversion注解的静态方式完成，以下：

	@BindingConversion
    public static ColorDrawable convertColorToDrawable(int color) {
       return new ColorDrawable(color);
    } 

但是，在绑定表达式中提供的值类型必须是一致的。你不能在相同的表达式中使用不同的类型，如下所示：
	
	<View
     android:background="@{isError ? @drawable/error : @color/white}"
     android:layout_width="wrap_content"
     android:layout_height="wrap_content"/>  