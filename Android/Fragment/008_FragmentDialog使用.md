# 1 FragmentDialog

DialogFragment是一种特殊的Fragment，使用DialogFragment来管理对话框，当旋转屏幕和按下后退键时可以更好的管理其生命周期，它和Fragment的生命周期基本一致。且DialogFragment也允许开发者把Dialog作为内嵌的组件进行重用，类似Fragment（可以在大屏幕和小屏幕显示出不同的效果）


# 2 基本使用


使用AppCompatDialogFragement需要实现以下方法




      public class Dialog extends AppCompatDialogFragment {

        private EditText mNameEt;
        private EditText mPwdEt;
        private Button mSubmitBtnl;


        /**
            这里可以Dialog的Style
        /
        public Dialog() {
            setStyle(DialogFragment.STYLE_NO_TITLE, R.style.full);
        }

        /**
        用于提供dialog的内容布局
        /
        @Nullable
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
            return inflater.inflate(R.layout.dialog_fragment, container, false);
        }

        @Override
        public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
            super.onViewCreated(view, savedInstanceState);
            mNameEt = (EditText) getView().findViewById(R.id.id_dialog_fragment_name_et);
            mPwdEt = (EditText) getView().findViewById(R.id.id_dialog_fragment_pwd_et);
            mSubmitBtnl = (Button) getView().findViewById(R.id.id_dialog_submit_btn);
        }

        /**
        用于提供一个dialog，可以在这里设置dialog的属性
        /
        @Override
        public android.app.Dialog onCreateDialog(Bundle savedInstanceState) {
            AppCompatDialog dialog = new AppCompatDialog(getActivity(), getTheme());
            Window dialogWindow = dialog.getWindow();
            WindowManager.LayoutParams lp = dialogWindow.getAttributes();
            dialogWindow.setGravity(Gravity.CENTER);
            lp.width = WindowManager.LayoutParams.MATCH_PARENT;
            lp.height = WindowManager.LayoutParams.MATCH_PARENT;
            dialogWindow.setWindowAnimations(R.style.dialog_anim);
            dialogWindow.setAttributes(lp);
            return dialog;
        }

onCreateDialog用于返回一个Dialog，默认已经实现，而onCreateView中返回的视图是作为dialog的contentView。

需要注意的地方：
1：window的属性设置使用`supportRequestWindowFeature()`。
2: theme最好继承`AlertDialog.AppCompat`之类的。


# 3 源码分析：

生命周期如下：

    onCreate
    getLayoutInflater
    onCreateDialog
    onCreateView
    onActivityCreated



getLayoutInflater在onCreate后执行：


        public LayoutInflater getLayoutInflater(Bundle savedInstanceState) {
            if (!mShowsDialog) {
                return super.getLayoutInflater(savedInstanceState);
            }
            mDialog = onCreateDialog(savedInstanceState);
            if (mDialog != null) {
                setupDialog(mDialog, mStyle);
                return (LayoutInflater) mDialog.getContext().getSystemService(
                        Context.LAYOUT_INFLATER_SERVICE);
            }
            return (LayoutInflater) mHost.getContext().getSystemService(
                    Context.LAYOUT_INFLATER_SERVICE);
        }

这里调用了onCreateDialog，然后是设置Dialog的属性 setupDialog(mDialog, mStyle);

        public void setupDialog(Dialog dialog, int style) {
            switch (style) {
                case STYLE_NO_INPUT:
                    dialog.getWindow().addFlags(
                            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE |
                                    WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE);
                    // fall through...
                case STYLE_NO_FRAME:
                case STYLE_NO_TITLE:
                    dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);
            }
        }

其实就是为Window添加一些标记，这里我可以看到`STYLE_NO_FRAME`和`STYLE_NO_TITLE`都设置了`Window.FEATURE_NO_TITLE`，然后如果设置了`STYLE_NO_FRAME`还会使用下面style


        public void setStyle(@DialogStyle int style, @StyleRes int theme) {
            mStyle = style;
            if (mStyle == STYLE_NO_FRAME || mStyle == STYLE_NO_INPUT) {
                mTheme = android.R.style.Theme_Panel;
            }
            if (theme != 0) {
                mTheme = theme;
            }
        }

     <style name="Theme.Panel">
            <item name="windowBackground">@color/transparent</item>
            <item name="colorBackgroundCacheHint">@null</item>
            <item name="windowFrame">@null</item>
            <item name="windowContentOverlay">@null</item>
            <item name="windowAnimationStyle">@null</item>
            <item name="windowIsFloating">true</item>
            <item name="backgroundDimEnabled">false</item>
            <item name="windowIsTranslucent">true</item>
            <item name="windowNoTitle">true</item>
        </style>

使用STYLE_NO_FRAME系统将不会限制Dialog的大小。






然后看一下onActiviyCreate方法

     @Override
        public void onActivityCreated(Bundle savedInstanceState) {
            super.onActivityCreated(savedInstanceState);
            if (!mShowsDialog) {
                return;
            }
            View view = getView();
            if (view != null) {
                if (view.getParent() != null) {
                    throw new IllegalStateException("DialogFragment can not be attached to a container view");
                }
                mDialog.setContentView(view);
            }
            mDialog.setOwnerActivity(getActivity());
            mDialog.setCancelable(mCancelable);
            mDialog.setOnCancelListener(this);
            mDialog.setOnDismissListener(this);
            if (savedInstanceState != null) {
                Bundle dialogState = savedInstanceState.getBundle(SAVED_DIALOG_STATE_TAG);
                if (dialogState != null) {
                    mDialog.onRestoreInstanceState(dialogState);
                }
            }
        }


其实就是把onCreateView中返回的View，设置为mDialog的ContentView，到此FragmentDialog的大概流程可以知道了：
根据风格创建一个Dialog，把onCreateView返回的View设置给Dialog，然后显示出来。


# 4 实现一个BottomSheet的FragmentDialog

    public  class BottomSheetFragmentDialog extends AppCompatDialogFragment {

        public static BottomSheetFragmentDialog newInstance() {
          return new BottomSheetFragmentDialog();
        }


        @Override
        public void onCreate(@Nullable Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setStyle(DialogFragment.STYLE_NO_TITLE, R.style.FragmentDialog);
        }

        @Nullable
        @Override
        public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
          return inflater.inflate(R.layout.dialog_bottom_sheet, container, false);
        }

        @NonNull
        @Override
        public Dialog onCreateDialog(Bundle savedInstanceState) {
            WindowManager wm = (WindowManager) getContext().getSystemService(Context.WINDOW_SERVICE);
            DisplayMetrics outMetrics = new DisplayMetrics();
            wm.getDefaultDisplay().getMetrics(outMetrics);
            AppCompatDialog dialog = new AppCompatDialog(getActivity(), getTheme());
            Window dialogWindow = dialog.getWindow();
            WindowManager.LayoutParams lp = dialogWindow.getAttributes();
            dialogWindow.setGravity(Gravity.BOTTOM);
            lp.width = outMetrics.widthPixels;
            lp.height = (int) (outMetrics.heightPixels*0.8);
            dialogWindow.setAttributes(lp);
            return dialog;
        }

        @Override
        public void onResume() {
          super.onResume();
          Log.d("BottomSheetFragmentDial", "getTargetFragment():" + getTargetFragment());
        }
    }


解析：

##### 1， 在onCreate中设置风格样式，因为在onCreate后Dialog就创建出来了，必须在Window创建前设置好窗口属性：

       setStyle(DialogFragment.STYLE_NO_TITLE, R.style.full);

第一个参数使用DialogFragment的内置参数：
- STYLE_NO_TITLE 不需要title
- STYLE_NORMAL 正常显示
- STYLE_NO_FRAME 系统将不会限制dialog的边界大小
- STYLE_NO_INPUT 同STYLE_NO_FRAME，并且获取不到输入焦点



第二个参数使用自定义的style

如果不设置那么dialog不会填充屏幕，即使设置了窗口宽度为屏幕宽度，所以设置一个style：

      <style name="full" parent="AlertDialog.AppCompat.Light">

            <item name="android:windowContentOverlay">@null</item>
                //边缘不需要阴影
            <item name="android:windowCloseOnTouchOutside">true</item>

        </style>
不使用自定义的style而使用`STYLE_NO_FRAME`也可以实现全屏，但是dialog显示出后没有阴影，体验不好，所以还是推荐使用theme。



        <!--Fragment Dialog Style-->
        <style name="FragmentDialog">
          <item name="android:windowAnimationStyle">@style/bottom_in_style</item>
          <item name="android:windowContentOverlay">@null</item>
          <item name="android:backgroundDimEnabled">true</item>
          <item name="android:backgroundDimAmount">0.4</item>
          <item name="android:windowNoTitle">true</item>
          <item name="android:windowCloseOnTouchOutside">true</item>
        </style>



        <style name="bottom_in_style">
            <item name="android:windowEnterAnimation">@anim/bottom_in</item>
            <item name="android:windowExitAnimation">@anim/bottom_out</item>
        </style>
