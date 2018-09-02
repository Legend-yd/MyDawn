## MyDawn
Developing a APP similar to dripping, using java to write a simple server can interact with the client and driver side with socket, and multithread technology to achieve a server and multiple clients. The API map of Baidu is used to locate, route and search peripheral services.


通过android利用socket连接客户端，乘客间的信息通过服务端进行转达，最后实现客户端服务端之间的交互

在写之前要先加入百度地图sdk的jar包从百度地图开发者官网上就能下载

[百度地图sdk下载](http://lbsyun.baidu.com/index.php?title=sdk/download&action#selected=mapsdk_basicmap,mapsdk_searchfunction,mapsdk_lbscloudsearch,mapsdk_calculationtool,mapsdk_radar
)

官网（可翻墙选择）：http://developer.android.com/sdk/index.html

不可翻墙选择：http://www.androiddevtools.cn/


而其中用到的一些与定位和路线规划有关的包可以在事例中查看导入

#首先是xml的布局文件，其中editText的background是采用的自定义的样式，而在edittext前面的小点是用来美化界面的，可以直接删除

```
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
```


接下来是java代码

主要实现功能为两个：定位和路线规划：


```
package com.ly.dawn;

import com.baidu.location.BDLocation;
import com.baidu.location.BDLocationListener;
import com.baidu.location.LocationClient;
import com.baidu.location.LocationClientOption;
import com.baidu.mapapi.SDKInitializer;
import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.BitmapDescriptor;
import com.baidu.mapapi.map.BitmapDescriptorFactory;
import com.baidu.mapapi.map.InfoWindow;
import com.baidu.mapapi.map.MapPoi;
import com.baidu.mapapi.map.MapStatus;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MyLocationConfiguration;
import com.baidu.mapapi.map.MyLocationConfiguration.LocationMode;
import com.baidu.mapapi.map.MyLocationData;
import com.baidu.mapapi.map.TextureMapView;
import com.baidu.mapapi.model.LatLng;
import com.baidu.mapapi.search.core.RouteLine;
import com.baidu.mapapi.search.core.SearchResult;
import com.baidu.mapapi.search.route.BikingRouteLine;
import com.baidu.mapapi.search.route.BikingRoutePlanOption;
import com.baidu.mapapi.search.route.BikingRouteResult;
import com.baidu.mapapi.search.route.DrivingRouteLine;
import com.baidu.mapapi.search.route.DrivingRoutePlanOption;
import com.baidu.mapapi.search.route.DrivingRouteResult;
import com.baidu.mapapi.search.route.OnGetRoutePlanResultListener;
import com.baidu.mapapi.search.route.PlanNode;
import com.baidu.mapapi.search.route.RoutePlanSearch;
import com.baidu.mapapi.search.route.TransitRouteLine;
import com.baidu.mapapi.search.route.TransitRoutePlanOption;
import com.baidu.mapapi.search.route.TransitRouteResult;
import com.baidu.mapapi.search.route.WalkingRouteLine;
import com.baidu.mapapi.search.route.WalkingRoutePlanOption;
import com.baidu.mapapi.search.route.WalkingRouteResult;
import com.ly.overlayutil.BikingRouteOverlay;
import com.ly.overlayutil.DrivingRouteOverlay;
import com.ly.overlayutil.OverlayManager;
import com.ly.overlayutil.TransitRouteOverlay;
import com.ly.overlayutil.WalkingRouteOverlay;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.Window;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageButton;
import android.widget.TextView;
import android.widget.Toast;

/**
 * 此demo用来展示如何进行驾车、步行、公交路线搜索并在地图使用RouteOverlay、TransitOverlay绘制
 * 同时展示如何进行节点浏览并弹出泡泡
 */
public class DriverMenu extends Activity implements BaiduMap.OnMapClickListener, OnGetRoutePlanResultListener {
    // 浏览路线节点相关
    Button mBtnPre = null; // 上一个节点
    Button mBtnNext = null; // 下一个节点
    int nodeIndex = -1; // 节点索引,供浏览节点时使用
    RouteLine route = null;
    private String loaclcity = null;
    OverlayManager routeOverlay = null;
    private Button requestLocButton, go;
    private ImageButton driver_inf;
    private LocationMode mCurrentMode;
    BitmapDescriptor mCurrentMarker;
    boolean useDefaultIcon = false;
    private TextView popupText = null, driver_city; // 泡泡view
    private EditText start_edit, end_edit;
    boolean isFirstLoc = true; // 是否首次定位

    // 地图相关，使用继承MapView的MyRouteMapView目的是重写touch事件实现泡泡处理
    // 如果不处理touch事件，则无需继承，直接使用MapView即可
    // 地图控件
    private TextureMapView mMapView = null;
    private BaiduMap mBaidumap;
    // 搜索相关
    RoutePlanSearch mSearch = null; // 搜索模块，也可去掉地图模块独立使用

    // 定位相关
    LocationClient mLocClient;
    public MyLocationListenner myListener = new MyLocationListenner();

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 在使用SDK各组件之前初始化context信息，传入ApplicationContext
        // 注意该方法要再setContentView方法之前实现
        this.requestWindowFeature(Window.FEATURE_NO_TITLE);// 设置标题栏不可用
        SDKInitializer.initialize(getApplicationContext());
        setContentView(R.layout.driver_menu);
        requestLocButton = (Button) findViewById(R.id.driver_change);

        mCurrentMode = LocationMode.COMPASS;
        requestLocButton.setText("罗盘");
        OnClickListener btnClickListener = new OnClickListener() {
            public void onClick(View v) {
                switch (mCurrentMode) {
                case NORMAL:
                    requestLocButton.setText("跟随");
                    mCurrentMode = LocationMode.FOLLOWING;
                    mBaidumap.setMyLocationConfigeration(
                            new MyLocationConfiguration(mCurrentMode, true, mCurrentMarker));
                    break;
                case COMPASS:
                    requestLocButton.setText("普通");
                    mCurrentMode = LocationMode.NORMAL;
                    mBaidumap.setMyLocationConfigeration(
                            new MyLocationConfiguration(mCurrentMode, true, mCurrentMarker));
                    break;
                case FOLLOWING:
                    requestLocButton.setText("罗盘");
                    mCurrentMode = LocationMode.COMPASS;
                    mBaidumap.setMyLocationConfigeration(
                            new MyLocationConfiguration(mCurrentMode, true, mCurrentMarker));
                    break;
                default:
                    break;
                }
            }
        };
        requestLocButton.setOnClickListener(btnClickListener);

        // 初始化地图
        inintmap();
        initview();
        // 地图点击事件处理
        mBaidumap.setOnMapClickListener(this);
        // 初始化搜索模块，注册事件监听
        mSearch = RoutePlanSearch.newInstance();
        mSearch.setOnGetRoutePlanResultListener(this);

    }

    public void inintmap() {
        // 地图初始化
        mMapView = (TextureMapView) findViewById(R.id.driver_mTexturemap);
        mBaidumap = mMapView.getMap();
        // 开启定位图层
        mBaidumap.setMyLocationEnabled(true);
        // 定位初始化
        mLocClient = new LocationClient(this);
        mLocClient.registerLocationListener(myListener);
        LocationClientOption option = new LocationClientOption();
        option.setOpenGps(true); // 打开gps
        option.setCoorType("bd09ll"); // 设置坐标类型
        option.setScanSpan(1000);
        option.setLocationMode(LocationClientOption.LocationMode.Hight_Accuracy);
        // 可选，默认高精度，设置定位模式，高精度，低功耗，仅设备
        option.setIsNeedAddress(true);// 可选，设置是否需要地址信息，默认不需要
        option.setIsNeedLocationPoiList(true);// 可选，默认false，设置是否需要POI结果，可以在BDLocation.getPoiList里得到
        mLocClient.setLocOption(option);
        mLocClient.start();
    }

    public void initview() {
        mBtnPre = (Button) findViewById(R.id.driver_pre);
        mBtnNext = (Button) findViewById(R.id.driver_next);
        mBtnPre.setVisibility(View.INVISIBLE);
        mBtnNext.setVisibility(View.INVISIBLE);
        start_edit = (EditText) findViewById(R.id.driver_start);
        end_edit = (EditText) findViewById(R.id.driver_end);
        driver_city = (TextView) findViewById(R.id.driver_city);
        driver_inf = (ImageButton) findViewById(R.id.driver_inf);
    }

    /**
     * 发起路线规划搜索示例
     *
     * @param v
     */
    public void searchButtonProcess(View v) {
        // 重置浏览节点的路线数据
        route = null;
        mBtnPre.setVisibility(View.INVISIBLE);
        mBtnNext.setVisibility(View.INVISIBLE);
        mBaidumap.clear();
        // 设置起终点信息，对于tranist search 来说，城市名无意义
        PlanNode stNode = PlanNode.withCityNameAndPlaceName(loaclcity, start_edit.getText().toString());
        PlanNode enNode = PlanNode.withCityNameAndPlaceName(loaclcity, end_edit.getText().toString());

        // 实际使用中请对起点终点城市进行正确的设定
        if (v.getId() == R.id.driver_go) {
            mSearch.drivingSearch((new DrivingRoutePlanOption()).from(stNode).to(enNode));
        } else if (v.getId() == R.id.transit) {
            mSearch.transitSearch((new TransitRoutePlanOption()).from(stNode).city(loaclcity).to(enNode));
        } else if (v.getId() == R.id.walk) {
            mSearch.walkingSearch((new WalkingRoutePlanOption()).from(stNode).to(enNode));
        } else if (v.getId() == R.id.bike) {
            mSearch.bikingSearch((new BikingRoutePlanOption()).from(stNode).to(enNode));
        }
    }

    /**
     * 节点浏览示例
     *
     * @param v
     */
    public void nodeClick(View v) {
        if (route == null || route.getAllStep() == null) {
            return;
        }
        if (nodeIndex == -1 && v.getId() == R.id.driver_pre) {
            return;
        }
        // 设置节点索引
        if (v.getId() == R.id.driver_next) {
            if (nodeIndex < route.getAllStep().size() - 1) {
                nodeIndex++;
            } else {
                return;
            }
        } else if (v.getId() == R.id.driver_pre) {
            if (nodeIndex > 0) {
                nodeIndex--;
            } else {
                return;
            }
        }
        // 获取节结果信息
        LatLng nodeLocation = null;
        String nodeTitle = null;
        Object step = route.getAllStep().get(nodeIndex);
        if (step instanceof DrivingRouteLine.DrivingStep) {
            nodeLocation = ((DrivingRouteLine.DrivingStep) step).getEntrance().getLocation();
            nodeTitle = ((DrivingRouteLine.DrivingStep) step).getInstructions();
        } else if (step instanceof WalkingRouteLine.WalkingStep) {
            nodeLocation = ((WalkingRouteLine.WalkingStep) step).getEntrance().getLocation();
            nodeTitle = ((WalkingRouteLine.WalkingStep) step).getInstructions();
        } else if (step instanceof TransitRouteLine.TransitStep) {
            nodeLocation = ((TransitRouteLine.TransitStep) step).getEntrance().getLocation();
            nodeTitle = ((TransitRouteLine.TransitStep) step).getInstructions();
        } else if (step instanceof BikingRouteLine.BikingStep) {
            nodeLocation = ((BikingRouteLine.BikingStep) step).getEntrance().getLocation();
            nodeTitle = ((BikingRouteLine.BikingStep) step).getInstructions();
        }

        if (nodeLocation == null || nodeTitle == null) {
            return;
        }
        // 移动节点至中心
        mBaidumap.setMapStatus(MapStatusUpdateFactory.newLatLng(nodeLocation));
        // show popup
        popupText = new TextView(DriverMenu.this);
        popupText.setBackgroundResource(R.drawable.popup);
        popupText.setTextColor(0xFF000000);
        popupText.setText(nodeTitle);
        mBaidumap.showInfoWindow(new InfoWindow(popupText, nodeLocation, 0));

    }

    /**
     * 切换路线图标，刷新地图使其生效 注意： 起终点图标使用中心对齐.
     */

    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
    }

    @Override
    public void onGetWalkingRouteResult(WalkingRouteResult result) {
        if (result == null || result.error != SearchResult.ERRORNO.NO_ERROR) {
            MyToast("抱歉，未找到结果");
        }
        if (result.error == SearchResult.ERRORNO.AMBIGUOUS_ROURE_ADDR) {
            // 起终点或途经点地址有岐义，通过以下接口获取建议查询信息
            // result.getSuggestAddrInfo()
            return;
        }
        if (result.error == SearchResult.ERRORNO.NO_ERROR) {
            nodeIndex = -1;
            mBtnPre.setVisibility(View.VISIBLE);
            mBtnNext.setVisibility(View.VISIBLE);
            route = result.getRouteLines().get(0);
            WalkingRouteOverlay overlay = new MyWalkingRouteOverlay(mBaidumap);
            mBaidumap.setOnMarkerClickListener(overlay);
            routeOverlay = overlay;
            overlay.setData(result.getRouteLines().get(0));
            overlay.addToMap();
            overlay.zoomToSpan();
        }

    }

    @Override
    public void onGetTransitRouteResult(TransitRouteResult result) {

        if (result == null || result.error != SearchResult.ERRORNO.NO_ERROR) {
            MyToast("抱歉，未找到结果");
        }
        if (result.error == SearchResult.ERRORNO.AMBIGUOUS_ROURE_ADDR) {
            // 起终点或途经点地址有岐义，通过以下接口获取建议查询信息
            // result.getSuggestAddrInfo()
            return;
        }
        if (result.error == SearchResult.ERRORNO.NO_ERROR) {
            nodeIndex = -1;
            mBtnPre.setVisibility(View.VISIBLE);
            mBtnNext.setVisibility(View.VISIBLE);
            route = result.getRouteLines().get(0);
            TransitRouteOverlay overlay = new MyTransitRouteOverlay(mBaidumap);
            mBaidumap.setOnMarkerClickListener(overlay);
            routeOverlay = overlay;
            overlay.setData(result.getRouteLines().get(0));
            overlay.addToMap();
            overlay.zoomToSpan();
        }
    }

    @Override
    public void onGetDrivingRouteResult(DrivingRouteResult result) {
        if (result == null || result.error != SearchResult.ERRORNO.NO_ERROR) {
            MyToast("抱歉，未找到结果");
        }
        if (result.error == SearchResult.ERRORNO.AMBIGUOUS_ROURE_ADDR) {
            // 起终点或途经点地址有岐义，通过以下接口获取建议查询信息
            // result.getSuggestAddrInfo()
            return;
        }
        if (result.error == SearchResult.ERRORNO.NO_ERROR) {
            nodeIndex = -1;
            mBtnPre.setVisibility(View.VISIBLE);
            mBtnNext.setVisibility(View.VISIBLE);
            route = result.getRouteLines().get(0);
            DrivingRouteOverlay overlay = new MyDrivingRouteOverlay(mBaidumap);
            routeOverlay = overlay;
            mBaidumap.setOnMarkerClickListener(overlay);
            overlay.setData(result.getRouteLines().get(0));
            overlay.addToMap();
            overlay.zoomToSpan();
        }
    }

    @Override
    public void onGetBikingRouteResult(BikingRouteResult bikingRouteResult) {
        if (bikingRouteResult == null || bikingRouteResult.error != SearchResult.ERRORNO.NO_ERROR) {
            MyToast("抱歉，未找到结果");
        }
        if (bikingRouteResult.error == SearchResult.ERRORNO.AMBIGUOUS_ROURE_ADDR) {
            // 起终点或途经点地址有岐义，通过以下接口获取建议查询信息
            // result.getSuggestAddrInfo()
            return;
        }
        if (bikingRouteResult.error == SearchResult.ERRORNO.NO_ERROR) {
            nodeIndex = -1;
            mBtnPre.setVisibility(View.VISIBLE);
            mBtnNext.setVisibility(View.VISIBLE);
            route = bikingRouteResult.getRouteLines().get(0);
            BikingRouteOverlay overlay = new MyBikingRouteOverlay(mBaidumap);
            routeOverlay = overlay;
            mBaidumap.setOnMarkerClickListener(overlay);
            overlay.setData(bikingRouteResult.getRouteLines().get(0));
            overlay.addToMap();
            overlay.zoomToSpan();
        }
    }

    // 定制RouteOverly
    private class MyDrivingRouteOverlay extends DrivingRouteOverlay {

        public MyDrivingRouteOverlay(BaiduMap baiduMap) {
            super(baiduMap);
        }

        @Override
        public BitmapDescriptor getStartMarker() {
            if (useDefaultIcon) {
                return BitmapDescriptorFactory.fromResource(R.drawable.icon_st);
            }
            return null;
        }

        @Override
        public BitmapDescriptor getTerminalMarker() {
            if (useDefaultIcon) {
                return BitmapDescriptorFactory.fromResource(R.drawable.icon_en);
            }
            return null;
        }
    }

    private class MyWalkingRouteOverlay extends WalkingRouteOverlay {

        public MyWalkingRouteOverlay(BaiduMap baiduMap) {
            super(baiduMap);
        }

        @Override
        public BitmapDescriptor getStartMarker() {
            if (useDefaultIcon) {
                return BitmapDescriptorFactory.fromResource(R.drawable.icon_st);
            }
            return null;
        }

        @Override
        public BitmapDescriptor getTerminalMarker() {
            if (useDefaultIcon) {
                return BitmapDescriptorFactory.fromResource(R.drawable.icon_en);
            }
            return null;
        }
    }

    private class MyTransitRouteOverlay extends TransitRouteOverlay {

        public MyTransitRouteOverlay(BaiduMap baiduMap) {
            super(baiduMap);
        }

        @Override
        public BitmapDescriptor getStartMarker() {
            if (useDefaultIcon) {
                return BitmapDescriptorFactory.fromResource(R.drawable.icon_st);
            }
            return null;
        }

        @Override
        public BitmapDescriptor getTerminalMarker() {
            if (useDefaultIcon) {
                return BitmapDescriptorFactory.fromResource(R.drawable.icon_en);
            }
            return null;
        }
    }

    private class MyBikingRouteOverlay extends BikingRouteOverlay {
        public MyBikingRouteOverlay(BaiduMap baiduMap) {
            super(baiduMap);
        }

        @Override
        public BitmapDescriptor getStartMarker() {
            if (useDefaultIcon) {
                return BitmapDescriptorFactory.fromResource(R.drawable.icon_st);
            }
            return null;
        }

        @Override
        public BitmapDescriptor getTerminalMarker() {
            if (useDefaultIcon) {
                return BitmapDescriptorFactory.fromResource(R.drawable.icon_en);
            }
            return null;
        }

    }

    @Override
    public void onMapClick(LatLng point) {
        mBaidumap.hideInfoWindow();
    }

    @Override
    public boolean onMapPoiClick(MapPoi poi) {
        return false;
    }

    @Override
    protected void onPause() {
        mMapView.onPause();
        super.onPause();
    }

    @Override
    protected void onResume() {
        mMapView.onResume();
        super.onResume();
    }

    @Override
    protected void onDestroy() {
        mSearch.destroy();
        mMapView.onDestroy();
        super.onDestroy();
    }

    public void MyToast(String s) {
        Toast.makeText(DriverMenu.this, s, Toast.LENGTH_SHORT).show();
    }

    /**
     * 定位SDK监听函数
     */
    public class MyLocationListenner implements BDLocationListener {

        @Override
        public void onReceiveLocation(BDLocation location) {
            // map view 销毁后不在处理新接收的位置
            if (location == null || mMapView == null) {
                return;
            }
            MyLocationData locData = new MyLocationData.Builder().accuracy(location.getRadius())
                    // 此处设置开发者获取到的方向信息，顺时针0-360
                    .direction(100).latitude(location.getLatitude()).longitude(location.getLongitude()).build();
            mBaidumap.setMyLocationData(locData);
            if (isFirstLoc) {
                isFirstLoc = false;
                LatLng ll = new LatLng(location.getLatitude(), location.getLongitude());
                MapStatus.Builder builder = new MapStatus.Builder();
                builder.target(ll).zoom(18.0f);
                mBaidumap.animateMapStatus(MapStatusUpdateFactory.newMapStatus(builder.build()));
                start_edit.setText(location.getAddrStr());
                MyToast("当前所在位置：" + location.getAddrStr());
                driver_city.setText(location.getCity());
                loaclcity = location.getCity();
            }
        }

        public void onReceivePoi(BDLocation poiLocation) {
        }
    }

}
```


代码的开头导入了大量的包可以从百度地图下载的sdk中查看到

下面附上运行之后的样例图片看看是不是想要的效果


<div align=center><img src="https://github.com/Legend-yd/MyDawn/blob/master/20171223205633274.png" width="300" height="450" ><div>
<div align=center><img src="https://github.com/Legend-yd/MyDawn/blob/master/20171223205617115.png" width="300" height="450"><div>
<div align=center><img src="https://github.com/Legend-yd/MyDawn/blob/master/20171223205540585.png" width="300" height="450"><div>

