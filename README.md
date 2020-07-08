This is a tutorial on _hctsa_ time-series classification using the [Bonn University EEG dataset](http://epileptologie-bonn.de/cms/front_content.php?idcat=193&lang=3).

It will walk you through a basic analysis of the data, and a classification of the different labeled states from time-series features of the EEG.

The paper that introduced this dataset is:

Andrzejak, Ralph G., et al. (2001)
"Indications of nonlinear deterministic and finite-dimensional structures in time series of brain electrical activity: Dependence on recording region and brain state."
_Physical Review E_ __64__(6): 061907.
[Link](https://doi.org/10.1103/PhysRevE.64.061907).

The dataset contains 100 examples each of five classes of EEG data: `eyesOpen`, `eyesClosed`, `epileptogenic`, `hippocampus`, `seizure`.

We're going to try to understand the patterns in this dataset, and use highly comparative time-series analysis to classify the different classes.

You're going to need [hctsa](https://github.com/benfulcher/hctsa) installed.

## Computing

Let's first check out how we would set this dataset up for an _hctsa_ analysis.
The first step is setting up the time series; you can see the input file that I've set up for this dataset ([cloudstor](https://cloudstor.aarnet.edu.au/plus/s/6sRD6IPMJyZLNlN)).

I need to specify the data I want to calculate features for, and add any additional metadata (such as class labels).
_hctsa_ needs three variables:
* A 500 x 1 `labels` cell (each element gives a unique identifying name to each time series);
* A 500 x 1 `timeSeriesData` cell (each element contains the time-series data);
* A 500 x 1 `keywords` cell (each element contains a comma-delimited set of keywords--in this case I've just given each time series a single keyword corresponding to one of the five classes we want to classify).

So all of this is saved in our time-series input file, `INP_Bonn_EEG.mat`, and defines everything we need to know about the time series.
There are two files in _hctsa_ (`INP_ops.txt` and `INP_mops.txt`) that do a similar job for the time-series features (as `Operations`, what we call the individual features, and `MasterOperations`, the specific pieces of code and their inputs that output sets of features).

This default feature library is used by default when we initialize our dataset:
```matlab
TS_Init('INP_Bonn_EEG.mat')
```
Running this will initialize the _hctsa_ analysis with a `HCTSA.mat` file containing, including tables defining the time series (`TimeSeries`) and the features (`Operations`), as well as matrices that contain the results of applying every operation to every time series (`TS_DataMat`).

So all we need now is to run all of these computations.
This is a big calculation, but you can test it by running `TS_Compute(false,1,1:50)`, which will load the data from `HCTSA.mat`, compute the first time series and the first 50 operations, and save the results back to `HCTSA.mat` (the `false` flag says not to use parallel processing across cores; you can switch this `true` if you want to try that).

Because of the size of the dataset, we distributed different sets of time series across different nodes of a computing cluster using some simple shell scripts contained in [this repository](https://github.com/benfulcher/distributed_hctsa).
The result is a fully computed `HCTSA.mat` file (initialized and computed using _hctsa_ v1.03), which you can cheat and download from [cloudstor](https://cloudstor.aarnet.edu.au/plus/s/PWUIAdIeHhtkxsF).
If you download this file as `HCTSA.mat` to this tutorial directory, you will have skipped the hefty computations and can get on with the fun stuff.


### Checkin it out

The first thing you do when you have a computed `HCTSA.mat` file on your hands is to check how the computation went.
This can be done as
```matlab
TS_InspectQuality()
```
which by default reads in from `HCTSA.mat`, lists out all the features that had any special-valued outputs across the dataset, and plots a summary of what type of special-valued outputs they were.
In this case, we have 494(/7702) features that produced some special-valued outputs:

![](img/TS_InspectQuality.png)



## Normalizing and visualizing
