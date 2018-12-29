---
title: 豆瓣API
date: 2018-09-02 10:10:56
tags: 数据
id: 1535854293
---
# 图书
## 搜索图书
GET  https://api.douban.com/v2/book/search

参数 | 意义 | 备注
-|-|-
q | 查询关键字 | q和tag必传其一
tag | 查询的tag | q和tag必传其一
start | 取结果的offset | 默认为0
count | 取结果的条数 | 默认为20，最大为100

返回：返回status=200

```json
{
      "start": 0,
      "count": 10,
      "total": 30,
      "books" : [Book, ]
    }
注：对于登录用户，若搜索结果图书在当前用户的图书收藏中，会在对应搜索结果信息中附加当前用户对此书的收藏信息，改部分的 Book 数据结构如下：

{
    … (图书信息的其他部分)
    "current_user_collection": {
        "status":"read",
        "rating": {
            "max":5,
            "value":"5",
            "min":0
        },
        "updated":"2012-11-2012:08:04",
        "user_id":"33388491",
        "book_id":"6548683",
        "id":605519800
    }
}
```
## 根据 id 获取图书信息
GET  https://api.douban.com/v2/book/:id

返回图书信息，返回status=200

对于授权用户，返回数据中会带有该用户对该图书的收藏信息：
```json
{
    … (图书信息的其他部分)
    "current_user_collection": {
        "status":"read",
        "rating": {
            "max":5,
            "value":"5",
            "min":0
        },
        "updated":"2012-11-2012:08:04",
        "user_id":"33388491",
        "book_id":"6548683",
        "id":605519800
    }
}
```

## 通过 isbn 获取图书信息
GET  https://api.douban.com/v2/book/:isbn

# 电影
## 搜索电影
GET https://api.douban.com/v2/movie/search?q=张艺谋

参数 | 描述 | 备注
-|-|-
q | 关键词 | q和tag必传其一
tag | tag | q和tag必传其一
start | start | 0
count | count | -

## 通过 id 获取电影
GET https://api.douban.com/v2/movie/25849049

## 通过 imdb 获取电影
GET https://api.douban.com/v2/movie/imdb/:imdb

# 音乐
## 搜索音乐
GET  https://api.douban.com/v2/music/search

参数 | 描述 | 备注
-|-|-
q | 查询关键字 | q和tag必传其一
tag | 查询的tag | q和tag必传其一
start | 取结果的offset | 默认为0
count | 取结果的条数 | -

返回：返回status=200，
```json
{
    "start": 0,
    "count": 10,
    "total": 30,
    "musics" : [Music, ]
}
```

## 获取音乐信息
GET  https://api.douban.com/v2/music/:id

返回音乐信息，返回status=200

## 小组
0. 请求的前缀是： https://api.douban.com/v2 
1. 获取小组基本信息：/group/:id 
如：https://api.douban.com/v2/group/husttgeek/ 
2. 获取话题列表： /group/:id/topics 
如：https://api.douban.com/v2/group/husttgeek/topics 
3. 新发话题估计是POST到上面那个地址，没测试 
4. 获取某话题评论列表： /group/topic/:id/comments 
如：https://api.douban.com/v2/group/topic/33488193/comments 
5. 发表评论估计是POST到上面那个地址，没测试 

------------------------------
参考：  
豆瓣官方客户端：https://github.com/douban/douban-client