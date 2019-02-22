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
	