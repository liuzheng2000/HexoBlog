---
title: Android结课作业 音乐播放器 视频播放 游戏 附带源码
abbrlink: 3434175137
date: 2021-11-29 10:14:20
tags:
  - Android
  - 学校 结课作业
categories: Android
---
项目简介

一：整合了三个小功能   

实现了音乐播放器（Service+Activity实现）

视频播放器（本地视频播放——进度条控制）

2048游戏（人生版2048——分数统计显示）

二：项目概述（创建Activity的方式均是以Android Studio 4.1.2 自动创建 ）

音乐播放器

1页面展示如下：

![image-20210419212318302](https://img-blog.csdnimg.cn/img_convert/cbd8ff9b7e2d02e07961d275cbe8b20a.png)



2设计分析：

项目包含五个类，五个布局文件

frag1、frag2为java文件

Music_Activity为Activity文件，
 MusicService为Service文件，
 MainActivity为主类文件。

activity_main为MainActivity的主布局文件，显示运行APP时的主界面

activity_music为Music_Activity的布局文件、显示音乐播放器界面

music_list和item_layout一起组成了frag1的主布局文件，也就是音乐界面（APP的默认显示页面）

frag2_layout就是frag2的布局文件，主要显示的是专辑封面图片。

![image-20210419213014651](https://img-blog.csdnimg.cn/img_convert/4c4495fcf832803dba42bf7643711c67.png)

3、资源文件分析

在res文件夹下创建raw文件、放置MP3文件、在drawable文件下夹粘贴了音乐封面文件和播放器背景图片music_bg.jpg。还有背景选择器的btn_bg_selector.xml文件，如图所示：



![image-20210419213406874](https://img-blog.csdnimg.cn/img_convert/243bfd350a64ef444e3aee17617f3752.png)

二、开发环境

```
Win10+AndroidStudio4.12+MUMU模拟器
```

三：准备工具

1、选择几首自己下载好的音乐文件，命名为music0、music1、music2等。

2、选择每首歌对应的歌手图片，剪成圆形并且保存好，命名为music0、music1、music2等。
（一定要剪成圆形，椭圆都会影响运行效果）如果不太清楚图片如何剪成圆形，可以看这篇博客：如何将图片剪成圆形

3、准备一张音乐播放器的背景图片，命名为music_bg，再找一张图片用作专辑图片，命名为bg。

四：详细设计

1搭建主页面布局

MainActivity类   包含两个菜单文件：frag1（歌曲菜单）和frag2（专辑菜单）

默认展示歌曲菜单  点击切换 显示 frag2专辑菜单（代码如下：）

```JAVA
package com.example.a2048application;

import android.content.Intent;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.FrameLayout;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.ActionBar;
import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.FragmentManager;
import androidx.fragment.app.FragmentTransaction;

import lz.sdut.Music.frag1;
import lz.sdut.Music.frag2;


public class MainActivity extends AppCompatActivity implements View.OnClickListener{
    private FrameLayout content;
    private TextView Music_Song,Music_Album;
    private FragmentManager fm;   //管理类
    private FragmentTransaction ft;  //回调传递
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ActionBar actionBar = getSupportActionBar();
        if (actionBar != null) {
            actionBar.setDisplayOptions(ActionBar.DISPLAY_SHOW_CUSTOM);
            actionBar.setCustomView(R.layout.tittle);//设置标题样式
            TextView textView = (TextView) actionBar.getCustomView().findViewById(R.id.display_title);//获取标题布局的textview
            textView.setText("音乐播放");//设置标题名称，menuTitle为String字符串
            actionBar.setHomeButtonEnabled(true);//设置左上角的图标是否可以点击
            actionBar.setDisplayHomeAsUpEnabled(true);//给左上角图标的左边加上一个返回的图标
            actionBar.setDisplayShowCustomEnabled(true);// 使自定义的普通View能在title栏显示，即actionBar.setCustomView能起作用
        }
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        /**
         * 设置当前页面功能
         */
        content=(FrameLayout)findViewById(R.id.content); //不改变原activity布局的情况下 在activity顶部覆盖一层蒙版（类似与蒙版）

        Music_Song= (TextView)findViewById(R.id.menu1);  //歌曲菜单
        Music_Album= (TextView)findViewById(R.id.menu2); //专辑菜单


        Music_Song.setOnClickListener(this);//设置歌曲菜单监听器
        Music_Album.setOnClickListener(this);//设置专辑菜单监听器（公用一个监听，通过ID实现切换）

        fm = getSupportFragmentManager(); //若是继承FragmentActivity，fm=getFragmentManger();
        ft = fm.beginTransaction();
        ft.replace(R.id.content,new frag1()); //将FrameLayout中的内容切换成Flag1的内容
        ft.commit();//实现
    }

    @Override
    public void onClick(View v) {
        ft = fm.beginTransaction();
        switch (v.getId()) {
            case R.id.menu1:
                ft.replace(R.id.content, new frag1()); //将FrameLayout中的内容切换成Flag1的内容
                break;
            case R.id.menu2:
                ft.replace(R.id.content, new frag2()); //将FrameLayout中的内容切换成Flag2的内容
                break;
            default:
                break;
        }
        ft.commit();
    }





    /**
     * 设置顶部菜单栏选项
     * (页面跳转)
     * @param menu
     * @return
     */
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.region_right_menu, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.music_menu:
                Toast.makeText(this,"音乐播放",Toast.LENGTH_SHORT).show();
                Intent Music_intent = new Intent();
                Music_intent.setClass(this, MainActivity.class);
                startActivity(Music_intent);
                break;
            case R.id.movie_menu:
                Toast.makeText(this,"电影播放",Toast.LENGTH_SHORT).show();
                Intent Movie_intent = new Intent();
                Movie_intent.setClass(this, MovieActivity.class);
                startActivity(Movie_intent);
                break;
            case R.id.game_menu:
                Toast.makeText(this,"2048",Toast.LENGTH_SHORT).show();
                Intent Game_intent = new Intent();
                Game_intent.setClass(this, GameActivity.class);
                startActivity(Game_intent);
                break;
            case android.R.id.home:
                Toast.makeText(MainActivity.this,"返回",Toast.LENGTH_SHORT).show();
                return true;
            default:
                break;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

对应的Activity文件

```XML
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="喜欢的音乐"
            android:textSize="35dp"
            android:textColor="#87CEFA">
        </TextView>
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <TextView
            android:id="@+id/menu1"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="歌曲"
            android:textSize="25dp"
            android:textColor="#87CEFA">
        </TextView>

        <TextView
            android:id="@+id/menu2"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="专辑"
            android:textSize="25dp"
            android:textColor="#87CEFA">
        </TextView>

    </LinearLayout>
    <FrameLayout
        android:id="@+id/content"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="9">
    </FrameLayout>

</LinearLayout>

```

activity_main是布局文件，主要显示主界面，最上面放置了“我喜欢”的TextView控件，在底下放置了“歌曲”和“专辑”的TextView控件,效果如图：

![image-20210419214706637](https://img-blog.csdnimg.cn/img_convert/a4fae6bc6e8fbb52ce3db90598620d10.png)

2.2

创建flag1类（显示歌曲列表）模拟器运行出来的默认界面，代码如下：

```JAVA
package lz.sdut.Music;

import android.content.Intent;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.ListView;
import android.widget.TextView;

import androidx.fragment.app.Fragment;

import com.example.a2048application.MusicActivity;
import com.example.a2048application.R;

/**
 * Fragment依赖于Activity，不能独立存在
 * 一个Activity可以有多个Fragment
 * 一个Fragment可以被多个Activity重用
 * Fragment有自己的生命周期，并能接收输入事件
 * 可以在Activity运行时动态地添加或删除Fragment
 *模块化组件
 *
 * onAttach()：Fragment和Activity相关联时调用。可以通过该方法获取Activity引用，还可以通过getArguments()获取参数。
 * onCreate()：Fragment被创建时调用
 * onActivityCreated()：当Activity完成onCreate()时调用
 * onStart()：当Fragment可见时调用。
 * onResume()：当Fragment可见且可交互时调用
 * onPause()：当Fragment不可交互但可见时调用。
 * onStop()：当Fragment不可见时调用。
 * onDestroyView()：当Fragment的UI从视图结构中移除时调用。
 * onDestroy()：销毁Fragment时调用。
 * onDetach()：当Fragment和Activity解除关联时调用。
 *
 */
public class frag1 extends Fragment {
    private View view;
    public String[] name={"邓紫棋——光年之外","蔡健雅——红色高跟鞋","Taylor Swift——Love Story"};
    public static int[] icons = {R.drawable.music0, R.drawable.music1, R.drawable.music2};


    @Override
//    onCreate()：Fragment被创建时调用
    public View onCreateView( final LayoutInflater inflater,  ViewGroup container, Bundle savedInstanceState) {
        view = inflater.inflate(R.layout.music_list,null);
        ListView listView = view.findViewById(R.id.lv);
        MyBaseAdapter adapter = new MyBaseAdapter();
        //数据适配方法
        listView.setAdapter(adapter);
        //点击事件监听
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                Intent intent = new Intent(frag1.this.getContext(), MusicActivity.class);//创建Intent对象，启动check
                //将数据放入Intent对象         传输对象
                intent.putExtra("name",name[position]);
                intent.putExtra("position",String.valueOf(position));
                startActivity(intent);
            }
        });
        return view;
    }

    /**
     * 学会BaseAdapter其实只需要掌握四个方法：
     * getCount, getItem, getItemId, getView
     *
     *     getCount : 要绑定的条目的数目，比如格子的数量
     *     getItem : 根据一个索引（位置）获得该位置的对象
     *     getItemId : 获取条目的id
     *     getView : 获取该条目要显示的界面
     *
     */
    class MyBaseAdapter extends BaseAdapter {

        //创建数量
    @Override
    public int getCount() {
        return name.length;
    }

    //条目名字
    @Override
    public Object getItem(int position) {
        return name[position];
    }

    //条目ID
    @Override
    public long getItemId(int position) {
        return position;
    }

    //返回视图
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        //设置布局并返回
        View view = View.inflate(frag1.this.getContext(), R.layout.item_layout, null);
        TextView tv_name = view.findViewById(R.id.item_name);
        ImageView iv= view.findViewById(R.id.iv);
        tv_name.setText(name[position]);
        iv.setImageResource(icons[position]);
        return view;
    }
}
}

```

定义Flag1的需要用到的activity文件：

music_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <ListView
        android:id="@+id/lv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</LinearLayout>
```

item_layout.xml   

```XML
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">
    <ImageView
        android:id="@+id/iv"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_centerVertical="true"/>
    <RelativeLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_toRightOf="@+id/iv"
        android:layout_centerVertical="true">
        <TextView
            android:id="@+id/item_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="歌曲"
            android:textSize="15sp"
            android:textColor="#87CEFA"/>
    </RelativeLayout>
</RelativeLayout>
```

界面如下:

![image-20210419215628732](https://img-blog.csdnimg.cn/img_convert/5daa7e50ca1c3abb6a757f34e4800314.png)

创建flag2类（显示专辑列表）代码较为简单：

```java
package lz.sdut.Music;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

import com.example.a2048application.R;


/**
 * 仅插入了一个视图
 * 视图为一张图片
 */
public class frag2 extends Fragment {
    private View zj;

    @Nullable
    @Override
    public View onCreateView(@NonNull final LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        zj = inflater.inflate(R.layout.frag2_layout, null);
        View listView = zj.findViewById(R.id.iv);
        return zj;
    }
}
```

frag2_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageView
        android:id="@+id/zj"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/bg"/>
</LinearLayout>
```

效果如下：

![image-20210419221830910](https://img-blog.csdnimg.cn/img_convert/ad7ea1bf0e3c4d3afc26dbe5fcdebfb5.png)

3.3

创建服务类

![image-20210419222107674](https://img-blog.csdnimg.cn/img_convert/f8c055cb9f3d609acda6f8cd3b0acd52.png)

![image-20210420142230552](https://img-blog.csdnimg.cn/img_convert/6cd710fffa29d048a1f46aede47491cc.png)

创建MusicService代码：

```JAVA
package lz.sdut.Music;

import android.app.Service;
import android.content.Intent;
import android.media.MediaPlayer;
import android.net.Uri;
import android.os.Binder;
import android.os.Bundle;
import android.os.IBinder;
import android.os.Message;


import com.example.a2048application.MusicActivity;

import java.util.Timer;
import java.util.TimerTask;

public class Music_Service extends Service {
    private MediaPlayer player; //音乐播放器
    private Timer timer;    //计时器（进度条）

    public Music_Service() {
    }

    /**
     * //绑定一个组件方法
     * //生成一个音乐播放器
     * 音乐服务启动主方法
     * @param intent
     * @return
     */
    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        return new MusicControl();    //音乐播放控制器
    }

    /**
     * 创建时，启动的创建方法
     */
    @Override
    public void onCreate() {
        super.onCreate();
        player = new MediaPlayer(); //创建音乐播放器
    }

    /**
     * //添加计时器，用于播放进度条
     *
     */
    public void addTimer(){
        if (timer == null){
             timer = new Timer();
             //定时器方法
             TimerTask tack = new TimerTask(){   //Timer来讲就是一个调度器,而TimerTask呢只是一个实现了run方法的一个类,而具体的TimerTask需要由你自己来实现,

                 @Override
                 public void run() {
                    if (player == null) return;
                     int duration = player.getDuration();  //获取歌曲总时长
                     int currentPosition = player.getCurrentPosition();//获取播放进度
                     Message msg = MusicActivity.handler.obtainMessage();//创建消息对象
                     //将音乐的总时长和播放进度封装到消息对象中
                     Bundle bundle = new Bundle();
                     bundle.putInt("duration",duration);
                     bundle.putInt("currentPosition",currentPosition);
                     msg.setData(bundle);
                     //将消息发送到主线程的消息队列  线程对象   计时器使用多线程传递信息
                     /**
                      * handler是Android给我们提供用来更新UI的一套机制，也是一套消息处理机制，我们可以发消息，也可以通过它处理消息。
                      * 、Android为什么要设计只能用handler机制更新UI呢？
                      *
                      *     答：最根本的目的就是为了解决多线程并发的问题！
                      *
                      *            打个比方，如果在一个activity中有多个线程，并且没有加锁，就会出现界面错乱的问题。
                      *            但是如果对这些更新UI的操作都加锁处理，又会导致性能下降。
                      *
                      *           处于对性能的问题考虑，Android给我们提供这一套更新UI的机制我们只需要遵循这种机制就行了。
                      *           不用再去关系多线程的问题，所有的更新UI的操作，都是在主线程的消息队列中去轮训的。
                      *
                      *           大家都知道handler的作用有两个，发送消息和处理消息。
                      *           而handler发送的消息必须被送到指定MessageQueue（消息队列）中，也就是说，
                      *           如果想让handler正常工作，就必须有一个MessageQueue（消息队列），不过MessageQueue（消息队列）是由Looper来关系。
                      *           所以也可以说想让handler正常工作，必须在当前线程中有一个Looper对象。（请认真读）
                      */
                     MusicActivity.handler.sendMessage(msg);
                 }
             };
             //开始计时人物的5秒钟，第一次执行task任务，以后每500毫秒执行一次
            timer.schedule(tack,5,500);
        }

    }

    public class MusicControl extends Binder{ //Binder是一种跨进程的通信方式
        public void play(int i){
            //String path  ma3位置文件地址
            Uri uri=Uri.parse("android.resource://"+getPackageName()+"/raw/"+"music"+i);
            try{
                player.reset();  //重置音乐播放器
                //加载多媒体文件
                player = MediaPlayer.create(getApplicationContext(), uri);
                player.start();//播放音乐
                addTimer();//添加计时器
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        public void pausePlay(){
            player.pause(); //暂停播放音乐
        }
        public void continuePlay(){
            player.start();//继续播放音乐
        }
        public void seekTo(int progress){
            player.seekTo(progress); //设置音乐的播放位置
        }
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        if (player == null){
            return;
        }
        if (player.isPlaying()){
            player.stop();
        }
        player.release();  //释放占用的音乐
        player = null;     //将player置为空
    }
}
```

![image-20210420144330182](https://img-blog.csdnimg.cn/img_convert/004925d3dcf176f311c8bdf59cefc8ab.png)

4.4创建Music_Activity（主音乐界面）

Music_Activity类：通过onClick方法控制着音乐的播放、暂停、继续播放和退出功能。它和MusicService进行绑定连接。在音乐播放时显示歌曲总时长，还有歌曲当前播放时长，控制滑动条的移动。代码如下：

```JAVA
package com.example.a2048application;

import android.animation.ObjectAnimator;
import android.content.ComponentName;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Build;
import android.os.Bundle;
import android.os.Handler;
import android.os.IBinder;
import android.os.Message;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.animation.LinearInterpolator;
import android.widget.ImageView;
import android.widget.SeekBar;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.RequiresApi;
import androidx.appcompat.app.AppCompatActivity;

import lz.sdut.Music.Music_Service;
import lz.sdut.Music.frag1;

import static java.lang.Integer.parseInt;

public class MusicActivity extends AppCompatActivity implements View.OnClickListener{
    private static SeekBar sb;
    private static TextView tv_progress,tv_total,name_song;
    private ObjectAnimator animator;
    private Music_Service.MusicControl musicControl;
    String name;
    Intent intent1,intent2;
    MyServiceConn conn;
    private boolean isUnbind = false;  //用于记录服务是否被解绑

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_music);
        /**
         * 对于getIntent()这个方法，androidAPI文档中是这样解释的：
         * Retrieve the raw Intent contained in this Item.
         * 意思就是将该项目中包含的原始intent检索出来，
         * 而Intent intent=getIntent();
         * 是将检索出来的intent赋值给一个Intent类型的变量intent
         * 上句中，第一个intent是本身就有的一个intent，而后面的是个变量名，需要赋值
         */
        intent1 = getIntent();
        //初始化启动
        init();
    }

    private void init() {
        /**
         * 页面赋值
         */
        tv_progress = (TextView)findViewById(R.id.tv_progress);
        tv_total = (TextView) findViewById(R.id.tv_total);
        sb = (SeekBar) findViewById(R.id.sb);
        name_song = (TextView)findViewById(R.id.song_name);

        findViewById(R.id.btn_play).setOnClickListener(this);
        findViewById(R.id.btn_pause).setOnClickListener(this);
        findViewById(R.id.btn_continue_play).setOnClickListener(this);
        findViewById(R.id.btn_exit).setOnClickListener(this);

        /**
         * 获取传递进来的MusicName
         */
        name = intent1.getStringExtra("name");
        name_song.setText(name);
        //创建 Music_Service.class
        intent2 = new Intent(this, Music_Service.class);   //创建意图对象
        conn  = new MyServiceConn(); //创建服务连接对象


        /**
         * bindService(intent,mConnection, Context.BIND_AUTO_CREATE);
         *
         *     第一个bindService()的参数是一个明确指定了要绑定的service的Intent．
         *
         *     第二个参数是ServiceConnection对象．
         *
         *     第三个参数是一个标志，它表明绑定中的操作．它一般应是BIND_AUTO_CREATE，
         *     这样就会在service不存在时创建一个．其它可选的值是BIND_DEBUG_UNBIND和BIND_NOT_FOREGROUND,不想指定时设为0即可．
         */
        bindService(intent2,conn,BIND_AUTO_CREATE);//绑定服务
        //为滑动监听条添加事件监听
        sb.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @RequiresApi(api = Build.VERSION_CODES.KITKAT)
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                //进度条改变时，调用此方法
                if (progress == seekBar.getMax()){  //当滑动条到末端时，结束事件
                    //转动事件停止
                    animator.pause(); //停止播放事件
                }
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {   //滑动条开始滑动时调用

            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {  // 滑动条停止时调用
                //根据拖动的进度改变音乐的播放进度
                int progress = seekBar.getProgress();  //获取seekBar的进度
                musicControl.seekTo(progress);  //改变播放进度
            }
        });
        ImageView iv_music = findViewById(R.id.iv_music);
        String position = intent1.getStringExtra("position");
        int i  = Integer.parseInt(position);
        iv_music.setImageResource(frag1.icons[i]);

        /**
         * 动画播放（图片旋转）
         */
        animator = ObjectAnimator.ofFloat(iv_music,"rotation",0f,360.0f);
        animator.setDuration(10000); // 动画旋转一周的时间为10秒
        animator.setInterpolator(new LinearInterpolator()); //匀速
        animator.setRepeatCount(-1);//表示动画无限循环播放
    }

    /**
     *  Handler机制也可叫异步消息机制，它主要由4个部分组成：Message,Handler,MessageQueue,Looper,
     *
     * 1.Message
     *   Message是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不同线程之间交换数据。
     * 使用Message的arg1和arg2便可携带int数据，使用obj便可携带Object类型数据。
     *
     * 2.Handler
     *   Handler顾名思义就是处理者的意思，它只要用于在子线程发送消息对象Message,在UI线程处理消息对象Message，
     * 在子线程调用sendMessage方法发送消息对象Message，而发送的消息经过一系列地辗转之后最终会被传递到Handler的handleMessage方法中,
     * 最终在handleMessage方法中消息对象Message被处理。
     *
     * 3.MessageQueue
     *   MessageQueue就是消息队列的意思,它只要用于存放所有通过Handler发送过来的消息。这部分消息会一直存放于消息队列当中，等待被处理。
     * 每个线程中只会有一个MessageQueue对象，请牢记这句话。其实从字面上就可以看出，MessageQueue底层数据结构是队列，而且这个队列只存放Message对象。
     *
     * 4.Looper
     *   Looper是每个线程中的MessageQueue的管家，调用Looper的loop()方法后，就会进入到一个无限循环当中，
     * 然后每当MesssageQueue中存在一条消息，Looper就会将这条消息取出，并将它传递到Handler的handleMessage()方法中。每个线程只有一个Looper对象。
     *
     *   了解了上述Handler机制的4个成员后，我们再来把思路理一遍：首先在UI线程我们创建了一个Handler实例对象，
     * 无论是匿名内部类还是自定义类生成的Handler实例对象，我们都需要对handleMessage方法进行重写，
     * 在handleMessage方法中我们可以通过参数msg来写接受消息过后UIi线程的逻辑处理，
     * 接着我们创建子线程，在子线程中需要更新UI的时候，新建一个Message对象，并且将消息的数据记录在这个消息对象Message的内部，比如arg1,arg2,obj等，
     * 然后通过前面的Handler实例对象调用sendMessge方法把这个Message实例对象发送出去，之后这个消息会被存放于MessageQueue中等待被处理，
     * 此时MessageQueue的管家Looper正在不停的把MessageQueue存在的消息取出来，通过回调dispatchMessage方法将消息传递给Handler的handleMessage方法，
     * 最终前面提到的消息会被Looper从MessageQueue中取出来传递给handleMessage方法，最终得到处理。这就是Handler机制整个的工作流程。
     * 应该都差不多懂了吧，感觉我写的很接地气啊。
     * ————————————————
     * 版权声明：本文为CSDN博主「ttxs99989」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
     * 原文链接：https://blog.csdn.net/ttxs99989/article/details/81814037
     *
     */
    //处理进度条
    public static Handler handler = new Handler(){  //创建消息处理对象

        @Override
        public void handleMessage(@NonNull Message msg) {
            Bundle bundle = msg.getData();//获取从子线程发送过来的音乐播放度
            int duration = bundle.getInt("duration");
            int currentPosition = bundle.getInt("currentPosition");
            sb.setMax(duration);
            sb.setProgress(currentPosition);
            //歌曲总时长
            int minute = duration/1000/60;
            int second = duration/1000%60;
            String strMinute = null;
            String strSecond = null;
            if (minute < 10){ //如果歌曲时长小于10分钟
                strMinute="0"+minute; //在分钟前面加0
            }else {
                strMinute = minute+"";
            }

            if (second < 10){//如果歌曲中的秒钟小于10
                strSecond="0"+second;//在秒钟前面加一个0
            }else {
                strSecond = second+"";
            }
            tv_total.setText(strMinute+":"+strSecond);
            //歌曲当前的播放时长
            minute = currentPosition/1000/60;
            second = currentPosition/1000%60;
            if (minute < 10){ //如果歌曲时长小于10分钟
                strMinute="0"+minute; //在分钟前面加0
            }else {
                strMinute = minute+" ";
            }

            if (second < 10){//如果歌曲中的秒钟小于10
                strSecond="0"+second;//在秒钟前面加一个0
            }else {
                strSecond = second+"";
            }
            tv_progress.setText(strMinute+":"+strSecond);
        }
    };

    /**
     * 创建服务连接对象
     */
    class MyServiceConn implements ServiceConnection{  // 用于实现连接服务

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            //赋值MusicControl 主控制类
            musicControl =  (Music_Service.MusicControl) service;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    }

    private void unbind(boolean isUnbind){
        if (!isUnbind){   //判断服务是否被解绑
            musicControl.pausePlay();  //暂停播放音乐
            unbindService(conn);      //解绑服务
        }
    }


    @RequiresApi(api = Build.VERSION_CODES.KITKAT)
    @Override
    //点击事件
    public void onClick(View v) {
            switch (v.getId()){
                case R.id.btn_play:  //设置播放按钮点击事件
                    String position = intent1.getStringExtra("position");
                    int i = parseInt(position);
                    musicControl.play(i);
                    animator.start();
                    break;
                case R.id.btn_pause:  //暂停播放事件
                    musicControl.pausePlay();
                    animator.pause();
                    break;
                case R.id.btn_continue_play: //继续播放按钮
                    musicControl.continuePlay();
                    animator.start();
                    break;
                case R.id.btn_exit:
                    unbind(isUnbind);
                    isUnbind = true;
                    finish();
                    break;
            }
    }
    //销毁事件
    //解除绑定
    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbind(isUnbind);
    }

    /**
     * 设置顶部菜单栏选项
     * (页面跳转)
     * @param menu
     * @return
     */
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.region_right_menu, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.music_menu:
                Toast.makeText(this,"音乐播放",Toast.LENGTH_SHORT).show();
                Intent Music_intent = new Intent();
                Music_intent.setClass(this, MainActivity.class);
                startActivity(Music_intent);
                break;
            case R.id.movie_menu:
                Toast.makeText(this,"电影播放",Toast.LENGTH_SHORT).show();
                Intent Movie_intent = new Intent();
                Movie_intent.setClass(this, MovieActivity.class);
                startActivity(Movie_intent);
                break;
            case R.id.game_menu:
                Toast.makeText(this,"2048",Toast.LENGTH_SHORT).show();
                Intent Game_intent = new Intent();
                Game_intent.setClass(this, GameActivity.class);
                startActivity(Game_intent);
                break;
            case android.R.id.home:
                Toast.makeText(this,"返回",Toast.LENGTH_SHORT).show();
                return true;
            default:
                break;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

对应的Activity视图

activity_music: 显示音乐播放器界面，包括图片转动，歌曲名传值，还有播放、暂停播放、继续播放和退出四个控制按钮。效果如图：

![image-20210420151306512](https://img-blog.csdnimg.cn/img_convert/9521f0e503e78e47ea344bda3452b49e.png)

代码如下:

```XML
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/music_bg"
    tools:context=".MusicActivity"
    android:gravity="center"
    android:orientation="vertical">
    <ImageView
        android:id="@+id/iv_music"
        android:layout_width="240dp"
        android:layout_height="240dp"
        android:layout_gravity="center_horizontal"
        android:layout_margin="15dp"
        android:src="@drawable/music0"/>
    <TextView
        android:id="@+id/song_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="歌曲名"
        android:textSize="20sp"/>
    <SeekBar
        android:id="@+id/sb"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="8dp"
        android:paddingRight="8dp">
        <TextView
            android:id="@+id/tv_progress"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="00:00"/>
        <TextView
            android:id="@+id/tv_total"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:text="00:00"/>
    </RelativeLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <Button
            android:id="@+id/btn_play"
            android:layout_width="0dp"
            android:layout_height="40dp"
            android:layout_margin="8dp"
            android:layout_weight="1"
            android:background="@drawable/btn_bg_selector"
            android:text="播放音乐"/>
        <Button
            android:id="@+id/btn_pause"
            android:layout_width="0dp"
            android:layout_height="40dp"
            android:layout_margin="8dp"
            android:layout_weight="1"
            android:background="@drawable/btn_bg_selector"
            android:text="暂停播放"/>
        <Button
            android:id="@+id/btn_continue_play"
            android:layout_width="0dp"
            android:layout_height="40dp"
            android:layout_margin="8dp"
            android:layout_weight="1"
            android:background="@drawable/btn_bg_selector"
            android:text="继续播放"/>
        <Button
            android:id="@+id/btn_exit"
            android:layout_width="0dp"
            android:layout_height="40dp"
            android:layout_margin="8dp"
            android:layout_weight="1"
            android:background="@drawable/btn_bg_selector"
            android:text="退出"/>
    </LinearLayout>

</LinearLayout>
```

5.5创建相应的资源类

在res文件夹下新建一个raw文件夹，将刚刚准备好的视频文件复制到raw文件夹中。对应的封面圆形图片、背景图片和专辑图片都复制到drawable中，这里博主只选择了三首歌曲和三张封面圆形图片，歌曲数量由大家自己设置，没有限制。

至此 		音乐播放器部分完成  整体效果如图

![image-20210420151627676](https://img-blog.csdnimg.cn/img_convert/0149295a75663d9b7e44fe7688eb523b.png)

主体代码完成，剩余部分请看具体的代码资源

二：视频播放器部分（仅实现本地播放器，且实现较为简单）

设计分析

主要包括一个类，一个Activity和一个资源文件

一：将相应的MP4文件保存至/raw文件夹中效果如图

![](https://img-blog.csdnimg.cn/img_convert/4c7fdd031273266721032d0a60a82414.png)

二：创建MovieActivity

MovieActivity中实现了进度条的设置，播放唯一的影片（可以改成相应的url地址）

绑定视频播放器、绑定进度条。等等

```JAVA
package com.example.a2048application;

import android.content.Intent;
import android.media.MediaPlayer;
import android.net.Uri;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.SeekBar;
import android.widget.TextView;
import android.widget.Toast;
import android.widget.VideoView;

import androidx.appcompat.app.ActionBar;
import androidx.appcompat.app.AppCompatActivity;

public class MovieActivity extends AppCompatActivity {

    private final String TAG = "main";
    private Button btn_play,btn_pause,btn_replay,btn_stop;
    private SeekBar seekBar;
    private VideoView vv_video;
    private boolean isPlaying;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_movie);

        seekBar = (SeekBar)findViewById(R.id.seekBar);
//        et_path = (EditText)findViewById(R.id.et_path);
        vv_video = (VideoView) findViewById(R.id.vv_videoview);


        /**
         * 绑定资源文件
         */
        btn_play = (Button) findViewById(R.id.btn_movie_play);
        btn_pause = (Button) findViewById(R.id.btn_movie_pause);
        btn_replay = (Button) findViewById(R.id.btn_movie_replay);
        btn_stop = (Button) findViewById(R.id.btn_movie_stop);
        /**
         * 绑定监听信息
         */
        btn_play.setOnClickListener(click);
        btn_pause.setOnClickListener(click);
        btn_replay.setOnClickListener(click);
        btn_stop.setOnClickListener(click);

        //为进度条添加进度改变事件
        seekBar.setOnSeekBarChangeListener(change);

        //设置顶部菜单栏
        ActionBar actionBar = getSupportActionBar();
        if (actionBar != null) {
            actionBar.setDisplayOptions(ActionBar.DISPLAY_SHOW_CUSTOM);
            actionBar.setCustomView(R.layout.tittle);//设置标题样式
            TextView textView = (TextView) actionBar.getCustomView().findViewById(R.id.display_title);//获取标题布局的textview
            textView.setText("电影播放");//设置标题名称，menuTitle为String字符串
            actionBar.setHomeButtonEnabled(true);//设置左上角的图标是否可以点击
//            actionBar.setDisplayHomeAsUpEnabled(true);//给左上角图标的左边加上一个返回的图标
            actionBar.setDisplayShowCustomEnabled(true);// 使自定义的普通View能在title栏显示，即actionBar.setCustomView能起作用
        }
    }

    private SeekBar.OnSeekBarChangeListener change = new SeekBar.OnSeekBarChangeListener() {
        @Override
        public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {

        }

        @Override
        public void onStartTrackingTouch(SeekBar seekBar) {

        }

        @Override
        public void onStopTrackingTouch(SeekBar seekBar) {
            //当进度条停止修改时触发
            //取得当前进度条的刻度
            int progress = seekBar.getProgress();
            if (vv_video != null && vv_video.isPlaying()){
                //视频播放位置改变  设置当前位置
                vv_video.seekTo(progress);
            }
        }
    };
    /**
     * 根据ID执行不同的操作
     */
    private View.OnClickListener click = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            switch (v.getId()){
                case R.id.btn_movie_play:
                    try {
                        play(0);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    break;
                case R.id.btn_movie_pause:
                    pause();
                    break;
                case R.id.btn_movie_replay:
                    replay();
                    break;
                case R.id.btn_movie_stop:
                    stop();
                    break;
                default:
                    break;
            }
        }
    };

    protected void play(int msec) throws InterruptedException {

        /**
         * 视频信息播放
         */
        Log.i(TAG, "指定文件路径");
        vv_video.setVideoURI(Uri.parse("android.resource://"+getPackageName()+"/raw/big_buck_bunny"));
        Log.i(TAG, "开始播放");
        vv_video.start();

        /**
         * 视频资源准备完成
         * 设置进度条的最大值为视频最长播放信息
         */
        vv_video.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
            @Override
            public void onPrepared(MediaPlayer mp) {
                vv_video.getDuration();
                System.out.println(vv_video.getDuration());
                seekBar.setMax(vv_video.getDuration());
            }
        });

        //按照初始位置进行播放
        vv_video.seekTo(msec);
        //设置进度条的最大长度为视频流的最大播放时长
        Thread.sleep(300);

        //开始线程，更新进度条的刻度
        new Thread() {
            @Override
            public void run() {
                try {
                    isPlaying = true;
                    while (true) {
                        //如果正在播放，没0.5豪秒更新一次进度条
                        if (isPlaying){
                            int currentPosition = vv_video.getCurrentPosition();
                            seekBar.setProgress(currentPosition);
                            sleep(500);
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }.start();

        btn_play.setEnabled(false);
        vv_video.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
            @Override
            public void onCompletion(MediaPlayer mp) {
                //在播放完成完毕后回调进度条
                btn_play.setEnabled(true);
            }
        });
        vv_video.setOnErrorListener(new MediaPlayer.OnErrorListener() {
            @Override
            public boolean onError(MediaPlayer mp, int what, int extra) {
                //发生错误时重新播放
                try {
                    play(0);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return false;
            }
        });
    }
    /**
     * 重新开始播放
     */
    protected void replay(){
        if (vv_video != null && vv_video.isPlaying()){
            vv_video.seekTo(0);
            int i = 0;
            Toast.makeText(this,"重新播放",i).show();
            btn_pause.setText("暂停");
            seekBar.setProgress(0);
            isPlaying = true;
            return;
        }
    }

    /**
     * 暂停或者继续
     */
    protected void pause(){
        if (btn_pause.getText().toString().trim().equals("继续")){
            btn_pause.setText("暂停");
            vv_video.start();
            int i = 0;
            Toast.makeText(this,"继续",i).show();
            isPlaying = true;
            return;
        }

        if (vv_video != null && vv_video.isPlaying()){
            vv_video.pause();
            int i = 0;
            btn_pause.setText("继续");
            Toast.makeText(this,"暂停",i).show();
            isPlaying = false;
            return;
        }
    }

    /**
     * 停止播放
     * @param
     * @return
     */
    protected  void stop(){
        if (vv_video != null && vv_video.isPlaying()){
            vv_video.stopPlayback();
            btn_play.setEnabled(true);
            seekBar.setProgress(0);
            isPlaying = false;
        }
    }

    /**
     * 设置顶部菜单栏选项
     * (页面跳转)
     * @param menu
     * @return
     */
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.region_right_menu, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.music_menu:
                Toast.makeText(this,"音乐播放",Toast.LENGTH_SHORT).show();
                Intent Music_intent = new Intent();
                Music_intent.setClass(this, MainActivity.class);
                startActivity(Music_intent);
                break;
            case R.id.movie_menu:
                Toast.makeText(this,"电影播放",Toast.LENGTH_SHORT).show();
                Intent Movie_intent = new Intent();
                Movie_intent.setClass(this, MovieActivity.class);
                startActivity(Movie_intent);
                break;
            case R.id.game_menu:
                Toast.makeText(this,"2048",Toast.LENGTH_SHORT).show();
                Intent Game_intent = new Intent();
                Game_intent.setClass(this, GameActivity.class);
                startActivity(Game_intent);
                break;
            case android.R.id.home:
                Toast.makeText(this,"返回",Toast.LENGTH_SHORT).show();
                return true;
            default:
                break;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

其对应的Activity视图如下:

![image-20210420194308561](https://img-blog.csdnimg.cn/img_convert/864f37e9d84f5e1fbc03e18bfb6a4d68.png)

代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    tools:context=".MovieActivity">

<!--    <EditText-->
<!--        android:id="@+id/et_path"-->
<!--        android:layout_width="match_parent"-->
<!--        android:layout_height="wrap_content"-->
<!--        android:text="src\main\res\raw\big_buck_bunny.mp4"/>-->

    <SeekBar
        android:id="@+id/seekBar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal" >

        <Button
            android:id="@+id/btn_movie_play"
            android:layout_width="0dip"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="播放" />

        <Button
            android:id="@+id/btn_movie_pause"
            android:layout_width="0dip"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="暂停" />

        <Button
            android:id="@+id/btn_movie_replay"
            android:layout_width="0dip"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="重播" />

        <Button
            android:id="@+id/btn_movie_stop"
            android:layout_width="0dip"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="停止" />
    </LinearLayout>

    <VideoView
        android:id="@+id/vv_videoview"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent" />

</LinearLayout>
```

在模拟器中展示如下：

![image-20210420194553768](https://img-blog.csdnimg.cn/img_convert/0acbc472e99a158ebc781a8d92adadcf.png)

主体代码如上所言，部分代码请参考项目

三：整合相应的2048人生版（不会实现）

具体代码请参考实例项目

链接：https://pan.baidu.com/s/1M-qK3rmM64xoMHdoNcuSmg 
提取码：44hh 
复制这段内容后打开百度网盘手机App，操作更方便哦

项目参考地址：https://blog.csdn.net/qq_42257666/article/details/105555550（基本抄袭）
