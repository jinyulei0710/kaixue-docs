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

     