How to Contribute
=================

This document describes how to contribute new models, feature extraction methods, and datasets to EEGEmoLib.

New Models
----------

Model Input
~~~~~~~~~~~

For most models, the input shape is ``[B, 1, C, F]``, where ``B`` denotes the batch size, ``C`` the number of channels, and ``F`` the feature length.

When constructing a model, we provide two arguments: ``num_classes`` and ``input_shape``, which represent the number of target classes and the input shape, respectively. If your model requires additional parameters, please assign them reasonable default values.

Ideally, your model should be able to adaptively construct its architecture based on the provided ``input_shape``. If your model requires a fixed feature length, you should still make an effort to support a variable number of channels. If the provided ``input_shape`` is incompatible with your model, you should raise a clear error indicating the expected input format.

Model Output
~~~~~~~~~~~~

We recommend that your model outputs logits with shape ``[B, num_classes]``. Tuple outputs are also supported, allowing you to return additional information. By default, EEGEmoLib uses the last element of the tuple as the classification output.

If your model outputs a 3D tensor of shape ``[B, T, num_classes]``, EEGEmoLib will automatically apply ``mean(dim=1)``.

Model Integration
~~~~~~~~~~~~~~~~~

1. Add a new file under ``src/eegemolib/models/``, e.g., ``MyNet.py``.
2. Define the model class (the class name must match ``model.name`` in the configuration).

Example: A Minimal Model MyNet
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # src/eegemolib/models/2d_analysis/MyNet.py
   import torch
   import torch.nn as nn

   class MyNet(nn.Module):
       def __init__(self, n_classes=3, input_shape=None, feature_dim=1050, hidden=128):
           super().__init__()
           self.net = nn.Sequential(
               nn.Flatten(),
               nn.Linear(input_shape[2] * feature_dim, hidden),
               nn.ReLU(),
               nn.Linear(hidden, n_classes),
           )

       def forward(self, x):  # x: [B, 1, C, F]
           return self.net(x)

New Feature Extraction Methods
------------------------------

Input and Output
~~~~~~~~~~~~~~~~

Typically, the input to a feature extraction method is a single-trial EEG signal with shape ``[channels, timesteps]``. In addition, an ``args`` dictionary is provided, which is configured via ``feature.args`` in the YAML file.

Your feature extraction method should output a 3D tensor of shape ``[channels, windows, bands]``. If your method is band-independent, you may set the third dimension (bands) to 1.

Compatibility Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want your method to support ``adaptive_band_weight``, the output must contain an interpretable bands dimension. Other than this, there are no strict compatibility requirements.

Integration
~~~~~~~~~~~

Add your function to ``src/eegemolib/preprocess/features.py``.

Example: Adding an rms Feature
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   def rms(data, args):
       window_length = args['win_length']
       sample_rate = args['down_sample']
       n_channels, n_samples = data.shape

       points = int(sample_rate * window_length)
       n_windows = n_samples // points

       out = np.zeros((n_channels, n_windows, 1), dtype=np.float32)
       for w in range(n_windows):
           seg = data[:, w * points:(w + 1) * points]
           out[:, w, 0] = np.sqrt(np.mean(seg ** 2, axis=1))
       return out

New Datasets
------------

In principle, we do not recommend manually adding new datasets to EEGEmoLib. If you would like to include a new dataset, please open an issue and provide the dataset link. We will integrate it into EEGEmoLib accordingly.

If you still wish to add a dataset yourself, please follow the instructions below.

Typically, adding a new dataset requires implementing two classes:

1. **Preprocess class** (handles conversion from raw data to cached data)
2. **Dataset class** (loads cached data and returns batches during training)

Required Methods
~~~~~~~~~~~~~~~~

Preprocess Class: inherit from ``BasePreprocess``

Suggested path: ``src/eegemolib/preprocess/yourset_preprocess.py``

Must implement:

- ``_load_data(self, path)``
- ``_load_feature(self, path, feature_list)``
- ``_preprocess(self, data)``

Optional overrides:

- ``_split(...)``
- ``_feature_compute(...)``, ``_feature_selection(...)``

Dataset Class: inherit from ``BaseDataset``

Suggested path: ``src/eegemolib/data/yourset_dataset.py``

Must implement:

- ``__init__(self, args, test=False)``
- ``__getitem__(self, index)`` (returns ``{'data': ..., 'label': ...}``)
- ``__len__(self)``
- ``__str__(self)``

Example: Minimal Dataset Skeleton
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # src/eegemolib/preprocess/myset_preprocess.py
   from .base_preprocess import BasePreprocess

   class MysetPreprocess(BasePreprocess):
       def _load_data(self, path):
           # return data_list, label_list
           pass

       def _load_feature(self, path, feature_list):
           pass

       def _preprocess(self, data):
           return data

.. code-block:: python

   # src/eegemolib/data/myset_dataset.py
   from data.base_dataset import BaseDataset

   class MysetDataset(BaseDataset):
       def __init__(self, args, test=False):
           super().__init__(args)
           ...

       def __getitem__(self, index):
           return {'data': data_tensor, 'label': label_tensor}

       def __len__(self):
           return self.length

       def __str__(self):
           return 'MYSET'
