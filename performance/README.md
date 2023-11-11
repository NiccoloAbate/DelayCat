# Performance Evaluation
[Methodology](#methodology)

[Primary Performance Affectors](#primary-performance-affectors)

[Other Metrics and Considerations](#other-metrics-and-considerations)

[Room for Improvement](#room-for-improvement)

# Methodology

The primary metric used for evaluation is the CPU core load of the plugin under different configurations. The CPU core load is a value from 0 to 1 which is the amount of time used by the process block call / the amount of time allowed by the DAW before the audio must be finished processing. This metric is used to measure the change in performance caused by each important parameter.

A sample rate of 44100 and a audio block size of 256 was used, and the tests were run on an Intel i7-6799HQ processor.

# Primary Performance Affectors

Knowing the Feature-Based Delay Line (FBDL) architecture, intuitively we think there are a 3 main affectors of performance: (1) [FFT / analysis](#fft-analysis), (2) [Targeting function](#targeting-function), and (3) [audio processing overhead](#audio-processing-overhead). This intuition is backed by measurements as well.

## FFT Analysis
This is shown to typically one of the largest affector of performance for the FBDL.

This is demonstrated below, where we see the CPU Load with different FFT overlap settings, where the different overlap values represent the number of FFTs overlapped, where the overlap percentage is (FFT Overlap - 1)/(FFT Overlap). The default FFT size of 2048 was used.

![CPU Core Load vs  FFT Overlap](https://github.com/NiccoloAbate/DelayCat/assets/27022723/836de897-381a-4032-8518-e46eac1a424e)

This shows the significant impact of the FFT / analysis on the performance of the FBDL.

## Targeting Function
The targeting function (segment selection) also can have significant impact on the performance of the FBDL. Roughly speaking there are 2 factors to consider here: (1) number / frequency of queries to the targeting function and (2) number of comparisons in the targeting function (number of segments in the delay line check for selection viability). For each query to the targeting function, all comparisons must be made, meaning these factors are multiplicative.

The main parameters which determine these factors are [Number of Segments](#number-of-segments) read concurrently, [Segment Size](#segment-size), [Delay Line Buffer Size](#delay-line-buffer-size), and [Segment Playback Pitch](#segment-playback-pitch).

There is also a notible multiplicative [worst case scenario](#worst-case-scenario) for these factors.

### Number of Segments
The Number of Segments read concurrently increases the number of queries to the targeting function, as more segments are being selected.

The graph below shows the CPU Load with increasing Number of Segments read concurrently increasing Number of Segments read concurrently. Default settings for other parameters were used.

![CPU Core Load vs  # Segments](https://github.com/NiccoloAbate/DelayCat/assets/27022723/d9771682-f792-4c7e-9457-0cb6bea18b6d)

As we can see, this increases CPU load at a linear rate.

It should be noted that the Number of Segments also affects performance outside of targeting function related processing, because each segment read concurrently is essentially an independent tap into the delay line and thus needs its own copy of many of the signal processing computations.

### Segment Size
The Segment Size increases the number of queries to the targeting function because, since each segment is shorter, new segments must be selected with higher frequency. Segment Size also increases the number of comparisons in the targeting function, as the the delay buffer is segmented more finely.

The graph below shows the CPU load with different segment sizes. Default settings for other parameters were used.

![CPU Core Load vs  Segment Size](https://github.com/NiccoloAbate/DelayCat/assets/27022723/10b21c0d-a128-4082-840e-ee39b7a270a1)

We can see some performance effect as the segment size shrinks, though it isn't always super visible at this level of granularity.

### Delay Line Buffer Size
The Delay Line Buffer Size increases the number of comparisons in the targeting function, as it increases the number of segments (together with the Segment Size).

The graph below shows the CPU Load with different Delay Line Buffer Sizes (seconds). A plot with default segment size of 0.25 seconds (11025 samples) and a plot with minimum segment size (1024 with the default FFT settings of 2048, 50% overlap) are displayed. Default settings for other parameters were used.

![CPU Load vs  Buffer Size](https://github.com/NiccoloAbate/DelayCat/assets/27022723/0e06c098-e829-43df-8eee-24e62c0e87d2)

Interestingly, the Delay Line Buffer Bize has no visible impact at this level of granularity when the default Segment Size of 0.25s / 11025 samples is used. However, with the smaller Segment Size of 1024 samples, there is visible affect. This is because factors that affect the number of queries to the targeting function and comparisons in the targeting function are multiplicative, meaning the affect of the larger number of comparisons introduced by the increased Delay Line Buffer Size is much more noticeable with more frequent queries to the targeting function.

### Segment Playback Pitch
The Segment Playback Pitch affects the number of queries to the targeting function because, if segments are being read faster or slower, the rate that new segments must be selected changes.

The graph below shows the CPU Load with different Segment Playback Pitches (semitones). A plot with default Segment Size of 0.25 seconds (11025 samples) and a plot with minimum Segment Size (1024 with the default FFT settings of 2048, 50% overlap) are displayed. Default settings for other parameters were used.

![CPU Load vs  Segment Playback Pitch](https://github.com/NiccoloAbate/DelayCat/assets/27022723/c4d0f361-d3e3-453f-b2cc-dc75c7ad2e1b)

Once again, like the Delay Line Buffer Size, the Segment Playback Pitch has no visible impact at this level of granularity when the default segment size of 0.25s / 11025 samples is used, but can be seen with smaller Segment Size of 1024 samples. This same logic applies in this case.

### Worst Case Scenario
As we can see, these targeting functions affectors are multiplicative. With that in mind, when they are all combined at there extremes, it can ramp to a high worst case scenario.

The graph below shows the CPU load under the worst case scenario (Segment Size = 1024, Buffer Size = 64, Pitch = 24) and increasing number of segments.

![CPU Core Load Worst Case](https://github.com/NiccoloAbate/DelayCat/assets/27022723/fe08ceb1-639d-4f89-a18a-1f49f3617a42)

The CPU load in this worst case is quite extreme (note that a CPU core load value of 1.0 is the maximum value before the process block call doesn't finish in time, causing lagging and artifacting). This worst case scenario could be avoided by limitting the multiplicative use of these parameters or by speeding up / limitting the querries in the targeting function. Additionally, this worst case scenario is by no means a common case scenario.

## Audio Processing Overhead
Some of the plugin performance, doesn't break down as easily, and simply falls into the gategory of general audio processing overhead, which includes all the audio processing operations which are necessitated for the plugins functioning.

We have no rigorous measure of this quantity currently, but an informal intuition of this can be gleaned by comparing all the other measurements.

# Other Metrics and Considerations
Other metrics and notable performance considerations for the FBDL include [Delay Time](#delay-time).

## Delay Time
Like a traditional delay line, Delay Time has no effect on performance.

This is illustrated in the graph below.

![CPU Load vs  DelayTime](https://github.com/NiccoloAbate/DelayCat/assets/27022723/11e1b185-eb2a-4653-a793-d1c89a6d7aa4)

# Room for Improvement
Surely there are lots of clever ways under to hood to improve the performance of our implementation. That being said, there are a couple larger scale areas where improvement could be made, namely:
* Faster FFT Analysis
* Faster comparisons in targeting method / tricks to reduce the number of comparisons
