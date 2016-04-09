# 1 Fragment介绍





## 1.1 Fragment是什么


Fragment包含在Activity中，Fragment只能存在于Activity的上下文（context）内，没有Activity就无法使用Fragment，因此Fragment只能在Activity的上下文（context）创建。Fragment可以作为Activity的一部分，Fragment和Activity非常相似，Fragment拥有一个与她相关的视图层次结构，拥有一个与活动非常相似的生命周期。



## 1.2 为何使用Fragment


Activity的使用局限：不能将多个Activity活动界面放在屏幕上一并显示。因此创建了Fragment来弥补Activity的局限。Fragment可以像Activity一样响应Back键等类似Activity的功能。



## 1.3 Fragment的默认构造方法

如果需要给Fragment设置初始化参数，应该使用`setArgument`,把参数放到Bundle中，当因为内存不足而导致Activity被回收时，Fragment自然也会被回收。当系统还原Fragment时，它调用默认的构造函数，然后将参数包还原到新建的Fragment。该Fragment执行的后续回调还能够访问这些参数，因此在使用Fragment时，一定要确保以下两点：



- 确保Fragment类存在默认的构造函数；
- 在Fragment创建后立即添加一个参数包（Bundle），使Fragment重建时可以正确设置Fragment，也使Android系统可以在必要时正确还原Fragment。

>参数其实都是跨进程保存的，这也是为什么需要把参数放入到Bundle中，Bundle可以存放类型都是可以跨进程传输的。

Fragment在重建时，其之前保存的状态会被存放在一个Bundle对象中，通过其回调方法，返回给开发者，下面是会回调的方法，(注意**这不是作为初始化参数而附加的包。可能在这个包中存储Fragment的当前状态，而不是应该用于初始化它的值**)，区分`getArguments`与生命周期回调的`Bundle`
- onInflate
- onCreate
- onCrateView
- onActivityCreate
- onViewCreate


在Fragment重建时，即使并没有显式的保存任何状态，状态包依然不是null值。




------
<br/><br/><br/>
# 2 Fragment生命周期方法回调


### onInflate
如果Fragment是由在xml中`<fragment>`标记定义的，Fragment的onInflate方法将被回调



### onAttach
onAttach()回调将在Fragment与其Activity关联之后调用,  onCreate方法将在之后调用，


初始化参数包（Bundle）可以从碎片的**`getArguments()`**方法获得。



    public void onAttach(Context context) {
        super.onAttach(context);
        Bundle arguments = getArguments();
        args = arguments.getString("Args");
    }


### onCreate
此方法表示Fragment被创建了，不要将此方法与Activity的onCreate方法混淆。


尽管Fragment现在可能已经与其Activity关联，但是我们还没有获得Activity的onCreate()已完成的通知，所以不能将依赖于Activity视图层次结构存在性的代码放入此回调方法中。



### onCreateView
此回调方法用于构建Fragment将管理的视图，不要将视图添加到ViewGroup中，系统会自定执行。

视图可以返回null，表示Fragment不需要管理视图，而是作为一个逻辑执行者。


     @Nullable

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {

        return inflater.inflate(R.layout.fragment_replace, container, false);//注意最后一个参数是false
    }



### onViewCreate
当 onCreateView中返回了View，此方法将被回调，此时view已经被添加到Activity的视图中，可以通过getActivity().findViewById()来获取子view




### onActivityCreate

终于到了与用户交互的时刻了，onActivityCreated()回调会在Activity完成其onCreate()回调之后调用。在调用onActivityCreated()之前，Activity的视图层次结构已经准备好了，**这是在用户看到用户界面之前你可对用户界面执行的最后调整的地方**。（备注：**如果Activity和它的Fragment是从保存的状态重新创建的，此回调尤其重要，也可以在这里确保此Activity的其他所有Fragment已经附加到该活动中了**）





### onStart()\onResume()\onPause()\onStop()
接下来的onStart()\onResume()\onPause()\onStop()回调方法将和Activity的回调方法进行绑定，也就是说与Activity中对应的生命周期相同




###onDestoryView

当Activity的视图结构与Fragment分离时被调用


###onDestory

不再使用Fragment时调用。（备注：**Fragment仍然附加到Activity并任然可以找到**，但是不能执行其他操作）





###onDetach

Fragment生命周期最后回调函数，调用后，Fragment不再与Activity绑定，释放资源   。





<br/><br/><br/>


# 3 处理屏幕旋转与销毁重建

看一下Fragment在屏幕旋转时被销毁重建的生命周期：


        BaseFragment: MainFragment-->onAttach
        BaseFragment: MainFragment-->onCreate
        BaseFragment: MainFragment-->onCreateView
        BaseFragment: MainFragment-->onViewCreated
        BaseFragment: MainFragment-->onActivityCreated
        BaseFragment: MainFragment-->onStart
        BaseFragment: MainFragment-->onResume
         //旋转

        BaseFragment: MainFragment-->onPause
        BaseFragment: MainFragment-->onStop
        BaseFragment: MainFragment-->onDestroyView
        BaseFragment: MainFragment-->onDestroy
        BaseFragment: MainFragment-->onDetach//完全被销毁
        BaseFragment: MainFragment-->onAttach//重建
        BaseFragment: MainFragment-->onCreate
        BaseFragment: MainFragment-->onCreateView
        BaseFragment: MainFragment-->onViewCreated
        BaseFragment: MainFragment-->onActivityCreated
        BaseFragment: MainFragment-->onStart
        BaseFragment: MainFragment-->onResume


默认的屏幕旋转（类似Activity在后台被回收），Fragment也会跟着Activity一样被销毁和重建，当Activity被重建Fragment也会被自动重建并和Activity关联，也就是说我们应该先判断fragment是否已经被添加，如果已经添加，则没有必要再次创建Fragment对象，而是通过tag在 fragmentManager中找到fragment，如下面代码所示


    @Override

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        if (savedInstanceState == null) {
            getSupportFragmentManager().beginTransaction()
                    .replace(R.id.container, Fragment.instantiate(this,MainFragment.class.getName() , null) , "fragmentTag").commit();
        }


    }




## 使用 setRetainInstance方法


Fragment有一个非常强大的功能——就是可以在Activity重新创建时可以**不完全销毁Fragment**，以便Fragment可以恢复。**在onCreate()方法中调用setRetainInstance(true/false)方法是最佳位置**。当在onCreate()方法中调用了setRetainInstance(true)后，当Activity被销毁时Fragment的实例不会被销毁，不需要被重建，所以Fragment恢复时会跳过onCreate()和onDestroy()方法，因此不能在onCreate()中放置一些初始化逻辑，但是被保存的Fragment实例不会保持太久，若长时间没有容器承载它，也会被系统回收掉的。

`setRetainInstance`的一个应用场景是，在Retained的Fragment中执行一个耗时的后台任务，即使Activity被销毁重建任务也不会被中断，当Activity被重建后，依然可以通过Retained的Fragment重新与任务连接。


### 需要注意的地方：

#### setRetainInstance与addToStack不能同时使用

#### setRetainInstance不能在嵌套的Fragmeng中使用

如果在嵌套的Fragment中使用，会有如下错误。


         public void setRetainInstance(boolean retain) {
           if (retain && mParentFragment != null) {
               throw new IllegalStateException(
                    "Can't retain fragements that are nested in other fragments");
          }
        mRetainInstance = retain;
    }



<br/><br/><br/>

## 4 Fragment的事务

一般添加一个Fragment的操作为：

       getSupportFragmentManager()
                        .beginTransaction()
                        .add(R.id.act_main_container_rl, page.newFragment(this), page.getTag())
                        .commit();

可以看到需要调用一连串的方法，还有事务。这是因为FragmentManager对于fragment的操作是一系列有顺序步骤的操作。就像数据库一样，任何一个环节都不能出错，否则会造成意想不到的后果，所以系统对fragment的操作是基于事务的



## 1commit方法


commit用于提交一个事务 (只有在Activity处于running状态才能保存)，如果Activity没有处于running状态，将会抛出异常，调用commit()后，事务不会马上执行。它会在activity的UI线程中等待直到UI线程能够执行这个事物为止。


## commitAllowingStateLoss方法

commitAllowingStateLoss 用于提交一个事务，并且允许Fragment的状态丢失。如果要在可能丟失状态的情況下提交事务，应该使用commitAllowingStateLoss()。


## executePendingTransactions方法

如果希望FragmentTransaction的conmmit的事务立即执行，可以在UI線程中调用FragmentManager的 `executePendingTransactions()`方法。
