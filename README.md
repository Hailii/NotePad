# 期中作业
## NotePad
## 1、时间戳
![无法显示](/images/9.png)  
通过```SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss aa")```先将格式转换为标准时间格式  
然后修改**SimpleCursorAdapter**适配器来添加时间戳的对应列名  
```
   String[] dataColumns = {NotePad.Notes.COLUMN_NAME_TITLE,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};
   int[] viewIDs = {android.R.id.text1, R.id.tv_time_stamp};
```
最后在**NodeEditor**中修改**updateNote**方法来实现保存笔记时，时间戳会被修改  
```
   long now = System.currentTimeMillis();
   SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss aa");  
   String nowTime=sdf.format(new Date(now));
   // Sets up a map to contain values to be updated in the provider.
   ContentValues values = new ContentValues();
   values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, nowTime);
```
## 2、查询功能
支持模糊查询  
![无法显示](/images/12.png)
![无法显示](/images/8.png)
![无法显示](/images/9.png)  
在**NotesList**的**onOptionsItemSelected**方法中，添加如下代码：
```
   String[] search = {"_id", NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes
                                .COLUMN_NAME_CREATE_DATE};
   String selection = NotePad.Notes.COLUMN_NAME_TITLE + " like ?";
   String[] selectionArgs = {input_word + "%"};
   Cursor cursors = managedQuery(getIntent().getData(), search, selection,
                                selectionArgs, NotePad.Notes.DEFAULT_SORT_ORDER);
   cursors.moveToFirst();
   String[] data = {NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes
                                .COLUMN_NAME_CREATE_DATE};
   final int[] viewID = {android.R.id.text1, R.id.tv_time_stamp};
   SimpleCursorAdapter adapters = new SimpleCursorAdapter(NotesList
                                .this, R.layout.noteslist_item, cursors, data, viewID);
   setListAdapter(adapters);
```
## 3、修改字体大小
![无法显示](/images/1.jpg)
![无法显示](/images/2.jpg)
![无法显示](/images/3.jpg)  
先在编辑菜单中添加对应菜单项
```
    <item
        android:title="字体大小"
        android:id="@+id/menu_font"
        >
        <menu>
            <group >
                <item
                    android:id="@+id/font_10"
                    android:title="小"/>
                <item
                    android:id="@+id/font_16"
                    android:title="中"/>
                <item
                    android:id="@+id/font_20"
                    android:title="大"/>
            </group>
        </menu>
    </item>
```
然后在**NoteEditor**中的**onOptionsItemSelected**方法中，对应相应的id，添加如下代码：
```
   TextView txt;
   txt = (TextView) findViewById(R.id.note);
```
```
   case R.id.menu_font:
          switch (item.getItemId()){
          case R.id.font_10:
             txt.setTextSize(20);
             break;
          case R.id.font_16:
             txt.setTextSize(32);
             break;
          case R.id.font_20:
             txt.setTextSize(40);
             break;
          }
          break;
```
## 4、修改字体颜色
![无法显示](/images/4.jpg)  
先在编辑菜单中添加对应菜单项
```
    <item
        android:title="字体颜色"
        android:id="@+id/menu_font_color"
        >
        <menu>
            <group>
                <item
                    android:id="@+id/font_red"
                    android:title="红" />
                <item
                    android:id="@+id/font_black"
                    android:title="黑"/>
            </group>
        </menu>
    </item>
```
然后在**NoteEditor**中的**onOptionsItemSelected**方法中，对应相应的id，添加如下代码：
```
   case R.id.menu_font_color:
     switch (item.getItemId()){
         case R.id.font_red:
              txt.setTextColor(Color.RED);
              break;
         case R.id.font_black:
              txt.setTextColor(Color.BLACK);
              break;
     }
     break;
```
## 5、按步撤销
![无法显示](/images/5.png)
![无法显示](/images/6.png)  
对**EditText**进行监听，这里使用addTextChangedListener()方法进行监听
```
etFirstSchedule.addTextChangedListener(new TextWatcher() {
    @Override
    public void afterTextChanged(Editable s) {
            if (change) {
                TextPeriousCache textPeriousCache = new TextPeriousCache();
                textPeriousCache.setView_name("etFirstSchedule");
                textPeriousCache.setContent(s.toString());
                textPeriousCache.save();
            }
    }
});
```
```
ivNoteEditBack.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        int count = DataSupport.count(TextPeriousCache.class);
        if (count>0) {
            change = false;
            TextPeriousCache last = DataSupport.findLast(TextPeriousCache.class);
            TextNextCache textNextCache = new TextNextCache();
            textNextCache.setView_name(last.getView_name());
            textNextCache.setContent(last.getContent());
            textNextCache.save();
            last.delete();
            if (count==1){
                etFirstSchedule.setText("");
                change = true;
            }else {
                String content = DataSupport.findLast(TextPeriousCache.class).getContent();
                etFirstSchedule.setText(content);
                etFirstSchedule.setSelection(content.length());
                change = true;
            }
        }
    }
});
```
## 6、查看笔记
通过输入标题直接查看笔记  
![无法显示](/images/12.png)
![无法显示](/images/10.png)
![无法显示](/images/11.png)
```
    private Boolean doSearch(String title) {
        String[] TitleselectionArgs = {title};
        Cursor cursor = managedQuery(getIntent().getData(), PROJECTION, null, null, NotePad.Notes
                .DEFAULT_SORT_ORDER);
        mUri = cursor.getNotificationUri();
        cursor.moveToFirst();
        do {
            if (title.equals(cursor.getString(cursor.getColumnIndex("title")))) {
                mUri = ContentUris.withAppendedId(mUri, cursor.getInt(0));
                cursor.close();
                return true;
            }
        } while (cursor.moveToNext());
        cursor.close();
        return false;
    }
```
## 7、UI

![无法显示](/images/11.png)
