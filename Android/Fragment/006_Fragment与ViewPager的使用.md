# 1 FragmentPagerAdapter


FragmentPagerAdapter一般只需要实现 getItem和getCount方法，getItem方法用于创建Fragment的对象，如果要设置args，只能在这里设置。 已经实现的`instantiateItem`方法用于找回被detach的fragment，如果没有找到才会调用getItem方法来创建Fragment对象，FragmentPagerAdapter会为它管理的Fragment指定一个tag，源码如下：



         private static String makeFragmentName(int viewId, long id) {
            return "android:switcher:" + viewId + ":" + id;
    }




FragmentPagerAdapter默认缓存两个页面，即当前显示页面的两边，超出范围的会调用其detach方法，让fragment的view与activity分离，只保留fragment的对象，当下次滑动回来时，会重新调用的attach方法，此时fragment会重建视图，生命周期方法如下:



    rDemoFragment 0-->setUserVisibleHint ==false
    rDemoFragment 1-->setUserVisibleHint ==false
    rDemoFragment 0-->onAttach
    rDemoFragment 0-->onCreate
    rDemoFragment 0-->setUserVisibleHint ==true
    rDemoFragment 0-->onCreateView
    rDemoFragment 0-->onViewCreated
    rDemoFragment 0-->onActivityCreated
    rDemoFragment 0-->onStart
    rDemoFragment 0-->onResume
    rDemoFragment 1-->onAttach
    rDemoFragment 1-->onCreate
    rDemoFragment 1-->onCreateView
    rDemoFragment 1-->onViewCreated
    rDemoFragment 1-->onActivityCreated
    rDemoFragment 1-->onStart
    rDemoFragment 1-->onResume


当滑动到第2个界面的时候，会初始化第三个界面的Fragment，加入到Activity中，此时设置第一个界面的Fragment的 `setUserVisibleHint ` 为flase，第二个界面的` setUserVisibleHint ` 为true


    rDemoFragment 2-->setUserVisibleHint ==false
    rDemoFragment 0-->setUserVisibleHint ==false
    rDemoFragment 1-->setUserVisibleHint ==true
    rDemoFragment 2-->onCreateView
    rDemoFragment 2-->onViewCreated
    rDemoFragment 2-->onActivityCreated
    rDemoFragment 2-->onStart
    rDemoFragment 2-->onResume


当滑动到第三个界面时,会销毁第一个界面的Fragment的视图，只保留其对象，设置第二个界面 `setUserVisibleHint ` 为flase，第三个界面的 `setUserVisibleHint`  为ture


    rDemoFragment 1-->setUserVisibleHint ==false
    rDemoFragment 2-->setUserVisibleHint ==true
    rDemoFragment 0-->onAttach
    rDemoFragment 0-->onStop
    rDemoFragment 0-->onDestroyView


滑回第2个界面，重建第1个界面的视图：


    rDemoFragment 0-->setUserVisibleHint ==false
    rDemoFragment 2-->setUserVisibleHint ==false
    rDemoFragment 1-->setUserVisibleHint ==true
    rDemoFragment 0-->onCreateView
    rDemoFragment 0-->onViewCreated
    rDemoFragment 0-->onActivityCreated
    rDemoFragment 0-->onStart
    rDemoFragment 0-->onResume


- 可以通过 mViewPager.setOffscreenPageLimit(int count);方法来设置viewPager缓存界面的个数，默认和最小值是1 ，表示缓存当前界面的两边各1个。
- 利用`setUserVisibleHint`可以实现Fragment的懒加载。
- 由于 FragmentPagerAdapter会保存Fragment对象，所以考虑到内存优化，只适合用于页面较少的ViewPager情况















# 2 FragmemtStatePagerAdapter

FragmemtStatePagerAdapter与FragmentPagerAdapter类似，同样实现getItem和getCount方法，getItem方法用于获取Fragment。不同点在于，当Fragment超出缓存范围是，是将Fragment remove掉，而不是detach，也就是说，系统不会保存Fragment的对象，我们可以通过 onSaveInstanceState保存Fragment的状态。

生命周期如下：


    rDemoFragment 0-->setUserVisibleHint ==false
    rDemoFragment 1-->setUserVisibleHint ==false
    rDemoFragment 0-->onAttach
    rDemoFragment 0-->onCreate
    rDemoFragment 0-->setUserVisibleHint ==true
    rDemoFragment 0-->onCreateView
    rDemoFragment 0-->onViewCreated
    rDemoFragment 0-->onActivityCreated
    rDemoFragment 0-->onStart
    rDemoFragment 0-->onResume
    rDemoFragment 1-->onAttach
    rDemoFragment 1-->onCreate
    rDemoFragment 1-->onCreateView
    rDemoFragment 1-->onViewCreated
    rDemoFragment 1-->onActivityCreated
    rDemoFragment 1-->onStart
    rDemoFragment 1-->onResume

    rDemoFragment 2-->setUserVisibleHint ==false
    rDemoFragment 0-->setUserVisibleHint ==false
    rDemoFragment 1-->setUserVisibleHint ==true
    rDemoFragment 2-->onAttach
    rDemoFragment 2-->onCreate
    rDemoFragment 2-->onCreateView
    rDemoFragment 2-->onViewCreated
    rDemoFragment 2-->onActivityCreated
    rDemoFragment 2-->onStart
    rDemoFragment 2-->onResume

    rDemoFragment 0-->onSaveInstanceState
    rDemoFragment 1-->setUserVisibleHint ==false
    rDemoFragment 2-->setUserVisibleHint ==true
    rDemoFragment 0-->onAttach
    rDemoFragment 0-->onStop
    rDemoFragment 0-->onDestroyView
    rDemoFragment 0-->onDestroy
    rDemoFragment 0-->onDetach


    rDemoFragment 0-->setUserVisibleHint ==false
    rDemoFragment 2-->setUserVisibleHint ==false
    rDemoFragment 1-->setUserVisibleHint ==true
    rDemoFragment 0-->onAttach
    rDemoFragment 0-->onCreate
    rDemoFragment 0-->onCreateView
    rDemoFragment 0-->onViewCreated
    rDemoFragment 0-->onActivityCreated
    rDemoFragment 0-->onStart
    rDemoFragment 0-->onResume





# 3 Adapter的notifyDataSetChanged

一般我们调用FragmentPagerAdapter和FragmentStatePagerAdapter的`notifyDataSetChanged`是没有效果的，在Adapter中有一个方法需要重写：

     public int getItemPosition(Object object) {
            return PagerAdapter.POSITION_UNCHANGED;
        }

方法的注释为：

    Called when the host view is attempting to determine if an item's position
    has changed. Returns {@link #POSITION_UNCHANGED} if the position of the given
    item has not changed or {@link #POSITION_NONE} if the item is no longer present
    in the adapter.

如果希望`otifyDataSetChanged`方法有效果，需要重写`getItemPosition`方法，然后返回`POSITION_NONE`。但是这种做法会让ViewPager移除所有的View，然后重新加载，代价还是有点高的。
