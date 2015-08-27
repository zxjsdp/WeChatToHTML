WeChatToHTML
============
Extract `text`, `image`, `audio`, and `video` info from WeChat SQLite db
and user content folder, generate **WeChat app style HTML5 files**.


Why there is no source code here?
---------------------------------
Since WeChat is a commercial software, I am not yet sure if there will be
any **legal risk** if I release the code which uses it's user data.

So I decided to remove all the source code. I will consider releasing the
code in the future. ^_^


Demo
----
#### Overview of my implementation
![overview](./data/demo_imgs/overall.png)

#### Some details
![img_and_gif](./data/demo_imgs/img_and_gif.png)

- You can see that `text message`, `emoji`, and `images` can be displayed
  normally.

![audio](./data/demo_imgs/audio.png)

- `Audios` can be played with HTML5 audio player provided by browsers
  (Firefox, Chrome, etc.)
- The background of `date infomation` is white bar.
- Text of `time information` uses light grey color.

![inline_emoji](./data/demo_imgs/inline_emoji.png)

- `Emoji` can be displayed **inline with other text**.

![enlarge](./data/demo_imgs/enlarge.png)

- If you click `text message`, you can see the enlarged version of it,
  along with `datetime information`.
- If you click user `avatar`, you can see enlarged version of avatar.
- If you click an `image`, you can see enlarged version of image.
  You can **zoom in** or **zoom out** as you wish.


Implementation
--------------

First, get WeChat `SQLite db file` and `user content folder` from mobile phone.

For Android, you can get your `EnMicroMsg.db` from this path:

     /data/data/com.tencent.mm/MicroMsg/de42c....8ed(32 chars length)/

This is an **encrypted SQLite db file**. You need to get the `IMEI` of your mobile
phone and your WeChat `UIN` (**password=md5(IMEI+UIN)[:7]**).

Then, execute:

    echo -n "$imei$uin" | md5sum | cut -c -7

Say the password of your `EnMicroMsg.db` is abc1234. Now you need to decipher
your db file. You can use sqlcipher2 (not 3 or newer).

After deciphering, you can now access your text message, user list, datetime
informations, media path, ... from db file. All the informations we need are known.

Now you can get your media files, attachment files, and other files in this
path:

    sdcard/Tencent/MicroMsg/de42c....8ed(32 chars length)/

Normally we will need these folders: `emoji`, `image`, `image2`, `video`,
`voice`...

Then we can start design our webpage.

Using HTML5 is much more convenient since
we can play some audio and video types directly in web browser. Since the audio
format `.amr` in `voice` folder is not supported by most web browsers, we can
convert them to `.mp3` format using ffmpeg. You can do similar things for
other media type.

    ffmpeg -i ./file.amr ./file.mp3

We need to parse contents from SQLite db file.

    msgId
    msgSvrId
    type
    status
    isSend
    isShowTimer
    createTime
    talker
    content
    imgPath
    reserved
    lvbuffer
    transContent
    ...

Now we need to concentrate on type.

        1        Normal text message
        3        Photos

                    # How the photo folder organize:
                    if image_path.startswith("THUMBNAIL_DIRPATH"):
                        image_name = image_path.split('//')[1]\
                            .replace('th_', '').strip()
                        image_path = os.path.join(
                            'image2',
                            '%s/%s/%s' % (image_name[:2],
                                          image_name[2:4], image_name))

                    ...

        34       Audios (.amr)
        47       Emoji
        43       Image (jpg)
        62       Video (%H%M%S%d%m%Y******.mp4)
        1048625  image

Emoji chars:

    u_keyword_list = [
        u"微笑", u"撇嘴", u"色", u"发呆", u"得意", u"流泪",
        u"害羞", u"闭嘴", u"睡", u"大哭", u"尴尬", u"发怒",
        u"调皮", u"呲牙", u"惊讶", u"难过", u"酷", u"冷汗",
        u"抓狂", u"吐", u"偷笑", u"愉快", u"白眼", u"傲慢",
        u"饥饿", u"困", u"惊恐", u"流汗", u"憨笑", u"悠闲",
        u"奋斗", u"咒骂", u"疑问", u"嘘", u"晕", u"疯了",
        u"哀", u"骷髅", u"敲打", u"再见", u"擦汗", u"抠鼻",
        u"鼓掌", u"糗大了", u"坏笔", u"左哼哼", u"右哼哼",
        u"哈欠", u"鄙视", u"委屈", u"快哭了", u"阴险", u"亲亲",
        u"吓", u"可怜", u"菜刀", u"西瓜", u"啤酒", u"篮球",
        u"乒乓", u"咖啡", u"饭", u"猪头", u"玫瑰", u"凋谢",
        u"嘴唇", u"爱心", u"心碎", u"蛋糕", u"闪电", u"炸弹",
        u"刀", u"足球", u"瓢虫", u"便便", u"月亮", u"太阳",
        u"礼物", u"拥抱", u"强", u"弱", u"握手", u"胜利",
        u"抱拳", u"勾引", u"拳头", u"差劲", u"爱你", u"NO",
        u"OK", u"爱情", u"飞吻", u"跳跳", u"发抖", u"怄火",
        u"转圈", u"磕头", u"回头", u"跳绳", u"投降", u"激动",
        u"乱舞", u"献吻", u"左太极", u"右太极"
    ]

    u_code_list = [
        u"/::)", u"/::~", u"/::B", u"/::|", u"/:8-)", u"/::<", u"/::$",
        u"/::X", u"/::Z", u"/::'(", u"/::-|", u"/::@", u"/::P", u"/::D",
        u"/::O", u"/::(", u"/::+", u"/:–b", u"/::Q", u"/::T", u"/:,@P",
        u"/:,@-D", u"/::d", u"/:,@o", u"/::g", u"/:|-)", u"/::!", u"/::L",
        u"/::>", u"/::,@", u"/:,@f", u"/::-S", u"/:?", u"/:,@x", u"/:,@@",
        u"/::8", u"/:,@!", u"/:!!!", u"/:xx", u"/:bye", u"/:wipe", u"/:dig",
        u"/:handclap", u"/:&-(", u"/:B-)", u"/:<@", u"/:@>", u"/::-O",
        u"/:>-|", u"/:P-(", u"/::'|", u"/:X-)", u"/::*", u"/:@x", u"/:8*",
        u"/:pd", u"/:<W>", u"/:beer", u"/:basketb", u"/:oo", u"/:coffee",
        u"/:eat", u"/:pig", u"/:rose", u"/:fade", u"/:showlove", u"/:heart",
        u"/:break", u"/:cake", u"/:li", u"/:bome", u"/:kn", u"/:footb",
        u"/:ladybug", u"/:shit", u"/:moon", u"/:sun", u"/:gift", u"/:hug",
        u"/:strong", u"/:weak", u"/:share", u"/:v", u"/:@)", u"/:jj",
        u"/:@@", u"/:bad", u"/:lvu", u"/:no", u"/:ok", u"/:love", u"/:<L>",
        u"/:jump", u"/:shake", u"/:<O>", u"/:circle", u"/:kotow", u"/:turn",
        u"/:skip", u"/:oY", u"/:#-0", u"/:hiphot", u"/:kiss", u"/:<&",
        u"/:&>"
    ]
