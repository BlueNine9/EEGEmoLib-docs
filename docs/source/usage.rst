Usage
=====

.. _installation:

Installation
------------

To use EEGEmoLib, first install it using pip:

.. code-block:: console

   (.venv) $ pip install placeholder

Reproducing Paper Experiments
-----------------------------

We provide part of the experiment configurations used in our paper under ``cfg/experiments`` in the repository. You can use these configurations to reproduce the experiments reported in the paper.

For example, we provide a ``seed_protocols`` folder that contains settings for experiments on the ``SEED`` dataset. If you want to reproduce the ``DeepConvNet`` experiment under the ``cross_session`` setting, run:

.. code-block:: bash

   python -m eegemolib.engine.protocol --cfg cfg/experiments/seed_protocols/cross_session/deepconvnet_psd.yaml

This corresponds to evaluating emotion recognition accuracy on the ``SEED`` dataset with ``DeepConvNet`` using the ``psd`` feature. After running, the program outputs metrics such as accuracy, variance, and confusion matrix for further analysis.

Simple Tutorial
---------------

In this section, we demonstrate how to use EEGEmoLib to load, preprocess, and utilize the SEED dataset for model training.

EEGEmoLib provides built-in scripts for loading and preprocessing a variety of datasets. To load the SEED dataset, one can use the following sample script:

.. code-block:: python

   from preprocess.seed_preprocess import SeedPreprocess

   args = load_yaml("cfg/datasets/SEED.yaml")
   processor = SeedPreprocess(args)
   data, label = processor._load_data("path/to/SEED/dataset")

We employ a YAML configuration file to specify the preprocessing and data-handling procedures in a fine-grained manner. A detailed explanation of the YAML configuration can be found in our repository.

Once the data is loaded, additional preprocessing steps can be applied. For the SEED dataset, we provide built-in procedures such as filtering and Independent Component Analysis (ICA):

.. code-block:: python

   data = processor._preprocess(data)

Subsequently, features can be extracted from the preprocessed data using our feature extraction module. For instance, to compute Differential Entropy (DE) and Power Spectral Density (PSD) features, one may proceed as follows:

.. code-block:: python

   import preprocess.features as pf

   de_feature = pf.de(data, args)
   psd_feature = pf.psd(data, args)

The library also provides visualization tools to facilitate the inspection of extracted features. For example, the following code visualizes segments of the DE and PSD features:

.. code-block:: python

   import utils.visual as uv

   uv.plot_frequency_feature(de_feature[0, 0, :, :2], ['Alpha', 'Beta', 'Gamma', 'Theta', 'Delta'], 'DE', args)
   uv.plot_frequency_heatmap_feature(psd_feature[0, 0, :, :2], ['Alpha', 'Beta', 'Gamma', 'Theta', 'Delta'], 'PSD', args)

Here, ``de_feature[0, 0, :, :2]`` and ``psd_feature[0, 0, :, :2]`` correspond to two selected time segments of the DE and PSD feature tensors, respectively.

The caption and axis labels of the generated figures are automatically derived from the arguments ``feature_label`` (e.g., ``'DE'`` or ``'PSD'``), ``band_label`` (e.g., ``['Alpha', 'Beta', 'Gamma', 'Theta', 'Delta']``).

Each figure consists of multiple subplots arranged according to the number of EEG channels.

- In ``plot_frequency_feature``, each subplot displays time-series curves of the extracted feature values for different frequency bands within one EEG channel. The x-axis represents time (in seconds), and the y-axis shows the feature magnitude. Different colored lines correspond to distinct frequency bands (e.g., Alpha, Beta, Gamma, etc.).
- In ``plot_frequency_heatmap_feature``, each subplot shows a time-frequency heatmap for a single channel, where the x-axis denotes time, the y-axis lists the frequency bands, and the color intensity encodes the feature value (e.g., PSD power or DE magnitude). A shared colorbar on the right indicates the numerical range of the feature values across all channels.

Additional visualization utilities can be found in our codebase.

Finally, models can be defined and trained using the built-in implementations in the repository. As an example, we demonstrate the definition and training of DeepConvNet:

.. code-block:: python

   from models.2d_analysis import DeepConvNet
   import torch
   from tqdm import tqdm

   model = DeepConvNet(args)
   optim = torch.optim.Adam(model.parameters(), lr=args.learning_rate)

   train_feature, test_feature, train_label, test_label = processor._split(data, label)

   for _ in tqdm(range(args.max_epoch)):
       # Training code omitted