# Performance Evaluation (ongoing)
[Primary Performance Affectors](#primary-performance-affectors)

[Other Metrics and Considerations](#other-metrics-and-considerations)

# Primary Performance Affectors

Knowing the Feature-Based Delay Line (FBDL) architecture, intuitively we think there are a 3 main affectors of performance: (1) [FFT / analysis](#fft-/-analysis), (2) [Targeting function](#targeting-function), and (3) audio processing overhead. This intuition is backed by measurements as well.

## FFT / Analysis
This is shown to typically one of the largest affector of performance for the FBDL.

This is demonstrated below, where we see the CPU Load with different FFT overlap settings, where the different overlap values represent the number of FFTs overlapped, where the overlap percentage is (FFTOverlap - 1)/(FFTOverlap). The default FFT size of 2048 was used.

![DAW CPU Load Metric vs  FFTOverlap](https://github.com/delaycattemp/delaycattemp/assets/105883026/e9eaac52-0c5a-4f2f-9ab2-205a0977745b)

This shows the significant impact of the FFT / analysis on the performance of the FBDL.

## Targeting Function
The targeting function (segment selection) also can have significant impact on the performance of the FBDL. Roughly speaking there are 2 factors to consider here: (1) number / frequency of queries to the targeting function and (2) number of comparisons in the targeting function (number of segments in the delay line check for selection viability). For each query to the targeting function, all comparisons must be made, meaning these factors are multiplicative.

The main parameters which determine these factors are [Number of Segments](#number-of-segments) read concurrently, [Segment Size](#segment-size), and [Delay Line Buffer Size](#delay-line-buffer-size).

### Number of Segments
The Number of Segments read concurrently increases the number of queries to the targeting function, as more segments are being selected. Default settings for other parameters are used.

The graphs below shows the CPU Load with increasing Number of Segments read concurrently, and the execution time of the audio ProcessBlock call (ms) with different buffer sizes and increasing Number of Segments read concurrently.

![CPU Load vs  # Segments](https://github.com/delaycattemp/delaycattemp/assets/105883026/82d2583f-28ab-4060-992a-2bbd2dcd68e7)

![PluginDoctorNumSegmentsPlots](https://github.com/delaycattemp/delaycattemp/assets/105883026/53aa1018-0bb3-4e9c-ba27-ca4adc8a399d)

As we can see, this increases CPU load at a linear rate.

### Segment Size
The Segment Size increases the number of queries to the targeting function because, since each segment is shorter, new segments must be selected with higher frequency. Segment Size also increases the number of comparisons in the targeting function, as the the delay buffer is segmented more finely.

### Delay Line Buffer Size
The Delay Line Buffer Size increases the number of comparisons in the targeting function, as it increases the number of segments (together with the Segment Size).

The graph below shows the CPU Load with different Delay Line Buffer Sizes (seconds). A plot with default segment size of 0.25 seconds (11025) and a plot with minimum segment size (1024 with the default FFT settings of 2048, 50% overlap) are displayed. Default settings for other parameters are used.

![CPU Load (Segment Size = 11025) and CPU Load (Segment Size = 1024)](https://github.com/delaycattemp/delaycattemp/assets/105883026/1635e056-4c81-4d88-b156-a650800dcd8a)

Interestingly, the buffer size has no visible impact when the default segment size of 0.25s / 11025 samples is used. However, with the smaller segment size of 1024 samples, there is visible affect. This is because factors that affect the number of queries to the targeting function and comparisons in the targeting function are multiplicative, meaning the affect of the larger number of comparisons introduced by the increased buffer size is much more noticeable with more frequent queries to the targeting function.

# Other Metrics and Considerations

## Delay Time
Like a traditional delay line, Delay Time has no effect on performance.

This is illustrated in the graph below.

![CPU Load vs  DelayTime](https://github.com/delaycattemp/delaycattemp/assets/105883026/3719ddaa-e76c-427f-a73a-d2c4fa6d4e74)
