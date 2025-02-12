quiet.py
========

[![](https://img.shields.io/pypi/v/quiet.py.svg)](https://pypi.org/project/quiet.py/)


Python ctypes bindings for libquiet to transmit data with sound.

## Requirements
+ numpy

## Install

+ For ARM platform, binary package is available on pypi, just use `pip` to install it:

  ```bash
  sudo apt install python-numpy
  pip install --no-deps quiet.py
  ```
  
  We install `numpy` separately, as installing `numpy` via pip requires compiling numpy from source.
  
+ For x86/amd64

  ```bash
  sudo apt install cmake
  git clone https://github.com/OpenVoiceOS/quiet.py && cd quiet.py
  ./scripts/libs.sh
  pip install .
  ```


## Usage
1. Encode a message, and then decode it
```python
from quiet import Encode, Decoder

def test():
    encoder = Encoder()
    decoder = Decoder()

    for chunk in encoder.encode('hello, world'):
        message = decoder.decode(chunk)
        if message is not None:
            print(message)


test()
```

2. decode messages from recording in realtime

```python
import sys
import numpy
import pyaudio
from quiet import Encode, Decoder

def decode():
    if sys.version_info[0] < 3:
        import Queue as queue
    else:
        import queue

    FORMAT = pyaudio.paFloat32
    CHANNELS = 1
    RATE = 44100
    CHUNK = 16384  # int(RATE / 100)

    p = pyaudio.PyAudio()
    q = queue.Queue()

    def callback(in_data, frame_count, time_info, status):
        q.put(in_data)
        return (None, pyaudio.paContinue)

    stream = p.open(format=FORMAT,
                    channels=CHANNELS,
                    rate=RATE,
                    input=True,
                    frames_per_buffer=CHUNK,
                    stream_callback=callback)

    count = 0
    with Decoder(profile_name='ultrasonic-experimental') as decoder:
        while True:
            try:
                audio = q.get()
                audio = numpy.fromstring(audio, dtype='float32')
                # audio = audio[::CHANNELS]
                code = decoder.decode(audio)
                if code is not None:
                    count += 1
                    print(code.tostring().decode('utf-8', 'ignore'))
            except KeyboardInterrupt:
                break


decode()
```


