# youtube-dl-patch
my patch to youtube-dl-patch

1. Fixed file name too long which is the most critical bug in youtube-dl especially to non-English video title and Facebook, now will truncate by max 250 bytes + ".part" = 255 bytes, or 242(docker), 143(eCryptfs) accurately(respect utf-8). Only fixed in Linux(No Mac to test) and python 3(python 2 need count length without `.encode('utf-8')` and need decide allow incomplete unicode or not).  (Windows need \/\/?/\/ prefix in full path) . 
I don't play with complicated format, only hard-coded my desired output format `'./%(title)s-%(upload_date)s-%(id)s.%(ext)s'`
Longer filename will truncated and endswith ... in title part.  
A small issue I noticed is .part (5 bytes) is waste the filename space, why not make it .p or .tmp when converting, or better pass another longer name after converted?  
  
        
    Patch based on file at:  
    - https://github.com/ytdl-org/youtube-dl/commit/fca6dba8b80286ae6d3ca0a60c4799c220a52650 (YoutubeDL.py)  
    - https://github.com/ytdl-org/youtube-dl/commit/9cd5f54e31bcfde1f0491d2c7c3e2b467386f3d6#diff-41e5a35fe85b286fe4b4f735f8ac8fae (utils.py)  
