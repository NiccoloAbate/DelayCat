# Performance Evaluation (ongoing)

Knowing the Feature-Based Delay Line (FBDL) architecture, intuitively we think there are a 3 main affectors of performance: (1) FFT / analysis, (2) Targeting function, and (3) audio processing overhead. This intuition is backed by measurements as well.

## FFT / Analysis
This is shown to typically one of the largest affector of performance for the FBDL.

This is demonstrated below, where we see the CPU Load with different FFT overlap settings, where the different overlap values represent the number of FFTs overlapped, where the overlap percentage is (FFTOverlap - 1)/(FFTOverlap). The default FFT size of 2048 was used.

![DAW CPU Load Metric vs  FFTOverlap](https://github.com/delaycattemp/delaycattemp/assets/105883026/e9eaac52-0c5a-4f2f-9ab2-205a0977745b)

This shows the significant impact of the FFT / analysis on the performance of the FBDL.
