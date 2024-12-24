# 基础 API

[TOC]

## 序言 

介绍一些 API 函数

## 创建 Mat 对象

**Mat 对象与 IplImage 对象**

Mat对象 OpenCV2.0 之后引进的图像数据结构、自动分配内存、 不存在内存泄漏的问题，是面向对象的数据结构。分了两个部分，头部与数据部分。

IplImage 是从 2001年 OpenCV 发布之后就一直存在，是 C语言风格的数据结构，需要开发者自己分配与管理内存，对大的程序使用它容易导致内存泄漏问题。

**Mat对象使用四个要点：**

1. 输出图像的内存是自动分配的
2. 使用OpenCV的C++接口，不需要考虑内存分配问题
3. 赋值操作和拷贝构造函数只会复制头部分
4. 使用clone与copyTo两个函数实现数据完全复制

**常见的几个创建 Mat 对象**

- Mat()构造函数

```c++
Mat()
Mat(int rows, int cols, int type)
Mat(Size size, int type)
Mat(int rows, int cols, int type, const Scalar &s)
Mat(Size size, int type, const Scalar &s)
Mat(int ndims, const int* sizes, int type)
Mat(int ndims, const int* sizes, int type, const Scalar &s)
```

- Matlab 式的初始化方式

```c++
Mat::zeros(int rows, int cols, int type)	// 常用于二值化图像
Mat::ones(int rows, int cols, int type)		// 常用于二值化图像
Mat::eye(int rows, int cols, int type)
```

- clone() or copyTo()

```c++
Mat image = imread(".....");
Mat M1, M2;
M1 = image.clone();
image.copyTo(M2);
```

## 图像的载入 imread()

```c++
Mat imread(const string& filename,int flags=1);
```

- 第一个参数，const string& 类型的 filename，填我们需要载入的**图片路径名**。
- 第二个参数，int 类型的 flags，为载入标识，它指定一个加载图像的颜色类型。可以看到它自带默认值1，所以有时候这个参数在调用时可以忽略。如果在调用时忽略这个参数，就表示载入三通道的彩色图像。
  - CV_LOAD_IMAGE_UNCHANGED——等价取值为-1，这个标识在新版本中已被废置，忽略。
  - CV_LOAD_IMAGE_GRAYSCALE——等价取值为 0，如果取这个标识的话，始终将图像转换成灰度再返回。
  - CV_LOAD_IMAGE_COLOR——等价取值为 1，如果取这个标识，总是转换图像到彩色再返回。
  - CV_LOAD_IMAGE_ANYDEPTH——等价取值为 2，如果取这个标识，且载入的图像的深度为16位或者32位，就返回对应深度的图像，否则，就转换为8位图像再返回。
  - 需要说明的是，如果输入有冲突的标志，将采用较小的数字值。比如CV_LOAD_IMAGE_COLOR|CV_LOADIMAGE_ANYCOLOR 将载入三通道图。
  - 而如果想要载入最真实无损的源图像，可以选择CV_LOAD_IMAGE_ANYDEPTH|CV_LOAD_IMAGE_ANYCOLOR。

**另外，需要额外注意，若以彩色模式载入图像，解码后的图像会以 BGR 的通道顺序进行存储，即蓝、绿、红的顺序，而不是通常的 RGB 的顺序。**

## 图像的显示 imshow()

```c++
void cv::imshow	(const String & winname, InputArray mat)	
```

- winname：要显示图像的窗口名称
- mat：待显示的图像

1. 如果图像数据类型是 8U（8位无符号），则直接显示
2. 如果图像数据类型是 16U（16位无符号）或 32S（32位有符号整数），则 imshow 函数内部会自动将每个像素值除以256（2的八次方）并显示，即将原图像素值的范围由 [0~255*256] 映射到 [0~255]
3. 如果图像数据类型是 32F（32位浮点数）或 64F（64位浮点数），则 imshow 函数内部会自动将每个像素值乘以 255 并显示，即将原图像素值的范围由 [0~1] 映射到 [0~255]（注意：原图像素值必须要归一化）

## 视频VideoCapture()

```c++
cv::VideoCapture capture(const string& filename);
```

- Filename：视频的路径；当为 0 时，从摄像机中读取视频，即开启摄像头。

opencv 读取到的摄像头画面大小一般为默认的 640x480，通过下面操作可改变画面大小。

```c++
capture.set(cv2.CAP_PROP_FRAME_HEIGHT, 1080)		// 设置摄像头分辨率的高
capture.set(cv2.CAP_PROP_FRAME_WIDTH, 1920)			// 设置摄像头分辨率的宽
width = capture.get(cv2.CAP_PROP_FRAME_WIDTH)		// 获取视频帧的宽
height = capture.get(cv2.CAP_PROP_FRAME_HEIGHT)		// 获取视频帧的高
fps = capture.get(cv2.CAP_PROP_FPS)				   // 获取视频帧的帧率
fcount = capture.get(cv2.CAP_PROP_FRAME_COUNT)  	// 获取视频流的总帧数
capture.get(CAP_PROP_FOURCC)				 	  // 获取视频流的编辑码
```

```c++
void QuickDemo::video_demo()
{
	VideoCapture capture("F:/大学文件/课外资料/Opencv/opencv_tutorial_data-master/images/01.mp4");
	// VideoCapture capture(0);//用摄像头
	// 设置摄像头画面的大小
	// capture.set(CAP_PROP_FRAME_WIDTH, 1080);
	// capture.set(CAP_PROP_FRAME_HEIGHT, 1920);
	// 获取视频的宽度 高度
	int video_width = capture.get(CAP_PROP_FRAME_WIDTH);
	int video_height= capture.get(CAP_PROP_FRAME_HEIGHT);
	cout << "width:" << video_width << endl;
	cout << "height:" << video_height << endl;
	// 获取视频的帧率 总帧率
	double fps = capture.get(CAP_PROP_FPS);
	int count = capture.get(CAP_PROP_FRAME_COUNT);
	cout << "fps:" << fps << endl;
	cout << "count:" << count << endl;
	VideoWriter writer("01.mp4", capture.get(CAP_PROP_FOURCC), fps, Size(video_width, video_height), true);
	Mat frame;
	while (true)
	{
		capture.read(frame);	// 读取视频
		if(frame.empty())		// 判断视频是否为空
			break;
		imshow("frame", frame);	// 演示视频
		writer.write(frame);	// 写入视频
		int c = waitKey(50);
		if (c == 27)
			break;
	}
	frame.release();	// 释放VideoCapture资源
	writer.release();	// 释放VideoWriter资源
}
```

## 保存图像 imwrite()

```c++
bool imwrite(const string& filename, InputArray img, const vector<int>& params = vector<int>());
```

- filename：填需要写入的文件名。注意要**带上后缀**，如“123.jpg”。
- img：一般填一个**Mat类型**的图像数据。
- params：表示为特定格式保存的参数编码。它有默认值 vector<int>（），所以**一般情况下不需要填写**。而如果要填写的话，有下面这些需要了解的地方：
  - 对于 JPEG 格式的图片，这个参数表示从 0 到 100 的图片质量（CV_IMWRITE_JPEG_QUALITY），默认值是 95。
  - 对于 PNG 格式的图片，这个参数表示压缩级别（CV_IMWRITE_PNG_COMPRESSION）从 0 到 9。较高的值意味着更小的尺寸和更长的压缩时间，默认值是3。
  -  对于 PPM，PGM，或 PBM 格式的图片，这个参数表示一个二进制格式标志（CV_IMWRITE_PXM_BINARY），取值为0或1，默认值是1。

imwrite 函数用于将图像保存到指定的文件。图像格式是基于文件扩展名的，可保存的扩展名和 imread 中可以读取的图像扩展名一致。

## copyTo()与mask()

```c++
image_in.copyTo(image_out)	// 把image_in的内容复制粘贴到image_out上
```

```c++
image_in.copyTo(image_out，mask)	// 把mask和image_in重叠以后,把mask中像素值为0（black）的点对应的image_in中的点变为透明，而保留其他点。
```

```c++
A.copyTo(B, mask)
```

mask 作为一个掩模板，如果在某个像素点 (i, j) 其值为 1（只看第一通道，所以mask单通道即可），则把 A.at(i, j) 处的值直接赋给 B.at(i, j)；如果其值为0，则 B.at(i, j) 处保留其原始像素值。这样能够使得图标边缘的颜色和原图保持一致。

## 图像色彩空间转换函数 cvtcolor()

cvtColor（）函数是OpenCV里的颜色空间转换函数，可以实现RGB颜色向HSV、HSI等颜色空间的转换，也可以转换为灰度图像。原型如下：

```c++
void cvtColor(InputArray src, OutputArray dat, int code, int dstCn = 0)
```

第一个参数为输入图像，第二个参数为输出图像，第三个参数为颜色空间转换的标识符（具体见表4.1），第四个参数为目标图像的通道数，若该参数是0，表示目标图像取源图像的通道数。

下面是一个调用示例：

![image-20230918210615221](基础API/image-20230918210615221.png)

OpenCV默认的图片通道存储顺序是 **BGR**，即蓝绿红，而不是RGB。

## 图像色彩空间转换

HSV 范围 H：0~180 S：0~255 V：0~255

|        | 黑   | 灰   | 白   | 红      | 橙   | 黄   | 绿   | 青   | 蓝   |  紫  |
| ------ | ---- | ---- | ---- | ------- | ---- | ---- | ---- | ---- | ---- | :--: |
| H(min) | 0    | 0    | 0    | 0,156   | 11   | 26   | 35   | 78   | 100  | 125  |
| H(max) | 180  | 180  | 180  | 10 ,180 | 25   | 34   | 77   | 99   | 124  | 155  |
| S(min) | 0    | 0    | 0    | 43      | 43   | 43   | 43   | 43   | 43   |  43  |
| S(max) | 255  | 43   | 30   | 255     | 255  | 255  | 255  | 255  | 255  | 255  |
| V(min) | 0    | 46   | 221  | 46      | 46   | 46   | 46   | 46   | 46   |  46  |
| V(max) | 46   | 220  | 255  | 255     | 255  | 255  | 255  | 255  | 255  | 255  |

------

OpenCV 中的 inRange() 函数可实现二值化功能，更关键的是可以同时针对多通道进行操作，使用起来非常方便！**主要是将在两个阈值内的像素值设置为白色（255），而不在阈值区间内的像素值设置为黑色（0）。**

```c++
void inRange(InputArray src, InputArray lowerb, InputArray upperb, OutputArray dst);
```

- InputArray src：输入图像，可以为单通道或多通道。
- InputArray lowerb：包含下边界的数组或标量。
- InputArray upperb：包含上边界数组或标量。
- OutputArray dst：输出图像，与输入图像src 尺寸相同且为CV_8U 类型。

## 图像像素的基本操作

### 图像像素的读写

在OpenCV中，提供了三种访问每个像素的方法。

1. 方法一 指针访问：C操作符[]；
2. 方法二 迭代器iterator；
3. 方法三 动态地址计算

这三种方法在访问速度上略有差异。debug模式下，这种差异非常明显，不过在release模式下，这种差异就不太明显了。我们通过一组程序来说明这几种方法。程序的目的是减少图像中颜色的数量，比如原来的图像是是256种颜色，我们希望将它变成64种颜色，那只需要将原来的颜色除以4（整除）以后再乘以4就可以了。

Mat类提供了一个at的方法用于取得图像上的点，它是一个模板函数，可以取到任何类型的图像上的点。**at<typename>(i,j)**

- 方法一 指针访问 (常用 速度最快)

```c++
void QuickDemo::pixel_visit_demo(Mat& image)
{
	int width = image.cols;			// cols列 也就是宽度
	int height = image.rows;		// rows行 也就是高度
	int dims = image.channels();	// 频道 维数
		for (int row = 0; row < height; row++)
	{
		uchar* current_row = image.ptr<uchar>(row);
		for (int col = 0; col < width; col++)
		{
			if (dims == 1) {	// 灰度图像
				int pv = *current_row;
				*current_row++ = 255 - pv;
			}
			if (dims == 3) {	// 彩色图像
				*current_row++ = 255 - *current_row;
				*current_row++ = 255 - *current_row;
				*current_row++ = 255 - *current_row;
			}
		}
	}	imshow("像素读写显示", image);
}

```

- 方法二 迭代器iterator (少用)



- 方法三 动态地址计算 (速度最慢)

uchar,Vec3b 都是图像像素值的类型，**<Vec3b> 是指 BGR 三通道 char 型的**，类似的 Vec3i 就是 int 型的三通道 Vec3f 就是 float 型的三通道

```c++
void QuickDemo::pixel_visit_demo(Mat& image)
{
	int width = image.cols;//cols列 也就是宽度
	int height = image.rows;//rows行 也就是高度
	int dims = image.channels();//频道 维数
	for (int row = 0; row < height; row++)
	{
		for (int col = 0; col < width; col++)
		{
			if (dims == 1) {//灰度图像
				int pv = image.at<uchar>(row, col);
				image.at<uchar>(row, col) = 255 - pv;//取反
			}
			if (dims == 3) {//彩色图像
				Vec3b bgr = image.at<Vec3b>(row, col);//一次性获取三个像素值
				image.at<Vec3b>(row, col)[0] = 255 - bgr[0];
				image.at<Vec3b>(row, col)[1] = 255 - bgr[1];
				image.at<Vec3b>(row, col)[2] = 255 - bgr[2];
			}
		}
	}
	imshow("像素读写显示", image);
}
```

### 图像像素的算术操作

- 加法：add(a, b, c) c = a + b; 
- 减法：subtract(a, b, c) c = a - b; 
- 乘法：multiply(a, b, c) c = a * b; 
- 除法：divide(a, b, c) c = a / b;

```c++
Mat dst = Mat::zeros(image.size(), image.type());
Mat m = Mat::zeros(image.size(), image.type());
m = Scalar(50,50,50);
add(image, m, dst);
imshow("加法操作"，dst);
subtract(image, m, dst);
imshow("减法操作",dst);
multiply(image, m, dst);
imshow("乘法操作"，dst);
divide(image, m, dst);
imshow("除法操作"，dst);
```

**Demo**

```c++
void QuickDemo::operators_demo(Mat& image)
{
	Mat dst1,dst2,dst3,dst4;
	dst1 = image + Scalar(50, 50, 50);
	dst2 = image - Scalar(50, 50, 50);
	imshow("加法操作", dst1);
	imshow("减法操作", dst2);
	//乘法 除法不能直接用以下相同操作
	//dst3 = image * Scalar(2, 2, 2);
    //dst4 = image / Scalar(2, 2, 2);
	//imshow("乘法操作", dst3);
	//imshow("除法操作", dst4);
//乘法操作 Opencv乘法所用的函数 multiply(a,b,c) a*b=c
Mat dst = Mat::zeros(image.size(), image.type());	// 初始化 dst为image图像的大小的和类型
	Mat m = Mat::zeros(image.size(), image.type());
	m = Scalar(2, 2, 2);
	multiply(image, m, dst);	// Opencv乘法所用的函数 multiply
	imshow("乘法操作", dst);
}
```

由于经过运算后可常常会出现溢出的情况，一般会用到 `saturate_cast` 函数，防止溢出。

![image-20230918211409956](基础API/image-20230918211409956.png)

### 图像像素的逻辑操作

```c++
void QuickDemo::bitwise_demo(Mat& image)
{
	Mat m1 = Mat::zeros(Size(256, 256), CV_8UC3);
	Mat m2 = Mat::zeros(Size(256, 256), CV_8UC3);
	rectangle(m1, Rect(100, 100, 80, 80), Scalar(255, 255, 0), -1, LINE_8, 0);
	rectangle(m2, Rect(150, 150, 80, 80), Scalar(0, 255, 255), -1, LINE_8, 0);
	imshow("m1", m1);
	imshow("m2", m2);
	Mat dst1,dst2,dst3,dst4;
	bitwise_and(m1, m2, dst1);	// 与
	bitwise_not(m1, m2, dst2);	// 取反
	bitwise_or(m1, m2, dst3);	// 或
	bitwise_xor(m1, m2, dst4);	// 异或
	imshow("逻辑与", dst1);
	imshow("取反操作", dst2);
	imshow("逻辑或", dst3);
	imshow("逻辑异或", dst4);
}
```

## 键鼠交互

### 创建滑动条 createTrackbar()

`createTrackbar()` 函数用于创建一个可以调整数值的滑动条（常常也称**轨迹条**），并将滑动条附加到指定的窗口上。它往往会和一个**回调函数**配合起来使用。

```c++
int createTrackbar(const String& trackbarname, const String& winname, int* value, int count, TrackbarCallback onChange = 0, void* userdata = 0);
```

- trackbarname：**轨迹条的名字**，用 来表示创建的轨迹条
- winname：**窗口的名字**，表示这个轨迹条会依附到哪个窗口上。特别注意：**窗口名字须与创建的窗口名字相同**，否则会报错
- value：一个指向整形的指针，表示滑块的位置。创建时，滑块的**初始位置就是该变量当前的值**
- count：表示**滑块可以达到的最大位置的值**。滑块最小位置的值始终为0。滑块可以**滑动的范围永远都是[0, count]**
- onChange：默认值为 0，**指向回调函数的指针**。每次滑块位置改变时，这个函数都会进行回调。且这个函数的原型必须为 void XXXX（int，void*）;其中第一个参数是**轨迹条的位置**，第二个参数是**用户数据**。如果回调函数是 NULL 指针，则表示没回调函数的调用，仅第三个参数 value 变化
- userdata：**默认值0**。这个参数是用户传递给回调函数的数据，用来处理轨迹条事件。如果使用的第三个参数value 是全局变量，完全可以不去管这个 userdata 参数

### 获取当前轨迹条位置getTrackbarPos ()

getTrackbarPos（），它用于获取当前轨迹条的位置并返回。

```c++
int getTrackbarPos(const& string trackbarname,const string& winname)
```

- trackbarname：轨迹条的名字
-  winname：轨迹条的父窗口的名称

### 用 waitKey() 实现键盘响应

waitKey()函数是一个**等待键盘事件**的函数，参数值 delay<=0 时等待时间无限长，delay 为正整数 n 时至少等待 n毫秒的时间才结束。在等待的期间按下任意按键时函数结束，返回按键的键值（**ascii码**），等待时间结束仍未按下按键则返回 -1。该函数用在处理 HighGUI 窗口程序，最常见的便是与显示图像窗口 imshow 函数搭配使用。

```c++
void QuickDemo::key_demo(Mat& image)
{
	Mat dst = Mat::zeros(image.size(), image.type());
	while (true)
	{
		int c = waitKey(100);
		if (c == 27) {	// ESC对应的ASCLL码
			break;
		}
		if (c == 49) {	// 对应1的ASCLL码
			cout << "you enter key is 1" << endl;
			cvtColor(image, dst, COLOR_BGR2GRAY);
		}
		if (c == 50) {	// 对应2的ASCLL码
			cout << "you enter key is 2" << endl;
			cvtColor(image, dst, COLOR_BGR2HSV);
		}
		if (c == 51) {	// 对应3的ASCLL码
			cout << "you enter key is 3" << endl;
			dst = Mat::zeros(image.size(), image.type());	// 直接由灰度图像变换会报错（可能是由于参数溢出）
			add(image, dst, dst);
		}
		imshow("键盘响应", dst);
	}
}

```

## 几何形状的绘制

### 线 Line()

```c++
void line（Mat& img， Point pt1， Point pt2， const Scalar& color， int thickness=1， int lineType=8， int shift=0);
```

- img：待操作的图像
- pt1：线的起点坐标
- pt2：线的终点坐标
- color：线条颜色
- thickness：线条粗细
- lineType：线条类型
  - 8：8 连接线，默认情况下
  - 4：4 连接线路
  - CV_AA：抗锯齿线

- shift：点坐标中的小数位数

### 带箭头的线 arrowedLine()

```c++
void arrowedLine(InputOutputArray img, Point pt1, Point pt2, const Scalar& color, int thickness = 1, int line_type = 8, int shift = 0, double tipLength = 0.1);
```

- img：待操作的图像
- pt1：线的起点坐标
- pt2：线的终点坐标
- color：线条颜色
- thickness：线条粗细
- lineType: 线条类型
- shift：线条的偏移量
- tipLength：箭头占线段的比例

### 矩形 Rectangle()

```c++
void cv::rectangle (InputOutputArray img, Point pt1, Point pt2, const Scalar &color, int thickness = 1, int lineType = LINE_8, int shift = 0)
    
void cv::rectangle (InputOutputArray img, Rect rec, const Scalar & color, int thickness = 1, int lineType = LINE_8, int shift = 0)	
```

- img：待操作的图像
- pt1：矩形的顶点
- pt2：对应pt1的矩形的顶点
- color：矩形颜色或亮度（灰度图像）
- thickness：构成矩形的线条的厚度。**负值（-1）为填充矩形**
- lineType：线的类型默认为 8
- shift：点坐标中的小数位数，一般为0
- rec：Rect(int x, int y, int width, int height); 参数含义： **Rect（左上角x坐标 ， 左上角y坐标，矩形的宽，矩形的高）**

### 圆Circle()

```c++
void circle(InputOutputArray img, Point center, int radius, const Scalar& color, int thickness = 1, int lineType = LINE_8, int shift = 0);
```

- img：待操作的图像
- center：圆心
- radius：半径
- color：圆形的颜色
- thickness：圆形边的大小
- lineType：圆形类型
- shift：中心坐标和半径值中的小数位数。

### 椭圆 ellipse()

```c++
void cv::ellipse( CvArr* img, CvPoint center, CvSize axes, double angle, double start_angle, double end_angle, CvScalar color, int thickness = 1, int line_type = 8, int shift = 0 );
```

- img：待操作的图像
- center：椭圆圆心坐标
- axes：轴的长度
- angle：偏转的角度
- start_angle：圆弧起始角的角度
- end_angle：圆弧终结角的角度
- color：线条的颜色
- thickness：线条的粗细程度
- line_type：线条的类型见CVLINE的描述
- shift：圆心坐标点和数轴的精度

### 绘制填充多边形 fillPoly()

```c++
void fillPoly(InputOutputArray  img,  InputArrayOfArrays pts,  const Scalar & color,  int lineType = LINE_8,  int shift = 0,  Point offset = Point() );
```

- img：待操作的图像
- pts：表示多边形的点
- color：表示颜色
- lineType：表示线渲染类型
- shift：表示相对位移

### 绘制轮廓多边形 polyLines()

```c++
void cv::polyLines( InputOutputArray  img, InputArrayOfArrays       pts, bool isClosed,  const Scalar & color,  int thickness = 1,  int   lineType = LINE_8,  int shift = 0);
```

- img：待操作的图像
- pts：表示多边形的点
- isClosed：表示是否闭合，默认闭合
- color：表示颜色
- thickness：表示线宽，必须是正数
- lineType：表示线渲染类型
- shift：表示相对位移

```c++
Mat image = Mat::zeros(Size(800, 800)， CV 8UC3);
Point p1(100,100);
Point p2(350,100);
Point p3(300,500);
Point p4(200，300);
Point p5(80，200);
vector<Point>pts;
// 将各个坐标存储到vector 形参点集
pts.push_back(pl);
pts.push_back(p2);
pts.push_back(p3);
pts.push_back(p4);
pts.push_back(p5);
// 第一种polyLines() 轮廓
polylines(image, pts, true, Scalar(0,120，255), 1, 8, 0);	// true 闭合
// 第二种fillPoly() 填充多边形
fillPoly(image, pts, Scalar(255, 255, 116), 8, 0);
// 第三种 drawContours 0 绘制和填充多边形轮廓
vector<vector<Point>>contours;	// 将点集的vector再存进vector 点聚成线
contours.push_back(pts);
drawContours(image. contours, -1, Scalar(255, 0, 0), 5);
imshow("多边形绘制", image):
```

### 绘制轮廓或者填充轮廓 drawContours()

```c++
void cv::drawContours( InputOutputArray image,  InputArrayOfArrays contours,  int contourIdx,  const Scalar & color,  int thickness = 1,  int lineType = LINE_8,  InputArray hierarchy = noArray(), int maxLevel = INT_MAX,  Point offset = Point() );
```

- image：目标图像
- contours：输入的所有轮廓（**每个轮廓以点集（一般用vector存储）的方式存储**）
- contoursIdx：指定绘制轮廓的下标（**若为负数，则绘制所有轮廓**）
- color：绘制轮廓的颜色
- thickness：绘制轮廓的线的宽度（**若为负数，则填充轮廓内部**）
- lineType：绘制轮廓的线型（4连通、8连通或者反锯齿）
- hierarchy：关于层级的**可选**信息，仅用于当你想要绘制部分轮廓的时候
- maxLevel：绘制轮廓的最大层级，若为 0，则仅仅绘制指定的轮廓；若为 1，则绘制该轮廓及其内嵌轮廓，若为 2，则绘制该轮廓、其内嵌轮廓以及内嵌轮廓的内嵌轮廓，依次类推。该参数只有在有层级信息输入时**才被考虑**。
- offset：**可选**的轮廓偏移参数，所有的轮廓将会进行指定的偏移

## convertTo  函数

把一个矩阵从一种数据类型转换到另一种数据类型，同时可以带上缩放因子和增量。

```c++
void convertTo(OutputArray M, int Rtype, double Alpha = 1, double Deta = 0 ) const;
```

- M ：目标矩阵。如果m在运算前没有合适的尺寸或类型，将被重新分配
- Rtype： 目标矩阵的类型。因为目标矩阵的通道数与源矩阵一样，所以 Rtype 也可以看做是目标矩阵的位深度。**如果 Rtype 为负值，目标矩阵和源矩阵将使用同样的类型**
- Alpha ：尺度变换因子（可选）。默认值是1。即把原矩阵中的每一个元素都乘以 alpha
- Beta ：附加到尺度变换后的值上的偏移量（可选）。默认值是 0。即把原矩阵中的每一个元素都乘以 alpha，再加上 beta

![image-20230918214819505](基础API/image-20230918214819505.png)

## 改变图像的亮度和对比度 

- 单个 Trackbar

  ```c++
  // 回调函数
  void on_track(int pos, void* userdata)
  {
  	Mat* user = (Mat*)userdata; // 将无类型指针转为Mat类型指针
  	Mat dst;
  	(* user).convertTo(dst, -1, pos * 0.01); // -1表示转换前后类型相同
  	imshow("对比度调整", dst);
  }
  void QuickDemo::tracking_bar_demo(Mat& image)
  {
  	namedWindow("对比度调整", WINDOW_FREERATIO);
  	int value = 50;		    // 初始值
  	int max_value = 100;	// 最大值
  	createTrackbar("Value Bar", "对比度调整", &value, max_value, on_track, (void*)(&image));
  	on_track(50, &image);	// 一定得是输出图像的地址，否则会报错 
  }
  ```

- 多个 Trackbar

  当需要分别修改图像的亮度与对比度时，要使用两个或多个 TrackBar。也需要创建两个 TrackBar。而TrackBar 传参时，只能传入一个int指针类型参数，如果使用两个不同的回调函数分别传参，则会造成两个TrackBar 互相无法联系，即修改亮度与对比度无法一起使用。

  这里使用全局变量的方法，将需要传入的变量声明为全局变量，回调函数无需使用传入的参数。两个 TrackBar使用同一个回调函数，实现对比度、亮度的同时修改。

  ```c++
  // 声明并定义全局变量
  int colorStart=0 ;
  int contast=100;
  int colorMax=255;
  int contastMax=100;
  Mat srcBar = imread("test.jpg");
  
  // 回调函数2，使用全局变量，不使用传入参数
  void mytrackBarCallback2(int, void*) 
  {
      Mat dst;
      srcBar.convertTo(dst, -1, contast * 0.01, colorStart);
      imshow("image show", dst);
  }
  
  void trackbarTest() 
  {
      namedWindow("image show", WINDOW_FREERATIO);
      // 调整亮度的TrackBar
      createTrackbar("亮度", "image show", &colorStart, colorMax, mytrackBarCallback2);
      // 调整对比度的TrackBar，他们使用同一个回调函数
      createTrackbar("对比度", "image show", &contast, contastMax, mytrackBarCallback2);
      waitKey(0);
  }
  ```

## 图像通道操作

### 分离通道 split()

**split函数**用于将一个多通道数组分离成几个单通道数组。

```c++
void split（InputArray src, OutputArrayOfArrays mv）
```

- src：填我们需要进行分离的多通道数组
- mv：填函数的输出数组或者输出的 vector 容器

```c++
vector<Mat>mv;	// Mat类型的vector容器
split(image, mv);
// BGR
imshow("Blue", mv[0]); 
imshow("Green", mv[1]);
imshow("Red", mv[2]);}  
// 当三个通道值相同时，则为灰度图。
mv[0] = 0;
mv[1] = 0;
```

根据 OpenCV 的 BGR 色彩空间（Bule、Green、Red，蓝绿红），其中**channels.at（0）**就表示引用取出channels中的**蓝色分量**，**channels.at（1）**就表示引用取出channels中的**绿色分量**，**channels.at（2）**就表示引用取出channels中的**红色分量**。

### 合并通道 merge()

**merge（）函数**是split（）函数的逆向操作——将多个数组合并成一个多通道的数组。它通过组合一些给定的单通道数组，将这些孤立的单通道数组合并成一个多通道的数组，从而创建出一个由多个单通道阵列组成的多通道阵列。

```c++
void merge（const Mat*mv, size_t count, OutputArray dst）
void merge（InputArrayOfArrays mv, OutputArray dst）
```

- mv：填需要被合并的输入矩阵或 vector 容器的阵列，这个mv参数中所有的矩阵必须有着一样的尺寸和深度
- count：当 mv 为一个空白的C数组时，代表输入矩阵的个数，这个参数显然必须大于 1
- dst：输出矩阵，和 mv[0] 拥有一样的尺寸和深度，并且通道的数量是矩阵阵列中的通道的总数

merge 函数的功能是将一些数组合并成一个多通道的数组。关于组合的细节，输出矩阵中的每个元素都将是输出数组的串接。其中，第 i 个输入数组的元素被视为 mv[i]

```c++
vector<Mat>mv;	// Mat类型的vector容器
split(image, mv);
// BGR
imshow("Blue", mv[0]);
imshow("Green", mv[1]);
imshow("Red", mv[2]);
Mat dst;
// mv[0]=0,mv[1]=0也就是B=0、G=0，剩下R为分割前的值，最终显示红色。
mv[0] = 0;
mv[1] = 0;
merge(mv, dst);
imshow("合并", dst);
```

**mixChannels（）函数**可多通道数组中提取出特定的单通道数组，实现一些复杂的通道组合。

```c++
mixChannels(const Mat* src, size_t nsrcs, Mat* dst, size_t ndsts, const int* fromTo, size_t npairs)
```

- src：**输入**数组或向量矩阵,所有矩阵的大小和深度必须相同。**(1个或者多个)**
- srcs：矩阵的数量
- dst：**输出**数组或矩阵**向量,大小和深度**必须与 src[0] **相同**
- dsts：矩阵的数量
- fromTo：指定被复制通道与要复制到的位置组成的索引对
- pairs ：fromTo中**索引对**的数目**（fromTo的数组元素个数除以2）**

```c++
vector<Mat>mv;	// Mat类型的vector容器
	split(image, mv);
	// BGR
	imshow("Blue", mv[0]);
	imshow("Green", mv[1]);
	imshow("Red", mv[2]);
	Mat dst;
	// mv[0]=0,mv[1]=0也就是B=0、G=0，剩下R为分割前的值，最终显示红色。
	mv[0] = 0;
	mv[1] = 0;
	merge(mv, dst);
	imshow("通道合并", dst);
int from_to[] = { 0,1,1,2,2,0 };//两个为一对，0到1，1到2，2到0.
mixChannels(&image, 1, &dst, 1, from_to, 3);
imshow("通道混合", dst);
```

## 最大值最小值minMaxLoc()、均值、标准差meanStdDev()

```c++
void minMaxLoc(InputArray src, double* minVal, double* maxVal = 0, Point* minLoc = 0, Point* maxLoc = 0, InputArray mask = noArray());
```

- src：输入的数组，若是图像，需为单通道图像
- minVal：返回**最小值**的指针。若无需返回，此值设为 NULL
- maxVal：返回**最大值**的指针。若无需返回，此值设为 NULL
- minLoc：返回**最小值位置**的指针（二维情况下）。若无需返回，此值设为 NULL
- maxVal：返回**最大值位置**的指针（二维情况下）。若无需返回，此值设为 NULL
- mask：可选的**掩膜操作**，非零掩码元素用于标记待统计元素，需要与输入图像集有相同尺寸

```c++
void cv::meanStdDev (InputArray src, OutputArray mean, OutputArray stddev, InputArray mask = noArray());
```

- src：输入的图像或矩阵
- mean：输出的均值矩阵，**Mat类型**
- stddev：输出的标准差矩阵，**Mat类型**
- mask：可选的掩码矩阵

```c++
void QuickDemo::statistic_demo(Mat& image)
{
	//定义最大值，最小值变量，用于接收函数查询的结果.
	double minVal, maxVal;
	//定义坐标，接收查询的最小值，最大值的坐标值.
	Point minLoc, maxLoc;
	//图像通道分离
	vector<Mat>mv;
	split(image, mv);
	for (int i = 0; i < mv.size(); i++)
	{
		//图像各通道像素及其位置的最小值和最大值
		minMaxLoc(mv[i], &minVal, &maxVal, &minLoc, &maxLoc);
		cout << "Channels:" << i << " the min value:" << minVal << " the max value:" << maxVal << endl;
		cout << "Channels:" << i << " the min point value:" << minLoc << " the max point value:" << maxLoc << endl;
	}
	//定义均值和标准差
	Mat mean, stddev;
	meanStdDev(image, mean, stddev);
	cout << "均值means:" << mean << endl;
	cout << "标准差stddev:" << stddev << endl;
}
```

也可以单独对每一个通道求均值和标准差:

```c++
// 均值方差
scalar s = mean(src);
printf("均值: %.2f %.2f %.2f\n",s[e],s[1],s[2]);
Mat mm,mstd;
meanStdDev(src,mm,mstd);
int rows = mstd.rows;
printf("rows:%d\n",rows);
printf("方差: %.2f %.2f%.2f\n",mstd.at<double>(8,0),mstd.at<double>(1,0),mstd.at<double>(2,0));
```

## 计算数组加权和addWeighted()

这个函数的作用是计算两个数组（图像阵列）的加权和。原型如下：

```c++
// 两个图像必须相同
void addWeighted(InputArray srcl, double alpha, InputArray src2, double beta, double gamma, OutputArray dst, int dtype = -l);
```

- src1：表示需要加权的第一个数组，常常填一个Mat
- alpha：表示第一个数组的权重
- src2：表示第二个数组，它需要和第一个数组拥有相同的尺寸和通道数
- beta：表示第二个数组的权重值
- gamma：一个加到权重总和上的标量值。其含义通过接下来列出的式子自然会理解
- dst：输出的数组，它和输入的两个数组拥有相同的尺寸和通道数
- dtype：输出阵列的可选深度，有默认值-1。当两个输入数组具有相同的深度时，这个参数设置为-1（默认值），即等同于src1.depth（）

下面的数学公式表示：用 addWeighted 函数计算以下两个数组（src1和src2）的加权和，得到结果输出给第四个参数，也就是 addWeighted 函数的作用的矩阵表达式。
$$
dst = src1[I] * alpha + src2[I] * beta + gamma
$$
其中 I 是多维数组元素的索引值。而且在遇到多通道数组的时候，每个通道都需要独立地进行处理。另外需要注意的是，当输出数组的深度为 CV_32S 时，这个函数就不适用了，这时候就会内存溢出或者算出的结果压根不对。

## 随机数 RNG

RNG类是 C++ 的随机数产生器。它可产生一个 64位的 int 随机数。目前可按均匀分布和高斯分布产生随机数。随机数的产生采用的是 Multiply-With-Carry 算法和 Ziggurat 算法。

RNG可以产生3种随机数：

```c++
RNG(int seed)         // 使用种子seed产生一个64位随机整数，默认-1
RNG::uniform( )       // 产生一个均匀分布的随机数
RNG::gaussian( )      // 产生一个高斯分布的随机数
RNG::uniform(a, b)    // 返回一个[a,b)范围的均匀分布的随机数，a,b的数据类型要一致，而且必须是int、float、double中的一种，默认是int。
RNG::gaussian( σ)     // 返回一个均值为0，标准差为σ的随机数。如果要产生均值为λ，标准差为σ的随机数，可以λ + RNG::gaussian(σ)
```

## 归一化函数 normalize()

归一化数据（为了方便数据的运算）。该函数分为范围归一化与数据值归一化。（先通过convertTo（）将数据类型转为浮点型（float），再进行归一化）。

```c++
void cv::normalize( InputArry src, InputOutputArray dst, double alpha = 1, double beta = 0, int norm_type = NORM_L2, int dtype = -1, InputArray mark = noArry());
```

- src：输入数组

- dst：输出数组，数组的大小和原数组一致

- alpha：要归一化的范数值或者为范围归一化的下边界

- beta：范围归一化的的下边界，在范数归一化时不会使用//为0时则为值归一化，否则为范围归一化

- norm_type：归一化选择的数学公式类型

- dtype：当为负，输出在大小深度通道数都等于输入，当为正，输出只在深度与输如不同，不同的地方游 dtype 决定

- mark：掩码。选择感兴趣区域，选定后只能对该区域进行操作

**归一化选择的数学公式类型介绍（norm_type）**





## 图像的基本变换

### 图像的缩放 resize()

```c++
void cv::resize(InputArray src,  OutputArray dst,
Size dsize, double fx=0, double fy=0,int interpolation = INTER_LINEAR)；
```

- InputArray src：输入图片
- OutputArray dst：输出图片
- dsize：输出图像的尺寸
- fx, fy  沿x轴，y轴的缩放系数（可选）
- interpolation：插入方式（可选）

| INTER_NEAREST  | 最近邻插值                   |
| -------------- | ---------------------------- |
| INTER_LINEAR   | 双线性插值（默认设置）       |
| INTER_AREA     | 使用像素区域关系进行重采样。 |
| INTER_CUBIC    | 4x4 像素邻域的双三次插值     |
| INTER_LANCZOS4 | 8x8 像素邻域的 Lanczos 插值  |

### 图像的翻转 flip()

```c++
void cv::flip(InputArray src, OutputArray dst, int flipCode);
```

- src:输入图像

- dst:输出图像，和src具有相同大小、类型。

- flipCode:旋转类型：

  - flipCode = 0：x轴方向旋转

  - flipCode > 0：y轴方向旋转

  - flipCode < 0：x轴y轴方向同时旋转

### 图像的旋转

```c++
void cv::rotate(InputArray src, OutputArray dst, int rotateCode)	
```

- src：输入图像
- dst：输出图像
- rotateCode
  - 0：90度
  - 1：180度
  - 2：270度

### 仿射变换

```c++
void cv::warpAffine (InputArray src, OutputArray dst, InputArray M, Size dsize, int flags = INTER_LINEAR, int borderMode = BORDER_CONSTANT, const Scalar& borderValue = Scalar());
```

- src：输入图像
- dst：输出图像，大小由 dsize 指定，图像类型与原图像一致
- M：2 X 3 的变换矩阵
- dsize：指定图像输出尺寸
- flags：插值算法标识符，有默认值 INTER_LINEAR，如果插值算法为 WARP_INVERSE_MAP, warpAffine 函数使用如下矩阵进行图像转换
- borderMode：边界像素模式，有默认值BORDER_CONSTANT
- borderValue: 边界取值，有默认值 Scalar() 即 0

**getRotationMatrix2D函数：用来创建转换矩阵 M**

```c++
Mat cv::getRotationMatrix2D(Point2f center, double angle, double scale); 
```

- center：图像的旋转中心
- angle：旋转角度（正的则向逆时针转）
- scale：一个各向同性的比例因子，根据提供的值将图像向上或向下缩放

$$
M = \left[
		\begin{array}{cc}
		cosθ & -sinθ	\\
		sinθ & cosθ		
		\end{array}
     \right]
$$

OpenCV 提供了为图像定义旋转中心的能力，以及一个缩放因子来调整图像的大小。在这种情况下，变换矩阵如下。其中 Cx 和 Cy 是图像旋转所沿的坐标。
$$
\left[
		\begin{array}{ccc}
		α & β &	(1 - α)c_x - βc_y      \\
		-β & α & βc_x + (1 - α)c_y 	
		\end{array}
\right] \\
α = scale·cosθ	\\
β = scale·sinθ
$$

```c++
Mat dst, M;
int width = image.cols;
int height = image.rows;
M = getRotationMatrix2D(Point2f(width / 2, height / 2), 45, 1.0); // 旋转45度
double cos = abs(M. at<double>(0, 0));
double sin = abs(M.at<double>(0, 1));
int nwidth = cos * width + sin * height;
int nheight = sin * width + cos * height;
M.at<double>(0, 2) += (nwidth / 2 - width / 2);
M.at<double>(1,2) += (nheight / 2 - height / 2);
warpAffine(image, dst, M, Size(nwidth, nheight), INTER LINEAR, 0, Scalar(165， 25， 89))
imshow("旋转演示", dst);
```

