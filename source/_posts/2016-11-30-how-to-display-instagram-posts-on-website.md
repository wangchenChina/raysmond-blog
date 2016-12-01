---
title: 如何在網站中同步顯示Instagram的照片
date: 2016-11-30 22:01:29
tags:
    - Instagram
    - 個人網站
---

![照片元數據信息](http://7b1fa0.com1.z0.glb.clouddn.com/20161201131233.png)

去年用Spring寫的博客程序（[SpringBlog](https://github.com/Raysmond/SpringBlog)），部署在阿里雲ECS上，用了一年多，到期，不想續費了。幾天前，把博客搬遷到Github，用Hexo生成靜態頁面，就是現在這個，效果不錯。主題用的是Litten寫的[Yilia](https://github.com/litten/hexo-theme-yilia)，黑白，簡潔，挺喜歡。另一個亮點是，他弄了個同步顯示Instagram照片的[相冊頁面](http://litten.me/photos/)，很酷。仔細看了代碼 ，發現是照片是寫死在頁面js裡的，圖片還緩存在了他的服務器上。不過，我喜歡折騰，所以決定自己實現一番。當然，頁面還是複用他寫的[index.ejs](https://github.com/litten/BlogBackup/blob/master/source/photos/index.ejs)，只不過需要動態把自己Instagram的照片信息通過json的方式傳進來。

## 如何獲取Instagram照片和信息？

調查一番，可以通過以下兩種方式：
- 通過Instagram的API獲取（折騰之後，發現不好用，棄）
- 爬蟲

### 通過Instagram的API獲取

這需要兩個步驟。第一步通過OAuth方式獲取access_token，然後用access_token通過API獲取照片信息（JSON）。具體如下：

<!-- more -->

1. 在Instagram的[開發者後台](https://www.instagram.com/developer/)註冊一個沙盒客戶端(sandbox client)，用於發起API請求。![Instagram的沙盒客戶端](http://7b1fa0.com1.z0.glb.clouddn.com/20161201124621.png)
2. 獲取access_token
通過瀏覽器訪問 https://api.instagram.com/oauth/authorize/?client_id=CLIENT-ID&redirect_uri=REDIRECT-URI&response_type=token ，其中CLIENT-ID和REDIRECT-URI自己填寫，REDIRECT-URI在客戶端中要事先註冊好。授權回來，就可以在瀏覽器網址欄中得到access_token。
3. 通過API獲取照片信息
API入口是 https://api.instagram.com/v1/users/{user-id}/media/recent/?client_id=CLIENT-ID&access_token=TOKEN 
獲取user-id，可以在[smashballoon.com]( https://smashballoon.com/instagram-feed/find-instagram-user-id) 通過用戶名獲取。返回的結果是JSON格式，例如：![Instagram API返回結果例子](http://7b1fa0.com1.z0.glb.clouddn.com/20161201130110.png)
結果里有很詳細的照片信息，如縮略圖大圖URL，上傳時間，標籤，點讚，評論，標題caption等。但是，問題來了。這個沙盒環境下的API只能獲取最新20張照片信息，pagination返回的永遠是空`{ }`。無法分頁獲取全部照片信息，失望，放棄。

### 第三方插件

有很方便的第三方插件可以直接在網站中顯示自己Instagram的照片，比如[Instamax](http://demos.codehandling.com/instamax_demo/instamax_live_edit.html)。問題是要錢，好像是6刀，不好自己定制開發。放棄。

### 爬蟲

Github上搜索一下， 很輕易就找到了開源爬蟲（[InstaLooter](https://github.com/althonos/InstaLooter)），可以下載任意用戶公開的Instagram照片，其原身是[InstaRaider](https://github.com/akurtovic/InstaRaider)，InstaRaider作者不再維護了，親測現在好像只有InstaLooter比較好用。

InstaLooter可以通過pip安裝，下載照片可以把作者，caption（備註）這些元數據都通過Exif寫入圖片文件。

```bash
pip install instaLooter
mkdir images
instaLooter raysmond29 ./images -v -m 
```



InstaLooter下載可以多線程，速度很快。但是，為了下載完照片和視頻之後，還不滿足。我要在網站上顯示他們，按照日期排序，展示caption，Instagram鏈接等，那我總不能通過js去提取文件的Exif信息吧。所以，我要在InstaLooter下載照片的同時，把元數據信息通過JSON的形式保存起來。InstaLooter還有這樣的接口，只能自己改源碼了。

1. 先把源代碼checkout到本地

    ```bash 
    git clone https://github.com/althonos/InstaLooter
    cd InstaLooter
    pip install .
    pip install -r requirements-metadata.txt
    ```
2. 修改`instaLooter.py`文件，我在以下兩個函數中分別把`media`信息打了出來。

    ```python
    def _download_photo(self, media):

        photo_url = media.get('display_src')
        photo_basename = os.path.basename(photo_url.split('?')[0])
        photo_name = os.path.join(self.directory, photo_basename)

        # save full-resolution photo
        self._dl(photo_url, photo_name)

        print json.dumps(media)
        print ','

        # put info from Instagram post into image metadata
        if self.use_metadata:
            self._add_metadata(photo_name, media)

    def _download_video(self, media):
        """
        """

        url = "https://www.instagram.com/p/{}/".format(media['code'])
        res = self.session.get(url)
        data = self.owner._get_shared_data(res)

        video_url = data["entry_data"]["PostPage"][0]["media"]["video_url"]
        video_basename = os.path.basename(video_url.split('?')[0])
        video_name = os.path.join(self.directory, video_basename)

        # save full-resolution photo
        self._dl(video_url, video_name)

        media['video_url'] = video_url
        print json.dumps(media)
        print ','
    ```
打出來的一條元數據信息結構如下：
    ```json
    {
        "code": "BNR6MsVAAg_", 
        "dimensions": {
            "width": 1080, 
            "height": 1080
        }, 
        "comments_disabled": false, 
        "owner": {
            "id": "673488830"
        }, 
        "comments": {
            "count": 0
        }, 
        "caption": "「井」很喜歡的一首歌 能唱上去很開心", 
        "likes": {
            "count": 1
        }, 
        "date": 1480177200, 
        "thumbnail_src": "https://scontent.cdninstagram.com/t51.2885-15/s640x640/sh0.08/e35/15101740_205434453238622_3933555745185857536_n.jpg?ig_cache_key=MTM5MjE0OTcxODc2MjUyMjY4Nw%3D%3D.2", 
        "is_video": false, 
        "id": "1392149718762522687", 
        "display_src": "https://scontent.cdninstagram.com/t51.2885-15/e35/15101740_205434453238622_3933555745185857536_n.jpg?ig_cache_key=MTM5MjE0OTcxODc2MjUyMjY4Nw%3D%3D.2"
    }
    ```
> 如果是視頻的話，我還加上了video_url。

3. 寫個腳本自動下載
    ```bash 
    #!/bin/bash 
    USER="raysmond29"
    DIR="images"
    RESULT="$DIR/result.json"
    
    mkdir $DIR
    
    echo [ > $RESULT
    python instaLooter.py $USER $DIR -m -v >> $RESULT
    echo ] >> $RESULT
    ```
    下載的`result.json`文件保存了所有照片的元數據信息。格式類似下面。這樣，我就可以把`result.json`文件上傳到一個web可以訪問的地方，個人網站通過Ajax去請求它，達到動態顯示Instagram照片的目的。每次上傳了新的照片，只需要重新更新`result.json`文件即可，不需要重新修改代碼。
    ```json 
    [
{"code": "BNR6MsVAAg_", "dimensions": {"width": 1080, "height": 1080}, "comments_disabled": false, "owner": {"id": "673488830"}, ...}
,
{"code": "BM51etvAnnf", "dimensions": {"width": 1080, "height": 1080}, "comments_disabled": false, "comments": {"count": 0}, "owner": {"id": "673488830"}, ...}
,
{"code": "BND6JxcACCM", "dimensions": {"width": 750, "height": 750}, "comments_disabled": false, "owner": {"id": "673488830"}, ...}
,
...
]
    ```
4. Instagram照片在國內訪問挺慢的，為了加速，可以把照片上傳到云存儲上，比如七牛。可以用[qrsctl](http://developer.qiniu.com/code/v6/tool/qrsctl.html)這個命令行工具上傳。
    ```bash 
    qrsctl login user_account user_password
    DIR="images"
    cd $DIR
    for file in *
    do
        if [[ -f $file ]]; then
            echo "uploading file: $file"
            qrsctl put raysmond instagram/images/$file $file
        fi
    done
    ```
    
> 這裡我所有上傳的所有照片的key都是以instagram/images/開頭的，例如http://7b1fa0.com1.z0.glb.clouddn.com/instagram/images/14498996_227601564319714_6416518897834393600_n.jpg .

## 在網站上顯示照片
通過上面的步驟，可以獲得`result.json`文件，描述了所有照片信息。在網站中，用Ajax獲取，然後排序，遍歷，通過日期聚合，組織成Litten寫的`index.ejs`中要求的object格式就行了。下面只展示部分代碼，詳細請看我的[index.ejs](https://github.com/Raysmond/raysmond-blog/blob/master/source/instagram/index.ejs)文件。

```js 
var result = [];
// 這裡我把result.json和所有照片都上傳到了七牛云存儲中，key前綴是instagram/images/
var dataFile = "http://7b1fa0.com1.z0.glb.clouddn.com/instagram/images/result.json";
var dataDir = "http://7b1fa0.com1.z0.glb.clouddn.com/instagram/images/";

function loadInstagram() {
    var xmlhttp = new XMLHttpRequest();

    xmlhttp.onreadystatechange = function() {
        if (xmlhttp.readyState == XMLHttpRequest.DONE ) {
           if (xmlhttp.status == 200) {
               showInstagram(eval('('+xmlhttp.responseText+')'));
           }
           else if (xmlhttp.status == 400) {
              alert('There was an error 400');
           }
           else {
               alert('something else other than 200 was returned');
           }
        }
    };

    xmlhttp.open("GET", dataFile, true);
    xmlhttp.send();
}

function compare(a,b) {
  return a.date > b.date ? -1 : (a.date < b.date ? 1 : 0);
}

function showInstagram(data){
  data.sort(compare);
  for(var i=0;i<data.length;i++){
    addPic(data[i]);
  }
}

// 通過日期（如2016-11）加入到result數組中
function addPic(item){
  // 在result中根據日期找到subArr，沒有的話創建，
  // ...
}


loadInstagram();
```

InstaLooter下載都是大圖，在頁面里一般先載入的都是縮略圖。通過七牛可以建立一個縮略圖樣式，輕鬆實現縮略圖功能。比如`240x240`，然後直接在圖片URL後面加上`!240x240`就可以自動居中裁剪圖片。

![縮略圖例子](http://7b1fa0.com1.z0.glb.clouddn.com/instagram/images/15046816_650775245093609_6799342947973201920_n.jpg!240x240)
[圖片鏈接](http://7b1fa0.com1.z0.glb.clouddn.com/instagram/images/15046816_650775245093609_6799342947973201920_n.jpg!240x240)


最終的結果就是 **[相冊](http://raysmond.com/instagram) **這個頁面啦。折騰了一個晚上。