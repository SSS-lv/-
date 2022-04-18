# 第五小组题目讲解
# 全球疫情形势动态地图展示
[toc]
# fetch_data函数

```livecodeserver
def fetch_data(file):
    df=pd.read_csv(file)# 用pd.read_csv读取csv文件
    #由于美国等国数据是按二级行政区划提供的，需按国家进行汇总,并且去掉了Lat,Long两列
    result=df.groupby(['Country/Region']).sum().drop(['Lat','Long'],axis=1).stack()
    # 重新定义索引
    result=result.reset_index()
    result.columns=['Country','date','value']
    result['date_']=pd.to_datetime(result.date)#生成时间索引
    result=result.sort_values(by='date_',axis=0,ascending=True)#按时间排序
    result=result.replace('\*','',regex=True)#有些国家名字中有*，去掉国名中的*
    return result
```


```gams
 ef fetch_data(file):
	 df=pd.read_csv(file)# 用pd.read_csv读取csv文件
```
    
>调入文件语句，通过参数的方法调入我们准备的文件，具体的调入操作在下面，这里我们写的是这个对文件进行预处理操作的函数，具体的处理操作见下面操作

```livecodeserver
result=df.groupby(['Country/Region']).sum().drop(['Lat','Long'],axis=1).stack()
```
>groupby:
    **df.groupby(['Country/Region'])** :通过列‘countr/region’分组，创建一个groupby对象
    **.sum()** 对分组后的数据进行求和运算
    **axis**:表示分组轴的方向，0是按行，1是按列----原文axis=1
    **.drop('lat','long')** 删除对应指定列
**.stack()** 对groupby数组进行堆叠
    结合本文：
    这句函数是对3个数据进行初步的处理，分别是对应Excel：confine，death，recover
    先将对应数据按‘countr/region’分组，创建一个groupby对象，接着进行求和，这样就是就可以得到一个国家记录的每一天的患病or死亡or康复总和
  在求和的数据上在进行删除lat，long这2列，具体调出Excel查看，再调用stack将处理好的数据进行堆叠，堆叠出来的结果就是将数据按日期分开，同一日期中所有的数据，这样处理数据方便后面进行处理
    详细解释链接: https://blog.csdn.net/weixin_44330492/article/details/100126774

```livecodeserver
result=result.reset_index()
result.columns=['Country','date','value']
```
> 这2句的代码就是通过调用reset_index()重新索引，
我们在重新赋值索引名为country date value
知识点：pandas 中的 reset_index()
**数据清洗时，会将带空值的行删除，此时DataFrame或Series类型的数据不再是连续的索引，可以使用reset_index()重置索引。**
**.columns 设置列标签**

```livecodeserver
 result['date_']=pd.to_datetime(result.date)#生成时间索引
```

>这里会再创建出一列出来数据是来自date中的数据
![请添加图片描述](https://img-blog.csdnimg.cn/57f20cf1d7e6460fa97d2ab28a6382a6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAU1NT6L-q,size_20,color_FFFFFF,t_70,g_se,x_16)

生成时间索引，原本中的数据不是时间格式的数据，是一种文本数据，我们将这些数据转化为时间类型是数据，
    就是本来只是一串文字，我们把改成为时间类型的数据，他就具有时间的属性了
pandas中的sort_values()函数原理类似于SQL中的order by，
        可以将数据集依照某个字段中的数据进行排序，该函数即可根据指定列数据也可根据指定行的数据排序。
        本题通过时间排序
        **.sort_values(by='',axis=0,ascending=True)**
            by:指定的列名
            **axis**：若axis=0或’index’，则按照指定列中数据大小排序；
            若axis=1或’columns’，则按照指定索引中数据大小排序，默认axis=0
            **ascending**：是否按指定列的数组升序排列，**默认为True，即升序排列** 	
```python
	result=result.sort_values(by='date_',axis=0,ascending=True)#按时间排序
```
    
  > **sort_values** ：这个方法啊，他和sql中的order by 命令类似哈，他是用来排序用的，那他排序中通过by：“名字”,这个参数来判断这个数据是通过什么来排序的，本文是通过时间这个索引来排序的，那是按时间是什么属性来排序的呢？这里是安装时间的数据大小来排序的，这里也可以看出我们前一句通过to_datetime来将数据改成为时间类型的数据的意义，没有定义类型我们怎么比较大小

```livecodeserver
result=result.replace('\*','',regex=True)#有些国家名字中有*，去掉国名中的*
```
> replace：替换函数。result.replace('\*','',regex=True)‘*’用‘’（空替换）
    意义这里的意思就是去掉国家后面带* 的避免 *号带来的影响。               
# draw_data函数

```python
# choropleth绘制地图
# 绘制动态图表
# 'Country': df中的国家列的列名，'date': df中的日期列的列名，'value': df中的人数值的列名
def draw_data(df, label, color):
    print(df)
    fig = px.choropleth(df,
                        locations='Country',  # 选择城市为坐标
                        # 按照国家名字绘制各个国家的轮廓，'country names'是locationmode参数所提供的选项，不能随意更改，
                        # 选项为['ISO-3', 'USA-states', 'country names', 'geojson-id']
                        locationmode='country name',
                        # 动画帧
                        animation_frame='date',  # 以时间为轴
                        # 地图以及右侧刻度的颜色设置的标准
                        color='value',  # 根据人数进行颜色的设置
                        # 颜色连续刻度
                        # 0表示最小值，1表示最大值，最小值为白色，最大值为传入的颜色设置，形成一个渐变颜色
                        color_continuous_scale=[[0, 'White'], [1, color]],  # 按提供的颜色作为最大值的颜色color
                        # 右侧刻度标签，按照传出的label设置
                        labels={'value': label},  # 按提供的label绘制图例
                        # 颜色范围
                        # 按照value的最大值、最小值绘制
                        range_color=[df.value.min(), df.value.max()]) 
                        # 按全程数据的最大值、最小值进行绘制，不采用autoscale
    
    fig.show()# 将绘制的地图进行显示
```
# resample函数解释
```python
def resample(df, period):
    #，函数均为默认参数，意思为将表格中的国家转化为列表，并在所有列中查找，去除重复的数据，不在原有数据上修改，创建新的变量
    country_list = df.Country.drop_duplicates()  # 计算城市,得到国家的列表
    #讲df复制并传给temp
    temp = df.copy()
    #创建Datafrme数据，Dataframe数据类似excel数据表格，是一种二维表，可以设置列名和行名，存储的数据可以是数值字符串等。
    #此处为创建一个空的Dataframe,并用result接收
    # 此处选择默认选择不设置idex和columns，即没有给定的列名和行名
    result = pd.DataFrame()
    #利用for循环，在每一个国家中进行相同的如下操作
    for i in country_list:  # 按国家分别进行重抽样，并合并数据
        # 按照国家标签进行选取，选取的数据为国家所对应的那行数据
        #其中的loc参数为按照标签选择数据，
        # 如：标签为canada，即为temp.Country == canada  那么选取的行就是canada所对应的数据
        r_temp = temp.loc[temp.Country == i]  # 选择对应的城市的行
        # 去掉数据中相同的元素，减少运算量
        r_temp = r_temp.drop_duplicates(['date_'])  # 对数据去重
      
	  """  
	   将数据中的列转化为行索引，如：
        a    b    c
    0   1   ....
    1   4   ....
    2   7   ....
    3   9   ....
    进行行索引转化set_index('a')后为：
    a   b   c
    1   ....
    4   ....
    7   ....
    9   ....
        """
		
        # 在此处就是将日期设置为索引，代替原来的0,1,2,3....
        r_temp = r_temp.set_index('date_')
        #重采样（Resampling）指的是把时间序列的频度变为另一个频度的过程。把高频度的数据变为低频度叫做降采样（downsampling），
        # 把低频度变为高频度叫做升采样
        # 其中的日期period表示的为时间区间，可以选择的参数有日D，周W，月M，季度S，年Y等，此处选择的是周W
        r_temp = r_temp.resample(period).asfreq().dropna()  # 重采样并且删除缺失值
        #将原来的index设置为列，与set_index（）方法的功能相反。
        # 在本处的实际意义是将日期所代替的索引变回原来的0,1,2,3,4....,方便后续的操作
        r_temp = r_temp.reset_index()
        #将进行上述操作的数据与空的Dataframe进行连接组成新的Dataframe表格
        result = pd.concat([result, r_temp])
        #将数据按照日期排序，并且正序排序，返回排序后的结果，其中的axis为指定列名，此处为默认选项，ascending为升序或者降序，默认为True升序，by是指定列
    return result.sort_values(by='date_', axis=0, ascending=True)
```

