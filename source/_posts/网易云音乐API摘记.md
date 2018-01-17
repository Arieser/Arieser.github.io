---
title: 网易云音乐API摘记
date: 2016-12-05 13:28:19
tags: 
---

简要记录一下网易云音乐API，因云音乐服务并没有开放接口，部分api可能在一段时间后失效。

<!-- more -->

1.  音乐搜索 √
    - API: `http://s.music.163.com/search/get/?src=lofter&type=1&filterDj=true&s=缘&limit=8&offset=0&callback=loft.w.g.cbFuncSearchMusic`
    - Method: GET
    - 参数：
        - s: keyword
        - limit: 数目限制
        - offset: 偏移量
        - src: lofter //可为空
        - type: 1 (歌曲1 专辑10 歌手100 歌单1000 用户1002 mv1004 歌词1006 主播电台1009: 除1外其他存疑)
        - filterDj: true|false //可为空
        - callback:  //为空时返回json，反之返回jsonp callback

2.  外链接口 √
    - API: `http://music.163.com/api/song/detail/?id=歌曲Id&ids=%5B歌曲Id%5D&csrf_token=`
    - Method: GET
    - 例子：
        - 歌曲：`http://music.163.com/#/song?id=30780524`
        - 用法：`http://music.163.com/api/song/detail/?id=30780524&ids=%5B30780524%5D&csrf_token=`

3.  歌曲信息 √
    - API: `http://music.163.com/api/song/detail/`
    - Method: GET
    - 参数：
        - id: 歌曲id
        - ids: 用[]括起来的歌曲ID
    - 用法：`http://music.163.com/api/song/detail/?id=28377211&ids=%5B28377211%5D`

4.  歌曲专辑
    - API: `http://music.163.com/api/artist/albums/歌手ID`
    - Method: GET
    - 参数：
        - limit: 获取数量
    - 用法：`http://music.163.com/api/artist/albums/166009?id=166009&offset=0&total=true&limit=5`

5.  专辑信息
    - API: `http://music.163.com/api/album/专辑ID`
    - Method: GET
    - 参数：
        - limit: 获取数量
    - 用法：`http://music.163.com/api/album/2457012?ext=true&id=2457012&offset=0&total=true&limit=10`

6.  歌单 √
    - API: `http://music.163.com/api/playlist/detail`
    - Method: GET
    - 参数：
        - id: 歌单id
    - 用法：`http://music.163.com/api/playlist/detail?id=37880978&updateTime=-1`

7.  歌词 √
    - API: `http://music.163.com/api/song/lyric`
    - Method: GET
    - 参数：
        - id: 歌曲id
        - lv: 值为-1，我猜测应该是判断是否搜索lyric格式
        - kv: 值为-1，这个值貌似并不影响结果，意义不明
        - tv: 值为-1，是否搜索tlyric格式
    - 用法：`http://music.163.com/api/song/lyric?os=pc&id=93920&lv=-1&kv=-1&tv=-1`

8.  MV
    - API: `http://music.163.com/api/mv/detail`
    - Method: GET
    - 参数：
        - id: 歌曲id
        - type: 值为mp4，视频格式
    - 用法：`http://music.163.com/api/mv/detail?id=319104&type=mp4`



Reference:
[网易云音乐常用API浅析](http://moonlib.com/606.html)
[应用实例之音乐搜索](http://blog.csdn.net/zhaoyazhi2129/article/details/9194483)
[网易音乐搜索API](http://mrasong.com/a/163-music-api)
[基于微信公众平台的Python开发——（网易云）音乐搜索](http://blog.csdn.net/u011439951/article/details/46433225)