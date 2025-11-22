---
title: AAC Decoder
html_meta:
    description: This article explains how you can use Transcoder to decode an AAC ADTS elementary stream.
taxonomy:
    category: docs
---

# AAC Decoder

This article explains how you can use [Transcoder.Pull](https://doc.avblocks.com/net/latest/class_primo_software_1_1_a_v_blocks_1_1_transcoder.html#adcea55e196dc32689f98e91fa154a0eb) to decode an AAC ADTS elementary stream.

## Source Audio

For source we use the `Hydrate-Kenny_Beltrey.aac` file from the [AVBlocks Assets](https://github.com/avblocks/avblocks-assets/releases) repository. After downloading and unzipping you will find `Hydrate-Kenny_Beltrey.aac` in the `aud` subdirectory.

## Code

This code takes an AAC ADTS stream and decodes it to uncompressed LPCM samples.

### Initialize AVBlocks

``` csharp
static void DecodeAACStream()
{
    Library.Initialize();

    DecodeAACStream("Hydrate-Kenny_Beltrey.aac");

    Library.Shutdown();
}
```

### Configure Transcoder

``` csharp
static void DecodeAACStream(string inputFile)
{
    // Create an input socket from file
    var inSocket = new MediaSocket()
    {
        File = inputFile
    };

    // Create an output socket with one YUV 4:2:0 video pin
    var outSocket = new MediaSocket()
    {
        StreamType = StreamType.LPCM
    };

    outSocket.Pins.Add(new MediaPin()
    {
        StreamInfo = new AudioStreamInfo()
        {
            StreamType = StreamType.LPCM,
            SampleRate = 44100,
            Channels = 2,
            BitsPerSample = 16
        }
    });

    // Create Transcoder
    using (var transcoder = new Transcoder())
    {
        transcoder.Inputs.Add(inSocket);
        transcoder.Outputs.Add(outSocket);

        if (transcoder.Open())
        {
            DecodeAACStream(transcoder);

            transcoder.Close();
        }
    }
}
```

### Call Transcoder.Pull

``` csharp
static void DecodeAACStream(Transcoder transcoder)
{
    int outputIndex = 0;
    var audioSample = new MediaSample();

    while (transcoder.Pull(out outputIndex, audioSample))
    {
        // Each call to Transcoder::pull returns one audio frame. 
        // One audio frame typically contains 512 audio samples.  
        ProcessAudioFrame(audioSample);
    }
}
```

## Complete Program

``` csharp
using PrimoSoftware.AVBlocks;

namespace AACDecoder
{
    class Program
    {
        static int sampleIndex = 0;
        static void ProcessAudioFrame(MediaSample sample)
        {
            // The samples of multi-channel audio (such as stereo and surround) are stored by 
            // cycling through the samples for each channel before advancing to the next sample time

            // sample.Buffer contains 512 interleaved samples.
            // In this example each sample is 16 bit signed integer (so 1024 bytes total) 

            Console.WriteLine("Sample Index : {0} StartTime : {1} Buffer.Start : {2} Buffer.DataSize : {3}", sampleIndex, sample.StartTime, sample.Buffer.Start, sample.Buffer.DataSize);
            sampleIndex++;
        }

        static void DecodeAACStream(Transcoder transcoder)
        {
            int outputIndex = 0;
            var audioSample = new MediaSample();

            while (transcoder.Pull(out outputIndex, audioSample))
            {
                // Each call to Transcoder::pull returns one audio frame. 
                // One audio frame typically contains 512 audio samples.  
                ProcessAudioFrame(audioSample);
            }
        }

        static void DecodeAACStream(string inputFile)
        {
            // Create an input socket from file
            var inSocket = new MediaSocket()
            {
                File = inputFile
            };

            // Create an output socket with one YUV 4:2:0 video pin
            var outSocket = new MediaSocket()
            {
                StreamType = StreamType.LPCM
            };

            outSocket.Pins.Add(new MediaPin()
            {
                StreamInfo = new AudioStreamInfo()
                {
                    StreamType = StreamType.LPCM,
                    SampleRate = 44100,
                    Channels = 2,
                    BitsPerSample = 16
                }
            });

            // Create Transcoder
            using (var transcoder = new Transcoder())
            {
                transcoder.Inputs.Add(inSocket);
                transcoder.Outputs.Add(outSocket);

                if (transcoder.Open())
                {
                    DecodeAACStream(transcoder);

                    transcoder.Close();
                }
            }
        }

        static void DecodeAACStream()
        {
            Library.Initialize();

            DecodeAACStream("Hydrate-Kenny_Beltrey.aac");

            Library.Shutdown();
        }

        static void Main(string[] args)
        {
            DecodeAACStream();
        }
    }
}
```

## How to run

Follow the steps to [create a C# console application in Visual Studio](../getting-started-windows/create-a-c-sharp-console-app-in-visual-studio) but in `Program.cs` use the code from this article. 

Copy the `Hydrate-Kenny_Beltrey.aac` file from the assets archive to `bin/x64/Debug/net8.0` under the project's directory.

Run the application in Visual Studio.
