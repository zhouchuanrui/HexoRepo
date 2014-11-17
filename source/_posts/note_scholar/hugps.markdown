title: GPS的正确用法
Status: Public
Tags: GPS 
date: 2013-9-23 15:53:19
---

[TOC]

# GPS模块与NMEA-1800协议

GPS模块的基本输入输出构成是：

- 天线，用来接收GPS卫星信号：
- PPS输出；
- 串口模块，用来配置模块的工作模式以及NMEA-1800码串口输出格式；

<!--more-->

NMEA-1800协议包含丰富的字段，具体有1.0版本支持的字段有：

字段名|内容
---|---
AAM|Waypoint Arrival Alarm 
ALM|Almanac data 
APA|Auto Pilot A sentence 
APB|Auto Pilot B sentence 
BOD|Bearing Origin to Destination 
BWC|Bearing using Great Circle route 
DTM|Datum being used. 
GGA|Fix information 
GLL|Lat/Lon data 
GRS|GPS Range Residuals 
GSA|Overall Satellite data 
GST|GPS Pseudorange Noise Statistics 
GSV|Detailed Satellite data 
MSK|send control for a beacon receiver 
MSS|Beacon receiver status information. 
RMA|recommended Loran data 
RMB|recommended navigation data for gps 
RMC|recommended minimum data for gps 
RTE|route message 
TRF|Transit Fix Data 
STN|Multiple Data ID 
VBW|dual Ground / Water Spped 
VTG|Vector track an Speed over the Ground 
WCV|Waypoint closure velocity (Velocity Made Good) 
WPL|Waypoint Location information 
XTC|cross track error 
XTE|measured cross track error 
ZDA|Date and Time

里面可以获取时间信息、卫星信号信息、经纬度地理位置信息、卫星速度信息等，协议直接有冗余部分，用哪个字段要看具体的应用场合。

# 只用ZDA是不对的

对时间同步的应用场合来讲，时间码是最重要的信息，因此ZDA字段是一定要的，ZDA字段的内容为：

	$GPZDA,hhmmss.ss,dd,mm,yyyy,xx,yy*CC
内容非常简单，但是是NMEA-1800里面唯一包含了年份编码的字段。

看起来用ZDA字段就已经满足了时间同步系统的时间源需求了。但是实际情况没这么简单。

之前把一个GPS模块设为只有ZDA格式输出，不接天线的时候输出是：

	$GPZDA,,,,,,*CC
这个很好理解，没有接到GPS信号的嘛，然后接了天线，ZDA时间码很快就出来了。当时以为这个时候时间就已经准了，但是用仪器一对模块的PPS输出，发现偏的很大，跟之前没接天线的时候差不多，然后仔细一看时间码，跟标准时间还差着10多秒呢。

后来，问了一个业内人士，才知道了问题所在。他说的是，ZDA时间出来的时候，GPS模块不一定就已经锁定了卫星信号了，因此还不一定是准确的时间。

按这个来说，ZDA字段是不能单独使用的，因为这个字段里面没有GPS模块锁星的信息。其实说起来，ZDA可能是NMEA-1800里面最不重要的字段了，没有最核心的信息，只是比其他字段的UTC时间码多了年份码。

因此，正确的使用方式是，**要把ZDA跟其他字段配合起来使用，比如GGA、RMC或者GSA，要在这些字段里面读到卫星锁定了，才能去用ZDA里面的时间信息，这样才能保证时间码和PPS信号的正确**。像是那些时间仪器里面一般都显示了GPS卫星数量，这些仪器里面肯定也是用了ZDA之外的字段的。

