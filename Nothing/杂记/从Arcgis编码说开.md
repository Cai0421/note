# 从ArcGIS编码说开

## 1. 前言

前段时间，遇见了如下的两个问题

1. power shell安装oh-my-posh后显示为方块
2. ArcGIS通过csv转换得到的point shapefile的属性信息同地理坐标发生错位

  而最终对于上述两个问题的解决，其问题的本质均指向了编码问题，同样也是在平时编写代码过程中最为重要但同时也即为容易被忽略的问题，下面如何解决上述问题以及就编码这一概念展开解释。

## 2. 编码

### 2.1 编码本质

  日常代码过程中遇见的绝大部分的乱码问题都可以归结到编码问题，而编码的本质其实就是对于给计算机的一本**字典**，即编码实现了如下两件事情

1. 给所有字符一个独一无二的数字编号，通过mapping的机制进行一一映射，如我将"00000001"映射给"蔡"这一中文字符
2. 该数字编号能用0、1表示

  对于2而言，由于编码的长度存在不定长的情况，如'才'映射给"00000010",而"菜"用"00000001 00000010"，而计算机如何分辨读取到的编码应该映射为“蔡才”还是“菜”便成了问题。（常见的UTF-8编码中的中文常为3-4个字节），而对于第二个问题的解决一般是采用**定长**的策略，即规定多少字节的代表一个字符，对于**未达到长度的字符前面补0**进行解决。

### 2.2 乱码的背后机理

  说起乱码对于大部分人影响最为深刻的应该为C以及C++命令行输出“烫烫烫”或者“屯屯屯”的乱码形式，其输出如下乱码的原因是VS studio部分的编辑器对于未分配的内存空间会默认填入相关的内容。

1. 未分配或者静态分配而未赋初值的内存采用0xCC填充

   ```c++
   int C; // default C = -858993460(0xCCCCCCC)
   	   // 烫（0xCCCC）
   ```

2. 动态分配(new, malloc)而未赋初值的内存采用0xCD填充，其中屯(0xCDCD)

  由上述例子我们可以发现乱码的本质就是**编码解码采用了不同的标准**。

  对于计算机而言计算机只能识别0/1进制数据，这就决定了我们需要实现一下两种过程

**编码**：文字符号 $\to$ 二进制数据

**解码**：二进制数据 $\to$ 文字符号

如下为将字符串先进行utf-8编码，再利用gbk解码，最终输出的结果即为乱码

```python
s = "蔡菜菜"
s.encode('utf-8').decode('gbk',errors='ignore') #‘ignore’ 去除非gbk框架内的字符
# output s = '钄¤彍鑿'
```

### 2.3 字体和编码

  从上面可以得知，不同的编码体系主要实现的内容是**字符编码**同**二进制存储码**之间转换的不同体系的“字典”，而字体库即是对于字符编码的可视化显示过程的字典，所以文字调用的过程可以解释为下列流程

1. 根据计算机提供的二进制数据，在确定的编码体系中（如UTF-8)，转换为目标的**文字编码**
2. 根据文字编码同程序中设置的文字样式（如Mono)等，提取出字体库中的目标符号进行展示

  所以字体库的主要目的是在于美观。同样的由于不同字体库能表示的文字符号是有限的，对于**不在库中的文字同样是无法显示出正确的符号**，而这部分符号的显示常常会用一些默认的符号（如方块）进行替换。

### 2.4. 中文编码

![img](https://pic4.zhimg.com/80/v2-0d0285e7b9433eeedf7e705d6e082d13_720w.jpg)

  常见的中文编码兼容性图如上，可见 GB18030 $\to$ GBK $\to$ GB2312 $\to$ ASCII $\leftarrow$ UTF8。UTF8同GB相关的中文字符除了ASCII有一定交集外没有任何的交集。这也是最为常见导致乱码的场景，UTF8同GBK的编解码混用。

  ASCII编码是最为常见的编码形式，但其能编码的字母和符号仅用128个，其能被大部分的编码体系兼容（除UTF-16，UTF32）

#### 2.4.1 GB18030 $\to$ GBK $\to$ GB2312

GB即国标，GBK即国标扩展，这三种编码的本质除了字节长度的不同，本质上都是前者对于后者由于汉字数目需要的扩展。

1. **GB2312**

   最先提出的兼容中文字符的编码集，其最先提出是在ASCII的基础上进行扩展，每个字占据了**2bytes**，同时输入法常见的半角全角的概念（全角即英文的字符长度同汉字等宽），而全角同半角的相同英文字符背后的编码是不同的（全角输入导致编辑器无法运行的原因）

2. **GBK**

   在GB2312的基础上进一步扩展了GB2312的6312个中文字符至20902个，同时添加了中文标点符号部首等等

3. **GB18030**

   在GBK字符基础上进一步扩展，但是由于2bytes的存储上限仅为65536，所以GB18030每个字节占据了**4bytes**。

具体的字节值域见下图，由于需要兼容ASCII所以第一个8bits的最高位置为0

![img](https://pic1.zhimg.com/80/v2-905e6e4840050444f64375ffd9dffc48_720w.jpg)

同时需要注意的是上述中文编码均采用了**定长编码的划分方式**

#### 2.4.2 UTF-8

  我对于UTF-8的初印象是来自于，对于大部分python2不兼容中文一般的解决方法通过在文件首通过设定为UTF-8进行解决。UTF-8编码的出现是为了能解决世界上的大部分文字，也是目前大部分代码过程中会采用的编码形式。其的主要目的便是将**Unicode（字符集）**设定一一映射的**编码规则**。

  但UTF-8为变长字符编码，这也是导致了其同GB系列的编码除ASCII外的所有均不兼容的主要原因。

  UTF-8采用了二进制**最高位连续1的个数**来决定该字是几字节编码（如110xxxxx代表的为双字节，同时这也意味着前n+1位是间隔判断字符），如果最高位为0则代表单字节（该举措是为了兼容ASCII编码）。如下图即为汉字字符从Unicode字符集中转换为UTF-8的过程。

![img](https://pic1.zhimg.com/80/v2-e65b436e01c4df151e496bf2acfab51c_720w.jpg)

## 3. 问题解决

### 3.1 power shell oh-my-posh

oh-my-posh类似于linux系统中的oh-my-zsh主要作用是对于命令行的美化，但对于oh-my-posh设置后往往会出现方块问题（该问题同样在WSL的oh-my-zsh中存在），问题示意如下图。

![image-20210111203210867](C:\Users\CRUN\AppData\Roaming\Typora\typora-user-images\image-20210111203210867.png)

由上述可知，出现方块的字符的主要原因是，字体库中不支持该字符编码导致的问题，具体的解决方案如下。

下载并安装https://github.com/ryanoasis/nerd-fonts内的字体并进行安装，将内部的字体设置更换为安装的字体。结果如下所示。

![image-20210111203645195](C:\Users\CRUN\AppData\Roaming\Typora\typora-user-images\image-20210111203645195.png)

解决问题的核心在于nerd-fonts提供的字体库能提供相应所需的符号。

### 3.2 Arcgis错位问题

  解决该部分的问题将近用了两天时间，下列主要是对于解决的过程和相关的方法进行总结。

#### 3.2.1 问题本质

   该问题的产生同样是因为编码问题导致，而这一问题是我利用python进行逐行写入后编码问题报错，从而意识到了所谓的错位问题本质是在于编码方式导致的直接问题。在对于逐行编码过程中，发现不管是采用**GB**系列亦或者是**UTF-8**均会产生无法编码的字符，这就表明了，原始数据中存在**不同编码方式**的数据，而Arcgis采用统一的编码形式进行编码，也就导致了该问题的发生。

​	同时需要注意的是，ArcGIS打开dBase文件时，会先对于.CPG文件进行读取，确定编码类型，如果缺失CPG文件，则Arcgis会默认将编码假设为Windows(ANSI/Multi-byte)，这也是导致dbf属性表乱码的原因之一。

#### 3.2.2 解决思路

   确定了问题是由于数据中存在不同形式的编码形式，但python的pandas读取数据后均为二进制的存储方式（python的str存储的为二进制码），所以为了解决编码不统一的问题主要采用了下列代码。

```python
s = "目标语句"
s = s.encode('utf-8',errors='ignore').decode('utf-8')
# errors ignore剔除了同utf-8无法表示的内容符号，并最后解码成utf-8的方式进行存储
```

#### 3.2.2 核心相关代码

   该部分主要介绍了csv等文件转换为shapefile的两种方式，以下代码均为核心代码。

1. **geopandas**

```python
import geopandas # geopandas的方式，具体安装流程参照https://blog.csdn.net/weixin_38333199/article/details/101761810
df= gpd.read_file(r'company_final_data.xlsx',encoding='utf-8') # 读取文件
df[['pointx','pointy']] = df[['pointx','pointy']].apply(pd.to_numeric) 
gdf = gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df.pointx, df.pointy))# 添加点位置
gdf.crs = pyproj.CRS.from_user_input('EPSG:4326') #给输出的shp增加投影
gdf.to_file('test.shp',
           driver='ESRI Shapefile',
           encoding='utf-8') # 写入shapefile文件
```

2. pyshp

```python
import shapefile # pip install pyshp
def get_shapefile(shape_path, csv_data):
    file = shapefile.Writer(shape_path, encoding='utf-8')
    #创建字段 C字符 100字符长度， N 双精度 D 日期
    colunms_list = csv_data.keys()
    for i in range(3):
        file.field(colunms_list[i],'C','100')
    file.field(colunms_list[3], 'D')
    for i in range(15):
        file.field(colunms_list[i+4], 'C', '100')
    for i in range(2):
        file.field(colunms_list[i+19], 'N', '31', decimal=4)
    # 读取csv每行文件进行逐一赋值
    for i in tqdm(range(len(csv_data['企业名称']))):
        if csv_data['lon'].values[i] == 0:
            continue
        # 添加点矢量
        file.point(csv_data['lon'].values[i], csv_data['lat'].values[i])
        # 添加属性表的记录
        file.record(str(csv_data['企业名称'].values[i]).encode('gbk',errors='ignore').decode('gbk').encode('utf-8').decode('utf-8'), 
                  #省略不细写
                   csv_data['lon'].values[i],
                   csv_data['lat'].values[i])
            
    file.close()
    # 定义投影
    proj = osr.SpatialReference() 
    proj.ImportFromEPSG(4326) # 4326-GCS_WGS_1984; 4490- GCS_China_Geodetic_Coordinate_System_2000
    wkt = proj.ExportToWkt()
    # 写入投影
    f = open(shape_path.replace(".shp", ".prj"), 'w') 
    f.write(wkt)#写入投影信息
    f.close()#关闭操作流
```

## 4. 参考资料

https://zhuanlan.zhihu.com/p/46216008

https://github.com/ryanoasis/nerd-fonts

https://blog.csdn.net/weixin_38333199/article/details/101761810



