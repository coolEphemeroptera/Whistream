# Whistream
A Whisper-based online video captioning tool

<img src="img/img2.gif" width="500" height="300">

## 1. Install

    pip install whishow
    pip install faster-whisper

## 2. Usage
Online captioning generation based on multithreaded interaction of whishow stream processing and faster-whisper speech recognition
```python
import threading
from whisper_api import WhisperModel
from whishow import STREAM
from whishow import PLAY
import time 

url = "rtmp://mobliestream.c3tv.com:554/live/goodtv.sdp"
url = "test.mp4"
language = "zh"

# init the stream reader, named stm.
stm = STREAM()
stm.init_state(url=url,
               cache_size=10*60,
               video_frame_quality=50)

# init the whisper model, and connect the audio stream of stm
asr = WhisperModel(model_size_or_path=r"tiny", 
                    device='cpu', 
                    compute_type="int8",
                    download_root=r"./models",
                    local_files_only=False)
asr.init_state(Q_audio_asr=stm.Q_audio_asr,
               read_size=16000)

# init the whishow player, and connect the audio/video stream of stm and the asr result
ply = PLAY()
ply.init_state(chunk_size=1,
                video_frame_shift=10,
                audio_fps=stm.AUDIO_FPS,
                video_fps=stm.VIDEO_FPS,
                Q_audio_play=stm.Q_audio_play,
                Q_video_play=stm.Q_video_play,
                asr_results=asr.asr_results)
                

# launch the stm
def process1():
    global stm
    stm.read(is_play=True,
             is_asr=True)

# launch the asr, modify your seeting
def recognition():
    global asr
    asr.transcribe(language=language,
                   task="transcribe",
                   condition_on_previous_text=True)
def process2():
    global asr
    p1 = threading.Thread(target=asr.read_data,args=())
    p2 = threading.Thread(target=recognition,args=())
    p1.start()
    p2.start()
    p1.join()
    p2.join()


# lanuch the player
def process3():
    global ply,stm
    delay = 60
    while stm.at < delay:
         print("wait for asr preprocess ..")
         time.sleep(1)
    ply.run()

# esc for exit
def engine():
        global asr,stm
        import keyboard
        while 1:
            if keyboard.is_pressed('esc'):
                print("exit ..")
                break
            time.sleep(0.1)
        stm.running = False
        asr.running = False
        ply.running = False

if __name__ == "__main__":

    p0 = threading.Thread(target=engine,args=())
    p1 = threading.Thread(target=process1,args=())
    p2 = threading.Thread(target=process2,args=())
    p3 = threading.Thread(target=process3,args=())

    p0.start()
    p1.start()
    p2.start()
    p3.start()
```

## 3. Contact us
605686962@qq.com
coolEphemeroptera@gmail.com


