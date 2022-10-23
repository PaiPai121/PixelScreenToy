# PixelScreenToy
## 硬件部分设计
### ESP8266系统
#### 最小系统
此处主要参考[【ESP8266】ESP12S/ESP12F最小系统设计及typeC自动下载电路设计](https://blog.csdn.net/qq_44078824/article/details/126149855)

处理器部分选用ESP-12S模块

<img width="821" alt="image" src="https://user-images.githubusercontent.com/53391594/197379575-de90a278-7c4f-4551-b094-6b82a39c4b5d.png">

其手册给出的应用图如下图

<img width="1044" alt="image" src="https://user-images.githubusercontent.com/53391594/197380073-5cad4e70-09d2-4b15-a835-504a2f4469a2.png">

仅需用10u和100n的电容并联对供电滤波即可。当然了，别的接口能用则用，还是都接上。

然后就是下载和运行模式的配置

<img width="1080" alt="image" src="https://user-images.githubusercontent.com/53391594/197380060-fff9d425-2315-442a-b48a-eaf6bb153dee.png">

这个时候需要EN拉高，RST拉高，GPIO2拉高，TXD0拉高（就是GPIO2），GPIO15拉低，GPIO0需要是可变的，毕竟我们要下载的，我们可以接个上拉电阻，同时再接其他的控制来切换下载状态。。

<img width="530" alt="image" src="https://user-images.githubusercontent.com/53391594/197380251-396f3066-28e4-46f8-b2d6-032ad0ae9ce7.png">

于是我们就得到了这样的最小系统模块。需要上拉或者下拉的引脚都已配置妥当。
其中nRST加入电容是为了在上电时有一段低电平时间，使模块开机复位。

#### typec接口
其实这是我头回画typec了，现在手头typec的线一般比较多，用c方便嘛
不过typec看上去可比以前的miniusb，microusb麻烦多了

<img width="565" alt="image" src="https://user-images.githubusercontent.com/53391594/197380469-5657940f-3ce9-447c-974d-c3bf8e2b685b.png">

我只准备启用usb2.0的部分，要让我做高速信号我一是不会，二是用不到啊。
接下来就看表吧。

<img width="317" alt="image" src="https://user-images.githubusercontent.com/53391594/197380513-61192326-f52b-48b2-8207-6c4613d77d1b.png">

SSTX*都是SS高速差分信号，通通不用。
D*就是我们的数据信号了，DP就是positive，DN就是negetive，该连的连起来，该接地接地，该供电供电。
CC1和CC2要拉低

#### 下载电路
下载还是用经典的CH340把usb的D+D-转换成Tx和Rx，这个芯片已经老生常谈了，这次参考开头提到的那个贴把他做成自动下载，其实就是自动的将eps12s调整到下载模式，也就是将GPIO拉低。
这时候就需要用到DTR和RTS两个引脚了。
其分别为MODEM 联络输出信号，数据终端就绪\ 以及\ MODEM 联络输出信号，请求发送，

下载的时候，我们需要把GPIO0拉低，然后复位设备，就可以下载了。虽然我也不知道为什么“推荐使⽤ CHIP_EN 进⾏芯⽚复位。”

<img width="302" alt="image" src="https://user-images.githubusercontent.com/53391594/197381076-d7068a5c-a669-4b10-af84-69bd73ee28d6.png">

这样一来两个信号便可通过控制三极管的基极电位来设置EN和GPIO0。

#### 供电
供电部分是电池+USB还是单USB其实我也没想好。
最好的话还是能够用电池独立工作吧，老拖着一根线太难看了。
usb输出是5V的，所以打算把电池供电也统一到5V来，准备整个纽扣电池升到5V。
但是单片机是3.3v的，肯定要一个5V-3.3V的芯片。
灯珠的部分就还是打算用5V供电了。

<img width="530" alt="image" src="https://user-images.githubusercontent.com/53391594/197381967-de2db9eb-ae6e-46da-90a2-58738410d91b.png">

不过啊，想用电池好麻烦啊，还得整充放电保护啥的。这次我就偷个懒

<img width="913" alt="image" src="https://user-images.githubusercontent.com/53391594/197393779-b6ab60dd-3dfe-44d6-b550-c687dc694bee.png">

直接用这个充放电好了

<img width="415" alt="image" src="https://user-images.githubusercontent.com/53391594/197395031-4c344c44-87b0-4d5b-8673-7a9f01b2249d.png">


然后修改一下，统一用这个5V供电

<img width="537" alt="image" src="https://user-images.githubusercontent.com/53391594/197394989-70614f43-eaa6-452d-a491-9c8d1d391b21.png">

### WS2812B5050灯珠阵列

这个阵列设置多大我一直没想太好
大了吧，物料又多又耗电，焊起来还麻烦
小了吧，你说8x8能显示个啥图像，啥也显不了。
我最开始计划是16*32
一算，512颗，得干死我。8*14好了。（想凑黄金分割比）

<img width="712" alt="image" src="https://user-images.githubusercontent.com/53391594/197398476-8d74b37e-4a69-4d93-94b2-fc4b7fcf0249.png">

大概就是这么样去排列灯珠了。

