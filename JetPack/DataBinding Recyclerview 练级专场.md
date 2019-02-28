# DataBinding Recyclerview 练级专场

##RecyclerView和Data Binding

使用ViewHolder对于ListView来说是十分常见的，并且Recyclerview强制了这种模式。如果你看下DataBinding的生成结果，你会看到它实际上为你生成了ViewHolder.它含有域、它了解视图。在Recyclerview中你也可以简便的使用。我们创建了基础的create方法，它是一个静态方法，它描述了UserItemBinding()（由user item 布局文件生成的类）。所以你调用了UserItemBinding的inflate方法。然后现在，你有了一个十分简单的ViewHolder,它仅仅保有了一个被生成绑定类的引用，并且绑定方法是像这样的：

	  public class UserViewHolder extends RecyclerView.ViewHolder {
      static UserViewHolder create(LayoutInflater inflater, ViewGroup parent) {
    UserItemBinding binding = UserItemBinding
        .inflate(inflater, parent, false);
    return new UserViewHolder(binding);
      } 
      private UserItemBinding mBinding;
      private UserViewHolder(UserItemBinding binding) {
         super(binding.getRoot());
         mBinding = binding;
      }
     public void bindTo(User user) {
        mBinding.setUser(user);
       mBinding.executePendingBindings();
     }
    }
    
一个要注意的小细节是调用这个executePendingBindings。这是很重要的，因为当你数据刷新的时候，DataBinding实际上是在下一动画帧来了之后才去设置布局的。这就是没有立即批量处理所有发生在数据中的改变并将其应用的原因，但是Recyclerview却不喜欢。RecyclerView调用了BindView,它想让你准备好视图从图它能对布局进行测量。这就是为什么我要调用executePendingBinings的原因，从而DataBinding能冲刷掉所有待处理的改变。否则，它会创建另一个布局失效。视觉上你可能不会注意到，但是操作的时候会体现出来。

对于onCreateViewHolder，它仅仅调用第一个方法，同时onBind传递对象到了ViewHolder。就这样。我们不要写任何findViewById,没有设置等等。所有的东西都包含在你的布局文件当中。

		public UserViewHolder onCreateViewHolder(ViewGroup viewGroup, int i) {
      return UserViewHolder.create(mLayoutInflater, viewGroup);
      }

     public void onBindViewHolder(UserViewHolder userViewHolder, int position) {
      userViewHolder.bindTo(mUserList.get(position));
     }    

在上述代码中，我们展示了一个简单直接的实现。举个例子，你的用户对象的姓名发生了改变，绑定系统意识到了它并在下一动画帧的时候对其自身进行了重布局。下一动画帧开始，计算什么发生了改变，然后对TextView进行了更新。然后，TextView说，“好的，我的文本发生了改变，我现在想要重建布局，因为我不知道我的新的大小。让我们告诉RecyclerView,它的其中一个孩子不开心了，它需要进行重布局。当这个发生了，你不会再有任何动画效果了，因为你在所有事情发生之后才告诉RecyclerView。Recyclerview会尝试自己解决。结果就是没有动画，但是这不是我们想要的。

我们想要发生的是，当用户的对象被刷新，我们告诉适配器item发生了改变。相应的，他会告诉RecyclerView,“你好，你的一个孩子将要发生变化了，做好准备。然后，RecylerView会对那些孩子发生变化的进行布局，它会指导他们并他们进行重新绑定。当重新绑定的时候，TextView会说，“好的，我的text被设好了，我需要布局。”Recyclerview会说，“好的，不要担心，我已经把你放好了，让我对你进行测量。结果是：大量的动画。你会获得所有的动画效果，因为所有事情发生在RecyclerView的掌控之中。

##重绑定和装载

这是我们需要发布成库的一部分，但是与此同时，我想要你知道你可以如何完成这件事情。在DataBinding中我们有着这么一个Api，我们可以在其中添加重绑定回调。从根本上来说，它是一个你可以附加的回调，当DataBinding要计算的时候会得到通知。例如，捏可能要冻结变化到界面上去。你可以钩到这个onPreBind()上去，这时候你会获得一个布尔类型的繁殖值，你会说，“不，还没有到绑定的时候。如果其中一个监听器说了，Databinding就将要调用，“你好，我取消了重绑定，现在是你的责任来调用我了，因为我不会做任何事情。

现在所有我们要做的事情是，当Recyclerview没有计算你的布局的时候，返回false。当RecyclerView不在做你的计算的时候，你不应该更新的视图。这就是计算布局，这个新的RecyclerView API是在这个夏天发布的的。并且当onCancled()来的时候，我们就会告诉适配器，“你好，item发生了变化，去搞定它，因为我们已经从holder得知了它们的位置。然后，让RecyclerView做它想要做的。

			public UserViewHolder onCreateViewHolder(ViewGroup viewGroup, int i) {
    final UserViewHolder holder = UserViewHolder.create(mLayoutInflater,   
      viewGroup);
    holder.getBinding().addOnRebindCallback(new OnRebindCallback() {
    public boolean onPreBind(ViewDataBinding binding) {
      return mRecyclerView != null && mRecyclerView.isComputingLayout();
    }
    public void onCanceled(ViewDataBinding binding) {
      if (mRecyclerView == null || mRecyclerView.isComputingLayout()) {
        return;
      }
      int position = holder.getAdapterPosition();
      if (position != RecyclerView.NO_POSITION) {
        notifyItemChanged(position, DATA_INVALIDATION);
      }
    }
    });
    return holder;
    }     
   
之前，我们只有一个onBind方法，当你要装载一个列表的时候，所以我们开始写这个新的RecyclerView API。这是发生在ViewHolder上的一系列事情。最酷的事情是你接受到装载，并且只有接受到装载的时候，RecyclerView重新绑定到相同的视图。你知道，视图已经表示了相同的视图，但是要做一些改变。数据对我们送来的装载进行刷新。如果数据是因为DataBinding而来的，我们仅仅调用了executePendingBindings()。还记得我们不让它自己进行更新吗，现在是时候按RecyclerView所说的做吧。

如果你疑惑这像什么，DataBinding仅仅对要装载的东西进行遍历，进行检查来看数据刷新是不是唯一的装载。例如，另外有人发送了你不知道的装载，你应该忽略这些，应为你不知道这些变化是什么。

		public void onBindViewHolder(UserViewHolder userViewHolder, int position) {
         userViewHolder.bindTo(mUserList.get(position));
      }

    public void onBindViewHolder(UserViewHolder holder, int position,
      List<Object> payloads) {
              notifyItemChanged(position, DATA_INVALIDATION);
        ...
       }   
我们将会把它弄成一个库，因为它会给予你性能，动画，是所有的事情变得更好，并且是Recyclerview更开心。

Data 刷新仅仅是一个简单的对象，但是如果你好奇的话，我也展示下：

    static Object DATA_INVALIDATION = new Object();
      private boolean isForDataBinding(List<Object> payloads) { 
       if (payloads == null || payloads.size() == 0) {
    return false;
      }
      for (Object obj : payloads) {
        if (obj != DATA_INVALIDATION) {
        return false;
      }
     }
    return true;
    }
##多种视图类型

另一种DataBinding的事情情形就是多种视图类型。这经常发生：你有一个顶部视图，或者你有一个展示来自google搜索结果的应用，其中可能有图片和地点。你如何在RecyclerView里面进行构造呢？你有着一个使用一个变量的布局，你把这个变量叫做“data”。这个名称data是很重要的，一味你要重用这个名称。你使用的是一个常规的布局文件：

    <layout>
    <data>
        <variable name="data" type="com.example.Photo"/>
    </data>
    <ImageView android:src="@{data.url}" …/>
    </layout>

如果你需要另一种类型的结果，例如叫做“place”的，然后你就需要一个完全不同的布局，另一个xml文件。

两个布局文件唯一工程的这个叫做data的变量名称。当我们这么做的时候，我们创建了一个dataBoundViewHolder。
		 
		 public class DataBoundViewHolder extends RecyclerView.ViewHolder {
       private ViewDataBinding mBinding;
       public DataBoundViewHolder(ViewDataBinding binding) {
          super(binding.getRoot());
          mBinding = binding;
        }
       public ViewDataBinding getBinding() {
           return mBinding;
        }
         public void bindTo( Place place) {
             mBinding.setPlace(place);
             mBinding.executePendingBindings();
         }
        }    
        
这跟之前的例子是相同的实现。它是持有绑定的真正绑定对象。实际的数据绑定是所生成类的基础类。这就是你可以保有这引用的原因。我们创建了这个绑定方法，之前它是绑定到user的，现在是绑定到pleace。

不幸的事是，也存在一个问题。在Real数据绑定类中不存在setPlace()方法，因为它是基础类。作为替代，基础类提供了另一个API,根本上来说是setVariable: 

	public void bindTo( Object obj) {
       mBinding.setVariable(BR.data, obj);
       mBinding.executePendingBindings();
    }

你可以提供变量的标识，然后不管是什么你想要的类型，就像一个常规的java对象。生成的类会去检查这个类型并对其赋值。

设置变量的过程看起来是这样的，他从根本上来说，如果过去的id是我知道的id，对它进行类型转换，并对其赋值。

    boolean setVariable(int id, Object obj) {
        if (id == BR.data) {
          setPhoto((Photo) obj);
          return true;
          }
         return false;
      }         

一但你在onBind()中这么做了，onCreate()方法就是一样的。我们所做的就是getItemViewType,所以在view type中我们返回了布局id作为type的id。这十分有效，因为当我们返回布局id并切获取item的类型，RecyclerView将其传到onCreateViewHolder,他会通过DataBindingUtil来为其创建正确的绑定类。每个item都有它自己的布局，你不需要去放置对象。

		DataBoundViewHolder onCreateViewHolder(ViewGroup viewGroup, int type) {
       return DataBoundViewHolder.create(mLayoutInflater, viewGroup, type);
     }

    void onBindViewHolder(DataBoundViewHolder viewHolder, int position) {
       viewHolder.bindTo(mDataList.get(position));
     }

     public int getItemViewType(int position) {
        Object item = mItems.get(position);
       if (item instanceof Place) {
         return R.layout.place_layout;
      } else if (item instanceof Photo) {
    return R.layout.photo_layout;
       }
       throw new RuntimeException("invalid obj");
     }       
 
 当然，如果你在一个生产应用中这么写，你可能会保留实例检查的工作。你应该有一个类，这个类之后如何来会返回布局，但是你有一个通用的想法。
     