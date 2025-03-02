.. _cli:

CLI Reference
=============

:code:`dbx` provides access to it's functions in a cli-oriented fashion.

Each individual command has a detailed help screen accessible via :code:`dbx command_name --help`.

We encourage you to use :code:`dbx` both for local development and CI/CD pipelines.

.. note::

    :code:`dbx` works with your PAT (Personal Access Token) in exactly the same way as `databricks-cli`_.
    This means that if the following environment variables:

    * :code:`DATABRICKS_HOST`
    * :code:`DATABRICKS_TOKEN`

    are defined, :code:`dbx` will use them to perform actions.
    It allows you to securely store these variables in your CI/CD tool and access them from within the pipeline.

.. click:: dbx.cli:cli
    :prog: dbx
    :nested: full


.. _databricks-cli: https://docs.databricks.com/dev-tools/cli/index.html
