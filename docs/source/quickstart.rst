.. _quickstart:

Quickstart
==========

Prerequisites
-------------

- Python >=3.6 environment on your local machine
- `databricks-cli`_ with a `configured profile <https://docs.databricks.com/dev-tools/cli/index.html#set-up-authentication>`_


In this instruction we're based on `Databricks Runtime 7.3 LTS ML <https://docs.databricks.com/release-notes/runtime/7.3ml.html>`_.
If you don't need to use ML libraries, we still recommend to use ML-based version due to :code:`%pip` magic `support <https://docs.databricks.com/libraries/notebooks-python-libraries.html>`_.

Installing dbx
--------------

Install :code:`dbx` via :code:`pip`:

.. code-block:: python

    pip install dbx

Starting from a template (Python)
---------------------------------
If you already have an existing project, you can skip this step and move directly to the next one.

For Python-based deployments, we recommend to use `cicd-templates <https://github.com/databrickslabs/cicd-templates>`_ for quickstart.
However, if you don't like the project structure defined in cicd-templates, feel free to use the instruction below for full customization.


Configuring environments
------------------------

Move the shell into the project directory and configure :code:`dbx`.

.. note::

    :code:`dbx` heavily relies on `databricks-cli`_ and uses the same set of profiles.
    Please configure your profiles in advance using :code:`databricks configure` command as described `here <https://docs.databricks.com/dev-tools/cli/index.html#set-up-authentication>`_.

Create a new environment configuration via given command:

.. code-block:: python

    dbx configure \
        --profile=test # name of your profile, omit if you would like to use the DEFAULT one

This command will configure a project file in :code:`.dbx/project.json` file. Feel free to repeat this command multiple times to reconfigure the environment.

Preparing Deployment Config
-------------------------

Next step would be to configure your deployment objects. To make this process easy and flexible, we support two options to define the configuration.

#. JSON: :code:`conf/deployment.json`: This is the default file which will be picked up automatically.
#. YAML: :code:`conf/deployment.yaml`: To use [ yaml | yml ] you will need to explicitly specify the file using the option :code:`--deployment-file=./conf/deployment.yaml`


.. note::

    Within the deployment config, if you find that you have duplicated parts like cluster definitions or retry config or permissions or anything else,
    and you are finding it hard to manage the duplications, we recommend you either use `YAML <http://yaml.org/spec/1.2/spec.html>`_ or `Jsonnet <https://jsonnet.org>`_.

    Yaml is supported by dbx where as with Jsonnet, you are responsible for generating the json file through Jsonnet compilation process.


JSON
****

By default, deployment configuration is stored in :code:`conf/deployment.json`.
The main idea of the deployment file is to provide a flexible way to configure job with it's dependencies.
You can use multiple different deployment files, providing the filename as an argument to :code:`dbx deploy` via :code:`--deployment-file=/path/to/file.json` option.
Here are some samples of deployment files for different cloud providers:

.. tabs::

   .. tab:: AWS

      .. literalinclude:: ../../tests/deployment-configs/aws-example.json
         :language: JSON

   .. tab:: Azure

      .. literalinclude:: ../../tests/deployment-configs/azure-example.json
         :language: JSON

   .. tab:: GCP

      .. literalinclude:: ../../tests/deployment-configs/gcp-example.json
         :language: JSON

Expected structure of the deployment file is the following:

.. code-block:: javascript

    {
        // you may have multiple environments defined per one deployment.json file
        "<environment-name>": {
            "jobs": [
                // here goes a list of jobs, every job is one dictionary
                {
                    "name": "this-parameter-is-required!",
                    // everything else is as per Databricks Jobs API
                    // however, you might reference any local file (such as entrypoint or job configuration)
                    "spark_python_task": {
                        "python_file": "path/to/entrypoint.py" // references entrypoint file relatively to the project root directory
                    },
                    "parameters": [
                        "--conf-file",
                        "conf/test/sample.json" // references configuration file relatively to the project root directory
                    ]
                }
            ]
        }
    }

As you can see, we simply follow the `Databricks Jobs API <https://docs.databricks.com/dev-tools/api/latest/jobs.html>`_ with one enhancement -
any local files can be referenced and will be uploaded to dbfs in a versioned way during the :code:`dbx deploy` command.


YAML
****

If you want to use yaml, you will have to specify the file using :code:`--deployment-file=/path/to/file.yaml` option
available on the :code:`dbx deploy` or :code:`dbx execute` commands.

You can define re-usable definitions in yaml. Here is an example yaml and its json equivalent:

.. note::
    The YAML file needs to have a top level :code:`environments` key under which all environments will be listed.
    The rest of the definition is the same as it is for config using json. It follows the
    `Databricks Jobs API <https://docs.databricks.com/dev-tools/api/latest/jobs.html>`_ with the same auto
    versioning and upload of local files referenced with in the config.

.. tabs::

    .. tab:: YAML

        .. literalinclude:: ../../tests/deployment-configs/02-yaml-with-vars-test.yaml
            :language: YAML

    .. tab:: JSON Equivalent

        .. literalinclude:: ../../tests/deployment-configs/02-yaml-with-vars-test.json
            :language: JSON



Interactive execution
---------------------

.. note::

    :code:`dbx` expects that cluster for interactive execution supports :code:`%pip` and :code:`%conda` magic `commands <https://docs.databricks.com/libraries/notebooks-python-libraries.html>`_.


The :code:`dbx execute` executes given job on an interactive cluster.
You need to provide either :code:`cluster-id` or :code:`cluster-name`, and a :code:`--job` parameter.

.. code-block:: python

    dbx execute \
        --cluster-name=some-name \
        --job=your-job-name

You can also provide parameters to install .whl packages before launching code from the source file, as well as installing dependencies from pip-formatted requirements file or conda environment yml config.

Deployment
----------

After you've configured the `deployment.json` file, it's time to perform an actual deployment:

.. code-block:: python

    dbx deploy \
        --environment=test

You can optionally provide requirements.txt file, all requirements will be automatically added to the job definition.
Please refer to the full description of deploy command in the CLI section for more options on setup.

Launch
------

Finally, after deploying all your job-related files, you can launch the job via the following command:

.. code-block:: python

    dbx launch --environment=test --job=sample

Please refer to the full description of launch command in the CLI section for more options.

.. _databricks-cli: https://docs.databricks.com/dev-tools/cli/index.html
