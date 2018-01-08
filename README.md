# BarbarianBot 野蛮答题机器人

基于“投屏识别”方案的超快超轻量答题机器人。

本文好多名词是瞎编的，好些个数据都基于不严谨的瞎测，但方案绝对是好用的。如果有错误烦请指正，不要喷 :)

## 引言
近来，类似于**HQ Trivia**模式的在线答题软件吸引了很多人的注意。主流的平台包括**百万英雄**、**冲顶大会**、**芝士超人**。

有很多童鞋提出了很多答题的辅助手段。其中如何**获取题目和答案内容**是第一步，也是最关键的，作者能想到的无非是**OCR识别**和**REST接口**直接获取。

其中Restful接口获取难度较大，需要进行比较多的前期工作，OCR进而成为一种比较可行的通杀方案。

本文将讨论Windows加Android情况下，如何加速题目的获取和解答。

## 为什么OCR法很慢

**截屏OCR法**是先对手机屏幕进行截图，然后传输到电脑，进行文字识别的一种方法。但是速度却很不理想，原因其实不难找到：

- 用 "adb shell screencap/screenshot" 指令使安卓机进行截图操作，截图结果是PNG格式，涉及到了格式转换的过程。即使配置优秀的手机也要耗费至少2秒
- 用 "adb pull" 指令将图片拉回Windows，又要消耗2秒
- 本地对截图局部识别，又要消耗1到2秒

比起本地识别所消耗的时间，实际上**截图**和**拉回图片**所消耗的时间和资源更多。

## 思考

- 能否省掉截图和拉回图片的时间？
- 截图清晰度可否降低？可否只截图一部分？
- 能否利用电脑的硬件和网速优势加速整个过程？

## 提出方案

利用手机屏幕投影软件，将屏幕投影到Windows上。取得投屏软件句柄后，**利用Win的原生API进行局部截图操作**，提交云端OCR进行识别，获取题目后进行答案预测的工作。

经过肉眼测试，不同网络进行投屏连接，sigma-rt.com<sup>1</sup>的TC DS已经快爆，看不到延迟如果使电脑和手机在同一局域网下，或者使用数据线连接电脑和手机，速度可能更快。

当然，直接对模拟器进行截图也是可以的，这样可能更快一步。使得整个过程**加速的秘诀**就在于*砍掉了安卓机自截图和传图的流程，使用快到爆的Windows原生API<sup>2</sup>进行局部截图*。

- [1] 它家还有投屏操控的软件，用电脑来玩手机游戏，对实时性要求要多高有多高。
- [2] 对TC-DS软件局部截图，每次平均0.2秒

## 实践前的准备
- Python3
- [Pywin32](https://sourceforge.net/projects/pywin32/files/pywin32/)
- [TC-DS安卓投屏软件](http://www.sigma-rt.com/tcds/)

>Pywin32请下载和你的Python对应的版本，比如你的Windows是64位，但是你的Python3.6是32位的，那么请下载Pywin32-Python3.6.exe。如果你的Python3.7是64位的，请下载Pywin32-AMD64-Python3.7.exe。

## 快速截图例子

先来一个简单例子，Python如何根据软件的标题，调用Win的原生API进行局部截图。这是加速整个OCR识别的核心所在。

参考pyfunc的回答：[Python实现屏幕截图的最快方式](https://stackoverflow.com/questions/3586046/fastest-way-to-take-a-screenshot-with-python-on-windows)。


``` 
import win32gui, win32ui, win32con, win32api

title="安卓投屏大师 TC DS - 镜像"
dlg=win32gui.FindWindow(None, title)

def window_capture(hwnd, filename, w=355, h=655):
    hwndDC = win32gui.GetWindowDC(hwnd)
    mfcDC = win32ui.CreateDCFromHandle(hwndDC)
    saveDC = mfcDC.CreateCompatibleDC()
    saveBitMap = win32ui.CreateBitmap()
    saveBitMap.CreateCompatibleBitmap(mfcDC, w, h)
    saveDC.SelectObject(saveBitMap)
    saveDC.BitBlt((0, 0), (w, h), mfcDC, (0, 0), win32con.SRCCOPY)
    saveBitMap.SaveBitmapFile(saveDC, filename)

window_capture(dlg,"2.jpg")
```

#### 未完待续...



