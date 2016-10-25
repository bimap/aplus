# iPOC30 常見問題集FAQ

## Q：如何增加一台主機到iPOC 中

**說明：** 當log 收集後， 如何增加到iPOC30 的處理流程中

**步驟：** 在/diskZ/log/uid 中，有幾個hostset.xx 的文件， 將收集到的主機log 依據log 格式增加到 其中即可

範例： uid 為acbellog， 收集文件PNT24 為ms 格式， 

```
$ cd /diskZ/log/acbellog
$ cat hostset.ms
PNT13
PNT52
PNT74
PNT16
PNT83
PNT19
PNT80
PNT36
PNT123
PNT124

$ vi hostset.ms 增加PNT24 到文件中即可
```

## Q：如何手動處理已經收集到的log 格式， 而不是自動執行

說明： 利用/opt/iPOC30/crontask/v30-acbel-one.sh 來執行， 執行方式為

#### step1. 修改v30-acbel-one.sh


```
＄ vi v30-acbel-one.sh

[polin@ipoc crontask]$ cat v30-acbel-one.sh
#!/bin/bash
ftpuid="acbellog"
uid="acbellog"
cfgDir="/opt/iPOC30/config"
today=`date +%Y%m%d`
Yesterday=`date -d "yesterday" +%Y%m%d`
DAY7=`date -d "7 days ago" +%Y%m%d`
month=`date -d "yesterday" +%Y%m`

if [ ! -f $cfgDir/setup.ini ] ; then
        echo "no $cfgDir/setup.ini, Please check again"
        exit
fi
for CODE in $(cat $cfgDir/setup.ini); do
        NAME=`echo $CODE | sed 's/=/ /g' | awk '{print $1}'`
        TYPE=`echo $CODE | sed 's/=/ /g' | awk '{print $2}'`
        VARS=`printf "${NAME}=\\${TYPE}\n"`
        eval `printf $VARS`
done

begindate=$Yesterday
  enddate=$Yesterday

if [ $# -eq 1 ] ; then
        datex=$1
        begindate=$datex
        enddate=$datex
fi
if [ $# -eq 2 ] ; then
        begindate=$1
        enddate=$2
fi

echo $0,$begindate, $enddate

while [ $begindate -le $enddate ] ; do
        datex=$begindate
        for host in $(cat $logDir/$uid/hostset.nmon)
#        for host in "ServerA"
        do
                $binDir/NMON2big.sh $uid $host $datex $cfgDir
                $binDir/awr2link.sh $uid $host $datex $cfgDir
                $binDir/awr_phraseDay.sh $uid $host $datex $cfgDir
a=1
        done

        for host in $(cat $logDir/$uid/hostset.ms)
#        for host in "ServerB"
        do
                $binDir/csv2big.sh $uid $host $datex $cfgDir
                $binDir/awr2link.sh $uid $host $datex $cfgDir
                $binDir/awr_phraseDay.sh $uid $host $datex $cfgDir
a=1
        done

        tempdate=`date -d "$begindate + 1 days" +%Y%m%d`
        begindate=$tempdate
done

        echo $datex > $bigDir/$uid/finalday
exit

```
#### step2. 重新執行所有server 的log 處理

```

$ cd /opt/iPOC30/crontask
$ ./v30-acbel-one.sh 20160901 20161024

```

其中20160901 為開始日期， 20161024 為結束日期， 可以查詢 /diskZ/log/uid/host/month 中的log， 確認一下log 的開始日期


#### step3. 執行單台的內容, 以單台PNT24 為例， 修改如下

```

#        for host in $(cat $logDir/$uid/hostset.nmon)
        for host in "ServerA"
        do
#                $binDir/NMON2big.sh $uid $host $datex $cfgDir
#                $binDir/awr2link.sh $uid $host $datex $cfgDir
#                $binDir/awr_phraseDay.sh $uid $host $datex $cfgDir
a=1
        done

#        for host in $(cat $logDir/$uid/hostset.ms)
        for host in "PNT24"
        do
                $binDir/csv2big.sh $uid $host $datex $cfgDir
                $binDir/awr2link.sh $uid $host $datex $cfgDir
                $binDir/awr_phraseDay.sh $uid $host $datex $cfgDir
a=1
        done
```

然後執行以下的程式即可， 使用一般用戶，記得使用sudo

```
$ sudo /opt/iPOC30/crontask/v30-acbel-one.sh 20160920 20161024 
$ ./v30-acbel-one.sh 20160920 20161024 
```

