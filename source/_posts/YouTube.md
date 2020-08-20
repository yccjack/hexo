---
title: 如何在YouTube Api限额的情况下获取更多视频
categories:  java
tags: [java] 
description: 如何在YouTube Api限额的情况下获取更多视频
mathjax: true
sidebar: [grid, toc, tags] # 放置任何你想要显示的侧边栏部件
---

# YouTube视频



谷歌限制了YouTube api v3的请求量，一天10000配额，这里不是10000次请求，每次请求根据不同参数消耗不同配额。为了摆脱这种限制而获得更多的新发布视频，做了以下内容的方案。



## 需求：

运营配置YouTube的channelId，后台需要根据这些channelId去获取最近发布的可以在小屏播放的video信息，以增加用户活度。

## 问题：

YouTube限额问题，谷歌限制域名只能使用一个ApiKey，配置多会被封禁，按照现有全部用api检索会导致频道越配越多，获得的视频越来越少。



## 解决：



### 思路1：

出于问题中关键点，系统不知道channel下面发布的情况，只能被动查询，这样可能会导致查询消耗了配置结果返回为空或者很少视频的情况；所以考虑使用订阅模式去事先得知频道的情况。

查找了很多资料；最坑的竟然是YouTube api官网给的方法。。。。([youtubeApi](https://developers.google.com/youtube/v3/getting-started))。我试着去使用它介绍的发布订阅，对于Google的集线器我研究了很久，毕竟不熟悉，而且没有相关的java实现。

### 方式1：

1.启动自己的回调服务器，随便弄个可以外网访问的服务返回200和请求参数中的hub_chanlenge即可。

2.订阅你需要订阅的频道的atom：类似：https://www.youtube.com/xml/feeds/videos.xml?channel_id=CHANNEL_ID 这种。

3.返回204即成功。



### 我的尝试：

我使用的自己的云服务器，使用谷歌的集线器，然后去订阅YouTube，发现509等错误，莫名其妙后使用了自己写的atom作为发布方，结果成功了。不过，可笑的是，这个集线器它并不能正常工作，我在修改atom再次发布的时候，它竟然没能好好工作；没向我的回调函数发送信息。我崩溃了，我去谷歌搜索了很多相关问题，发现YouTube已经不将视频信息发布到上面所说的xml中了，而且在这之前YouTube为了用户体验，每个频道只发送3条消息给订阅用户(YouTube自带的那个铃铛订阅)我去你.....



### 方式2：

再对问题思考，依然摆脱不了需要提前得知频道下视频的发布情况，我试着去YouTube网站videos下查看视频与api返回的视频做对照，发现可以使用解析http的标签获取发布的视频和时间(其实一开始也想过使用爬虫，奈何怕蹲牢啊。。)。我试着使用httpClient解析这个页面，果然得到了我想要的答案。

这样我就可以提前知道频道的发布情况，进而对使用api检索得到的结果有了大的优化。相关代码如下：



YouTubeTest

```java
public class YoutubeTest {
  
    private static String matching = "</li><li>";
    private final static String CONTENT = "class=\"yt-lockup-content\"";

    private static final String GET_VEDIO_INFO_PRE = "https://youtube.com/get_video_info?video_id=";

    public static void main(String[] args) throws Exception {
        http("UC24_Z2L-8Ki183AI9zJJzNQ");
    }




    private static void http(String channelId) throws Exception {
        String url = "https://www.youtube.com/channel/" + channelId + "/videos";
        CloseableHttpClient httpclient = HttpClients.createDefault();
        HttpGet httpget = new HttpGet(url);
        CloseableHttpResponse response = httpclient.execute(httpget);
        try {
            HttpEntity entity = response.getEntity();
            String responseYoutube = EntityUtils.toString(entity);
            List<String> countList = new ArrayList<>(100);
            int length = responseYoutube.length();
            int i1, i2 = 0;
            long startTime = System.currentTimeMillis();
            for (int i = 0; i < length; i++) {
                i1 = responseYoutube.indexOf(CONTENT, i);
                if (i1 > 0) {
                    i2 = responseYoutube.indexOf("</div>", i1);
                    if (i2 > 0) {
                        countList.add(responseYoutube.substring(i1, i2));
                        i = i2;
                    }
                } else {
                    break;
                }
            }
            long endEachTime = System.currentTimeMillis();
            System.out.println("遍历耗时：" + (endEachTime - startTime) + "ms");
            List<VideoInfo> videoInfos = new ArrayList<>(30);
            countList.forEach((s) -> {
                int hrefStart = s.indexOf("v=");
                int hrefEnd = s.indexOf("\"", hrefStart);
                VideoInfo videoInfo = new VideoInfo();
                int lastIndex = s.indexOf("</li></ul>");
                if (lastIndex > 0) {
                    String substring = s.substring(s.indexOf(matching) + matching.length(), s.indexOf("</li></ul>"));
                    int time = analysisTime(substring);
                    if (time == -2) {
                        System.out.println(channelId + "返回参数中有解析错误的html标签:" + s);
                    }
                    videoInfo.setPublishTime(time);
                    videoInfo.setVideoId(s.substring(hrefStart + 2, hrefEnd));
                    System.out.println(substring);
                    videoInfos.add(videoInfo);
                }
            });

            videoInfos.forEach(System.out::println);
            System.out.println("打印耗时：" + (System.currentTimeMillis() - endEachTime) + "ms");
            System.out.println(countList.size());
        } finally {
            response.close();
        }

    }

    private final static String CH_SECONDS_PRE = "秒前";
    private final static String CH_MINUTES_PRE = "分鐘前";
    private final static String CH_HOURS_PRE = "小時前";
    private final static String CH_DAYS_PRE = "天前";

    private static int analysisTime(String substring) {
        boolean matches = substring.substring(0, 2).trim().matches("^[0-9]*[1-9][0-9]*$");
        int time=-2;
        if(matches){
            time = Integer.parseInt(substring.substring(0, 2).trim());
            if (substring.contains(CH_SECONDS_PRE)) {
                time = time + 0;
            } else if (substring.contains(CH_MINUTES_PRE)) {
                time = time + 100;
            } else if (substring.contains(CH_HOURS_PRE)) {
                time = time + 200;
            } else if (substring.contains(CH_DAYS_PRE)) {
                time = time + 300;
            } else {
                time = -1;
            }
        }

        return time;
    }
```

VideoInfo

```java
@Setter
@Getter
@ToString
public class VideoInfo {
    private String videoId;
    private int publishTime;

}
```



这里使用的是香港，所以这里匹配获取时间的时候使用了繁体，解释下这里面的匹配规则。

<font color=red>class="yt-lockup-content"</font>是返回的html中视频主题标签的class，从此开始一个个获取。

<font color=red>analysisTime</font> 秒则直接使用，分钟则为100起，以此类推。

其实在F12调试的时候，这个URL请求获得的是一段json，不知道为什么变成了html，对这方面不是很熟悉，之后会想办法去优化这块。

<font color=red>GET_VEDIO_INFO_PRE</font>这个地址是YouTube的公共API，目前还是可以使用的，可以检索一些视频的信息。



