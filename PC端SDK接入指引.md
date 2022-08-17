本文介绍Windows端SDK的集成指引。

## 一、集成准备

- 开发工具使用VS2019（建议）
- 运行环境依赖 QT5.15.2  + QT VS插件 

环境配置如下

1. 下图说明qt插件已经安装成功

![](https://qcloudimg.tencent-cloud.cn/raw/43dbf2c136561024a3716386269f835c.png)

2. 配置qt插件版本，选择对应的QT版本

![](https://qcloudimg.tencent-cloud.cn/raw/6ab9d1a152406edffda7adc5c7345a35.png)

   

3. 配置工程依赖的QT组件

![](https://qcloudimg.tencent-cloud.cn/raw/72f359809d75205a4b6a6bf9f0755711.png)

4、添加头文件和Xmagic的dll库

![](https://qcloudimg.tencent-cloud.cn/raw/fbaa2ebfc557df069aadd0e80c96b1c9.png)



## 二、接口使用说明



| API            | 描述                                                 |
| -------------- | ---------------------------------------------------- |
| setTELicense   | 鉴权接口                                             |
| createXmagic   | Xmagic创建接口                                       |
| destroyXmagic  | Xmagic销毁接口                                       |
| updateProperty | 设置美颜参数（如美颜，动效，化妆等等）               |
| process        | 处理美颜接口，返回对应的美颜处理数据（如像素，纹理） |
| setSegmentBg   | 设置自定扣背接口                                     |
| setRenderSize  | 重置输入数据大小                                     |
| onPasue        | 暂停美颜                                             |
| onResume       | 开始美颜                                             |



**鉴权接口**

```c++
	XMAGIC_API void setTELicense(const char* url, const char* key, int (*TELicenseCallback)(int, const char*));

```

参数含义

| 参数              | 含义                             |
| ----------------- | -------------------------------- |
| url               | 对应的鉴权证书链接（在后台获取） |
| key               | 对应的鉴权证书key（在后台获取）  |
| TELicenseCallback | 鉴权成功或者失败的回调           |

鉴权回调code的含义

| 错误码 | 说明                                                    |
| :----- | :------------------------------------------------------ |
| 0      | 成功。Success                                           |
| -1     | 输入参数无效，例如 URL 或 KEY 为空                      |
| -3     | 下载环节失败，请检查网络设置                            |
| -4     | 从本地读取的 TE 授权信息为空，可能是 IO 失败引起        |
| -5     | 读取 VCUBE TEMP License文件内容为空，可能是 IO 失败引起 |
| -6     | v_cube.license 文件 JSON 字段不对。请联系腾讯云团队处理 |
| -7     | 签名校验失败。请联系腾讯云团队处理                      |
| -8     | 解密失败。请联系腾讯云团队处理                          |
| -9     | TELicense 字段里的 JSON 字段不对。请联系腾讯云团队处理  |
| -10    | 从网络解析的TE授权信息为空。请联系腾讯云团队处理        |
| -11    | 把 TE 授权信息写到本地文件时失败，可能是IO失败引起      |
| -12    | 下载失败，解析本地 asset 也失败                         |
| -13    | 鉴权失败                                                |
| 其他   | 请联系腾讯云团队处理                                    |



**创建Xmagic接口**

```c++
	XMAGIC_API IXmagic* createXmagic(std::string& resDir, int width, int height);

```

| 参数   | 含义                                |
| ------ | ----------------------------------- |
| resDir | 对应的资源文件（sdk依赖的资源路径） |
| width  | 处理数据宽                          |
| height | 处理数据高                          |



**销魂Xmagic接口**

```c++
	XMAGIC_API void destroyXmagic(IXmagic** xmagic);

```



**设置美颜参数**

```c++
		virtual void updateProperty(XmagicProperty* property) = 0;

```



参数含义

| 参数     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| category | 类型(BEAUTY = 0,//美颜<br/>		BODY_BEAUTY,//美体<br/>		LUT,//滤镜<br/>		MOTION,//动效<br/>		SEGMENTATION,//扣背<br/>		MAKEUP//化妆) |
| id       | id                                                           |
| resPath  | 对应设置资源的地址（如动效，化妆，扣背等）                   |
| effKey   | 美颜的key                                                    |
| effValue | 美颜的值                                                     |
| isAuth   | 默认true                                                     |



**处理美颜(像素)**

```c++
		virtual void process(YTImagePixelData* srcImage, YTImagePixelData* dstImage) = 0;

```



参数说明

| 参数     | 说明           |
| -------- | -------------- |
| srcImage | 输入的美颜数据 |
| dstImage | 输出的美颜数据 |



YTImagePixelData结构说明

```c++
struct YTImagePixelData {
		/// 【字段含义】像素格式
		PixelFormat pixelFormat;

		/// 【字段含义】
		uint8_t* data;

		/// 【字段含义】数据的长度，单位是字节
		int32_t length;

		/// 【字段含义】宽度
		int32_t width;

		/// 【字段含义】高度
		int32_t height;

		YTImagePixelData() : pixelFormat(PixelFormat::PixelFormatRGBA32), data(nullptr), length(0), width(0), height(0) {}
	};
```



**处理美颜纹理，返回对应的处理纹理**

```
		virtual int process(int textureId, int width, int height) = 0;

```

参数说明 

| 参数      | 说明         |
| --------- | ------------ |
| textureId | 输入的纹理id |
| width     | 纹理width    |
| height    | 纹理height   |



**设置自定义扣背景**

```c++
		virtual void setSegmentBg(std::string &segmentBgName ,int segmentBgType ,int timeOffset) = 0;

```

参数说明

| 参数          | 说明                             |
| ------------- | -------------------------------- |
| segmentBgName | 自定扣背路径                     |
| segmentBgType | 设置背影类型（0为图片，1是视频） |
| timeOffset    | 如果为视频，设置视频播放时长     |



**重置输入大小**

```c++
		virtual void setRenderSize(int width, int height) = 0;

```



参数说明

| 参数   | 说明       |
| ------ | ---------- |
| width  | 重置后的宽 |
| height | 重置后的高 |



**暂停**

```c++
		virtual void onPasue() = 0;

```



**开始**

```
		virtual void onResume() = 0;

```



## 三、代码示例(具体参考demo)

```c++
//鉴权接口
	auto respCallback = [](
		int ret, const char* data) -> int {
			int retCode = ret;
			const char* msg = data;
			return 0;
	};

	setTELicense("url", "key", respCallback);
```



```c++
//创建Xmaigc
	std::string exeFilePath = "资源位置";
    IXmagic* xmagic = createXmagic(exeDirectory, 720, 1280);

```



```c++
//设置属性和输出
    YTImagePixelData src,dst;
	src.width = 720;
	src.height = 1280;
	src.length = 4 * 720 * 1280;
	src.pixelFormat = PixelFormat::PixelFormatRGBA32;
	//uint8_t* rgbaBuffer = (uint8_t*)malloc(src.length);
	//int w = 0, h = 0, comp = 0;
	uint8_t* imageData;
	int w = 0, h = 0, comp = 0;
	stbi_set_flip_vertically_on_load(false);  // SDK要求输入的图是倒置的，否则有问题
	imageData = stbi_load(path.c_str(), &w, &h, &comp, STBI_rgb_alpha);
	src.data = imageData;

	dst.width = 720;
	dst.height = 1280;
	dst.length = 4 * 720 * 1280;
	dst.pixelFormat = PixelFormat::PixelFormatRGBA32;
	uint8_t* rgbaBuffer = (uint8_t*)malloc(dst.length);//这个要释放
	dst.data = rgbaBuffer;

    std::string bgPath = exeDirectory + "bg.jpg";
	xmagic->setSegmentBg(bgPath, 0, 0);

	XmagicProperty property;
	property.category = Category::SEGMENTATION;
	property.resPath = exeDirectory + "segmentMotionRes.bundle/video_empty_segmentation/template.json";
	xmagic->updateProperty(&property);


	for(int i =0; i < 2; i++){
		xmagic->process(&src, &dst);
	}
```



```c++
//销毁
if (xmagic) {
    destroyXmagic(&xmagic);
}
```

