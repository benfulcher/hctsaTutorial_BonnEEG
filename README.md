This is a tutorial on _hctsa_ time-series classification using the [Bonn University EEG dataset](http://epileptologie-bonn.de/cms/front_content.php?idcat=193&lang=3).

The paper describing this dataset is:

Andrzejak, Ralph G., et al. (2001)
"Indications of nonlinear deterministic and finite-dimensional structures in time series of brain electrical activity: Dependence on recording region and brain state."
_Physical Review E_ __64__(6): 061907.
[Link](https://doi.org/10.1103/PhysRevE.64.061907).

The dataset contains 100 examples each of five classes of EEG data: `eyesOpen`, `eyesClosed`, `epileptogenic`, `hippocampus`, `seizure`.

We're going to try to understand the patterns in this dataset, and use highly comparative time-series analysis to classify the different classes.

### Download the HCTSA datafile
You can download the results of _hctsa_ (v1.03) computation on this dataset from [figshare](), or by running `downloadBonnEEGData`.
This should download `HCTSA.mat` to the current directory.

###
