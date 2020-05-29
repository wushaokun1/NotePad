## NotePad对于时间戳和搜索框的添加
### 1.时间戳的显示
#### 1.1第一步：
首先修改NoteList文件中PROJECTION的内容，添加modif字段，这样在后面的搜索中才能从SQLite中读取修改时间的字段。

```java
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
    };
```
#### 1.2第二步：
修改适配器内容，增加dataColumns中装配到ListView的内容，所以要同时增加一个文本框来存放时间。

```java
final String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE , NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
int[] viewIDs = { android.R.id.text1, R.id.text2};
```

#### 1.3第三步：
修改layout文件夹中notelist_item的内容，增加一个textview组件，因为有两个组件，所以我们也要相应的为他们添加一个布局。

```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">

    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
    />
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:singleLine="true"
        />
</LinearLayout>
```
这样之后便能显示出long类型的时间，也就是创建距离现在的毫秒数，所以我们还要修改对于的时间类型，我的做法是从存入数据库就修改。
#### 1.4第四步：
修改NoteEditor中updateNote()方法中的时间类型.
首先获得long类型的now时间，在运用SimpleDateFormat类设置时间类型，这里我用的是“年/月/天 时/分”，可以根据不同的显示精度进行调整。将now转化成Date类型时间，其实做到这一步，将d放进数据库即可，不过显示得太精密了，太长了，所以我将他修改成我想要得sf类型。String format = sf.format(d)
```java
        Long now = Long.valueOf(System.currentTimeMillis());
        SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");
        Date d = new Date(now);
        String format = sf.format(d);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);

```
实验结果如下：
<img src="https://img-blog.csdnimg.cn/20200526184215942.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tra2tra3dz,size_16,color_FFFFFF,t_70" width="60%">

### 2.搜索框的添加
#### 2.1第一步：
新增布局文件searchview.xml，设置一个搜索的布局文件，由searchview，textview，listview组件组成，为搜索后所表现的样子

```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:queryHint="请输入搜索内容"
        >
    </SearchView>

    <ListView
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:drawSelectorOnTop="false" />
    <TextView
        android:paddingLeft="10dp"
        android:paddingRight="10dp"
        android:id="@android:id/empty"
        android:gravity="center"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:text="无搜索结果" />
</LinearLayout>
```
#### 2.2第二步：
修改NotesList一开始加载的布局文件

```java
 setContentView(R.layout.searchview);
```
#### 2.3第三步：设置searchview监听器
首先找到search组件，创造searchView得响应时间，根据传入得s进行搜索，然后写下selection语句NotePad.Notes.COLUMN_NAME_TITLE + " GLOB '*" + s + "*'"，运用getContentResolver().query对SQLite里面得数据进行搜索，PROJECTION则是在设置时间戳得时候已经书写好，如果s为空，则是把所有信息显示出来。最后更新adapter
```java
private void SearchView(final SimpleCursorAdapter adapter) {
        SearchView searchView = findViewById(R.id.search2);
        searchView.setSubmitButtonEnabled(true);
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                return false;
            }

            @Override
            public boolean onQueryTextChange(String s) {
                Cursor newCursor;

                if (!s.equals("")) {
                    String selection = NotePad.Notes.COLUMN_NAME_TITLE + " GLOB '*" + s + "*'";
                    newCursor = getContentResolver().query(
                            getIntent().getData(),
                            PROJECTION,
                            selection,
                            null,
                            NotePad.Notes.DEFAULT_SORT_ORDER
                    );
                } else {
                    newCursor = getContentResolver().query(
                            getIntent().getData(),
                            PROJECTION,
                            null,
                            null,
                            NotePad.Notes.DEFAULT_SORT_ORDER
                    );
                }
                adapter.swapCursor(newCursor); // 视图将同步更新！
                return true;
            }
        });
    }
```
实验结果如下：

<img src="https://img-blog.csdnimg.cn/2020052618503485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tra2tra3dz,size_16,color_FFFFFF,t_70" width="60%">

<img src="https://img-blog.csdnimg.cn/20200526185116629.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tra2tra3dz,size_16,color_FFFFFF,t_70" width="60%">


<img src="https://img-blog.csdnimg.cn/20200526185157435.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tra2tra3dz,size_16,color_FFFFFF,t_70" width="60%">
