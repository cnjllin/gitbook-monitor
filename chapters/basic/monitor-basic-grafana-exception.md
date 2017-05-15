# Grafana 异常


## 没有数据也没有报错

检查Group By time interval是否设置了值,如果group by time interval的值小于数据的收集周期,数据会显示不出来。

解决: 设置Group By time interval,大于数据的抓取周期,比如">10s"


## Invalid dimensions for plot, width = 1867.625, height = 0

在编辑图表时,不小心调整了图表了大小,返回Dashboard后提示这个异常。

看了一下正常图表中"General"标签中的Dimensions的Span是5,我误操作图表的这个是是12;直接修改为5还是不行,后来将图表的大小调整了一下就可以了。


