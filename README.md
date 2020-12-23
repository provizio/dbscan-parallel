## Overview

This repository contains the fastest Python package for parallel Euclidean DBSCAN clustering. The code automatically uses all available POSIX threads to speedup DBSCAN clustering. It stems from a paper presented in SIGMOD'20: [Theoretically Efficient and Practical Parallel DBSCAN](https://arxiv.org/abs/1912.06255).

It is much faster than the DBSCAN package in Sklearn on a single thread, and provides additional speedup via multi-threading. We provide some simple benchmark showing the speedup over Sklearn tested on a 4-core computer on a toy data set, and a visualization of the result.

<p float="left">
<img src="compare.png" alt="timing" width="300"/>
<img src="example.png" alt="example" width="300"/>
</p>

## Installation

The software is written using C++ and made available in Python 3.8+ using a Cython wrapper. It is tested, and should work directly on any fresh copy of Ubuntu 20.04. There are two ways to install it:
* Install it using PyPI: ``pip3 install --user dbscan``
OR
* Compile it yourself: First install dependencies ``pip3 install --user Cython numpy`` and ``sudo apt install libpython3-dev``. Navigate to ``dbscan-python/dbscan/``, and run the ''make'' script ``./make.sh``, The compilation will take a few minutes, and generate a ''.so'' library containing the ''DBSCAN'' module.

## Tutorial

An example API call:

```
from dbscan import DBSCAN
labels, core_samples_mask = DBSCAN(X, eps=0.3, min_samples=10)
```

##### Input

* ``X``: A 2-D Numpy array (``dtype=np.float64``) containing the input data points. The first dimension of ``X`` is the number of data points ``n``, and the second dimension is the data set dimensionality (the maximum supported dimensionality is 20).
* ``eps``: The epsilon parameter (default 0.5).
* ``min_samples``: The minPts parameter (default 5).

##### Output

* ``labels``: A length ``n`` Numpy array (``dtype=np.int32``) containing cluster IDs of the data points, in the same ordering as the input data. Noise points are given a pseudo-ID of ``-1``.
* ``core_samples_mask``: A length ``n`` Numpy array (``dtype=np.bool``) masking the core points, in the same ordering as the input data.

We provide a complete example below that generates a toy data set, computes the DBSCAN clustering, and visualizes the result as shown in the plot above. Before running the example, first install packages for generating the data set and visualizing the result ``pip3 install --user sklearn matplotlib``.

```
import numpy as np
from sklearn.datasets import make_blobs
from sklearn.preprocessing import StandardScaler

# #############################################################################
# Generate sample data
centers = [[1, 1], [-1, -1], [1, -1]]
X, labels_true = make_blobs(n_samples=750, centers=centers, cluster_std=0.4,
                            random_state=0)
X = StandardScaler().fit_transform(X)

# #############################################################################
# Compute DBSCAN
from dbscan import DBSCAN
labels, core_samples_mask = DBSCAN(X, eps=0.3, min_samples=10)

# #############################################################################
# Plot result
import matplotlib.pyplot as plt

n_clusters_ = len(set(labels)) - (1 if -1 in labels else 0)
n_noise_ = list(labels).count(-1)
unique_labels = set(labels)
colors = [plt.cm.Spectral(each)
          for each in np.linspace(0, 1, len(unique_labels))]

for k, col in zip(unique_labels, colors):
    if k == -1:
        # Black used for noise.
        col = [0, 0, 0, 1]
    class_member_mask = (labels == k)
    xy = X[class_member_mask & core_samples_mask]
    plt.plot(xy[:, 0], xy[:, 1], 'o', markerfacecolor=tuple(col),
             markeredgecolor='k', markersize=14)
    xy = X[class_member_mask & ~core_samples_mask]
    plt.plot(xy[:, 0], xy[:, 1], 'o', markerfacecolor=tuple(col),
             markeredgecolor='k', markersize=6)

plt.title('Estimated number of clusters: %d' % n_clusters_)
plt.show()
```

## Help and Support

Please feel free to contact the developers or the paper authors if you encounter any problems, we are happy to patch/fix the program.

## Citation

If you use our work in a publication, we would appreciate citations:

    @inproceedings{wang2020theoretically,
      author = {Wang, Yiqiu and Gu, Yan and Shun, Julian},
      title = {Theoretically-Efficient and Practical Parallel DBSCAN},
      year = {2020},
      isbn = {9781450367356},
      publisher = {Association for Computing Machinery},
      address = {New York, NY, USA},
      url = {https://doi.org/10.1145/3318464.3380582},
      doi = {10.1145/3318464.3380582},
      booktitle = {Proceedings of the 2020 ACM SIGMOD International Conference on Management of Data},
      pages = {2555–2571},
      numpages = {17},
      keywords = {parallel algorithms, spatial clustering, DBScan},
      location = {Portland, OR, USA},
      series = {SIGMOD ’20}
    }
