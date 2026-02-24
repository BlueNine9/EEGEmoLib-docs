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