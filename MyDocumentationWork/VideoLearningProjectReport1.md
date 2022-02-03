# Investigating the Existing Video Learning Project With NeocortexAPI:

## Introduction:
The Project "Video Learning With HTM CLA" is built upon the concept of Brain Theory that how a specific portion of our brain which is neocortex can learn multiple sequences of videos and can successfully predict the next frame from a given input frame of a video. The whole project is built on the basis of [NeoCortex Api](https://github.com/ddobric/neocortexapi). It uses Hierarchical Themporal Memory with Cortical Learning Algorithm as a learning mechanism. As we know HTM systems work with the data which is nothing but streams of zeros and ones(Binary Data). So the video is converted into the sequences of bit arrary and then it has been pushed to the HTM model to learn significant patterns from the video. Afterwards, when the model is ready for test phase, an arbitrary image is passed through the network and the model then tries to recreate the video by predicting the proceeding frame from the input frame(arbitrary image).

## How to Run the Existing Project:

The whole project has been programmed with C# and Microsoft Visual Studio is used as IDE. Two Experiment have been performed in this project which are basically two functions named Run1() and Run2() in the [VideoLearning.cs](https://github.com/ddobric/neocortexapi/blob/SequenceLearning_ToanTruong/Project12_HTMCLAVideoLearning/HTMVideoLearning/HTMVideoLearning/VideoLearning.cs). Though these functions are designed with the same concept of HTM model, they have some discrepancies if we visualize them in terms of how they are being programmed. We can toggle these experiment in the [Program.cs](https://github.com/ddobric/neocortexapi/blob/SequenceLearning_ToanTruong/Project12_HTMCLAVideoLearning/HTMVideoLearning/HTMVideoLearning/Program.cs). Function Run1() and Run2() has individual training, learning+predicting and testing phases. The following steps should be carried out to successfully build and run the project.
1. Run the main program [Program.cs](https://github.com/ddobric/neocortexapi/blob/SequenceLearning_ToanTruong/Project12_HTMCLAVideoLearning/HTMVideoLearning/HTMVideoLearning/Program.cs) by selecting only one function at a time (either Run1() or Run2()).
2. Then open the Command Prompt and put the directory of the executable file (HTMVideoLearning.exe) in the cmd. Run the executable file in the command prompt. Then the program will ask to insert or drag the folder in the cmd that contains the training files(HTMVideoLearning\VideoLibrary\SmallTrainingSet). After that the HTM Model will start running and start printing relevant information in the Command Prompt.
3. The similar process must be followed if we want to run the project with experiment 2 (Run2()).

## Current HTM Configuration Checked:

The following code segment depicts the configuration of the Hierarchical Themporal Memory Model that have been designed to learn video. Run1() and Run2() uses same configuration. I have tested some combination of these parameters especially the PermanenceDecrement and the PermanenceIncrement parameter. It takes significantly long time to train the model with the given input set(HTMVideoLearning\VideoLibrary\SmallTrainingSet). As these two parameters are responsible for manupulating the overlap connections in the SP column, these parameters changes the accuracy of the model. I have also analyzed the [SequenceLearningExperiment]https://github.com/ddobric/neocortexapi/blob/master/source/SequenceLearningExperiment/Program.cs and compare it with our project. Though we are learning videos, I found that above two parameters are essential when we want to learn sequences of numbers. Punishing of segments has not been initialized here to drop the connection of SP given a particular input space.
The following code segment depicts the configuration of the Hierarchical Themporal Memory Model that have been designed to learn video. Run1() and Run2() uses same configuration. I have tested some combination of these parameters especially the PermanenceDecrement and the PermanenceIncrement parameter. It takes significantly long time to train the model with the given input set(HTMVideoLearning\VideoLibrary\SmallTrainingSet). As these two parameters are responsible for manupulating the overlap connections in the SP column, these parameters changes the accuracy of the model. I have also analyzed the SequenceLearningExperiment [Program.cs]https://github.com/ddobric/neocortexapi/blob/master/source/SequenceLearningExperiment/Program.cs and compare it with our project. Though we are learning videos, I found that above two parameters are essential when we want to learn sequences of numbers. Punishing of segments has not been initialized here to drop the connection of SP given a particular input space.

```csharp
private static HtmConfig GetHTM(int[] inputBits, int[] numColumns)
{
    HtmConfig htm = new(inputBits, numColumns)
    {
        Random = new ThreadSafeRandom(42),
        CellsPerColumn = 30,
        GlobalInhibition = true,
        //LocalAreaDensity = -1,
        NumActiveColumnsPerInhArea = 0.02 * numColumns[0],
        PotentialRadius = (int)(0.15 * inputBits[0]),
        //InhibitionRadius = 15,
        MaxBoost = 10.0,
        //DutyCyclePeriod = 25,
        //MinPctOverlapDutyCycles = 0.75,
        MaxSynapsesPerSegment = (int)(0.02 * numColumns[0]),
        //ActivationThreshold = 15,
        //ConnectedPermanence = 0.5,
        // Learning is slower than forgetting in this case.
        //PermanenceDecrement = 0.15,
        //PermanenceIncrement = 0.15,
        // Used by punishing of segments.
    };
    return htm;
}
```
## Encoding and Reading of Videos:

For the purpose of encoding and reading video data from the SmallTrainingSet a library has been created. This library consists of three subclasses named **VideoSet**, **NVideo** and **NFrame**. In the following section the detail analysis of these classes has been added.

- [**1. VideoSet**](https://github.com/ddobric/neocortexapi/blob/SequenceLearning_ToanTruong/Project12_HTMCLAVideoLearning/HTMVideoLearning/VideoLibrary/VideoSet.cs):
The following code section in the VideoSet class reads video from the video folder path and the label has been added in accordance with the folder name.

```csharp
public VideoSet(string videoSetPath, ColorMode colorMode, int frameWidth, int frameHeight, double frameRate = 0)
        {
            nVideoList = new List<NVideo>();
            Name = new List<string>();
            // Set the label of the video collection as the name of the folder that contains it 
            this.VideoSetLabel = Path.GetFileNameWithoutExtension(videoSetPath);

            // Read videos from the video folder path 
            nVideoList = ReadVideos(videoSetPath, colorMode, frameWidth, frameHeight, frameRate);
        }
 ```
This class also defines a function named CreateConvertedVideos which is used to create a video with defined frameRate, frameWidth and frameHeight and put that video in a folder. Afterwards, a folder has been created which consists nothing but the converted frames in png format. These images (frames) can be used for the prediction purpose.

```csharp
public void CreateConvertedVideos(string videoOutputDirectory)
        {
            foreach (NVideo nv in nVideoList)
            {
                string folderName = $"{videoOutputDirectory}" + @"\" + $"{nv.label}";
                if (!Directory.Exists(folderName))
                {
                    Directory.CreateDirectory(folderName);
                }
                NVideo.NFrameListToVideo(nv.nFrames, $"{folderName}" + @"\" + $"{nv.name}", (int)nv.frameRate, new Size(nv.frameWidth, nv.frameHeight), true);
                if (!Directory.Exists($"{folderName}" + @"\" + $"{nv.name}"))
                {
                    Directory.CreateDirectory($"{folderName}" + @"\" + $"{nv.name}");
                }
                for (int i = 0; i < nv.nFrames.Count; i += 1)
                {
                    nv.nFrames[i].SaveFrame($"{folderName}" + @"\" + $"{nv.name}" + @"\" + $"{nv.nFrames[i].FrameKey}.png");
                }
            }
        }
```

- [**2. NVideo**](https://github.com/ddobric/neocortexapi/blob/SequenceLearning_ToanTruong/Project12_HTMCLAVideoLearning/HTMVideoLearning/VideoLibrary/NVideo.cs):
This class has been defined to read a video in different frame rates, frame height , frame width and color modes. The following code segment will show the basic settings of the parameters to manupulate a video.

```csharp
// Video Parameter 
            int frameWidth = 18;
            int frameHeight = 18;
            ColorMode colorMode = ColorMode.BLACKWHITE;
            // frame rate of 10 or smaller is possible
            double frameRate = 10;
 ```
For the experimental purpose the frameWidth and frameHeight parameters are chaged to 20, 22 and 24 to test the various segments of the existing project. The NFrameListToVideo function is used to compress and write a video after the prediction part had been done which returns the calculated predicted frame and all other frames that come after this frame. The VideoWriter function in the following code segments is used to compress the video in the mp4 format. Also in the parameter,-1 which automatically defines a codec for the purpose compression. Here, fourcc = -1. fourcc means 4-character code of codec to compress the frames. For example, VideoWriter::fourcc('P','I','M','1') is a MPEG-1 codec, VideoWriter::fourcc('M','J','P','G') is a motion-jpeg codec etc.

```csharp
public static void NFrameListToVideo(List<NFrame> bitmapList, string videoOutputPath, int frameRate, Size dimension, bool isColor)
        {
            using (VideoWriter videoWriter = new($"{videoOutputPath}.mp4", -1, (int)frameRate, dimension, isColor))
            {
                foreach (NFrame frame in bitmapList)
                {
                    Bitmap tempBitmap = frame.IntArrayToBitmap(frame.EncodedBitArray);
                    videoWriter.Write(tempBitmap.ToMat());
                }
            }
        }
```

- [**NFrame**](https://github.com/ddobric/neocortexapi/blob/SequenceLearning_ToanTruong/Project12_HTMCLAVideoLearning/HTMVideoLearning/VideoLibrary/NFrame.cs):
In this class different color modes have been defined such as BLACKWHITE, BINARIZEDRGB and PURE. The resolution of the actual video has been reduced for the purpose of testing. With the help of this class each bitmap has been encoded to an int array and binary array by iterating through every pixel of the frame. The function can also convert a bit array into bitmap.The Frame key parameter in this class is used to index the frames which is defined by **Framkey = (label)\_(VideoName)\_(index)**. This frame key is used for the classification purpose in the [HTMClassifier](https://github.com/ddobric/neocortexapi/blob/master/source/NeoCortexApi/Classifiers/HtmClassifier.cs).








































