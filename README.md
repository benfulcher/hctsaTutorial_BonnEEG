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

### Initializing an _hctsa_ computation (`TS_Init`)

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
Have a peek inside at these tables and matrices to get a better sense of how these _hctsa_ files are structured :eyes:.

### Computing features (`TS_Compute`)

So all we need now is to run all of these computations specified in our initialized `HCTSA.mat` file.
This is a big calculation, but you can test it by running, say, `TS_Compute(false,1,1:50)`, which will load the data from `HCTSA.mat`, compute the first time series and the first 50 operations, and save the results back to `HCTSA.mat` (the `false` flag says not to use parallel processing across cores; you can switch this `true` if you want to try that).

Because of the size of the dataset, we distributed different sets of time series across different nodes of a computing cluster using some simple shell scripts contained in [this repository](https://github.com/benfulcher/distributed_hctsa).
The result is a fully computed `HCTSA.mat` file (initialized and computed using _hctsa_ v1.03), which you can cheat and download from [cloudstor](https://cloudstor.aarnet.edu.au/plus/s/PWUIAdIeHhtkxsF).
If you download this file as `HCTSA.mat` to this tutorial directory, you will have skipped the hefty computations and can get on with the fun stuff.

#### Aside, _catch22_
For your own dataset, it's often a good idea to get a full method pieced together to test the usefulness of a feature-based analysis, but the computation time can be a barrier.
[_catch22_](https://github.com/chlubba/catch22) is a C-coded reduced set of 22 features, which (after installing) can be run very quickly from within Matlab run for a given dataset as, for example:

```matlab
TS_Init('INP_Bonn_EEG.mat','INP_mops_catch22.txt','INP_ops_catch22,txt');
TS_Compute(false);
```

Because computation is near-instant, it can be a good first step to work through an analysis using _catch22_ features.
The same analysis can then be repeated with the full power of the comprehensive _hctsa_ feature library if required.

### Checkin it out (`TS_InspectQuality`)

The first thing you do when you have a computed `HCTSA.mat` file on your hands is to check how the computation went.

This can be done as:

```matlab
TS_InspectQuality()
```

which by default reads in from `HCTSA.mat`, lists out all the features that had any special-valued outputs across the dataset, and plots a summary of what type of special-valued outputs they were.

In this case, we have 494 (/7702) features that produced some special-valued outputs (and they are listed to the command-line for reference):

![](img/TS_InspectQuality.png)

For example, we can see that some features gave `NaN` outputs for most of the dataset, and from the command-line list we see that these include features that capture properties of data distributions for positive-only data (returning a `NaN` because our EEG data are not positive-only).

So next we're going to want to clean up these types of poor-performing features.

## Normalizing and visualizing

### Filtering and normalizing (`TS_Normalize`)

We may next want to take the diverse set of >7000 features and:
1. Remove bad-performing features
2. Normalize all features to be on a similar scale

The application in mind will guide which settings to use: for example when performing classification we may not tolerate any missing values in our data at all, and some classifiers will be more or less sensitive to vast differences in scale between features (e.g., one features might be a _p_-value that varies from 0–1, while another might vary up to 10^8).

We can do both of these steps by running:
```matlab
TS_Normalize()
```
which by default filters on `[0.70,1]` (require 70%-good time series, and 100%-good features) and normalizes 'good' features using a `'mixedSigmoid'`.

This saves the new data to a new file: `HCTSA_N.mat`.
We can tell other _hctsa_ functions to load data from this file by using the `'norm'` shorthand.

### Setting up class labeling for classification (`TS_LabelGroups`)

Here we're solving a classification problem using the labels assigned to the data matrix.
We can tell _hctsa_ what labels (from the `Keywords` assigned to `TimeSeries`) we're interested in classifying.
We can run it to get a default labeling, which is assigned to the `Group` column of the `TimeSeries` table:

```matlab
TS_LabelGroups('norm')
```

In this case it can automatically assign the five groups (cf. `TS_WhatKeywords()`), because each time series only has a single keyword attached to it.

We could also focus on a two-class problem, for example, as:
```matlab
TS_LabelGroups('norm',{'eyesOpen','seizure'})
```

And we could be explicit in setting up the five-class problem, as:
```matlab
TS_LabelGroups('norm',{'eyesOpen','eyesClosed','epileptogenic','hippocampus','seizure'})
```

This stored group information can now be leveraged by other _hctsa_ plotting and analysis tools.

### Checking out the data (`TS_PlotTimeSeries`)

Now that the data are labelled, functions like `TS_PlotTimeSeries` will use this information automatically, to, for example, plot a fixed number of examples from each class:
```matlab
TS_PlotTimeSeries('norm')
```

![](img/TS_PlotTimeSeries.png)

### Visualizing the data matrix (`TS_PlotDataMatrix`)

Now that all features are on a similar scale, we can try plotting it using `TS_PlotDataMatrix('norm')`.
This plots time series as rows and features as columns, with the result of evaluating each feature on each time series shown as color, from low (blue) to high (red).

Even though features are not ordered in any particular way, we can see some structure by eye, and can zoom in to see individual feature values (and some time-series segments of each row).

![](img/TS_PlotDataMatrix_norm.png)

We can reorder features (and time series) to put similar rows and columns close to each other (and save this information back to `HCTSA_N.mat`):

```matlab
TS_Cluster()
```

And now we can plot the exact same matrix, but now we can visually see much more structure:

![](img/TS_PlotDataMatrix_cl.png)

### Visualizing the dataset in a low-dimensional feature space (`TS_PlotLowDim`)

```matlab
TS_PlotLowDim('norm','pca')
TS_PlotLowDim('norm','tsne')
```

You can see how your dataset appears when projected into a two-dimensional embedding space of the full feature space:

| PCA | _t_-SNE |
| :::: | :::: |
| ![](img/TS_PlotLowDim_PCA.png) | ![](img/TS_PlotLowDim_tSNE.png) |

It will also tell you how well each individual component classifies the class labels, and an overall classification rate in the full feature space.
Speaking of feature-based time-series classification...

## Classifying time series

### Setting the parameters (`GiveMeDefaultClassificationParams`)

When training and evaluating a classifier, there are many decisions to make about the classification algorithm (e.g., decision tree, SVM, neural network, ...), how to train and evaluate performance when there is class imbalance, and how to properly evaluate on unseen data (e.g., using cross-validation).
We can set the default parameters for classification in `GiveMeDefaultClassificationParams`, which outputs a `cfnParams` structure.
This can be modified and passed into other functions, or you can modify this function to set more appropriate defaults.

You can check the defaults (applied to the labeling in `HCTSA_N.mat`) as:
```matlab
cfnParams = GiveMeDefaultClassificationParams('norm');
TellMeAboutClassification(cfnParams)
```

### Running a feature-based classification (`TS_Classify`)
