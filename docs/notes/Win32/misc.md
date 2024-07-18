这里有一些奇妙的小技巧。

## 字太糊

以 VS 提供的窗口程序模板为例。这是默认的效果：

![image.png](https://s2.loli.net/2024/07/18/LYMzeymwBoKGUfQ.png)

可以看出，窗口里的字非常的糊。这是由于窗口默认是无 DPI 感知的，系统认为窗口的 DPI 是 96，并在此基础上对窗口进行放缩来适配显示器的 DPI，也就导致字很糊。我们可以在程序开头使用 `SetProcessDpiAwarenessContext` 函数来声明窗口的 DPI 感知来解决此问题，用法为：

```cpp
SetProcessDpiAwarenessContext(DPI_AWARENESS_CONTEXT value);
```

其中，`value` 的取值可以是：

|                   `value`                    |                                                       含义                                                       |
| :------------------------------------------: | :--------------------------------------------------------------------------------------------------------------: |
|       `DPI_AWARENESS_CONTEXT_UNAWARE`        |                                                   无 DPI 感知                                                    |
|     `DPI_AWARENESS_CONTEXT_SYSTEM_AWARE`     | 系统 DPI 感知，应用程序只在创建时查询 DPI，如果在使用过程中 DPI 改变，程序不会调整大小。此时系统会自动缩放窗口。 |
|  `DPI_AWARENESS_CONTEXT_PER_MONITOR_AWARE`   |                窗口不仅能在创建时查询 DPI，还能在 DPI 改变时作出适应。此时系统不会自动缩放窗口。                 |
| `DPI_AWARENESS_CONTEXT_PER_MONITOR_AWARE_V2` |                          比 `DPI_AWARENESS_CONTEXT_PER_MONITOR_AWARE` 功能更强大一些。                           |
|  `DPI_AWARENESS_CONTEXT_UNAWARE_GDISCALED`   |                              无 DPI 感知，但在高 DPI 显示器上能提高 GDI 绘图质量。                               |

除了 `DPI_AWARENESS_CONTEXT_UNAWARE`，其他的选项都可以使字不糊。值得注意的是，这个函数是向系统声明程序的 DPI 感知，程序适配 DPI 的操作需要我们自己实现。~~建议向系统说实话~~ 这是设置 DPI 感知为 `DPI_AWARENESS_CONTEXT_SYSTEM_AWARE` 后的窗口：

![image.png](https://s2.loli.net/2024/07/18/HvZln1az87UwXCt.png)

## 按钮太原始、工具提示不好用

!!! Tip "按钮太原始是什么样子？"

    ![image.png](https://s2.loli.net/2024/07/18/jFKrub8s695cxHU.png)

!!! Tip "什么是工具提示？"

    ![image.png](https://s2.loli.net/2024/07/18/XbMmvI6lAS4R7Kd.png)

程序开头加上这个就好用 ~~但是我不知道这是什么原理~~

```cpp
#if defined _M_IX86
#pragma comment(linker,"/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='x86' publicKeyToken='6595b64144ccf1df' language='*'\"")
#elif defined _M_IA64
#pragma comment(linker,"/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='ia64' publicKeyToken='6595b64144ccf1df' language='*'\"")
#elif defined _M_X64
#pragma comment(linker,"/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='amd64' publicKeyToken='6595b64144ccf1df' language='*'\"")
#else
#pragma comment(linker,"/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")
#endif
```

## 控件标识符、句柄的查询

### 已知父窗口的句柄、控件的标识符查询控件句柄

```cpp
HWND GetDlgItem(HWND hwnd, int id);
```

|  参数  |     含义     |
| :----: | :----------: |
| `hwnd` |  父窗口句柄  |
|  `id`  | 控件的标识符 |

这意味着我们不需要存储控件的句柄，只需要在使用的时候根据标识符查询即可。

### 已知控件句柄查询控件标识符

```cpp
int GetDlgCtrlID(HWND hwnd);
```

|  参数  |   含义   |
| :----: | :------: |
| `hwnd` | 控件句柄 |

这两个函数虽都以 `Dlg` （即对话框）命名，但任何窗口的控件都可以用这两个函数查询。