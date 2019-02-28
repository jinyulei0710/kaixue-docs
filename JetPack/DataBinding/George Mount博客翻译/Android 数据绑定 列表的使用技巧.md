Android 数据绑定：列表的使用技巧- 针对小数量的列表

我在思考之前的文章，在这篇文章中我写了如何在RecyclerView中使用Android DataBinding。但是如果你有一个元素的列表，并不需要用RecyclerView去处理它。毕竟，如果你只是要在屏幕上展示三四个元素并且他们永远不会被回收，那就没有必要拿出重武器了。

通常开发者会遍历它们的条目帮手动的创建视图：

	for (Account item : items) {
    ItemBinding itemBinding =
        ItemBinding.inflate(getLayoutInflater(), parent, true);
    itemBinding.setData(item);
    }

这很简单。如果我们能把绑定条目到XML中的列表。像这样就很好了：
			
	<LinearLayout
    app:entries="@{entries}"
    app:layout="@{@layout/item}"
    .../>
    
##简单的列表绑定适配器

我想要使用条目列表来在LinearLayout中创建视图帮切绑定这些视图到列表中的值。每个不同的布局它自己的生成绑定类，那么如果我想要做出一个通用的BindingAdapter,我不能仅仅调用常规的setter。我无疑不会使用反射-它的花销很大。作为替换，就像使用RecyclerView一样，我们可以使用约束来解决这个问题。

我们会使用这样一个约束，这个约束有着一个变量并且变量的名称是始终如一的。不管列表中的什么，列表中会一个名叫data的变。然后我们就可以在ViewDataBinding.setVariable()方法中绑定数据到布局中去。

    @BindingAdapter({"entries", "layout"})
      public static <T> void setEntries(ViewGroup viewGroup,
                                  List<T> entries, int layoutId) {
    viewGroup.removeAllViews();
    if (entries != null) {
        LayoutInflater inflater = (LayoutInflater)
            viewGroup.getContext()      
                .getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        for (int i = 0; i < entries.size(); i++) {
            T entry = entries.get(i);
            ViewDataBinding binding = DataBindingUtil
                .inflate(inflater, layoutId, viewGroup, true);
            binding.setVariable(BR.data, entry);
        }
    }
    }    
 
并且你可以把它像这样的绑定到你的ViewGroup：
 	
 	 <LinearLayout
    app:entries="@{entries}"
    app:layout="@{@layout/item}"
    .../>


以上的布局会使用item.xml布局和设置到条目列表中的项到“data”变量。这对于所有的ViewGroup都是适用的，因为addView()对于管理子视图是足够的。

