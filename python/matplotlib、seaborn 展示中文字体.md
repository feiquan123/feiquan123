# matplotlib、seaborn 展示中文字体

我们经常使用`matplotlib`、`seaborn`绘制各种图标但是失望的是他们对中文字符不支持。

![](https://img2020.cnblogs.com/blog/1297862/202102/1297862-20210222232814836-156240878.png)


## matplotlib 展示中文

查找系统上中文字体的安装位置，我选用的是宋体，然后拷贝到可读目录下`~/fonts/Songti.ttc`

```sh
mac: /System/Library/Fonts/Supplemental/Songti.ttc
window: C:\WINDOWS\Fonts\中文字体.ttf
linux: /usr/share/fonts
```

python 代码：

```python
import matplotlib.font_manager as fm,os
import matplotlib.pyplot as plt

# 添加字体
myfont = fm.FontProperties(fname=os.path.join(os.getenv('HOME'),'fonts/Songti.ttc'),size=10)
plt.xlabel('长度',fontproperties=myfont)
plt.ylabel('宽度',fontproperties=myfont)
```

![](https://img2020.cnblogs.com/blog/1297862/202102/1297862-20210222232843296-857854073.png)


## seaborn 展示中文

### 方法一：提供字体文件

1. 查找系统上中文字体的安装位置，我选用的是宋体，然后拷贝到可读目录下`~/fonts/Songti.ttc`

   ```
   mac: /System/Library/Fonts/Supplemental/Songti.ttc
   window: C:\WINDOWS\Fonts\中文字体.ttf
   linux: /usr/share/fonts
   ```

2. Python 代码：

   ```python
   import os
   import matplotlib.font_manager as fm
   import seaborn as sns
   import pandas as pd
   
   # 添加字体
   myfont = fm.FontProperties(fname=os.path.join(os.getenv('HOME'),'fonts/Songti.ttc'),size=10)
   sns.set(font=myfont.get_name())
   
   tips = pd.DataFrame({
       'time':["午餐","晚餐"],
       'total_bill':[1,2],
   })
   sns.pointplot(x="time", y="total_bill", data=tips)
   ```

   ![](https://img2020.cnblogs.com/blog/1297862/202102/1297862-20210222232855911-456319410.png)


### 方法二：添加字体

1. 查看 `matplotlib` 默认字体

   ```python
   from matplotlib.font_manager import findfont, FontProperties  
   
   findfont(FontProperties(family=FontProperties().get_familys
   # out: /home/xxx/venv/lib/python3.6/site-packages/matplotlib/mpl-data/fonts/ttf/DejaVuSans.ttf
   ```

2. 查找 `matplotlibrc`的路径

   ```python
   import matplotlib
   
   matplotlib.matplotlib_fname()
   # out: /home/xxx/venv/lib64/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc 
   ```

3. 修改`matplotlibrc`

   ```sh
   vim /home/xxx/venv/lib64/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc 
   ```

   取消注释`font.family`和`font.sans-serif` ,然后在`font.sans-serif`后追加你刚才的字符集名：

   ```python
   # 获取字符集名
   import matplotlib.font_manager as fm
   
   myfont = fm.FontProperties(fname=os.path.join(os.getenv('HOME'),'fonts/Songti.ttc'),size=10)
   myfont.get_name()
   
   # out: Songti SC
   ```

   修改：

   ```python
   font.family: sans-serif 
   #font.style:   normal
   #font.variant: normal
   #font.weight:  normal
   #font.stretch: normal
   #font.size:    10.0
   
   #font.serif:      DejaVu Serif, Bitstream Vera Serif, Computer Modern Roman, New Century Schoolbook, Century Schoolbook L,Utopia, ITC Bookman, Bookman, Nimbus Roman No9 L, Times New Roman, Times, Palatino, Charter, serif
   font.sans-serif: Songti SC, DejaVu Sans, Bitstream Vera Sans, Computer Modern Sans Serif, Lucida Grande, Verdana, Geneva, Lucid, Arial, Helvetica, Avant Garde, sans-serif
   #font.cursive:    Apple Chancery, Textile, Zapf Chancery, Sand, Script MT, Felipa, cursive
   #font.fantasy:    Comic Neue, Comic Sans MS, Chicago, Charcoal, ImpactWestern, Humor Sans, xkcd, fantasy
   #font.monospace:  DejaVu Sans Mono, Bitstream Vera Sans Mono, Computer Modern Typewriter, Andale Mono, Nimbus Mono L, Courier New, Courier, Fixed, Terminal, monospace
   ```

4. 移除`matplotlib`缓存

   ```sh
   rm -rf ~/.matplotlib
   ```

5. 拷贝你的字体文件到`matplotlib`字体目录下

   ```sh
   cp /System/Library/Fonts/Supplemental/Songti.ttc  /Users/Andrew/miniconda3/envs/technical-note/lib/python3.6/site-packages/matplotlib/mpl-data/fonts/ttf
   ```

6. 重启`jupyter Notebook` 内核，然后就可以了。

### 方法三：直接替换默认字体

1. 查看`matplotlib`默认字体

   ```python
   from matplotlib.font_manager import findfont, FontProperties  
   
   findfont(FontProperties(family=FontProperties().get_familys
   # out: /home/xxx/venv/lib/python3.6/site-packages/matplotlib/mpl-data/fonts/ttf/DejaVuSans.ttf
   ```

2. 备份字体

   ```
   cp /home/xxx/venv/lib/python3.6/site-packages/matplotlib/mpl-data/fonts/ttf/DejaVuSans.ttf /home/xxx/venv/lib/python3.6/site-packages/matplotlib/mpl-data/fonts/ttf/DejaVuSans.ttf.bp
   ```

3. 替换

   ```sh
   cp /System/Library/Fonts/Supplemental/Songti.ttc /home/xxx/venv/lib/python3.6/site-packages/matplotlib/mpl-data/fonts/ttf/DejaVuSans.ttf
   ```

4. 重启`jupyter Notebook` 内核，然后就可以了