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

##动态列表
以上的绑定适配器对于静态的列表是很好用的，但是如果你的列表是变化的呢？可能用户添加了一个新的选择，那一项要被添加到radio Button列表中去。ObservableList给予我们观察改变并对其做出响应的能力。我们使用了一个OnListChangedCallback来观察列表中的变化：

    @BindingAdapter({"entries", "layout"})
    public static <T> void setEntries(ViewGroup viewGroup,
        ObservableList<T> oldEntries, int oldLayoutId,
        ObservableList<T> newEntries, int newLayoutId) {
    if (oldEntries == newEntries && oldLayoutId == newLayoutId) {
        return; // nothing has changed
    }

    EntryChangeListener listener =
            ListenerUtil.getListener(viewGroup, R.id.entryListener);
    if (oldEntries != newEntries && listener != null) {
        oldEntries.removeOnListChangedCallback(listener);
    }

    if (newEntries == null) {
        viewGroup.removeAllViews();
    } else {
        if (listener == null) {
            listener =
                    new EntryChangeListener(viewGroup, newLayoutId);
            ListenerUtil.trackListener(viewGroup, listener,
                    R.id.entryListener);
        } else {
            listener.setLayoutId(newLayoutId);
        }
        if (newEntries != oldEntries) {
            newEntries.addOnListChangedCallback(listener);
        }
        resetViews(viewGroup, newLayoutId, newEntries);
    }
    }
在setEntries（）绑定适配器中有一些值得注意的事情。首先我使用了data binding的特性来让我能获取旧的值的同时也获取新的值。通过提供两倍于属性的数据参数，第一组参数接收旧值，第二组接收新值。我用这个方式从旧条目列表中删除了监听器。

第二点，Android DataBinding 通常会观察列表的改变，并且当改变发生的时候，它会对表达式重新进行评估。我想要在BindingAdapter中去管理改变，所以它不会在没有情况发生的时候做任何事情。我使用了ListenerUtil去记录EntryChangeListener以及OnListChangedCallback。Listener保存了listener的记录，所以它能在调用直接被获取，并且通过使用它能移除旧的监听器，以及可能把它添加到新的列表上。我需要提供一个标识作为key，所以我创建了一个：

<resources>
    <item type="id" name="entryListener"/>
</resources>

第三，如果只有如菊变化的时候，setEntries()依赖于EntryChangeListener来对子视图进行更新。否则的话，他会完全替换掉子视图。例如。，当layout ID发生变化，我们抛弃了旧的孩子然后就完全重新进行填充。

出此之外，它就是相当简单了。这里是它使用的其他方法：

    private static ViewDataBinding bindLayout(LayoutInflater inflater,
        ViewGroup parent, int layoutId, Object entry) {
    ViewDataBinding binding = DataBindingUtil.inflate(inflater,
            layoutId, parent, false);
    binding.setVariable(BR.data, entry);
    return binding;
    }

    private static void resetViews(ViewGroup parent, int layoutId,
        List entries) {
    parent.removeAllViews();
    if (layoutId == 0) {
        return;
    }
    LayoutInflater inflater = (LayoutInflater) parent.getContext()
            .getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    for (int i = 0; i < entries.size(); i++) {
        Object entry = entries.get(i);
        ViewDataBinding binding = bindLayout(inflater, parent,
                layoutId, entry);
        parent.addView(binding.getRoot());
    }
    }

它们做的基本上都是在原有的绑定适配器中完成的。resetViews()方法首先从ViewGroup中移除了所有的视图，然后填充视图并绑定到列表中的数据。

你能够用一个简单的EventChangeListener，它的功能任何时候仅仅是重置视图。
		
	 private static class EntryChangeListener
            extends ObservableList.OnListChangedCallback {
    private final ViewGroup mTarget;
    private int mLayoutId;

    public EntryChangeListener(ViewGroup target, int layoutId) {
        mTarget = target;
        mLayoutId = layoutId;
    }

    public void setLayoutId(int layoutId) {
        mLayoutId = layoutId;
    }

    @Override
    public void onChanged(ObservableList observableList) {
        resetViews(mTarget, mLayoutId, observableList);
    }

    @Override
    public void onItemRangeChanged(ObservableList observableList,
                                   int start, int count) {
        resetViews(mTarget, mLayoutId, observableList);
    }

    @Override
    public void onItemRangeInserted(ObservableList observableList,
                                    int start, int count) {
        resetViews(mTarget, mLayoutId, observableList);
    }

    @Override
    public void onItemRangeMoved(ObservableList observableList,
                                 int from, int to, int count) {
        resetViews(mTarget, mLayoutId, observableList);
    }

    @Override
    public void onItemRangeRemoved(ObservableList observableList,
                                   int start, int count) {
        resetViews(mTarget, mLayoutId, observableList);
    }
    }

老实说，这对于大部分使用情形可能已经够好了。你不会把它用作大量视图，大量视图是留给RecyclerView的。但是，如果我想要在发生变化的时候添加视图的动画效果，我会对改变事件进程更好的处理。你可能想到的是change监听器中像这样的东西：

    @Override
    public void onItemRangeChanged(ObservableList observableList,
                               int start, int count) {
    TransitionManager.beginDelayedTransition(mTarget);
    final int end = start + count;
    for (int i = start; i < end; i++) {
        Object data = observableList.get(i);
        View view = mTarget.getChildAt(i);
        ViewDataBinding binding = DataBindingUtil.getBinding(view);
        binding.setVariable(BR.data, data);
    }
    }

它仅仅是从当前视图对数据进行重新绑定。不幸的是，在我们有十分智能的转换的时候才能有效，但是默认的转换在数据发生变化的时候是不知道怎么做的。作为替代，我们要去替换掉视图：
		
	@Override
     public void onItemRangeChanged(ObservableList observableList,
                               int start, int count) {
    LayoutInflater inflater = (LayoutInflater) mTarget.getContext()
            .getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    TransitionManager.beginDelayedTransition(mTarget);
    final int end = start + count;
    for (int i = start; i < end; i++) {
        Object data = observableList.get(i);
        ViewDataBinding binding = bindLayout(inflater, 
            mTarget, mLayoutId, data);
        binding.setVariable(BR.data, observableList.get(i));
        mTarget.removeViewAt(i);
        mTarget.addView(binding.getRoot(), i);
    }
    }
    
现在我们在它们发生变化的时候，看到了一个非常好的渐出和渐入的视图效果 。实现方法的剩余部分就很直接了，使用TransitionManager来使视图动起来。

	@Override
    public void onItemRangeInserted(ObservableList observableList,
                                int start, int count) {
    TransitionManager.beginDelayedTransition(mTarget);
    final int end = start + count;
    LayoutInflater inflater = (LayoutInflater) mTarget.getContext()
            .getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    for (int i = end - 1; i >= start; i--) {
        Object entry = observableList.get(i);
        ViewDataBinding binding =
            bindLayout(inflater, mTarget, mLayoutId, entry);
        mTarget.addView(binding.getRoot(), start);
    }
    }

      @Override
    public void onItemRangeMoved(ObservableList observableList,
                             int from, int to, int count) {
    TransitionManager.beginDelayedTransition(mTarget);
    for (int i = 0; i < count; i++) {
        View view = mTarget.getChildAt(from);
        mTarget.removeViewAt(from);
        int destination = (from > to) ? to + i : to;
        mTarget.addView(view, destination);
    }
    }

    @Override
    public void onItemRangeRemoved(ObservableList observableList,
                               int start, int count) {
    TransitionManager.beginDelayedTransition(mTarget);
    for (int i = 0; i < count; i++) {
        mTarget.removeViewAt(start);
    }
    }

##思考

你可能想要用这个列表绑定技术来替换RecyclerView。不要这么做。数据绑定列表不是RecyclerView的替代方案。作为替换，使用它绑定到去绑定到能在你布局中可以的一小组视图。首要原则是如果你要滑动列表，那就使用RecyclerView。如果你不需要，就使用Data Binding。

刚开始的时候我的例子只有四行代码，但是不知道怎么的我写了一个将近150行代码的BindingAdapter.但是现在它已经写完了，我可以在应用中的任何地方来使用它来生成我的小量数据驱动的界面列表。当数据变化的时候，它甚至有动画效果并且我从不用担心直接更新视图。现在你也同样不用担心了。

你可以在[DataBoundList项目](https://github.com/google/android-ui-toolkit-demos)中看到代码。在这个项目中，用户被从列表中添加和移除并且他能动态地改变LinearLayout。你永远不应该在一个能够滑动的视图列表上使用这个。你应该使用RecyclerView,但这只是一个Demo。我希望你能把这个绑定列表到ViewGroup的方式对你的应用有帮助。
    