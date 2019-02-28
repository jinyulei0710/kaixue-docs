# Android 数据绑定 Recyclerview

虽然有时我喜欢这样认为，数据绑定作为一个术语并不总是意味着Android数据绑定。RecyclerView有着它自己绑定数据到UI上的方式。RecyclerView有着一个带有两个非常重要方法的适配器，这两个方法对于实现绑定数据是十分重要的。

	 RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent,
                                           int viewType);

     void onBindViewHolder(RecyclerView.ViewHolder holder, int position);

RecyclerView暴露常见的ViewHolder模式作为它的API的首位公民。在onCreateViewHolder()中，视图被创建了并且ViewHolder包含了对视图的引用，所以视图能够被很快设置。然后在onBindView()中，特定的数据被赋值到了视图上。

##RecyclerView中的Android DataBinding

正如在之前的一篇文章中所描述了，Android Data Binding 能够被当作ViewHolder模式。
理想情况下，我们仅仅需要从我们的onCreateViewHolder()中返回生成的绑定类，但是它没有继承RecyclerView.ViewHolder。所以，绑定类换而由ViewHolder含有。
		
	public class MyViewHolder extends RecyclerView.ViewHolder {
    private final ItemBinding binding;

    public MyViewHolder(ItemBinding binding) {
        super(binding.getRoot());
        this.binding = binding;
    }

    public void bind(Item item) {
        binding.setItem(item);
        binding.executePendingBindings();
    }
}

现在，我们的适配器就能使用Android DataBinding来进行创建和绑定了：

	public MyViewHolder onCreateViewHolder(ViewGroup parent,
                                       int viewType) {
    LayoutInflater layoutInflater =
        LayoutInflater.from(parent.getContext());
    ItemBinding itemBinding = 
        ItemBinding.inflate(layoutInflater, parent, false);
    return new MyViewHolder(itemBinding);
    }

    public void onBindViewHolder(MyViewHolder holder, int position) {
         Item item = getItemForPosition(position);
        holder.bind(item);
     }
如果你看的够仔细的话，在MyViewHolder.bind()方法的末尾你会看到executePendingBinds()这个方法。这强制绑定必须立即执行而不是延迟到下一帧。Recyclerview会在onBindViewHolder之后立即对视图进行measure。如果绑定到等到下一帧，错误的数据就会出现在视图中，它就不能正确地measure。所以，executePendingBinds()是很重要的！

##重用ViewHolder
如果你之前使用过RecyclerView的ViewHolder的话，你会知道我们已经节省了一大堆的模版代表，因为这些被设置到了视图当中了。不幸的事情是，我们仍旧要写一堆的ViewHolder来针对不同的RecyclerView.对于多种不同的视图类型的情况，如何去继承这个ViewHolder也是不清晰的。我们能解决这个问题。

仅仅传递一个数据对象到数据绑定类中是很常见的，就像以上的item。当你有这个模式之后，你可以使用命名规约来为所有的RecyclerView和视图创建单个ViewHolder。我们要使用的规约是把视图模型对象命名为“obj"。你可能会使用“item"或者“打他”，但是我选择使用“obj”，因为更容易从例子中辨认出来。

	public class MyViewHolder extends RecyclerView.ViewHolder {
    private final ViewDataBinding binding;

    public MyViewHolder(ViewDataBinding binding) {
        super(binding.getRoot());
        this.binding = binding;
    }

    public void bind(Object obj) {
        binding.setVariable(BR.obj, obj);
        binding.executePendingBindings();
    }
    } 
 在MyViewHolder中，我使用的是ViewDataBinding,它是所有生成绑定类的基类，而不是特定的ItemBinding。这样的话，我就能在我的ViewHolder中支持所有布局。我也使用setVariable()代替了类型安全但是针对特定类的方法setObject,所以我能够赋值不管什么我想要的视图模型对象类型。最重要的部分是变量必须叫做”obj",因为我使用BR.obj作为setVariable()中的key。这就意味着在你的布局文件中，你必须有一个像这样的变量标签：
 
 <variable name="obj" type="Item"/>

当然，你的变量可以是你的数据绑定布局所需要的任何类型，而不是仅仅是Item。

我然后就能创建一个基础类，而这个类能被我所有的RecyclerView适配器所使用。

	public abstract class MyBaseAdapter
                extends RecyclerView.Adapter<MyViewHolder> {
    public MyViewHolder onCreateViewHolder(ViewGroup parent,
                                           int viewType) {
        LayoutInflater layoutInflater =
                LayoutInflater.from(parent.getContext());
        ViewDataBinding binding = DataBindingUtil.inflate(
                layoutInflater, viewType, parent, false);
        return new MyViewHolder(binding);
    }

    public void onBindViewHolder(MyViewHolder holder,
                                 int position) {
        Object obj = getObjForPosition(position);
        holder.bind(obj);
    }
    @Override
    public int getItemViewType(int position) {
        return getLayoutIdForPosition(position);
    }

    protected abstract Object getObjForPosition(int position);

    protected abstract int getLayoutIdForPosition(int position);
    }
 在这个适配器中，布局id被用作视图类型，这样使inflate正确的绑定变得简单。这是让Adapter处理多个布局的，但是最常见的单个布局的RecyclerView，为此我们创建一个基础类：
 
    public abstract class SingleLayoutAdapter extends MyBaseAdapter {
    private final int layoutId;
    
    public SingleLayoutAdapter(int layoutId) {
        this.layoutId = layoutId;
    }
    
    @Override
    protected int getLayoutIdForPosition(int position) {
        return layoutId;
    }
    }  
##还剩下什么 

所有来自RecyclerView的模版代码已经被处理了，给你剩下的要做的部分就是最难的部分，加载数据到UI线程，通知适配器数据发生了改变，等等。Android DataBinding只是减少了无聊的那部分。

你可以继承这个技术点来支持多个变量，提供一个事件处理器对象来处理点击事件来说是很正常的，并且你可能会和View Model类一起传出。如果总是传入Activity或Fragment，你可以添加这些变量。只要你使用始终如一的命名，你就可以在你所有的RecyclerView上使用这个技术。

在RecyclerView上使用Andorid DataBinding是简单的，并且十分明显的减少了模版代码。可能你的应用只需要一个ViewHolder，并且你再也不用写什么onCreateViewHolder()和onBindViewHolder()了。   