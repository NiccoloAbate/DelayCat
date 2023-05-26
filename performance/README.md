# Performance Evaluation (ongoing)
[Primary Performance Affectors](#primary-performance-affectors)

[Other Metrics and Considerations](#other-metrics-and-considerations)

# Primary Performance Affectors

Knowing the Feature-Based Delay Line (FBDL) architecture, intuitively we think there are a 3 main affectors of performance: (1) [FFT / analysis](#fft-/-analysis), (2) [Targeting function](#targeting-function), and (3) [audio processing overhead](#audio-processing-overhead). This intuition is backed by measurements as well.

## FFT / Analysis
This is shown to typically one of the largest affector of performance for the FBDL.

This is demonstrated below, where we see the CPU Load with different FFT overlap settings, where the different overlap values represent the number of FFTs overlapped, where the overlap percentage is (FFTOverlap - 1)/(FFTOverlap). The default FFT size of 2048 was used.

![CPU Load vs  FFTOverlap](https://github.com/delaycattemp/delaycattemp/assets/105883026/bdb990e2-f690-435e-b3de-9e84af16c533)

This shows the significant impact of the FFT / analysis on the performance of the FBDL.

## Targeting Function
The targeting function (segment selection) also can have significant impact on the performance of the FBDL. Roughly speaking there are 2 factors to consider here: (1) number / frequency of queries to the targeting function and (2) number of comparisons in the targeting function (number of segments in the delay line check for selection viability). For each query to the targeting function, all comparisons must be made, meaning these factors are multiplicative.

The main parameters which determine these factors are [Number of Segments](#number-of-segments) read concurrently, [Segment Size](#segment-size), [Delay Line Buffer Size](#delay-line-buffer-size), and [Segment Playback Pitch](#segment-playback-pitch).

### Number of Segments
The Number of Segments read concurrently increases the number of queries to the targeting function, as more segments are being selected.

The graphs below shows the CPU Load with increasing Number of Segments read concurrently, and the execution time of the audio ProcessBlock call (ms) with different buffer sizes and increasing Number of Segments read concurrently. Default settings for other parameters were used.

![CPU Load vs  # Segments](https://github.com/delaycattemp/delaycattemp/assets/105883026/1079ad29-2a4c-44cf-9d5f-4f36447830eb)

![PluginDoctorNumSegmentsPlots](https://github.com/delaycattemp/delaycattemp/assets/105883026/53aa1018-0bb3-4e9c-ba27-ca4adc8a399d)

As we can see, this increases CPU load at a linear rate.

It should be noted that the Number of Segments also affects performance outside of targeting function related processing, because each segment read concurrently is essentially an independent tap into the delay line and thus needs its own copy of many of the signal processing computations.

### Segment Size
The Segment Size increases the number of queries to the targeting function because, since each segment is shorter, new segments must be selected with higher frequency. Segment Size also increases the number of comparisons in the targeting function, as the the delay buffer is segmented more finely.

The graph below shows the CPU load with different segment sizes. Default settings for other parameters were used.

![CPU Load vs  Segment Size](https://github.com/delaycattemp/delaycattemp/assets/105883026/ceaf8b51-5fa0-4b58-80a3-8b981ae3c1dc)

We can see some performance effect as the segment size shrinks, though it isn't always super visible at this level of granularity.

### Delay Line Buffer Size
The Delay Line Buffer Size increases the number of comparisons in the targeting function, as it increases the number of segments (together with the Segment Size).

The graph below shows the CPU Load with different Delay Line Buffer Sizes (seconds). A plot with default segment size of 0.25 seconds (11025 samples) and a plot with minimum segment size (1024 with the default FFT settings of 2048, 50% overlap) are displayed. Default settings for other parameters were used.

![CPU Load vs  Buffer Size](https://github.com/delaycattemp/delaycattemp/assets/105883026/71a77b2a-31a5-4e0f-9348-34a646333917)

Interestingly, the Delay Line Buffer Bize has no visible impact at this level of granularity when the default Segment Size of 0.25s / 11025 samples is used. However, with the smaller Segment Size of 1024 samples, there is visible affect. This is because factors that affect the number of queries to the targeting function and comparisons in the targeting function are multiplicative, meaning the affect of the larger number of comparisons introduced by the increased Delay Line Buffer Size is much more noticeable with more frequent queries to the targeting function.

### Segment Playback Pitch
The Segment Playback Pitch affects the number of queries to the targeting function because, if segments are being read faster or slower, the rate that new segments must be selected changes.

The graph below shows the CPU Load with different Segment Playback Pitches (semitones). A plot with default Segment Size of 0.25 seconds (11025 samples) and a plot with minimum Segment Size (1024 with the default FFT settings of 2048, 50% overlap) are displayed. Default settings for other parameters were used.

![CPU Load vs  Segment Playback Pitch](https://github.com/delaycattemp/delaycattemp/assets/105883026/0aa56e67-f1a2-4fc3-86f2-26d048c901e6)

Once again, like the Delay Line Buffer Size, the Segment Playback Pitch has no visible impact at this level of granularity when the default segment size of 0.25s / 11025 samples is used, but can be seen with smaller Segment Size of 1024 samples. This same logic applies in this case.

## Audio Processing Overhead
Some of the plugin performance, doesn't break down as easily, and simply falls into the gategory of general audio processing overhead, which includes all the audio processing operations which are necessitated for the plugins functioning.

We have no rigorous measure of this quantity currently, but an informal intuition of this can be gleaned by comparing all the other measurements.

# Other Metrics and Considerations
Other metrics and notable performance considerations for the FBDL include [Delay Time](#delay-time).

## Delay Time
Like a traditional delay line, Delay Time has no effect on performance.

This is illustrated in the graph below.

![CPU Load vs  DelayTime](https://github.com/delaycattemp/delaycattemp/assets/105883026/3719ddaa-e76c-427f-a73a-d2c4fa6d4e74)
