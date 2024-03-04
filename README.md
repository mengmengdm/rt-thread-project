**RT-Thread x 大学生夏令营 华南理工大学第二十三组 智能家居终端的技术分享帖**

1.项目背景：
本次方案基于星火一号开发板开发，使用RT-Thread Studio进行工程创建，代码编辑，RT-Thread配置，调试配置，程序下载等功能。
项目成员均来自华南理工大学大二集成电路设计与集成系统专业，组长：方浩然，组员：谢天宇。

由于项目时间短暂，主要结合了板载资源，并基于一些现有的例程开发相关功能……

最终该项目的整体规划为实现一个小型的智能家居终端，利用一些板载资源采集数据/代表部分现实中的家居。主要功能包括以下几项：
1.实时温湿度采集并上传onenet，在onenet中实现数据可视化
2.通过板载按钮/云端指令控制LED灯阵，模拟控制家居灯阵
3.显示屏显示当前温湿度以及选择灯阵的相关信息
4.没用的功能：~~实现开启动画~~

------------

2.项目成品展示：
本项目通过五个主线程来控制和实现所有功能，分别是
system_start_thread	系统启动线程
led_matrix_thread	灯阵控制线程
get_color_thread	获取灯阵颜色、模式线程
temp_humi_thread	温湿度获取线程
lcd_show_thread		lcd显示线程

部分项目图片实际展示：
1.温湿度数据实时上传onenet
![onenet_temp.png](https://oss-club.rt-thread.org/uploads/20230725/ff042778373e5b2e1e7f2e5f9e5df8da.png)

![onenet_humi.png](https://oss-club.rt-thread.org/uploads/20230725/120b4e62a3175db5e805e6902758189a.png)

实现高温警告
![onenet_hitempwarning.png](https://oss-club.rt-thread.org/uploads/20230725/3995316c1ffd4b61dc23ba34649a334b.png)
工作效果
![shijixiaoguo.jpg](https://oss-club.rt-thread.org/uploads/20230725/7e21f7fbf3cbe07d279cc1896c29dcf7.jpg)

------------
3.项目具体实现
线程间通信
例子：实现不同数据在lcd屏幕上的显示，实现实时灯阵颜色和模式的选择。
首先发送mode与color
void get_color_entry()
{
    rt_thread_mdelay(100);
    rt_mb_send(&mode_choice, (rt_ubase_t)mode);
    rt_mb_send(&color_choice,(rt_ubase_t)yanse);

    rt_pin_mode(PIN_KEY_LEFT, PIN_MODE_INPUT_PULLUP);
    rt_pin_mode(PIN_KEY_DOWN, PIN_MODE_INPUT_PULLUP);
    rt_pin_mode(PIN_KEY_RIGHT, PIN_MODE_INPUT_PULLUP);
    rt_pin_mode(PIN_KEY_UP, PIN_MODE_INPUT_PULLUP);


       while (1)
       {
           /* 读取按键 KEY0 的引脚状态 并发送*/
           if (rt_pin_read(PIN_KEY_LEFT) == PIN_LOW)
           {
              rt_kprintf("left\n");
              mode--;
              xunhuan();
              rt_mb_send(&mode_choice, (rt_ubase_t)mode);
              rt_mb_send(&color_choice,(rt_ubase_t)yanse);
接受并实现led灯阵的控制：
void led_matrix_control_entry(void *parameter)
{
    RGBColor_TypeDef chozen_color;
    int *pmode;
    int *pyanse;

    rt_err_t mbRet1 = RT_EOK;
    rt_err_t mbRet2 = RT_EOK;

    led_matrix_clear();

    while(1)
    {
        mbRet1 =rt_mb_recv(&mode_choice, (rt_ubase_t*)&pmode, RT_WAITING_FOREVER);
        mbRet2 =rt_mb_recv(&color_choice, (rt_ubase_t*)&pyanse, RT_WAITING_FOREVER);

        color_parameter = enter_color(pyanse);
        mode_choice_funtion(pmode, color_parameter);

        if(mbRet1==RT_EOK)
        {
            rt_kprintf("mode_choice:%d",pmode);
        }

------------

4.问题与未来方向
问题1：例程中的引脚定义不统一
例程中的引脚定义不统一导致一些程序能够正常编译但是不能灯阵不能正常展示
需要通过STM32CubeMX进行调整

问题2：使用led_matrix[i].io_ctl(&led_matrix[i],COLOR);在对多个LED同时进行更改时会出现延时，例如当同时使用该代码点亮所有外圈LED时会先亮一部分再亮另一部分。
如果使用Set_LEDColor(laite,RED); RGB_Reflash();则能够让所有灯同时亮起。

未来方向1：开发利用其他板载资源
星火一号的板载资源，相关软件包非常丰富，未来可以进一步利用未使用的板载资源进行进一步的开发。

未来方向2：开发移动端/桌面端应用
可以直接从移动端/桌面端向开发板发送控制指令，整个方案更加完善。

------------

ps.实现开场动画
运用img2lcd软件将图片变为c数组，调整合适大小后调用lcd_show_image()函数显示图片。
通过循环，实际效果为动图。

![xiduo.png](https://oss-club.rt-thread.org/uploads/20230725/a556e8b6ebfeea57b83e3cbd7e08a081.png.webp)

发帖人：方浩然
时间2023.07.25
