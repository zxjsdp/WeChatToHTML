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
