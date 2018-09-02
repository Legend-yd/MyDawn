# MyDawn
Developing a APP similar to dripping, using java to write a simple server can interact with the client and driver side with socket, and multithread technology to achieve a server and multiple clients. The API map of Baidu is used to locate, route and search peripheral services.


通过android利用socket连接客户端，乘客间的信息通过服务端进行转达，最后实现客户端服务端之间的交互

在写之前要先加入百度地图sdk的jar包从百度地图开发者官网上就能下载

百度地图sdk下载：http://lbsyun.baidu.com/index.php?title=sdk/download&action#selected=mapsdk_basicmap,mapsdk_searchfunction,mapsdk_lbscloudsearch,mapsdk_calculationtool,mapsdk_radar

官网（可翻墙选择）：http://developer.android.com/sdk/index.html

不可翻墙选择：http://www.androiddevtools.cn/


而其中用到的一些与定位和路线规划有关的包可以在事例中查看导入

首先是xml的布局文件，其中editText的background是采用的自定义的样式，而在edittext前面的小点是用来美化界面的，可以直接删除


<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FCFCFC" >

    <RelativeLayout
        android:id="@+id/driverb_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="5dp" >

        <ImageButton
            android:id="@+id/driver_inf"
            android:layout_width="20dp"
            android:layout_height="25dp"
            android:layout_marginLeft="5dip"
            android:layout_marginTop="5dip"
            android:background="@drawable/customer"
            android:src="@drawable/transparent_mask" />

        <TextView
            android:id="@+id/driver_city"
            style="@style/customer"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerHorizontal="true" />
    </RelativeLayout>

    <com.baidu.mapapi.map.TextureMapView
        android:id="@+id/driver_mTexturemap"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/driverb_layout" >
    </com.baidu.mapapi.map.TextureMapView>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignWithParentIfMissing="false"
        android:layout_below="@id/driverb_layout"
        android:layout_centerHorizontal="true"
        android:background="#00000000" >

        <Button
            android:id="@+id/driver_pre"
            android:layout_width="50dp"
            android:layout_height="40dp"
            android:layout_marginLeft="60dip"
            android:layout_marginRight="2dip"
            android:background="@drawable/pre_"
            android:onClick="nodeClick" />

        <Button
            android:id="@+id/driver_next"
            android:layout_width="50dp"
            android:layout_height="40dp"
            android:layout_marginLeft="85dip"
            android:layout_marginRight="2dip"
            android:background="@drawable/next_"
            android:onClick="nodeClick" />
    </LinearLayout>

    <Button
        android:id="@+id/driver_change"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_below="@id/driverb_layout"
        android:layout_marginLeft="15dp"
        android:layout_marginTop="70dp" />

    <LinearLayout
        android:id="@+id/edit_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_marginBottom="10dp"
        android:layout_marginLeft="15dp"
        android:layout_marginRight="15dp"
        android:background="@drawable/rectangle_radius_fen"
        android:orientation="vertical" >

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="10dp"
            android:layout_marginTop="10dp"
            android:gravity="center"
            android:text="起点····终点"
            android:textSize="16sp"
            android:textStyle="bold" />

        <EditText
            android:id="@+id/driver_start"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="7dp"
            android:background="@drawable/creat_normal_edittext"
            android:drawableLeft="@drawable/radio_red"
            android:drawablePadding="5dp"
            android:textColor="#303030"
            android:textSize="15dp" />

        <EditText
            android:id="@+id/driver_end"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="7dp"
            android:background="@drawable/creat_normal_edittext"
            android:drawableLeft="@drawable/radio_blue"
            android:drawablePadding="5dp"
            android:hint="您要去哪儿"
            android:textSize="15dp" />

        <Button
            android:id="@+id/driver_go"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="5dp"
            android:layout_marginTop="5dp"
            android:background="@drawable/transparent_mask"
            android:gravity="center"
            android:onClick="searchButtonProcess"
            android:text="现在出发"
            android:textSize="18sp"
            android:textStyle="bold" />
    </LinearLayout>

</RelativeLayout>



