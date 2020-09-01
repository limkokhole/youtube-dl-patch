# youtube-dl-patch
my patch to youtube-dl

1. Fixed file name too long which is the most critical bug in youtube-dl especially to non-English video title and Twitter, now will truncate by max 250 bytes + ".part" = 255 bytes, or 242(docker), 143(eCryptfs) accurately(respect utf-8). Only fixed in Linux(No Mac to test) and python 3(python 2 need count length without `.encode('utf-8')` and need decide allow incomplete unicode or not).  (Windows need \/\/?/\/ prefix in full path) . 
I don't play with complicated format, only hard-coded my desired output format `'./%(title)s-%(upload_date)s-%(id)s.%(ext)s'`
Longer filename will truncated and endswith ... in title part.  

To make the patch simpler, my patch always sacrifices 5 bytes for ".part" even though final filename doesn't include ".part".

To use this patch, you can simply diff the files of .py (not py.orig) with your youtube-dl files and replace directly if both same. Or else you need run `patch your_file <my.pt.patch` to patch, or diff to manually copy-paste. 

If you not sure where the library is use, you may try `strace -f -e open,openat youtube-dl video_id` `OR strace -f -e file youtube-dl video_id` to figure out.

Patch based on file at:  
    - https://github.com/ytdl-org/youtube-dl/commit/fca6dba8b80286ae6d3ca0a60c4799c220a52650 (YoutubeDL.py, [or my backup YoutubeDL.py.orig](https://github.com/limkokhole/youtube-dl-patch/blob/master/YoutubeDL.py.orig))  
    - https://github.com/ytdl-org/youtube-dl/commit/9cd5f54e31bcfde1f0491d2c7c3e2b467386f3d6#diff-41e5a35fe85b286fe4b4f735f8ac8fae (utils.py, [or my backup utils.py.orig](https://github.com/limkokhole/youtube-dl-patch/blob/master/utils.py.orig))  
    - https://github.com/ytdl-org/youtube-dl/commit/3bce4ff7d96d845ec67ffe8e9e2715474f190d89 (downloader/fragment.py, [or my backup fragment.py.orig](https://github.com/limkokhole/youtube-dl-patch/blob/master/fragment.py.orig))  

#### Assume you installed this way (Not `https://yt-dl.org/latest/youtube-dl` which is python 2 binary):
    xb@dnxb:/tmp/yt$ python3 -m pip install -U youtube_dl
    Collecting youtube_dl
      Using cached https://files.pythonhosted.org/packages/61/1c/a86837929eff24827b117d577584cc1a2a85dfdb5a91465d17c8b298f0d0/youtube_dl-2020.7.28-py2.py3-none-any.whl
    Installing collected packages: youtube-dl
    Successfully installed youtube-dl-2020.7.28

#### Download will failed:
    xb@dnxb:/tmp/yt$ youtube-dl 'TqA2WVwbI6Y'
    [youtube] TqA2WVwbI6Y: Downloading webpage
    [youtube] TqA2WVwbI6Y: Downloading MPD manifest
    WARNING: Requested formats are incompatible for merge and will be merged into mkv.
    [dashsegments] Total fragments: 4
    [download] Destination: 𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑-TqA2WVwbI6Y.f247.webm
    ERROR: unable to download video data: [Errno 36] File name too long: '𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑-TqA2WVwbI6Y.f247.webm.ytdl'

#### strace to know the library path (the path probably at e.g. `~/.virtualenvs/youtubedlbuild/lib/python3.6/site-packages/youtube_dl-2020.7.28-py3.6.egg/youtube_dl/` if you use virtualenvs):
    xb@dnxb:/tmp/yt$ strace -f -e file /home/xiaobai/.local/bin/youtube-dl 'TqA2WVwbI6Y' |& grep YoutubeDL.py
    stat("/home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/YoutubeDL.py", {st_mode=S_IFREG|0664, st_size=111152, ...}) = 0
    stat("/home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/YoutubeDL.py", {st_mode=S_IFREG|0664, st_size=111152, ...}) = 0
    xb@dnxb:/tmp/yt$ 

#### Diff the relevant .py files with .py.orig files of this patch to know not much different with the original file of this patch: 
    xb@dnxb:/tmp/yt$ diff /home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/YoutubeDL.py ~/youtube-dl-patch/YoutubeDL.py.orig
    xb@dnxb:/tmp/yt$ diff /home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/utils.py ~/youtube-dl-patch/utils.py.orig 
    xb@dnxb:/tmp/yt$ diff /home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/downloader/fragment.py ~/youtube-dl-patch/downloader/fragment.py.orig
    xb@dnxb:/tmp/yt$ 

#### patch the files:
    xb@dnxb:/tmp/yt$ patch /home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/YoutubeDL.py <~/youtube-dl-patch/YoutubeDL.patch 
    patching file /home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/YoutubeDL.py
    xb@dnxb:/tmp/yt$ patch /home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/utils.py <~/youtube-dl-patch/utils.patch 
    patching file /home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/utils.py
    xb@dnxb:/tmp/yt$ patch /home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/downloader/fragment.py <~/youtube-dl-patch/downloader/fragment.patch
    patching file /home/xiaobai/.local/lib/python3.6/site-packages/youtube_dl/downloader/fragment.py
    xb@dnxb:/tmp/yt$

#### Download again will success:
    xb@dnxb:/tmp/yt$ ls -la
    total 344
    drwxrwxr-x  2 xiaobai xiaobai   4096 Sep   1 22:07 .
    drwxrwxrwt 31 root    root    122880 Sep   1 22:03 ..
    -rw-rw-r--  1 xiaobai xiaobai 220912 Sep   1 22:07 𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑...-20200630-TqA2WVwbI6Y.mkv

#### Copy the filename above, touch a new file by adding suffix `.part𪍑`, it shows file too long:
    xb@dnxb:/tmp/yt$ touch '𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑...-20200630-TqA2WVwbI6Y.mkv.part𪍑'
    touch: cannot touch '𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑...-20200630-TqA2WVwbI6Y.mkv.part𪍑': File name too long

#### Once remove the `𪍑` but remains the temporary extension `.part`, now it able to touch. What's that means is this patch cut the filename precisely.
    xb@dnxb:/tmp/yt$ touch '𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑𪍑...-20200630-TqA2WVwbI6Y.mkv.part'
    xb@dnxb:/tmp/yt$ 



