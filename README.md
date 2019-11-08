# Miix700-OSX-Hackintosh-Clover

## 配置信息（Miix4，Lenovo Ideapad Miix700-12isk）

| CPU    | Intel Core m3-6y30 (Skylake)             |
|:------:|:----------------------------------------:|
| 核芯显卡   | HD515                                    |
| RAM    | 4GB DDR3                                 |
| 键盘&触摸板 | Miix700 USB keyboard                     |
| 触摸屏    | ELAN222F                                 |
| 声卡     | ALC236                                   |
| 屏幕     | 12英寸 2160x1440                           |
| 硬盘     | 128GB SATA m.2 SSD                       |
| 网卡&蓝牙  | Intel Wireless 8260AC （替换联想版本BCM94350ZAE无法驱动WIFI，DW1820A用08PKF4版本可以，但是睡眠后死机） |

## 正常工作

1. CPU变频（其他CPU型号使用[one-key-cpufriend](https://github.com/stevezhengshiqi/one-key-cpufriend)生成对应的kext）

2. 键盘&触摸板（键盘固件版本`V0010`,键盘映射按照需要手动调整，触摸板模拟鼠标工作）

3. 外放耳机正常

4. 麦克风

5. **触摸屏(需要手动修改DSDT)**

6. **电量显示(需要手动给DSDT打补丁)**

7. 盒盖关屏（休眠不太理想）

8. 亮度按键、音量按键（包括机身上的音量按键）  

9. HiDPI，使用一键[HiDPI](https://github.com/xzhih/one-key-hidpi)脚本，在最后手动输入几个3:2分辨率，注意纵向分辨率最好不要超过`800`，避免出现雪花等问题。





## 不工作&小毛病

1. I2C摄像头、重力感应、光线感应

2. 读卡器

3. Intel网卡&蓝牙（暂时使用USB网卡代替）

4. 休眠

5. 压感笔和触摸屏冲突，在使用压感笔后导致触屏的点击动作失效

6. 手写笔的压感

7. HDMI（未测试）

8. 休眠唤醒后键盘有一定几率失灵,重新插拔键盘可以解决  

## Todo

1. ~~替换并驱动联想版 `BCM94350ZAE`网卡，FRU PN: `00JT943`/`00JT944` ~~  （失败）
   
   ~~* 需要注意的是，联想大多数Skylake机型存在`网卡白名单`，在不清楚换什么网卡之前，请在联想官网查找你对应机型的`硬件维护手册（Hardware Maintenance Manual）`。 
   
   ~~* 对于`Miix700-12ISK`来说，可用的网卡只有`联想版Intel 8260AC`与`联想版BCM94350ZAE`。
   
   ~~* 所以市面上的DW1820A不要随便买
   
     * 尝试了 `0VW3T3` 的DW1820A,ID`1028:0021`,反面是`BCM94356ZEPA50DX_2`,无法驱动WIFI
     * 尝试 `08PKF4` 的DW1820A,可以驱动WIFI和BT，但是合盖睡眠后100%死机
     * 尝试 `00JT493` 的联想版BCM94350ZAE，无法驱动WIFI
     * 使用五个针脚屏蔽的方式无法找到PCI设备，请不要屏蔽5根针脚

2. I2C与触摸屏Hotpatch

3. 电池Hotpatch

4. 其他完善

5. <u>English verison of this document.</u>

## 许多内容都参考自小米Air 12.5的EFI， [github仓库地址](https://github.com/johnnync13/EFI-Xiaomi-Notebook-air-12-5),感谢[@johnnync13](https://github.com/johnnync13)  


## DSDT的修改

    由于个人水平有限，制作电池hotpatch、I2C hotpatch还有一段距离，所以把修改DSDT的来源贴出来。由于机型的DSDT的某些奇葩结构，粗暴地应用`VoodooI2C Patch`中的`SKL Controller Patch`会造成很多冲突，所以我们需要手动修改它们。

  在提取DSDT之前，建议在联想官网升级的BIOS和ACPI相关程序，并在BIOS设置中关闭 `Secure Boot`，当然，我还关闭了`Intel Platform Trust Technology`。

#### 电池补丁

使用clover提取整个ACPI表后，使用`iasl`对DSDT & SSDT进行联合反编译，  

使用RehanMan编译的`MaciASL`打开你的DSDT, 在`RehabMan Laptop`补丁源中找到  `[bat] Lenovo Ideapad-Y700` 这一个补丁，应用它。   

当然，在编译时可能出现一个错误，直接把出现问题的`Arg0 Arg1 Arg2 Arg3`注释/删除就好。

#### 触摸屏补丁

这个可能比较复杂，在此之前请保证电池补丁可以正常工作，再对DSDT进行进一步的修改。  

方法在在VoodooI2C的gitter聊天室中找到的，由[@blankmac](https://github.com/blankmac) 提出  

直接看聊天聊天记录就可以，至于前面的 **XCRS** 更名可以无视，从后面给出的 **I2C1**与**TSC2**操作就可以。

[链接在这里](https://gitter.im/alexandred/VoodooI2C/archives/2016/12/30)  

Miix710、Miix720用户也可以参考这一方法来实现触摸，因为这一系列的机型ACPI表中关于I2C的结构是类似的。

---

###### 下面给出具体的过程

1. ###### 给设备`TSC2`增加语句
   
   在MaciASL中搜索`TSC2`,找到以下描述：  
   
   ```javascript
   Device (TSC2)
   {
       Name (HID3, Zero)
       Method (_INI, 0, NotSerialized)  // _INI: Initialize
       {
           Store ("ELAN222F", _HID)
           Store (One, HID3)
           Return (Zero)
       }
   ```
   
   在第一句`Name (HID3, Zero)`下面增加一句`Name (_ADR, Zero)`  
   
   保证修改成这样：  
   
   ```javascript
   Device (TSC2)
   {
       Name (HID3, Zero)
       Name (_ADR, Zero)
       Method (_INI, 0, NotSerialized)  // _INI: Initialize
       {
           Store ("ELAN222F", _HID)
           Store (One, HID3)
           Return (Zero)
       }
   ```

2. ###### 修改`I2C1`中的描述
   
   搜索`I2C1`,找到以下语句：
   
   ```javascript
   Scope (_SB.PCI0)
    {
        Device (I2C1)
        {
            Name (LINK, "\\_SB.PCI0.I2C1")
        }
    }
   
   If (LNotEqual (SMD1, 0x02))
   {
       Scope (_SB.PCI0.I2C1)
       {
           Name (_HID, "INT3443")  // _HID: Hardware ID
           Method (_HRV, 0, NotSerialized)  // _HRV: Hardware Revision
           {
               Return (LHRV (SB11))
           }
   
           Method (_CRS, 0, NotSerialized)  // _CRS: Current Resource Settings
           {
               Return (LCRS (SMD1, SB01, SIR1))
           }
   
           Method (_PSC, 0, NotSerialized)  // _PSC: Power State Current
           {
               GETD (SB11)
           }
   
           Method (_PS0, 0, NotSerialized)  // _PS0: Power State 0
           {
               LPD0 (SB11)
           }
   
           Method (_PS3, 0, NotSerialized)  // _PS3: Power State 3
           {
               LPD3 (SB11)
           }
   
           Method (_STA, 0, NotSerialized)  // _STA: Status
           {
               Return (LSTA (SMD1))
           }
       }
   }
   
   If (LEqual (SMD1, 0x02))
   {
       Scope (_SB.PCI0.I2C1)
       {
           Name (_ADR, 0x00150001)  // _ADR: Address
           Method (_DSM, 4, Serialized)  // _DSM: Device-Specific Method
           {
               If (PCIC (Arg0))
               {
                   Return (PCID (Arg0, Arg1, Arg2, Arg3))
               }
   
               Return (Zero)
           }
       }
   }
   ```
   
   把`_PSC` ，`_PS0` 与` _PS3`从`scope`中`剪切`出来，并把他们放到`device`中，像这样：  

```javascript
   Scope (_SB.PCI0)
    {
        Device (I2C1)
        {
           Name (LINK, "\\_SB.PCI0.I2C1")
           Method (_PSC, 0, NotSerialized) // _PSC: Power State Current
            {
                GETD (SB11)
            }
            
           Method (_PS0, 0, NotSerialized)  // _PS0: Power State 0
           {
               LPD0 (SB11)
           }

           Method (_PS3, 0, NotSerialized)  // _PS3: Power State 3
           {
               LPD3 (SB11)
           }
       }
   }

   If (LNotEqual (SMD1, 0x02))
   {
       Scope (_SB.PCI0.I2C1)
       {
           Name (_HID, "INT3443")  // _HID: Hardware ID
           Method (_HRV, 0, NotSerialized)  // _HRV: Hardware Revision
           {
               Return (LHRV (SB11))
           }

           Method (_CRS, 0, NotSerialized)  // _CRS: Current Resource Settings
           {
               Return (LCRS (SMD1, SB01, SIR1))
           }

           Method (_STA, 0, NotSerialized)  // _STA: Status
           {
               Return (LSTA (SMD1))
           }
       }
   }

   If (LEqual (SMD1, 0x02))
   {
       Scope (_SB.PCI0.I2C1)
       {
           Name (_ADR, 0x00150001)  // _ADR: Address
           Method (_DSM, 4, Serialized)  // _DSM: Device-Specific Method
           {
               If (PCIC (Arg0))
               {
                   Return (PCID (Arg0, Arg1, Arg2, Arg3))
               }

               Return (Zero)
           }
       }
   }
```

   再进行编译，如果没有错误，那就另存为AML文件，放在Clover中，重启就可以了。

______
