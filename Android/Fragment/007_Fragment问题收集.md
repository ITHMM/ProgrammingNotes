### 1 Fragment的onActivityResult的问题


多层嵌套Fragment， onActivity Result问题这样解决。 在最新的`Support v4-23.2`中已经对此问题进行修复。

### 2 Fragment对Menu菜单操作


- 首先在Fragment的onCreate方法中设置      setHasOptionsMenu(true);


然后实现下面方法，需要注意的是对于菜单的处理，Activity的优先级高于Fragment



    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        inflater.inflate(R.menu.menu_frag_rep, menu);
        super.onCreateOptionsMenu(menu, inflater);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        Toast.makeText(getActivity(), "id = " + item.getTitle(), Toast.LENGTH_SHORT).show();
        return super.onOptionsItemSelected(item);
    }



### 3 Fragement通讯
- 对于Fragement与Activity通讯，建议使用接口，而不是getActivity强转，因为这样fragment的复用性太差，代码也有耦合。
- 对于Fragement间的通讯，可是使用Fragmen的`setTargetFragment`方法，或者通过Activity实现。
