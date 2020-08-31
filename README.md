# youtube-dl-patch
my patch to youtube-dl

1. Fixed file name too long which is the most critical bug in youtube-dl especially to non-English video title and Twitter, now will truncate by max 250 bytes + ".part" = 255 bytes, or 242(docker), 143(eCryptfs) accurately(respect utf-8). Only fixed in Linux(No Mac to test) and python 3(python 2 need count length without `.encode('utf-8')` and need decide allow incomplete unicode or not).  (Windows need \/\/?/\/ prefix in full path) . 
I don't play with complicated format, only hard-coded my desired output format `'./%(title)s-%(upload_date)s-%(id)s.%(ext)s'`
Longer filename will truncated and endswith ... in title part.  

To make the patch simpler, my patch sacrifice 10 bytes of ".part.part" for '.mp4.part-Frag0.part' regardless of part really exist or not.

Patch based on file at:  
    - https://github.com/ytdl-org/youtube-dl/commit/fca6dba8b80286ae6d3ca0a60c4799c220a52650 (YoutubeDL.py, [or my backup YoutubeDL.py.orig](https://github.com/limkokhole/youtube-dl-patch/blob/master/YoutubeDL.py.orig))  
    - https://github.com/ytdl-org/youtube-dl/commit/9cd5f54e31bcfde1f0491d2c7c3e2b467386f3d6#diff-41e5a35fe85b286fe4b4f735f8ac8fae (utils.py, [or my backup utils.py.orig](https://github.com/limkokhole/youtube-dl-patch/blob/master/utils.py.orig))  
    - https://github.com/ytdl-org/youtube-dl/commit/3bce4ff7d96d845ec67ffe8e9e2715474f190d89 (downloader/fragment.py, [or my backup fragment.py.orig](https://github.com/limkokhole/youtube-dl-patch/blob/master/fragment.py.orig))  

