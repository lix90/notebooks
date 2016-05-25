---
title: some tips regarding ica & filtering
author: lix
date: 2016-05-25
tags: [ica, filtering]
---

# ICA tricks

> Stephan Debener's solution: **apply 1-Hz high-pass on the data, run ICA, copy the weight matrix to the 0.1-Hz high-passed data.**

> Makoto, using **ICA + mutual information reduction** to evaluate the goodness of preprocessing, but this is kind of ICA-centric view and may not be acceptable for others.

# Too much drift with just 0.1hz high-pass filtering?

> Dear Kevin, Makoto and list,

> I fully agree with Makoto's comments. Further, I would like to emphasize that **there cannot be a general recommendation** or answer to this question (how to filter ERPs). The answer to this question always depends on the data, **the data quality** and **signal-to-noise ratio (SNR)**, **the noise characteristics**, **the paradigm**, **the parameter(s)** you want to estimate from the data, and other variables.

> I would like to argue for two criteria:

> * Filters should be applied **to improve the SNR (reduce the noise)** in order to be able to estimate a particular parameter (that is, if there is no noise there is no need to filter!).
> * **Applied filters must not introduce distortions or artifacts systematically biasing your estimated parameter.**

> Two examples:

* With high frequency noise you might need a lowpass filter to get a valid estimate of component peak latency. However, a lowpass filter might also lead to a systematic underestimation of component onset latency (due to smoothing; VanRullen, 2011). To estimate time window means you usually do *not* need a lowpass filter.
* Many ERP components are not of oscillatory nature but rather introduce a strong DC and low frequency component (in particular P3 and other late/slow components). Highpass filtering will necessarily spread/smear this activity and systematically bias the amplitude of preceding (we use non-causal filters!) and subsequent components. In a paradigm with a P3 in one but not in another condition (e.g. active oddball) the peak/mean amplitude of the preceding N1/N2 might be systematically biased between conditions after highpass filtering (e.g. with a higher 1 Hz cutoff while the effects might be negligible with a 0.1 Hz cutoff).

My recommendation is to systematically explore the effects a filter has on your data. One approach is to inspect and analyze the signal that was removed from the data by filtering, i.e. the difference between filtered and unfiltered data. Another important approach is to apply filters with different parameters and to compare the results (what you already did) and the effects on your parameter estimate. You can also filter test signals modeling your data and explore the observed filter effects/artifacts/distortions.

For your particular question you might want to read the already mentioned Acunzo paper and the recent paper by Tanner and colleagues (2015, Psychophysiology, DOI: 10.1111/psyp.12437). The point is whether your observed „drift" is really noise or evoked activity (I would rather guess that it is the latter) and whether removing it by filtering would bias your estimated parameter (following your description I would guess yes but this is really guessing without seeing the averaged data and knowing the dependent variable).

Hope this helps! Best,
Andreas

# how to select channels for back-projection of ERP?

Makoto is correct that the term 'back projection' of an IC to the scalp can be viewed also (and more fundamentally)  as a forward projection from cortical source to the scalp sensors.

I blieve the only EEG phenomena of interest are the source data activities (e.g., IC activations) -- the source projections to the scalp channels (whose sum we record with scalp electrodes) are essentially epiphenomena (more or less arbitrary source sums and differences, in themselves of no fundamental interest).

Now, the most correct unit of measurement for cortical sources is source current density per cortical area (for example, per mm^2).  However, to convert scalp measurements into that unit one needs a high-resolution electrical forward head model (sources to scalp electrodes) and an ideal inverse model (source scalp projection map to cortical source patch).  The former must include accurate values for tissue conductivities within the individual subject, not something we currently have access to (though see below).

Therefore, the most reliable unit of measurement we have for the activity of an identified source in scalp-recorded data is  'RMS uV' -- the root mean-square average projection from the source to the scalp. Even this has ambiguity -- To the whole scalp, even places where we did not place electrodes??   A simple alternative is to measure IC activity (unaveraged or averaged) as the RMS (mean) projection of the source to the actual scalp electrodes:

e.g., If IC is the index of an IC in an EEG data structure.

```{matlab}
RMS_uv(IC) = EEG.icaact(IC) / RMS(EEG.icawinv(IC));  % divide the IC activation by
% the RMS value of its
```

Unfortunately, the function `RMS()` I used in the meta-code above is not a Matlab function (by that name)... I am away from a Matlab installation at present and forget the actual function call.

It may be that this normalization `(making RMS(EEG.icawinv) = ones())` is already performed in the default output of runica / binica / AMICA - To be safe, however, run the meta-code above explicitly or test the equality above ...

All units of measurement for IC activations (activities) should be labelled 'RMS uV/chan' -- I will work with Arno and Ramon to see that this is corrected in the current functions.

Note: This removes the problem, 'Which channel projection is the best one to use to measure the contribution of a source to the scalp (ERP or raw) data?'

Scott Makeig

p.s. To see which ICs contribute most strongly, learn to use and interpret envtopo() -- called in the EEGLAB GUI by 'Plot > Component ERPs > With Scalp Maps'.  Makoto has recently supervised building of a plug-in to perform this function at the STUDY level (here the question is, 'Which IC clusters contribute most strongly to the grand mean ERP or ERP difference?')

p.p.s  Zeynep Akalin Acar and I have developed a new method for estimating skull conductivity from the EEG data (given a subject MR head image allowing development of an individual electrical forward head model). Our report on this should be accepted soon - more details then or at the upcoming EEGLAB workshop in UK...

---

ICs whose epoch-averaged activity time courses are ~orthogonal to some scalp channel ERP may likely have projections with opposite polarities to channel subsets (e.g., have scalp maps with both red and blue projection weights).

Remember that the sum of the IC projections to a channel is the channel value -- either in the continuous data or in data averages.  There was an EEGLAB function (chantopo() ?) I wrote to show the largest IC projections to a given channel epoch. It did not get into the EEGLAB GUI, but is likely still in the EEGLAB misc_funcs folder...

Scott Makeig

# STUDY Bootstrap statistical analysis

I assume you use statcond(data, 'method', 'bootstrap').

The bootstrap process is done by surrogdistrib(). It randomizes the indices for conditions to construct surrogate data (i.e. null hypothesis that the difference of the labels has no effect on data). Then it jumps to `ttest_cell()` to run a standard t-test to test on both real data difference and surrogate data difference separately to obtain t-values. Finally it runs `stat_surrogate_pvals()` using these data to determine the significant data point/ pixel/ voxel in a non-parametric way (i.e. counting from the end to determine X-percentile).

Makoto

# Adaptive Mixture ICA (AMICA)

AMICA被证明为已有ICA算法中最有效的算法

# ICA前基线校正（归一化）

- IC reliability for these data could be improved dramatically by removing the mean of each epoch instead of the mean of the 100 ms prestimulus baseline (a common EEG/MEG preprocessing step) before applying ICA.

- removing the mean of each epoch appears to have enhanced the strength of the latent independent components so that they could be extracted from less data. Presumably, preprocessing the data so as to enhance the ability of ICA to find latent components increases the accuracy of IC activations and scalp topographies as well.

- Removing the mean of each epoch acts as a leaky high-pass filter, zeroing the DC component and dampening low frequency components.

- Using only a small window to normalize the voltage of each epoch may enhance that variability and mask the latent sources ICA is sensitive to. (Groppe, D. M. et al. 2009)

# IC_MARC 独立成分分类算法

将ICA得到的独立成分分为neural和五种nonneural成分（eye blinks, heartbeat, lateral eye movements, muscle, and mixed neural and artifactual activity）
(Frølich, L. et al. 2015)
