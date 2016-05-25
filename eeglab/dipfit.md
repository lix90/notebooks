---
title: dipfit
author: lix
date: 2016-05-25
---

# Scalp topos represents normalized RMS amplitude of the IC.

Q：什么是等效偶极子分析时所提到的残差方法？残差方法意味着什么？
A：偶极子匹配中，残差方差（RV）指，由所估计到的偶极子“投射”回去所得到的地形图分布的没能解释的误差。如果RV很大，表示偶极子不能较好地解释地形图。

Q：连续数据和分段数据的ICA效果有没有差异？数据之间的突然的变化（abrupt transitions）是否影响ICA的结果？
A：ICA并不受数据连续与分段的影响，ICA关注的是通道间在某个时间点上的关系，而不受时间点之间关系的影响。（No, because ICA shuffles all the datapoints for every iteration. Are you surprised? It means that ICA does not know extension of time, it only cares inter-channel relation for each time points. ）

Q：About boundary marks
A：If you have boundary events, eeglab filter process will not go across it but uses windowing function to fade out/fade in before and after the boundary. Epoching will reject any epoch that contain boundary, etc.

Q: Now I´m looking for a tool that helps me to find brain regions (preferably, with corresponding functionality) based on talairach coordinates of the cluster centroids.
A: If you use Measure Projection Toolbox, it will provide Brodmann Area and AAL-defined anatomical region labels. See the MPT manual.

Hey Jianwei,

ICA doesn't care whether or not the provided data is continuous or segmented or even scrambled along the time dimension, so long as the data across channels for each time point is left intact. I would recommend against scrambling your data in the time domain though, as your computed ICA time-series will be quite difficult to interpret . It will depend somewhat on your preprocessing steps leading up to the point where you are ready to compute ICA activations. I believe it is commonly accepted that the best component activations come from reasonably stationary data (a 1Hz highpass filter usually recommended to achieve this) and reasonably clean data (reject stretches of data that contain rare and disruptive artifacts like head movements that contaminate the signal of many channels).

I filter my data as one continuous time-series just to minimize the amount of artifacts contributed by filtering edges of a time-series. However, for my cursory data cleaning following filtering, I find it easiest and most convenient to epoch the data and reject epochs instead of rejecting continuous stretches of data. That way it is easier to keep track of data that have been excluded (you only need to retain one epoch number instead of the index of every rejected sample or the start/stop index of every rejected stretch of continuous data) and you kind of have a more
appropriately sized standard unit for the amount of data excluded (# epochs rejected instead of # samples rejected).

With all that being said, that is simply my preference. If you find it easier/more convenient to work with the continuous data or find something unsettling in arbitrarily defining epochs (since you have no events) then by all means stick with the continuous data.

Hope that helps,
Michael
