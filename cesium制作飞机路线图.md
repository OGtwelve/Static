cesium实现飞行路线随时间移动
[https://cesium.com/learn/cesiumjs-learn/cesiumjs-flight-tracker/](https://cesium.com/learn/cesiumjs-learn/cesiumjs-flight-tracker/)
官方文档制作飞机路线图

# 首先

 登录网站获取token([https://cesium.com/ion/signin/tokens](https://cesium.com/ion/signin/tokens))
![token](https://img-blog.csdnimg.cn/img_convert/c25e9bb92e00d988c20223ccad94753f.png#pic_center)

```
{eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIzNDVkOWIyZS03M2UzLTQ3YWUtOWM5MS1hYWJmYTFkZWU4NjgiLCJpZCI6NjA3MjQsImlhdCI6MTYyNTM4NDM1M30.5BTOMlLA_XGdnKvmmfTPYHU5i_TMnh2cl242FSpdn9o19630303}
```

这个为我的token

#### 第一步

点击 [https://glitch.com/edit/#!/remix/cesium-template](https://glitch.com/edit/#!/remix/cesium-template)
1.创建组件表
2.点击左侧index.html进入编辑页面
3.把刚才复制的token赋值在your_token_here这个位置

![yourtokenhere](https://img-blog.csdnimg.cn/img_convert/281f589f3ed1804be6afbc83f7c70b91.png#pic_center)
4.点击左上角带着眼镜标识的show然后选择子选项右侧
![buildMode](https://img-blog.csdnimg.cn/img_convert/7f6ea31530f2f98cf957bc9feeb4f8e6.png#pic_center)


#### 第二步

[https://cesium.com/platform/cesium-ion/content/cesium-world-terrain/](https://cesium.com/platform/cesium-ion/content/cesium-world-terrain/) (high resolution terrain with up to 1 meter accuracy.)精准到1米内的图形

[https://cesium.com/platform/cesium-ion/content/cesium-osm-buildings/](https://cesium.com/platform/cesium-ion/content/cesium-osm-buildings/)(over 350 million buildings derived from OpenStreetMap data.)巨多建筑等可用.

Bing Maps Aerial Imagery — global satellite imagery with up to 15 cm resolution.(第一步建设的是自带这个图层的 这个应该是底层地图图片的意思)

1.以下代码放入index.html中 注意 要放在token后
![firstCopy](https://img-blog.csdnimg.cn/img_convert/a8ff1d0e2c22beb1ae7dc3a8b1ec3f9e.png#pic_center)


2.随便的拖拽着看看效果,可以通过按住Ctrl再拖拽 可以获得以定点展开的相机范围.

```
// STEP 3 CODE (first point)
// This is one of the first radar samples collected for our flight.
const dataPoint = { longitude: -122.38985, latitude: 37.61864, height: -27.32 };
// Mark this location with a red point.
const pointEntity = viewer.entities.add({
  description: `First data point at (${dataPoint.longitude}, ${dataPoint.latitude})`,
  position: Cesium.Cartesian3.fromDegrees(dataPoint.longitude, dataPoint.latitude, dataPoint.height),
  point: { pixelSize: 10, color: Cesium.Color.RED }
});
// Fly the camera to this point.
viewer.flyTo(pointEntity);
```

2.随便的拖拽着看看效果,可以通过按住Ctrl再拖拽 可以获得以定点展开的相机范围.

注意事项:
Notice that higher levels of detail are loaded as you zoom in. This allows us to visualize the entire planet with the accuracy required for making real world decisions.

This is possible because we’re using 3D Tiles, an open standard we pioneered for streaming 3D content to any device. Learn how to convert your own data to 3D Tiles.

#### 第三步

甚至可以根据真实的飞行数据模拟出真实飞机飞行的情况,[https://s3.amazonaws.com/cesiumjs/downloads/FlightRadar24_SFO_to_CPH_SK936.csv](https://s3.amazonaws.com/cesiumjs/downloads/FlightRadar24_SFO_to_CPH_SK936.csv)这个网站内有大致的真实数据 有兴趣的可以导入试试

1.将下列代码加入刚才的部分,其实跟刚才加入的代码一致

```
// STEP 3 CODE (first point)
// This is one of the first radar samples collected for our flight.
const dataPoint = { longitude: -122.38985, latitude: 37.61864, height: -27.32 };
// Mark this location with a red point.
const pointEntity = viewer.entities.add({
  description: `First data point at (${dataPoint.longitude}, ${dataPoint.latitude})`,
  position: Cesium.Cartesian3.fromDegrees(dataPoint.longitude, dataPoint.latitude, dataPoint.height),
  point: { pixelSize: 10, color: Cesium.Color.RED }
});
// Fly the camera to this point.
viewer.flyTo(pointEntity);
```

2.可以通过点击红点来看对应的描述,这个描述被用于显示相关信息或者被创建的时间点等.

#### 第四步

将以下代码把上面所添加的红点代码所替换

第一个:[https://github.com/OGtwelve/Static/blob/main/firstFlightData.txt](https://github.com/OGtwelve/Static/blob/main/firstFlightData.txt)

![inDown1](https://img-blog.csdnimg.cn/img_convert/ca726f42d10e078932596a399db96395.png#pic_center)

现在可以看到整个飞行的路线
坐标是通过Cartesian3来控制的 在这个部分里 origin 也就是原点(0,0,0) 为地球的中心.
(X,Y,Z) 前两个值为经纬度的表示,最后一个值为飞行的高度

#### 第五步

通过时间设置移动
目前为止,我们已经实现了标点的功能;CesiumJS中有内置的功能可以帮我们把飞机通过时间移动
我们会通过创建一个SamplePositionProperty对象来记录时间戳和位置([https://cesium.com/docs/cesiumjs-ref-doc/SampledPositionProperty.html](https://cesium.com/docs/cesiumjs-ref-doc/SampledPositionProperty.html)),刚才的代码中不包含时间戳,但我们知道飞机的号码是SK936,它被规定在2020年3月9日下午4:10离港,我们只需要控制位置之间有30秒的间隔.

1.把所有index.html中 你复制的代码(除了token)替换为下方代码
第二个:[https://github.com/OGtwelve/Static/blob/main/secondFlightData.txt](https://github.com/OGtwelve/Static/blob/main/secondFlightData.txt)
目前这部分结束后,我们就有了一个可运行的代码

#### 第六步

上传飞机的模型
1.[https://s3.amazonaws.com/cesiumjs/downloads/Cesium_Air.glb](https://s3.amazonaws.com/cesiumjs/downloads/Cesium_Air.glb)
2.[https://cesium.com/ion/assets/](https://cesium.com/ion/assets/) 去到自己账号的仪表盘,把飞机文件拉入此页面
3.选择3D Model(Covert to gltf),然后上传
4.当上传结束后 找到这个飞机的id并记录(505069)

#### 第七步

1.把从// STEP 4 CODE (green circle entity)开始到结尾的所有代码删除
2.替换为下方代码
```
// STEP 6 CODE (airplane entity)
async function loadModel() {
  // Load the glTF model from Cesium ion.
  const airplaneUri = await Cesium.IonResource.fromAssetId(your_asset_id);
  const airplaneEntity = viewer.entities.add({
    availability: new Cesium.TimeIntervalCollection([ new Cesium.TimeInterval({ start: start, stop: stop }) ]),
    position: positionProperty,
    // Attach the 3D model instead of the green point.
    model: { uri: airplaneUri },
    // Automatically compute the orientation from the position.
    orientation: new Cesium.VelocityOrientationProperty(positionProperty),    
    path: new Cesium.PathGraphics({ width: 3 })
  });

  viewer.trackedEntity = airplaneEntity;
}

loadModel();
```
3.把your_asset_id替换为飞机的id
(如果你有3d模型,可以上传为自己的从而不使用已经给出的默认飞机模型)

----------------------------------------------------------------------------------------------------

//此文档由OGtwelve撰写,如有引用请标注来源,谢谢大家.
