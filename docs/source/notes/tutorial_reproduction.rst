.. _fast_model_reproduction:

Fast Model Reproduction
==========================================

To facilitate the communication among researchers, we equip ``cotk`` with command line tools which allow you to

- download public code from github repositories or the cotk `dashboard <http://coai.cs.tsinghua.edu.cn/dashboard/>`__
- reproduce others' model if the code satisfy with a small set of rules
- publish model performance and code to the dashboard

Reproduce public code
----------------------------------------

The command line ``cotk download`` helps download public code from github repositories or the dashboard.

Download from public github repositories
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Command line:

.. code-block:: none

    cotk download <model>

``<model>`` should be a string that indicates the code path. It can be:

.. code-block:: none

    * url of a git repo / branch / commit.
        Example:    https://github.com/USER/REPO
                    https://github.com/USER/REPO/tree/BRANCH
                    https://github.com/USER/REPO/commit/COMMIT (full commit id is needed)
    * a string specifying a git repo / branch / commit.
        Example:    USER/REPO
                    USER/REPO/BRANCH
                    USER/REPO/COMMIT (full commit id is needed)

The command line above will create the following files under the current working directory:

.. code-block:: none

    .
    ├── REPO // the root of the code directory
    │   ├── .model_config.json
    │   └── ... // code files
    └── run_model.sh

.. note::

    ``run_model.sh`` will be created only if the root of the git repository includes
    a file named ``.model_config.json`` that specifies run configurations
    (Refer to `Make Your Model Reproducible`_ for more details).

By running ``run_model.sh``, the model will run and then generated a results file,
usually named ``result.json``.

Download from the dashboard
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Command line:

.. code-block:: none

    cotk download <model> --result <path_to_dump_information>

* **model**: An ID (a sequence of digits) on the dashboard.
* **path_to_dump_information**: The path where you want to dump the information from the dashboard.
  This file includes evaluation results of the model as well as the run configurations for reproducing them.
  Default: ``dashboard_result.json``.

The command line above will create the following files under the current working directory:

.. code-block:: none

    .
    ├── REPO // the root of the code directory
    │   └── ... // code files
    ├── dashboard_result.json
    └── run_model.sh

The ``REPO`` here is the git repository associated with the specified model.

.. note::

    Different from `Download from public github repositories`_, ``run_model.sh`` can be created
    without ``.model_config.json``.

Make Your Model Reproducible
-------------------------------

Publish your model to github
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Suppose you have written your model under a directory named ``PROJECT``.
By complying with the following protocols, you are able to make your model easily reproduced by anyone with ``cotk``:

- There exists an entry file named ``<entry>.py`` somewhere in ``PROJECT``. This file defines an entry function named ``run(*args)``.
  
- A file named ``result.json`` should be created, when the entry function ``run()`` is called with ``<args>``.
  The ``result.json`` contains the scores from :ref:`metric functions <metric_ref>`. For example:

.. code-block:: none

    {'bw-bleu': 0.04871277607530735,
     'fw-bleu': 0.22873635755754274,
     'fw-bw-bleu': 0.08032018568655393,
     'fw-bw-bleu hashvalue': '3018dc317f82b6013f011c1f8ccd90c5affed710b7d7d06a7235cf455c233542',
     'self-bleu': 0.07416490324471028,
     'self-bleu hashvalue': '9f1121d3988ef4789943ef18c1c0b749eec0d8eee3f12270671605ce670225f6'}

- There may optionally exist a file named ``.model_config.json`` right below ``PROJECT``.
  This file specifies run configurations which consist of ``entry``, ``args``, ``working_dir``, ``cotk_record_information``.

  - **entry** (str): The name of your entry file. Default: "main"
  - **args** (:class:`list`): The arguments passed to run(). For example: ``["--batch_size", 32]``. Default: ``[]``.
  - **working_dir** (str): The path relative to ``PROJECT`` where ``<entry>.py`` locates. Default: ``./``.
  - **cotk_record_information** (dict, optional): some information for the support of dashboard.

- ``PROJECT`` should be push to a github public repository with all your codes and configs.
  The remote origin should be set, like ``git remote add origin master https://github.com/USERNAME/REPONAME.git``.

.. note::

    ``.model_config.json`` can be generated by ``cotk run --only-run`` commands described in the :ref:`following section <publish_your_model>`.


.. _publish_your_model:

Publish your model to dashboard
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The command line ``cotk run`` help publish your model to the `dashboard <http://coai.cs.tsinghua.edu.cn/dashboard/>`__.
Besides showing evaluation results of models, this dashboard also tells
whether models refer to the same data, which aims at fair comparisons
(Refer to :ref:`Metric <hash_ref>` for more details).

- Same with `Publish your model to github`_, you have to initialize a git repository under a directory named ``PROJECT``.

- There exists an entry file named ``<entry>.py`` somewhere in ``PROJECT``. This file defines an entry function named ``run(*args)``.

- A file named ``result.json`` should be created, when the entry function ``run()`` is called with ``<args>``.
  The ``result.json`` contains the scores from :ref:`metric functions <metric_ref>`. For example:

.. code-block:: none

    {'bw-bleu': 0.04871277607530735,
     'fw-bleu': 0.22873635755754274,
     'fw-bw-bleu': 0.08032018568655393,
     'fw-bw-bleu hashvalue': '3018dc317f82b6013f011c1f8ccd90c5affed710b7d7d06a7235cf455c233542',
     'self-bleu': 0.07416490324471028,
     'self-bleu hashvalue': '9f1121d3988ef4789943ef18c1c0b749eec0d8eee3f12270671605ce670225f6'}

- ``PROJECT`` should be push to a github public repository with all your codes and configs.
  The remote origin should be set, like ``git remote add origin master https://github.com/USERNAME/REPONAME.git``.

- Running the ``cotk run --entry <path_to_entry>``. The command will invoke your program and automatically publish your model to the dashboard. A URL of the online report will be shown.

For the usage of ``cotk run``:

.. code-block:: none

    usage: cotk run [-h] [--token TOKEN] [--result RESULT] [--only-run]
                    [--only-upload] [--entry [ENTRY]]

    Run model and report performance to cotk dashboard.**More args can added at
    end of the command to pass to your model**.

    optional arguments:
    -h, --help       show this help message and exit
    --token TOKEN    Use a temporary token. (Specified your accounts on dashboard.)
    --result RESULT  Path to result file that your model generated. Default:
                     result.json
    --only-run       Just run my model, save running information to local
                     folder but do not upload anything.
    --only-upload    Don't run my model, just upload the existing result.
    --entry [ENTRY]  Entry file of your model, suffix name '.py' should not be
                     included. Default: main

* When ``--only-run`` is set, you choose to run your model locally without publishing results to the dashboard.
  A ``.model_config.json`` will be generated.

* When ``--only-upload`` is set, you can upload your results of generated before by ``--only-run``.

* When neither ``--only-run`` nor ``--only-upload`` is set, the model will run and then
  the results will be uploaded to the dashboard.

``TOKEN`` is a non-empty string for identification on the dashboard.
You can set it as a global variable and avoid typing it many times.

.. code-block:: none

    cotk config set token TOKEN

.. note::

    If you are to publish your model on dashboard, all the protocols in `Publish your model to github`_
    should be satisfied except ``.model_config.json`` (which will generated by ``cotk run`` automatically and
    not necessarily be committed to github).

You can find example at https://github.com/thu-coai/seq2seq-pytorch/