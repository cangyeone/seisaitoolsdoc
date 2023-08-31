CREDIT-X1local数据集
======================

介绍
----

高质量的数据集对于开发地震学领域的先进机器学习算法至关重要。在这里，我们展示了根据 ChinArray 一期(X1) 的记录构建的地震数据集，该数据集于 2011 年至 2013 年期间部署在南北地震带南部，拥有 355 个便携式宽带地震台站。作为 ChinArray 创新技术参考地震数据集（CREDIT）的首次发布，CREDIT-X1local 整理了南北地震带南部（20°-32°N，95°-110°E）发生的 105,455 个地震事件的综合信息。原始 100 Hz 采样的三个分量波形最大震中距离达 1,000 公里，每个波形包含至少 200 秒的记录。提供两种震相标签。第一个包括手动挑选的 5,999 个震级大于 ML2.0 的事件的标签，提供 66,507 个 Pg、42,310 Sg、12,823 Pn 和 546 Sn 相。第二个包含 105,442 个事件的自动标记震相，震级范围为 -1.6 到 7.6。使用 RNN 自动挑选的震相，并使用相应的走时曲线进行筛选，得到 1,179,808 Pg、884,281 Sg、176,089 Pn 和 22,986 Sn 相。此外，初动极性也附加到 31,273 个 Pg 中。数据集提供事件和站点位置，以便可以训练和验证用于传统震相选取和震相关联的深度学习网络。 CREDIT-X1local数据集是首个由密集地震台阵构建的百万级数据集，可作为各种多站深度学习方法以及高精度震源机制反演和地震层析成像研究的基础。此外，受益于中国南北地震带南部的高地震活动性，CREDIT-X1local数据集在未来的科学发现方面具有巨大的潜力。


下载地址
-------

待定

数据使用
--------

.. code:: python
   import h5py 
   import matplotlib.pyplot as plt 
   import datetime 
   import numpy as np 
   file_name = "path/to/creditx1local"
   h5file = h5py.File(file_name, "r") 

   for ekey in h5file:
      event = h5file[ekey]
      print(f"event:{ekey}")
      for key in event.attrs:
         if "strs" in key:continue 
         print(f"|-{key}:{event.attrs[key]}")
      for skey in event:
         station = event[skey]
         print(f"|-station:{skey}")
         for key2 in station:
               wave = station[key2]
               if "strs" in key:continue 
               print(f"  |-Data:{key2}:{wave[:].shape}")
               for key3 in wave.attrs:
                  print(f"    |-{key3}:{wave.attrs[key3]}")
         for key in station.attrs:
               if "strs" in key:continue 
               print(f"  |-{key}:{station.attrs[key]}")
      break 
   #绘图
   for ekey in ["YN.201103130305.0001"]:
      event = h5file[ekey]
      count = 0 
      for skey in event:
         station = event[skey]
         for key2 in station:
               wave = station[key2]
               for key3 in wave.attrs:
                  if "btime" in key3:
                     btime = datetime.datetime.strptime(wave.attrs[key3], "%Y/%m/%d %H:%M:%S.%f")
         w = wave[:] 
         w = w.astype(np.float32)
         w -= np.mean(w) 
         w /= np.max(np.abs(w)) 
         plt.plot(w*0.4+count, c="k", alpha=0.5, lw=1)
         for key in station.attrs:
               if "strs" in key:continue 
               if key.endswith("Pg") or key.endswith("Sg"):
                  #if "/" not in wave.attrs[key3]:continue 
                  ptime = datetime.datetime.strptime(station.attrs[key], "%Y/%m/%d %H:%M:%S.%f")
                  delta = (ptime-btime).total_seconds() * 100 
                  if "Pg" in key:
                     c = "r"
                  else:
                     c = "b"
                  plt.plot([delta, delta], [count-0.5, count+0.5], c=c)
         count += 1
      #plt.xlim((2500, 5000))
      plt.savefig("odata/demo.png")

波形数据库使用（mseedindex）
=============

介绍
----

用于构建MSEED文件索引，可以方便快速的根据台站名、时间等截取任意长度的波形数据。
用于将mseed数据和地震目录整合成独立的h5格式数据，方便后续分析处理工作.

软件架构
--------

软件整合了obspy中的Clint功能和震相数据读取分析功能。软件功能为： -
makeidex.py 产生索引 - makeh5.py 制作数据 - testh5.py 测试数据 - base.py
基础库 - utils 工具文件夹 - 修改来自于obspy - 原始版本存在死锁

安装教程
--------

依赖库包括：obspy,h5py,tqdm 请编译安装sqlite ## 使用说明

创建索引
~~~~~~~~

1. 使用命令’makeindex.py -r /path/to/data -o index.db’ 创建索引
2. 程序会自动搜索目录下文件
3. 如果索引文件过大，建议分开存储。比如按年分隔，否则截取时速度较慢
4. 如果分开存储数据库数据无法截取。 ### 制作数据
5. 使用命令’makeh5.py -i index.db -o out.h5 -c /path/to/ctlg -s
   /path/to/location’
6. 震相位置需要修改
7. 多线程程序需要自行修改 ### 测试数据
8. 使用命令’testh5.py -i out.h5 -o stats.txt -c /path/to/ctlg’
9. 用于测试h5数据完整性

通过数据库读取mseed数据
-----------------------

.. code:: python

   from obspy import UTCDateTime
   import datetime  
   from utils.dbdata import Client 
   clint = Client("path/to/index")
   time1 = etime + datetime.timedelta(seconds=-10)# 截取开始时间-10秒
   time2 = etime + datetime.timedelta(seconds=20)# 截取结束时间+20秒
   t1 = UTCDateTime(time1.strftime("%Y/%m/%dT%H:%M:%S.%f"))  #转换为obspy时间
   t2 = UTCDateTime(time2.strftime("%Y/%m/%dT%H:%M:%S.%f"))  
   cha = "?HZ"# 获取分量中有HZ的
   net = "X1" # 台网名
   sta = "00111" # 台站名
   loc = "00" # 位置名
   st = clint.get_waveforms(net, sta, loc, cha, t1, t2)# 获取波形，obspy.Stream

参与贡献
--------

如是：cangye@Hotmail.com
