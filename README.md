# Decklink -> ffmpeg -> HLS streaming example

This demo is used to create a HLS stream locally using a 

- Blackmagic Decklink compatible device
- ffmpeg as streaming encoder

To run `ffmpeg` must be compiled with `decklink`
I use Homebrew

      brew install homebrew-ffmpeg/ffmpeg/ffmpeg $(brew options
      homebrew-ffmpeg/ffmpeg/ffmpeg | grep -vE '\s' | grep -- '--with-' | grep -vi 'chromaprint\|libzvbit\|libflite' | tr '\n' ' ')

After successul installation `cd` into this repo and start your webserver. 



The following is the command used to generate the m3u8 file and the segments.
The command captures the video and audio from the UltraStudio 4K and encodes it into three different resolutions: 1080p, 720p, and 480p.
The command uses the HLS (HTTP Live Streaming) format to create the segments and the playlist file.

Just copy paste this into the terminal and edit it accordingly:

To list sources use:

    ffmpeg --list_devices decklink

eg output:

    Auto-detected sources for decklink:
        64:4060058d:00000000 [UltraStudio 4K] (none)

To list the formats use (I assume we have "Ultrastudio 4K" as result from the previous command):

      ffmpeg -f decklink -list_formats 1 -i "UltraStudio 4K"

eg output:

        [decklink @ 0x1397d0f90] Supported formats for 'UltraStudio 4K':
            format_code	description
            ntsc		720x486 at 30000/1001 fps (interlaced, lower field first)
            pal 		720x576 at 25000/1000 fps (interlaced, upper field first)
            23ps		1920x1080 at 24000/1001 fps
            24ps		1920x1080 at 24000/1000 fps
            Hp25		1920x1080 at 25000/1000 fps
            Hp29		1920x1080 at 30000/1001 fps
            Hp30		1920x1080 at 30000/1000 fps
            Hp50		1920x1080 at 50000/1000 fps
            Hp59		1920x1080 at 60000/1001 fps
            Hp60		1920x1080 at 60000/1000 fps
            Hi50		1920x1080 at 25000/1000 fps (interlaced, upper field first)
            Hi59		1920x1080 at 30000/1001 fps (interlaced, upper field first)
            Hi60		1920x1080 at 30000/1000 fps (interlaced, upper field first)
            hp50		1280x720 at 50000/1000 fps
            hp59		1280x720 at 60000/1001 fps
            hp60		1280x720 at 60000/1000 fps
            2d23		2048x1080 at 24000/1001 fps
            2d24		2048x1080 at 24000/1000 fps
            2d25		2048x1080 at 25000/1000 fps
            4k23		3840x2160 at 24000/1001 fps
            4k24		3840x2160 at 24000/1000 fps
            4k25		3840x2160 at 25000/1000 fps
            4k29		3840x2160 at 30000/1001 fps
            4k30		3840x2160 at 30000/1000 fps
            4d23		4096x2160 at 24000/1001 fps
            4d24		4096x2160 at 24000/1000 fps
            4d25		4096x2160 at 25000/1000 fps
        [in#0 @ 0x6000020d9000] Error opening input: Immediate exit requested
        Error opening input file UltraStudio 4K.

> Be careful that you have to match framerate and size to your incoming decklink signal and set that accordingly in the following ffmpeg command
> in my case I feed a 25 frames per second progressive video which results in `25p`

Now you can run the command:


      ffmpeg -y -f decklink -format_code Hp25 -i "UltraStudio 4K" \ -map 0:v
      -map 0:a -s:v:0 1920x1080 -b:v:0 4500k -c:v:0 libx264 -c:a:0 aac -f hls \
      -hls_time 4 -hls_list_size 5 -hls_flags delete_segments \
      -hls_segment_filename hls/1080p/seg-%03d.ts hls/1080p/playlist.m3u8 \ -map
      0:v -map 0:a -s:v:1 1280x720 -b:v:1 2500k -c:v:1 libx264 -c:a:1 aac -f hls
      \ -hls_time 4 -hls_list_size 5 -hls_flags delete_segments \
      -hls_segment_filename hls/720p/seg-%03d.ts hls/720p/playlist.m3u8 \ -map
      0:v -map 0:a -s:v:2 854x480 -b:v:2 1000k -c:v:2 libx264 -c:a:2 aac -f hls
      \ -hls_time 4 -hls_list_size 5 -hls_flags delete_segments \
      -hls_segment_filename hls/480p/seg-%03d.ts hls/480p/playlist.m3u8;

I use python and run: 

    python3 -m http.server 8888

Now you can open http://localhost:8888/




####  explanation of the command:


- -map 0:v -map 0:a: maps the video and audio streams from the input
- -s:v:0 1920x1080 -b:v:0 4500k: sets the resolution and bitrate for 1080p
- -s:v:1 1280x720 -b:v:1 2500k: sets the resolution and bitrate for 720p
- -s:v:2 854x480 -b:v:2 1000k: sets the resolution and bitrate for 480p
- -hls_time 4: sets the segment duration to 4 seconds
- -hls_list_size 5: sets the maximum number of segments in the playlist
- -hls_flags delete_segments: deletes the segments after they are played
- -hls_segment_filename hls/1080p/seg-%03d.ts: sets the segment filename pattern for 1080p 
- hls/1080p/playlist.m3u8: sets the playlist filename for 1080p

