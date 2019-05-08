========
 ink-id
========

``inkid`` is a Python package and collection of scripts for identifying ink in a document via machine learning.

From:
"From invisibility to readability: Recovering the ink of Herculaneum",
Parker CS, Parsons S, Bandy J, Chapman C, Coppens F, et al. (2019) From invisibility to readability: Recovering the ink of Herculaneum. PLOS ONE 14(5): e0215775. https://doi.org/10.1371/journal.pone.0215775

Requirements
============

Python >=3.4 is supported by this package.

Python 2 is not supported.

Installation
============

To install the package you can use `pipenv <https://docs.pipenv.org/>`_:

.. code-block:: bash

   $ pip install pipenv                   # If needed, install pipenv
   $ pipenv --three                       # Create a new virtual environment with Python 3
   $ pipenv install Cython numpy          # These are required before inkid can be installed
   $ python setup.py build_ext --inplace  # Build the Volume class
   $ pipenv install -e .                  # Install inkid to the virtual environment and make it editable

The install command will find ``setup.py`` and install the dependencies for ``inkid``.

The default installation assumes you have already installed ``tensorflow`` on your machine. If you wish to install ``tensorflow`` along with ``inkid``, you can run either of these commands depending on whether or not you wish to include GPU support:

.. code-block:: bash

   $ pipenv install -e .[tf]      # Install inkid and install tensorflow (CPU only)
   $ pipenv install -e .[tf_gpu]  # Install inkid and install tensorflow-gpu

You could also just use pip:

.. code-block:: bash
   
   $ pip install -e .

Usage
=====

The package can be imported into Python programs, for example:

.. code-block:: python

   import inkid

   params = inkid.ops.load_default_parameters()
   regions = inkid.data.RegionSet.from_json(region_set_filename)

There are also some console scripts included, for example:

::

   $ inkid-train-and-predict
   usage: inkid-train-and-predict [-h] input-file output-path [options]

Examples
--------

Grid Training
~~~~~~~~~~~~~

To perform grid training, create a RegionSet JSON file for the PPM with only one training region (with no bounds, meaning it will default to the full size of the PPM).

Then use ``scripts/misc/split_region_into_grid.py`` to split this into a grid of the desired shape. Example:

.. code-block:: bash

   $ python scripts/misc/split_region_into_grid.py \
		~/data/lunate-sigma/lunate-sigma.json \
		lunate-sigma-grid-2x5.json \
		-columns 2 \
		-rows 5

Then use this region set for standard k-fold cross validation and prediction.

K-Fold Cross Validation (and Prediction)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   
``train_and_predict.py`` typically takes a region set file as input and trains on the specified training regions, evaluates on the evaluation regions, and predicts on the prediction regions. However if the ``-k`` argument is passed, the behavior is slightly different. In this case it expects the input region set to have only a set of training regions, with evaluation and prediction being empty. The kth training region will be removed from the training set and added to the evaluation and prediction sets. Example:

.. code-block:: bash

   $ inkid-train-and-predict ~/data/lunate-sigma/grid-2x5.json ~/data/out/ -k 7 --final-prediction-on-all

After performing a run for each value of k, each will have created a directory of output. If these are all in the same parent directory, there is a script to merge together the individual predictions into a final prediction image. If ``--best-f1`` is passed, it will take the prediction with the best f1 score for each individual region, rather than the final prediction for that region. Example:

.. code-block:: bash

   $ python scripts/misc/add_k_fold_prediction_images.py --dir ~/data/out/carbon_phantom_col1_test/

License
=======

This package is licensed under GPLv3 - see ``LICENSE`` for details.
