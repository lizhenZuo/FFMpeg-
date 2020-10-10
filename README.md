


## 视频部分
### 视频信息获取

- 显示文件信息  
```
    ffmpeg -i prL8-2-g.mp4 -hide_banner
```

-  获取流的信息   
```
   ffprobe prL8-2-g.mp4 2>&1 | findstr Stream （Windows写法，在mac上是没有findstr命令的，把findstr 改成grep）
```

- 获取视频时长  
```
   ffprobe -i prL8-2-g.mp4 -show_entries format=duration -v quiet -of csv="p=0"
```

- 获取帧率  
```
ffprobe -v error -select_streams v:0 -show_entries stream=avg_frame_rate -of default=noprint_wrappers=1:nokey=1 prL8-2-g.mp4
```

- 获取pts信息   
```
ffprobe  -show_frames -select_streams v prL8-2-g.mp4 | findstr pkt_pts （Windows写法，在mac上是没有findstr命令的，把findstr 改成grep）
```

- 获取视频帧数  
```
ffprobe -v error  -count_frames -select_streams v:0 -show_entries stream=nb_read_frames -of default=nokey=1:noprint_wrappers=1 prL8-2-g.mp4
```
### 视频处理

- 改变视频的分辨率 [-s 1280x720  -filter:v scale=1280:720]  -qscale 0   
```
ffmpeg -i prL8-2-g.mp4  -s 1280x720  -c:a copy -y  ./resize.mp4
```
- 改变视频帧率   
```
ffmpeg -i prL8-2-g.mp4  -r 25 ./25.mp4
```

- 分离出无声的mp4  -y 存在覆盖 -n 存在退出    
```
ffmpeg  -i   prL8-2-g.mp4  -v 0  -an -y -vcodec copy  ./prL8-2-g_sil.mp4
```

- 只取视频并缩放  
```
    ffmpeg -i prL8-2-g.mp4 -filter_complex "[0:0] scale=320x180[smaller_sized]"  -map "[smaller_sized]" ./out.mp4
```
- 视频解析为帧序列  
```
ffmpeg  -i   prL8-2-g.mp4 -vsync 0 -qscale:v 2  -y  ./pngs/%05d.png
```

- 帧序列合成为视频
```
ffmpeg  -v error -r 15 -start_number 1 -i  ./pngs/%05d.png   -vframes  177 -maxrate 400k  -c:v libx264  -preset veryslow -crf 15 -pix_fmt yuv420p  ./compose.mp4  （这种合成方式没有声音，因为只是合成图片，如果想有声音，需要将mp3和mp4再合并一次）
```

- 视频裁剪一段
```
ffmpeg -i   prL8-2-g.mp4  -ss 1.1   -t 3.5 -c:v libx264  -c:a copy  -y  ./part.mp4
```

- 调整音量
```
ffmpeg   -i  prL8-2-g.mp4 -v 0 -c:v copy -af volume=7  -acodec aac -ar 48k -ab 96k -ac 1 -y ./louder.mp4
```

- 获取视频中的一帧
```
ffmpeg   -i  prL8-2-g.mp4 -v 0 -vf  select='eq(n,0)' -vframes 1 -y  ./oneframe.png  ##从0开始 (??执行这个指令，未找到oneframe.png)
上面指令错误，应该是如下：
ffmpeg   -i  prL8-2-g.mp4 -v 0 -vf  "select='eq(n,0)'" -vframes 1 -y  ./oneframe.png  ##select='eq(n,0)'需要加一个双引号
```

- 单帧图片生成视频  
```
ffmpeg  -framerate 1/2 -i  oneframe.png -pix_fmt yuv420p -r 15 -y  ./onemp4.mp4 （上一步未完成）
上一步正常，这一步也正常了
```

- 添加静音音频
```
ffmpeg  -f lavfi -i anullsrc=channel_layout=mono:sample_rate=48000 -i onemp4.mp4 -shortest -c:v copy -c:a aac -map 0:a -map 1:v ./oneaudio.mp4 （上一步未完成）
上一步正常，这一步也正常了
```

- 获取指定的帧,从0开始算起取到第3帧,共4帧. 重命名从66开始
```
ffmpeg  -i  prL8-2-g.mp4 -vf select='between(n,0,3)' -vsync 0 -start_number  66  ./pngs/%05d.png  ##（报错：“Missing ')' or too many args in 'between(n'”）
上面指令错误，应该是如下：
ffmpeg  -i  prL8-2-g.mp4 -vf "select='between(n,0,3)'" -vsync 0 -start_number  66  ./pngs/%05d.png ###select='between(n,0,3)'需要加一个双引号
```

- 给图片添加文字   添加中文
```
ffmpeg  -i oneframe.png  -vf drawtext=text='dingdongketang':fontcolor=red:fontsize=20:x=260:y=200:  ./text.png ##（上一步未完成）
## 上一步正常，这一步也正常了
```

- mp4 转avi格式
```
ffmpeg  -i  prL8-2-g.mp4  -vf unsharp=luma_msize_x=5:luma_msize_y=5:luma_amount=0.3 -v 0 -c:v libx264  -preset slow -crf 22 -c:a copy ./prL8-2-g.avi
(执行命令，等了一会才生成prL8-2-g.avi文件)
```

-  对mp4归一化处理
```
ffmpeg  -i prL8-2-g.mp4 -acodec aac -ar 48000 -ab 128k -ac 1  -vf "scale=960x540,vaguedenoiser=threshold=2,unsharp=luma_msize_x=3:luma_msize_y=3:luma_amount=1.1"  -c:v libx264  -profile:v high -level 4.1 -preset:v slower -r 15 -keyint_min 15 -g 15   -b:v 800k -minrate:v 700k -maxrate:v 900k -bufsize:v 800k  -tune psnr -pix_fmt yuv420p -f mp4 -y prL8-2-g_norm.mp4 
```

- 压缩mp4
```
ffmpeg  -i prL8-2-g.mp4 -v 0 -b:v 280k -minrate:v 230k -maxrate:v 330k -bufsize:v 330k  -tune psnr -pix_fmt yuv420p -f mp4 -y prL8-2-g_compact.mp4
```

- 多个视频按列表顺序合并  
```
ffmpeg  -f concat -safe 0 -i  filelocal.txt -map 0 -c copy -y  ./merge.mp4
```

- 视频播放变快变慢
```
ffmpeg -i prL8-2-g.mp4  -r 15 -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]"   -y   ./perturb.mp4
```

- 合并mp4和m4a   
```
ffmpeg  -i  prL8-2-g_sil.mp4  -i prL8-2-g.m4a -v 0 -c:v copy -c:a copy  ./prL8-2-g_recon.mp4
```

- 合并avi和m4a  
```
ffmpeg  -i  prL8-2-g.avi -i prL8-2-g.m4a -vf unsharp=luma_msize_x=5:luma_msize_y=5:luma_amount=0.3 -v 0  -c:v libx264  -preset slow -crf 22 -c:a copy  ./prL8-2-g_re2.mp4
```

- 获取alpha通道  
```
ffmpeg  -i  A103-2B.mov -vf alphaextract -pix_fmt yuv420p -y ./alpha.mp4
```


-  画中画  -itsoffset 2.5 播放2.5秒后显示logo
```
ffmpeg -i prL8-2-g.mp4 -itsoffset 2.5  -i ring_100x87.png -filter_complex overlay=W-w:56 -max_muxing_queue_size 1024  -y ./ring_logo_t.mp4
```


## 音频处理部分

### 音频信息获取
- 获取音频时长  
```
ffprobe -v error  -select_streams a:0 -show_entries stream=duration -of default=noprint_wrappers=1:nokey=1 prL8-2-g.mp3
```
- 获取音量  
```
ffmpeg -i prL8-2-g.mp3  -filter_complex volumedetect -c:v copy  -f null /dev/null  2>&1 | grep mean_volume
```
- 获取采样率  
```
ffprobe  -v error -select_streams a -show_entries stream=sample_rate -of default=nokey=1:noprint_wrappers=1 prL8-2-g.m4a
```
- 获取通道数  
```
ffprobe  -v error -select_streams a -show_entries stream=channel_layout -of default=nokey=1:noprint_wrappers=1 prL8-2-g.m4a
```
- 获取比特率  数值/1000  
```
ffprobe  -v error -select_streams a -show_entries stream=bit_rate -of default=nokey=1:noprint_wrappers=1 prL8-2-g.m4a
```
### 音频处理
-  分离出[pcm m4a wav flac mp3] 注:wav mp3时长略有差异  
```
ffmpeg  -i   prL8-2-g.mp4 -v 0  -vn -ar 48000 -ac 1 -f s16le  -y ./prL8-2-g.pcm
```
```
ffmpeg  -i   prL8-2-g.mp4 -v 0 -vn  -acodec copy  -y  ./prL8-2-g.m4a
```
```
ffmpeg  -i   prL8-2-g.mp4 -f wav   ./prL8-2-g.wav
```
```
ffmpeg  -i   prL8-2-g.mp4 -v 0 -vn -ab 96k -ac 1 -ar 48000  ./96k_mono.wav
```
```
ffmpeg  -i   prL8-2-g.mp4 -v 0 -f flac -acodec flac -ac 1 -ar 48000 -y ./prL8-2-g.flac
```
```
ffmpeg  -i  prL8-2-g.mp4 -v 0 -acodec libmp3lame -ar 48k -ab 96k -ac 1  ./prL8-2-g.mp3
```

- 不同格式之间相互转换  
```
ffmpeg   -i  prL8-2-g.wav  -f mp3  -y ./wav2mp3.mp3
```
```
ffmpeg   -i  prL8-2-g.wav -v 0  -ar 16000   -y  ./convert.wav
```
```
ffmpeg   -i  prL8-2-g.wav -v 0  -ac 1 -ar 48000 -f s16le -acodec pcm_s16le -y ./wav2pcm.pcm
```
```
ffmpeg   -i  prL8-2-g.mp3 -v 0  -acodec pcm_s16le -f s16le -ac 1  -y  ./mp32pcm.pcm
```
```
ffmpeg   -i  prL8-2-g.mp3 -v 0  -f wav  ./mp32wav.waff
```
```
ffmpeg   -v 0  -ac 1 -ar 48000 -f s16le -acodec pcm_s16le -i  prL8-2-g.pcm -y ./pcm2wav.wav   ###-i 放在后面
```
```
ffmpeg   -i  prL8-2-g.m4a -v 0  -f wav  -ar 16000  -y ./m4a2wav.wav
```
```
ffmpeg   -i  prL8-2-g.wav -ab 128k -acodec aac   -y  ./wav2aac.aac
```
```
ffmpeg   -i  apple.aifc -v 0  -f mp3 -acodec libmp3lame -ab 256000 -ar 16000 -y ./aifc2mp3.mp3   ##mac 音频格式aifc
```
- 截取音频片段  
```
ffmpeg   -i  prL8-2-g.wav  -v 0 -acodec copy -t 3.5  -ss 1.1 -y  ./part.wav
```

- 两个音频文件合并  
```
ffmpeg  -i   prL8-2-g.mp4 -f wav   ./prL8-2-g.wav
```
```
ffmpeg  -i   prL8-2-g-r.mp4 -f wav   ./prL8-2-g-r.wav
```
```
ffmpeg  -i  prL8-2-g.mp4 -v 0 -acodec libmp3lame -ar 48k -ab 96k -ac 1  ./prL8-2-g.mp3
```
```
ffmpeg  -i  prL8-2-g-r.mp4 -v 0 -acodec libmp3lame -ar 48k -ab 96k -ac 1  ./prL8-2-g-r.mp3
```
```
ffmpeg  -i prL8-2-g.wav  -i prL8-2-g-r.wav -filter_complex "[0:0] [1:0] concat=n=2:v=0:a=1 [a]" -map "[a]" -y ./audio_m2.wav
-map [a] 需要改为  -map "[a]"
同时，如果是两个视频合并可以如下：
ffmpeg  -i prL8-2-g.mp4  -i prL8-2-g-r.mp4 -filter_complex "[0:0] [1:0] concat=n=2:v=1:a=1 [v] [a]" -map "[v]" -map "[a]" -y ./video_m0.mp4
```
```
ffmpeg  -f concat -safe 0 -i fileaudio.txt -c copy  -y ./audio_m3.mp3
"-f concat -safe 0 -i fileaudio.txt"这个的顺序不能更改，我本来想把-i的指令放到-f之前，结果运行不通过
```

- 渐入渐出命令
```
ffmpeg   -i prL8-2-g.wav -i prL8-2-g-r.wav  -vn -filter_complex acrossfade=d=0.5:c1=tri:c2=tri -acodec libmp3lame -ar 48k -ab 128k -ac 1 -y ./fade_m.mp3
```

- 开头或结尾添加静音 或两个之间添加静音     
```
ffmpeg -f lavfi -t 2.5 -i anullsrc=channel_layout=mono:sample_rate=48000 -i prL8-2-g.wav -filter_complex "[0:a][1:a]concat=n=2:v=0:a=1" -y  ./sil_b.wav
```
```
ffmpeg -f lavfi -t 5   -i anullsrc=channel_layout=mono:sample_rate=48000  -i prL8-2-g.wav -filter_complex "[1:a][0:a]concat=n=2:v=0:a=1"  -y  ./sil_e.wav
```
```
ffmpeg -i prL8-2-g.wav -i prL8-2-g-r.wav -filter_complex "aevalsrc=exprs=0:d=1.5[silence],[0:a] [silence] [1:a] concat=n=3:v=0:a=1 [outa]" -map "[outa]" -y ./sil_m.wav
-map [outa] 需要改为 -map "[outa]"
```

- 去除开头或结尾的静音 开头+结尾  结尾去除不准确,故做了翻转处理  
```
ffmpeg -i sil_b.wav  -af silenceremove=1:0:-50dB  -y  ./sil_b_remove.wav
```
```
ffmpeg -i sil_e.wav -v 0 -af areverse,silenceremove=1:0:-50dB,areverse   -y  ./sil_e_remove.wav
```
```
ffmpeg -i  prL8-2-g.wav -af silenceremove=1:0:-50dB,areverse,silenceremove=1:0:-50dB,areverse  -y ./sil.wav
```
```
ffmpeg -i prL8-2-g.wav -af silenceremove=stop_periods=-1:stop_duration=1:stop_threshold=-50dB -y ./sil1.wav ##时间有差异
```

-  去除inwav1尾和inwav2头的静音，中间添加dur时长的静音
```
ffmpeg  -y  -i prL8-2-g.wav -i prL8-2-g-r.wav -filter_complex "[0:a] silenceremove=stop_periods=0:stop_duration=0:stop_threshold=-50dB [first], [1:a] silenceremove=start_periods=1:start_duration=0:start_threshold=-50dB [second],aevalsrc=exprs=0:d=1.5[silence],[first] [silence] [second] concat=n=3:v=0:a=1 [outa]" -map "[outa]"  -y  ./sil_m2.wav
-map [outa] 需要改为 -map "[outa]"
第一个 -y 可以不要
```




