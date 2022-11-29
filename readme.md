
而且还是跨平台的，无论是在 Linux、Windows 还是 macOS 上都能使用。

项目地址：

https://github.com/Genymobile/scrcpy

这里以 Windows 为例，安装起来非常简单：

图片
直接下载安装包并解压就行，里面还带了adb调试工具。

然后添加一下系统的环境变量。

图片
用 USB 线连接手机和电脑，将手机调整为开发者模式。

图片
这样，在电脑上运行 Scrcpy 指令：

scrcpy
就能将手机画面投放到电脑上了。

图片
我们可以通过这个投放的画面，在电脑上，操纵这台手机。

但是，如果想要让代码自动化控制，那就还需要另外一款工具。

uiautomator2
UiAutomator 是 Google 提供的用来做安卓自动化测试的一个 Java 库，基于 Accessibility 服务。功能很强，可以对第三方 App 进行测试，获取屏幕上任意一个 APP 的任意一个控件属性，并对其进行任意操作，但有两个限制：

测试脚本只能使用 Java 语言
测试脚本要打包成 jar 或者 apk 包上传到设备上才能运行
于是有了 UiAutomator2，逻辑可以用 Python 编写，能够在电脑上控制手机。

项目地址：

https://github.com/openatx/uiautomator2

安装方法也非常简单，直接 pip 安装即可，不过为了方便环境的管理，还是先创建一个 Conda 虚拟环境。

conda create -n android
然后激活这个虚拟环境：

conda activate android
安装 uiautomator2 和 weditor。

python -m pip install uiautomator2 weditor
然后用手机打开想要操控的 App，比如 BiliBili，打开软件后，使用 Weditor 审查元素。

python -m weditor
这样就开启了一个 Web 界面，在这个界面里，能够审查元素，定位一些想要点击的点。

图片
比如我想要给一个视频三连，那就审查三连的元素，然后将操作用代码流程化。

import uiautomator2 as u2
import time
from PIL import Image
import cv2
import numpy as np

all_videos = []

def get_images(device):
    views = device.xpath('//*[@resource-id="tv.danmaku.bili:id/recycler_view"]/android.view.ViewGroup/android.widget.FrameLayout[1]')
    for idx, view_box in enumerate(views.all()[:5]):
        print("视频{} 封面的中心坐标:".format(idx+1), view_box.center())
        image = view_box.screenshot()
        image.save("{}.jpg".format(idx+1))
        all_videos.append(view_box)

def refresh(device):
    device.swipe_ext("down")

def like_the_video(device):
    like_icon = device.xpath('//*[@resource-id="tv.danmaku.bili:id/recommend_icon"]')
    like_icon.click()
    print("视频点赞成功")

def pay_for_the_video(device):
    coin_icon = device.xpath('//*[@resource-id="tv.danmaku.bili:id/coin_icon"]')
    coin_icon.click()
    time.sleep(0.1)
    pay_icon = device.xpath('//*[@resource-id="tv.danmaku.bili:id/pay_coins"]')
    pay_icon.click()
    print("视频投币成功")

def follow_the_up(device):
    follow_icon = device.xpath('//*[@resource-id="tv.danmaku.bili:id/follow"]')
    follow_icon.click()
    print("关注成功")

def back(device):
    back_icon = device.xpath('//*[@content-desc="转到上一层级"]')
    back_icon.click()
    print("已退出视频")

if __name__ == "__main__":
    _DEVICE_ID = 'da317199'
    d = u2.connect(_DEVICE_ID) # connect to device
    get_images(d)
    all_videos[3].click()
    print("点进去了！")
    time.sleep(0.1)
    like_the_video(d)

    time.sleep(0.1)
    pay_for_the_video(d)

    time.sleep(0.1)
    follow_the_up(d)
效果是这样的：


代码里有个 device_id 可以通过 adb 工具查询，手机连接电脑后，再用如下指令查询：

adb devices
根据这个设备号，就能操纵这台手机。

絮叨
这东西，还能用来做什么？

你知道，为啥抢茅台你总抢不到吗？抢个演唱会门票抢不到？挂号挂不到？

票贩子为啥总能抢到票？

很多平台限制必须手机上抢，这个时候，这项技术就能排上用场了。

我只能帮你到这了，剩下的，可以自己慢慢体验。

所以，家里的旧手机不要扔！写个脚本，能帮你干很多事~
