---
title: AAC Encoder
html_meta:
    description: This article explains how you can use Transcoder to encode an AAC ADTS audio stream from raw audio frames.
taxonomy:
    category: docs
---

# AAC Encoder

How to use [Transcoder.Push](https://doc.avblocks.com/net/latest/class_primo_software_1_1_a_v_blocks_1_1_transcoder.html#a60429623c9613c8e6f0c0f3cf6ec598c) to encode an AAC ADTS audio stream from raw audio frames.

## Source Audio

As audio input we use the `equinox-48KHz.wav` file from the [AVBlocks Assets](https://github.com/avblocks/avblocks-assets/releases) archive. After downloading and unzipping you will find `equinox-48KHz.wav` in the `aud` subdirectory.


## Code

This code shows how you can encode raw uncompressed audio frames into an AAC ADTS stream. Two Transcoder objects are used, one to read the raw LPCM frames from a WAV file, and another to encode the raw frames to AAC ADTS stream. The encoding is done via the `Transcoder.Push` method.   

### Initialize AVBlocks

``` csharp
static void EncodeAacAdtsStream()
{
    Library.Initialize();

    EncodeAacAdtsStream("equinox-48KHz.wav", "equinox-48KHz.aac");

    Library.Shutdown();
}
```

### Configure WAV Reader

``` csharp
static Transcoder CreateWavReader(string inputFile)
{
    // Create MediaSocket describing the WAV input using MediaInfo
    MediaInfo mediaInfo = new MediaInfo();
    mediaInfo.Inputs[0].File = inputFile;
    
    if (!mediaInfo.Open())
        throw new Exception($"Could not parse {inputFile}.");

    MediaSocket inputSocket = MediaSocket.FromMediaInfo(mediaInfo);

    mediaInfo.Close();

    // Create AudioStreamInfo, MediaPin, and MediaSocket describing the LPCM output
    // This is the same as the input, but no output file is set on the MediaSocket, 
    // because we want to pull frames one by one using Transcoder::pull

    // MediaPin and AudioStreamInfo
    var outputPin = new MediaPin() {
        StreamInfo = new AudioStreamInfo() {
            StreamType = StreamType.LPCM,
            Channels = 2,
            SampleRate = 48000,
            BitsPerSample = 16
        }
    };

    // MediaSocket
    var outputSocket = new MediaSocket();
    outputSocket.StreamType = StreamType.LPCM;
    outputSocket.Pins.Add(outputPin);

    // Create Transcoder
    var wavReader = new Transcoder();
    wavReader.Inputs.Add(inputSocket);
    wavReader.Outputs.Add(outputSocket);

    return wavReader;
}
```

### Configure AAC Encoder

``` csharp
static Transcoder CreateAacEncoder(string outputFile)
{
    // Create AudioStreamInfo, MediaPin, and MediaSocket describing the LPCM input

    // MediaPin and AudioStreamInfo
    var inputPin = new MediaPin() {
        StreamInfo = new AudioStreamInfo() {
            StreamType = StreamType.LPCM,
            SampleRate = 48000,
            Channels = 2,
            BitsPerSample = 16,
            BytesPerFrame = 4 // = (BitsPerSample / 8) * Channels
        }
    };

    // MediaSocket
    var inputSocket = new MediaSocket() {
        StreamType = StreamType.LPCM
    };

    inputSocket.Pins.Add(inputPin);

    // Create AudioStreamInfo, MediaPin, and MediaSocket describing the AAC / M4A output

    // MediaPin and AudioStreamInfo
    var outputPin = new MediaPin() {
        StreamInfo = new AudioStreamInfo() {
            StreamType  = StreamType.Aac,
            StreamSubType = StreamSubType.AacAdts,
            Channels = 2,
            SampleRate = 48000,
            BitsPerSample = 16
        }
    };

    // MediaSocket
    var outputSocket = new MediaSocket() {
        StreamType = StreamType.Aac,
        StreamSubType = StreamSubType.AacAdts,
        File = outputFile
    };

    outputSocket.Pins.Add(outputPin);

    // Transcoder
    var aacEncoder = new Transcoder();
    aacEncoder.Inputs.Add(inputSocket);
    aacEncoder.Outputs.Add(outputSocket);

    return aacEncoder;
}
```

### Open Transcoders

``` csharp
static void EncodeAacAdtsStream(string inputFile, string outputFile)
{
    // Create a reader to simulate raw video frames. In reality you will likely
    // have a different raw video source, for example some kind of video capture device.
    Transcoder wavReader = CreateWavReader(inputFile);

    // Create a H.264 encoder. We will pass the raw video frames to it 
    // to encode them as AVC / H.264
    Transcoder aacEncoder = CreateAacEncoder(outputFile);

    if (wavReader.Open())
    {
        if (aacEncoder.Open())
        {
            EncodeAacAdtsStream(wavReader, aacEncoder);

            aacEncoder.Close();
        }

        wavReader.Close();
    }
}
```

### Call Transcoder.Pull and Transcoder.Push

``` csharp
static void EncodeAacAdtsStream(Transcoder wavReader, Transcoder aacEncoder)
{
    int inputIndex = 0;
    MediaSample audioFrame = new MediaSample();

    while (true)
    {
        // Simulate raw frames
        // Each call to Transcoder.Pull returns one video frame.
        if (!wavReader.Pull(out inputIndex, audioFrame))
            break;

        // Pass the raw video frame to Transcoder.Push to encode it as AVC / H.264
        if (!aacEncoder.Push(0, audioFrame))
            break;
    }

    aacEncoder.Flush();
}
```

## Complete Program

``` csharp
using System;
using System.Linq;

using PrimoSoftware.AVBlocks;

namespace AacEncoder
{
    class Program
    {
        static Transcoder CreateWavReader(string inputFile)
        {
            // Create MediaSocket describing the WAV input using MediaInfo
            MediaInfo mediaInfo = new MediaInfo();
            mediaInfo.Inputs[0].File = inputFile;
            
            if (!mediaInfo.Open())
                throw new Exception($"Could not parse {inputFile}.");

            MediaSocket inputSocket = MediaSocket.FromMediaInfo(mediaInfo);

            mediaInfo.Close();

            // Create AudioStreamInfo, MediaPin, and MediaSocket describing the LPCM output
            // This is the same as the input, but no output file is set on the MediaSocket, 
            // because we want to pull frames one by one using Transcoder::pull

            // MediaPin and AudioStreamInfo
            var outputPin = new MediaPin() {
                StreamInfo = new AudioStreamInfo() {
                    StreamType = StreamType.LPCM,
                    Channels = 2,
                    SampleRate = 48000,
                    BitsPerSample = 16
                }
            };

            // MediaSocket
            var outputSocket = new MediaSocket();
            outputSocket.StreamType = StreamType.LPCM;
            outputSocket.Pins.Add(outputPin);

            // Create Transcoder
            var wavReader = new Transcoder();
            wavReader.Inputs.Add(inputSocket);
            wavReader.Outputs.Add(outputSocket);

            return wavReader;
        }

        static Transcoder CreateAacEncoder(string outputFile)
        {
            // Create AudioStreamInfo, MediaPin, and MediaSocket describing the LPCM input

            // MediaPin and AudioStreamInfo
            var inputPin = new MediaPin() {
                StreamInfo = new AudioStreamInfo() {
                    StreamType = StreamType.LPCM,
                    SampleRate = 48000,
                    Channels = 2,
                    BitsPerSample = 16,
                    BytesPerFrame = 4 // = (BitsPerSample / 8) * Channels
                }
            };

            // MediaSocket
            var inputSocket = new MediaSocket() {
                StreamType = StreamType.LPCM
            };

            inputSocket.Pins.Add(inputPin);

            // Create AudioStreamInfo, MediaPin, and MediaSocket describing the AAC / M4A output

            // MediaPin and AudioStreamInfo
            var outputPin = new MediaPin() {
                StreamInfo = new AudioStreamInfo() {
                    StreamType  = StreamType.Aac,
                    StreamSubType = StreamSubType.AacAdts,
                    Channels = 2,
                    SampleRate = 48000,
                    BitsPerSample = 16
                }
            };

            // MediaSocket
            var outputSocket = new MediaSocket() {
                StreamType = StreamType.Aac,
                StreamSubType = StreamSubType.AacAdts,
                File = outputFile
            };

            outputSocket.Pins.Add(outputPin);

            // Transcoder
            var aacEncoder = new Transcoder();
            aacEncoder.Inputs.Add(inputSocket);
            aacEncoder.Outputs.Add(outputSocket);

            return aacEncoder;
        }

        static void EncodeAacAdtsStream(Transcoder wavReader, Transcoder aacEncoder)
        {
            int inputIndex = 0;
            MediaSample audioFrame = new MediaSample();

            while (true)
            {
                // Simulate raw frames
                // Each call to Transcoder.Pull returns one video frame.
                if (!wavReader.Pull(out inputIndex, audioFrame))
                    break;

                // Pass the raw video frame to Transcoder.Push to encode it as AVC / H.264
                if (!aacEncoder.Push(0, audioFrame))
                    break;
            }

            aacEncoder.Flush();
        }

        static void EncodeAacAdtsStream(string inputFile, string outputFile)
        {
            // Create a reader to simulate raw video frames. In reality you will likely
            // have a different raw video source, for example some kind of video capture device.
            Transcoder wavReader = CreateWavReader(inputFile);

            // Create a H.264 encoder. We will pass the raw video frames to it 
            // to encode them as AVC / H.264
            Transcoder aacEncoder = CreateAacEncoder(outputFile);

            if (wavReader.Open())
            {
                if (aacEncoder.Open())
                {
                    EncodeAacAdtsStream(wavReader, aacEncoder);

                    aacEncoder.Close();
                }

                wavReader.Close();
            }
        }

        static void EncodeAacAdtsStream()
        {
            Library.Initialize();

            EncodeAacAdtsStream("equinox-48KHz.wav", "equinox-48KHz.aac");

            Library.Shutdown();
        }

        static void Main(string[] args)
        {
            EncodeAacAdtsStream();
        }
    }
}
```

## How to run

Follow the steps to [create a C# console application in Visual Studio](../getting-started-windows/create-a-c-sharp-console-app-in-visual-studio) but in `Program.cs` use the code from this article. 

Copy the `equinox-48KHz.wav` file from the assets archive to `bin/x64/Debug/net8.0` under the project's directory.

Run the application in Visual Studio.
