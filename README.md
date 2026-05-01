# Lane-Departure-Warning-Lane-Keeping-Assist-Lane-Marking-Detection-Graduation-Project
本项目使用车载摄像头检测当前车道、判断车辆偏离情况并分析前方道路状况。可在界面中展示车道的状态、车辆的位置偏移、车道的曲率、车道线的鸟瞰图、车道线的二值图、以及系统的运行帧率等信息。可用于毕设、课设、工业实际项目，可提供整套代码、测试视频和详细说明文档。可以部署到树莓派、香橙派、Jetson Nano、瑞芯微RK3588等开发板上，也可调用摄像头输入视频流实时推理。

# 引言
**本项目使用车载摄像头检测当前车道、判断车辆偏离情况并分析前方道路状况。可在界面中展示车道的状态、车辆的位置偏移、车道的曲率、车道线的鸟瞰图、车道线的二值图、以及系统的运行帧率等信息。可用于毕设、课设、工业实际项目，可提供整套代码、测试视频和详细说明文档。可以部署到树莓派、香橙派、Jetson Nano、瑞芯微RK3588等开发板上，也可调用摄像头输入视频流实时推理。**

 [**效果展示视频：https://www.bilibili.com/video/BV123rqYxEcj/?share_source=copy_web&vd_source=138d2e7f294c3405b6ea31a67534ae1a**](https://www.bilibili.com/video/BV123rqYxEcj/?share_source=copy_web&vd_source=138d2e7f294c3405b6ea31a67534ae1a) (点击观看完整视频)
 
  [**操作演示视频：https://www.bilibili.com/video/BV16srqY2ELZ/?share_source=copy_web&vd_source=138d2e7f294c3405b6ea31a67534ae1a**](https://www.bilibili.com/video/BV16srqY2ELZ/?share_source=copy_web&vd_source=138d2e7f294c3405b6ea31a67534ae1a) (点击观看完整视频)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6c313c8b59aa41519b494901d0cdd485.gif#pic_center)


---
# 一、代码和文件
## 1. 项目文件说明
* calibration.py校准车载相机和保存校准结果
* main.py 演示程序的主代码
* lane.py包含车道类 
* camera_cal该文件夹中包含用于摄像机校准的图像和校准结果 
* examples该文件夹中包含样本图像和视频
* environment-gpu.yml GPU的环境配置
* README.md 说明文档（需要的包，包的版本）

## 2. 项目的依赖包和环境

* OpenCV3, Python3.5 
* 可以按照提供的GPU的环境配置：environment-gpu.yml，来安装依赖项。
* 操作系统：Windows（也适用于Ubuntu等其他平台）。

## 3. 运行代码
直接运行main.py这个脚本即可。这个脚本有三种工作模式：
1. **图片处理模式**。对单张图片进行车道偏离预警判断并显示处理结果；
2. **视频处理模式**。对本地视频逐帧处理，实现车道偏离预警判断，并保存处理后的新视频（原始视频路径和处理后的视频保存路径均可在代码中设置）；
3. **实时摄像头检测模式**。利用电脑或车载摄像头获取实时视频流，对每一帧进行车道偏离预警判断并显示处理结果，同时保存处理后的视频（视频保存路径可在代码中设置）。

代码中使用了 OpenCV 进行图像处理和显示，MoviePy 用于视频处理，支持窗口动态显示和结果保存功能。
```python
"""Departure Warning System with a Monocular Camera"""

from lane import *  # 导入自定义的车道线处理相关模块
import cv2  # OpenCV，用于图像处理
from moviepy.editor import VideoFileClip  # 用于视频处理

if __name__ == "__main__":
    demo = 2  # 设置模式，1表示处理单张图片，2表示处理视频，3表示实时摄像头检测
    if demo == 1:  # 图片处理模式
        imagepath = 'examples/test3.jpg'  # 图片路径
        img = cv2.imread(imagepath)  # 读取图片
        img_aug = process_frame(img)  # 调用处理函数，对图片进行处理
        # 创建一个可调整大小的窗口
        cv2.namedWindow("Original Image", cv2.WINDOW_NORMAL)  # 设置窗口为可调整大小
        cv2.namedWindow("Augmented Image", cv2.WINDOW_NORMAL)  # 设置窗口为可调整大小
        # 显示原始图片和处理后的图片
        cv2.imshow("Original Image", img)  # 显示原始图片
        cv2.imshow("Augmented Image", img_aug)  # 显示处理后的图片
        # 按下任意键关闭窗口
        cv2.waitKey(0)
        cv2.destroyAllWindows()  # 销毁所有窗口

    elif demo == 2:  # 视频处理模式
        video_path = 'examples/project_video.mp4'  # 输入视频路径
        video_output = 'examples/project_video_augmented.mp4'  # 输出视频路径
        # 定义一个回调函数，用于实时显示视频帧处理结果
        def process_and_display(frame):
            """
            处理每一帧并实时显示，同时保证视频保存一致性。
            """
            processed_frame = process_frame(frame)  # 调用处理函数，处理每一帧
            # 创建一个可调整大小的窗口
            cv2.namedWindow("Departure Warning System", cv2.WINDOW_NORMAL)  # 设置窗口为可调整大小
            # 实时显示结果
            cv2.imshow("Departure Warning System", processed_frame)  # 仅显示处理后的帧
            if cv2.waitKey(1) & 0xFF == ord('q'):  # 按下 'q' 键退出
                cv2.destroyAllWindows()
                exit()
            return processed_frame  # 返回处理后的帧供保存视频

        # 定义一个函数，用于处理并保存视频帧，同时实时显示
        def process_video_frame(frame):
            """
            用于 MoviePy 的逐帧处理函数，同时进行实时显示。
            """
            processed_frame = process_and_display(cv2.cvtColor(frame, cv2.COLOR_RGB2BGR))  # 转换为 BGR
            return cv2.cvtColor(processed_frame, cv2.COLOR_BGR2RGB)  # 转换回 RGB 格式以保证输出视频的颜色一致性
        # 使用 MoviePy 处理视频
        clip1 = VideoFileClip(video_path)  # 读取视频文件
        clip = clip1.fl_image(process_video_frame)  # 逐帧处理视频
        clip.write_videofile(video_output, audio=False)  # 保存处理后的视频
        cv2.destroyAllWindows()  # 销毁所有OpenCV窗口

    elif demo == 3:  # 实时摄像头检测模式
        camera_index = 0  # 摄像头索引，默认使用第一个摄像头
        video_output = 'examples/camera_output.mp4'  # 保存视频路径
        cap = cv2.VideoCapture(camera_index)  # 打开摄像头
        # 检查摄像头是否成功打开
        if not cap.isOpened():
            print("Error: Cannot open camera.")
            exit()
        # 设置摄像头分辨率为1280x720
        cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
        cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)
        # 手动设置帧率（确保帧率有效）
        fps = 30
        frame_width = 1280
        frame_height = 720
        # 定义视频保存格式和编码器（确保兼容MP4格式）
        fourcc = cv2.VideoWriter_fourcc(*'mp4v')
        out = cv2.VideoWriter(video_output, fourcc, fps, (748, 450))
        print("Press 'q' to exit...")
        while True:
            ret, frame = cap.read()  # 读取摄像头帧
            if not ret:
                print("Error: Cannot read frame from camera.")
                break
            # 调整每一帧的大小为1280x720
            frame_resized = cv2.resize(frame, (1280, 720))
            # 调用处理函数，处理当前帧
            processed_frame = process_frame(frame_resized)
            if processed_frame is None:
                print("Error: Processed frame is empty!")
                break
            # 确保 processed_frame 是 uint8 类型，三通道
            processed_frame = processed_frame.astype(np.uint8)
            if len(processed_frame.shape) == 2:
                processed_frame = cv2.cvtColor(processed_frame, cv2.COLOR_GRAY2BGR)
            # 实时显示处理后帧
            cv2.imshow("Processed Frame", processed_frame)
            # 写入视频文件
            try:
                out.write(processed_frame)
            except Exception as e:
                print(f"Error writing frame to file: {e}")
                break
            if cv2.waitKey(1) & 0xFF == ord('q'):  # 按下 'q' 键退出
                break
        # 释放资源
        cap.release()
        out.release()
        cv2.destroyAllWindows()
```

如果想使用代码校准自己的摄像头并测试，请将摄像头拍摄的多张棋盘格图像保存到 camera_cal文件夹，并用calibration.py这个脚本进行校准。

---
# 二、整体代码逻辑
## 1. 相机校准
(1) 用摄像机收集一组棋盘图像。
(2) 利用给定的一组图像计算摄像机校准矩阵和失真系数。
(3) 对原始图像进行失真校正。
## 2. 车道检测/跟踪
(1) 使用颜色变换、渐变梯度等创建阈值化的二值图像。
(2) 应用透视变换矫正二值图像（得到"鸟瞰图"）。
(3) 检测/跟踪车道像素并拟合，找到车道线边界。
## 3. 车道状态分析
(1) 计算得到车道曲率。
(2) 计算车辆相对于车道中心的位置。
## 4. 车道映射
(1) 将检测到的车道边界映射回原始图像上。
(2) 将车道偏离预警结果打印输出到图像中。

---
# 三、 相机校准
## 1. 计算摄像机矩阵和失真系数
该步骤的代码：calibration.py。首先要得到 "object points"，也就是真实世界中棋盘角点的 (x, y, z) 坐标。 这里假设棋盘固定在 z=0 的 (x, y) 平面上，这样，每个校准图像的object points都是相同的。因此，`objp` 只是一个复制的坐标数组，每次成功检测到图像中的所有棋盘角点时，`objpoints` 都会添加一个副本。每次成功检测到棋盘角点后，`imgpoints` 都会在图像平面上添加每个角点的（x, y）像素位置。然后，使用输出的 `objpoints` 和 `imgpoints` 函数计算相机校准和畸变系数。使用 `cv2.undistort()`函数对测试图像进行失真校正，得到以下结果：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/16208bb1eb4b4c6eb0845f474aed3e71.jpeg#pic_center)

---
# 四、图像和视频的处理步骤
项目脚本lane.py中的核心函数process_frame()的解读。
## 1. 进行图像失真校正
在此步骤中，cv2.undistort 与之前的校准结果一起使用（见 process_frame() 函数的第 2 行）。 测试图像的示例如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/285a76bad2de4cd99377ac17bfcc2361.jpeg#pic_center)
## 2. 使用颜色和梯度阈值化得到二值图像
使用颜色和梯度（阈值）组合得到二值图像（见 `lane.py`中的函数 `find_edges()`）。
首先将图像转换到 HLS 空间，然后使用 S 通道对图像进行过滤，使得算法在不同的光照条件下都能很好地定位黄色和白色车道。然后使用 sobel 滤波器（沿 x 轴方向）和梯度方向的滤波器滤除大部分水平线。 最后，为了处理发现两条以上候选车道的情况，为 S 通道滤波器设定了两倍于梯度滤波器的权重。 因此，黄色车道线比道路的边缘更加显著。 下面是我这一步的输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/eb7994a604d44bc5a2c6bed2d8d7accf.png#pic_center)
## 3. 使用透视变换得到鸟瞰图
项目中的透视变换代码包含一个名为 `warper()` 的函数。 首先，选择一帧车辆直线行驶的图像，并选择沿两条车道线形成梯形的 4 个原始像素点。然后定义了另外 4 个目标点，这样就可以使用`cv2.getPerspectiveTransform` 函数进行透视变换（见下面两张图片）。 得到原始像素点和目标点如下表格所示。
| Source        | Destination   | 
|:-------------:|:-------------:| 
| 194, 719      | 290, 719        | 
| 1117, 719      | 990, 719      |
| 705, 461    | 990,  0     |
| 575, 461      | 290, 0        |

再将 "Source"点和 "Destination"点绘制到测试图像及其变换后的对应图像上，验证透视变换的有效性。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8473c30509c340bfa572e1684ce1a068.png#pic_center)
大家可能会注意到，图像的长宽比发生了改变（车道被压扁），但如果这里用恰当的反变换将图像变换回来，不会对最终的结果图像产生任何影响。如果想看到比例更合适的图像，也可以根据梯形图中车道宽度和长度的合适比例来选择目标点。 如图：
![!\[alt image\]\[image-warper2\]](https://i-blog.csdnimg.cn/direct/19a46cb93bc14232af19ea3889cd8845.png#pic_center)

## 4. 提取车道线的像素区域并用多项式拟合其位置
在对道路图像进行校准、阈值处理和透视变换后，可以得到一张车道线清晰可见的二值图像。
但是，仍需要通过算法确定哪些像素属于车道线，哪些属于左侧车道线，哪些属于右侧车道线。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3ee461e9864f43fdb6154fd04ff5f1f9.png#pic_center)
## 5. 单张图像的处理
如果输入是单张图像或视频的第一帧，则使用`detector` 函数（lane.py 中）。它对图像的处理过程如下：

(1) 步骤一
首先沿图像下半部分的所有列绘制像素统计直方图。该直方图中最突出的两个峰值的位置作为寻找车道线的起点。如下图形：
![!\[alt image\]\[image-hist\]](https://i-blog.csdnimg.cn/direct/3f31a9c4c1334778b008881e2073081b.png#pic_center)

(2) 步骤二
从这一点出发，使用一个滑动窗口，放在线条中心周围，来寻找和跟踪线条。 它从底部开始并向上搜索到图片的顶部。 如下图所示：
![!\[\]\[image-win\]](https://i-blog.csdnimg.cn/direct/2760e0ea9ed147a18bcdc9c067bbe727.png#pic_center)

(3) 步骤三
最后，使用lane.py中的 `np.polyfit` 对车道线进行二阶多项式拟合，如下：

```python
    # 对左右车道线像素分别拟合二次多项式
    left_fit = np.polyfit(lefty, leftx, 2)
    right_fit = np.polyfit(righty, rightx, 2)
```
## 6. 视频的处理
(1) 步骤一
如果输入是视频（本地离线视频和实时摄像头视频同理），则第一帧可与单个图像一样处理。这样就能估计出左右车道的位置，将这个位置作为查找下一帧图像中车道的初始位置。 "lane.py "中名为 "tracker "的函数用于处理这种情况。

(2) 步骤二
为了在连续帧中得到平滑的车道，代码中使用了帧缓冲区来保存前 N 帧的车道位置，如下所示：

```python
        # 车道线最近 N 次拟合的 x 坐标
        self.prev_fitx = []
```

(3) 步骤三
如果跟踪失败（左右车道之间的距离标准偏差过大），要么保留之前的车道位置，要么开始新的检测。

# 五、计算车道的曲率半径和车辆相对于车道中心的位置

(1)曲率半径
曲线在某一点的曲率半径定义为近似圆的半径。[参考文献链接：http://www.intmath.com/applications-differentiation/8-radius-curvature.php](http://www.intmath.com/applications-differentiation/8-radius-curvature.php)

![!\[\]\[image-cur\]](https://i-blog.csdnimg.cn/direct/c6646608ae5b468e917c96b31d22f17e.png#pic_center)

`lane.py`中的函数`measure_lane_curvature`用于计算车道的曲率半径。

(2)车辆位置
假设摄像头大致安装在前车窗的中央位置，并且已经计算出了左右车道的位置。因此，可以只取左右车道的底部位置，然后与图像帧的中间位置进行比较，进行判断。 此外，还可以利用每条车道左右车道线之间的距离的先验知识（例如国内为3.5~3.75米）来估计实际距离。项目中的 `lane.py` 脚本中的 `off_center` 函数实现了这个算法逻辑。

# 六、车道线绘制和车道偏离预警
最后，将检测到的车道区域变换回原始输入帧图像。如果车辆离车道边缘太近，将车道颜色从**绿色**变为**红色**，从而警告驾驶人员（参见 `lane.py` 中的函数 `create_output_frame`）。

```python
def create_output_frame(offcenter, pts, undist_ori, fps, curvature, curve_direction, binary_sub, threshold=0.3):
    # 上面的threshold阈值可以根据视频效果调整。如果汽车偏离中心量的绝对值大于阈值（单位：米）绘制红色车道报警，否则绘制绿色车道
    """
    :param offcenter:
    :param pts:
    :param undist_ori:
    :param fps:
    :param threshold:
    :return:
    """
    undist_ori = cv2.resize(undist_ori, (0, 0), fx=1 / output_frame_scale, fy=1 / output_frame_scale)
    # 获取 undist_ori 图像的宽度
    w = undist_ori.shape[1]
    # 获取 undist_ori 图像的高度
    h = undist_ori.shape[0]
    # 对 undist_ori 图像进行缩放并进行透视变换，使用 M_b 变换矩阵
    undist_birdview = warper(cv2.resize(undist_ori, (0, 0), fx=1 / 2, fy=1 / 2), M_b)
    # 创建一个与 undist_ori 形状相同的全零数组，数据类型为 uint8，用于绘制颜色信息
    color_warp = np.zeros_like(undist_ori).astype(np.uint8)
    # 创建一个用于容纳所有图像的帧，尺寸根据 h 和 w 进行扩展，数据类型为 uint8
    whole_frame = np.zeros((int(h * 2.5), int(w * 2.34), 3), dtype=np.uint8)
    # 如果偏离中心量的绝对值大于阈值（汽车偏离中心超过 0.6 米）
    if abs(offcenter) > threshold:
        # 绘制红色车道
        cv2.fillPoly(color_warp, np.int_([pts]), (0, 0, 255))  # 红色
    else:  # 绘制绿色车道
        cv2.fillPoly(color_warp, np.int_([pts]), (0, 255, 0))  # 绿色
```

---
# 七、总结
由于涉及交通安全，对于像车道偏离这样的预警系统，误报率要达到极低的水准才能实际应用。当前，驾驶者可接受的误报率的具体数值仍是一个重要研究课题[1]。此外，道路和车道线的检测需要应对的场景多样，如下图所示，车道线和道路外观会发生巨大变化。
![!\[\]\[image-hard_case\]](https://i-blog.csdnimg.cn/direct/9ad8c3ad9867402187fbdd4f84ed7b74.png#pic_center)

本项目的车道偏离预警算法基于opencv实现的图像预处理、透视变换、滑动窗口搜索、多项式拟合、车道线跟踪、曲率半径和偏移计算等数字图像处理技术和计算机视觉技术，涉及到硬编码透视变换和多个手动调整的车道线参数。该技术路线在例如高速公路、白天、清晰的车道标志等良好路况下能满足精度需求。未来，可以尝试使用多种传感模式，如立体摄像机、激光雷达、雷达等（不仅只是车载相机），实现多模态融合。

参考资料: [1] [道路和车道线检测的最新进展--Review：https://pdfs.semanticscholar.org/223c/087455c1adc23562d5ea2ebe47cd077feb68.pdf](https://pdfs.semanticscholar.org/223c/087455c1adc23562d5ea2ebe47cd077feb68.pdf)

---
# 八、资源获取(可提供整套代码、测试视频、详细说明文档)
代码有详细注释，包全程指导，任何问题都可以随时问我。不过有的时候我太忙，可能不会及时回复消息，看到了肯定回你哈。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5d0ce42288524d52b8caec9423d8a311.png#pic_center)

包含完整word版本说明文档（共12页，4114字），可用于写论文、课设报告等参考。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2a0b90cf09fd47bf9dcda4fa9fb1c555.png#pic_center)

包含代码、测试视频、详细说明文档等。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/edc9ca89aeec49448405052cfb6d016d.png#pic_center)

**联系我：资源获取。** 

```python
获取整套代码、测试视频、训练好的权重和说明文档(有偿)
上交硕士，技术够硬，也可以指导深度学习毕设、大作业等。
--------------->qq------------
           3582584734
------------------------------
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c7e9309a04bd6f22ae3f1138149f65ea.png)
