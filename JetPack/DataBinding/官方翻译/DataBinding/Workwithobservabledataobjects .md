####使用可观察的数据对象

可观察性指的是一个对象通知其它对象它数据发生变化的能力。Data Binding库能让你使对象，域、以及对象变得可观察。

任何普通的对象都能用于data binding，但是修改对象并不会自动导致UI的更新。Data binding可以被用来给予你的数据对象通知其他对象的能力，例如，当它的数据发生改变时的监听器。有三种不同类型的可观察类：objects,fields以及collections。

当这些可观察数据对象绑定到界面，数据对象的一个属性发生变化，界面就会自动更新。

#####Observable fields

在创建实现Observable接口的类的时候是有一定工作量的，如果你的类中只有少量几个属性是得不偿失的。在这种情形下，你可以使用通用的可观察类以及以下的基础类来是域变得可观察：

* ObservableBoolean
* ObservableByte
* ObservableChar
* ObservableShort
* ObservableInt
* ObservableLong
* ObservableFloat
* ObservableDouble
* ObservableParcelable

可观察域是有单个域的自给自足的可观察对象。基础版本避免了访问操作的装箱和拆箱。为了使用这个机制，创建一个公开的final属性，如下例子所示：

	private static class User {
      public final ObservableField<String> firstName = new ObservableField<>();
      public final ObservableField<String> lastName = new ObservableField<>();
      public final ObservableInt age = new ObservableInt();
    }

为了访问域的值，使用set()和get()的存取方法，如下所示：

    user.firstName.set("Google");
    int age = user.age.get();
注意：在Android Studio 3.1以及更高版本上，你可以使用LiveData对象替代Observable域，它能够为你的应用带来额外的好处。想要知道跟多的信息，查看使用LiveDtat通知UI数据发生了变化。

#####Observalble 集合

一些应用使用动态结构来持有数据。可观察的集合允许通过key来访问这些结构。ObservableArrayMap类在key是一个引用类型的十分有用的，例如String，如以下的例子所示：

	ObservableArrayMap<String, Object> user = new     	ObservableArrayMap<>();
	user.put("firstName", "Google");
	user.put("lastName", "Inc.");
	user.put("age", 17);

在布局中，map可用用字符串key找到，如下：
	
	<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
    </data>
      …
    <TextView
    android:text="@{user.lastName}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
    <TextView
    android:text="@{String.valueOf(1 + (Integer)user.age)}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>

当key是整形的时候，ObservableArrayList是很有用的，如下：

	ObservableArrayList<Object> user = new ObservableArrayList<>();
    user.add("Google");
    user.add("Inc.");
    user.add(17);
在布局中，列表能够通过索引访问，如下例所示：

     <data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList<Object>"/>
     </data>
     …
    <TextView
    android:text='@{user[Fields.LAST_NAME]}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
    <TextView
    android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>   
    
 
 可观察对象
 ___
 
 实现Observable接口的类，允许注册当可观察对象的属性发生变化的监听器。为了使开发变得简单，DataBinding库提供了BaseObservable类，他实现了监听注册机制。这个实现BaseObservable的数据类是负责在属性改变的时候进行通知的。这是通过赋值一个Bindable对象到getter以及在setter对象中调用notifyProperytyChanged(),如下例所示：
 	  
 	private static class User extends BaseObservable {
    private String firstName;
    private String lastName;

    @Bindable
    public String getFirstName() {
        return this.firstName;
    }

    @Bindable
    public String getLastName() {
        return this.lastName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
        notifyPropertyChanged(BR.firstName);
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
        notifyPropertyChanged(BR.lastName);
    }
    }

在模块包中，Data binding 生成了一个名叫BR的类，这个类包含了用做数据绑定的资源id。Bindable 注解在编译时在BR类文件中生成了一个入口。如果用于数据的基础类不能发生改变，Observable 接口可以通过使用一个PropertyChangeRegistry对象来有效地注册和通知监听器的方式实现。

