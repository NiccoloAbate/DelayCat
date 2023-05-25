# Performance Evaluation (ongoing)

Knowing the Feature-Based Delay Line (FBDL) architecture, intuitively we think there are a 3 main affectors of performance: (1) FFT / analysis, (2) Targeting function, and (3) audio processing overhead. This intuition is backed by measurements as well.

## FFT / Analysis
This is shown to typically one of the largest affector of performance for the FBDL.

![DAW CPU Load Metric vs  FFTOverlap](https://github.com/delaycattemp/delaycattemp/assets/105883026/e9eaac52-0c5a-4f2f-9ab2-205a0977745b)

