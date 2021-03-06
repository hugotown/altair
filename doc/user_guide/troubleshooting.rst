.. _display-troubleshooting

Display Troubleshooting
=======================
Altair has a number of moving parts: it creates data structures in Python, those
structures are passed to front-end renderers, and the renderers run JavaScript
code to generate the output. This complexity means that it's possible to get
into strange states where things don't immediately work as expected.

This section summarizes some of the most common problems and their solutions.

.. _trouble-shooting-jupyterlab

Trouble-shooting Altair with JupyterLab
---------------------------------------

.. _jupyterlab-vega-lite-2-object

VegaLite 2 Object
~~~~~~~~~~~~~~~~~
**If you are using the Jupyter notebook rather than JupyterLab, see
:ref:`notebook-vega-lite-2-object` for information on this error**

If you are using JupyterLab (not Jupyter notebook) and see the following output::

    <VegaLite 2 object>

it can mean one of several things is wrong:

1. You are using too old a version of JupyterLab. Altair requires JupyterLab version
   0.31 or later; check the version with::

       $ jupyter lab --version
       0.31.10

   If this is the problem, then use ``pip install -U jupyterlab`` or
   ``conda update jupyterlab`` to update JupyterLab, depending on how you
   first installed it.

2. You have not correctly installed the ``vega3`` lab extension mentioned
   in the installation steps above. You can list currently installed lab
   extensions with::

       $ jupyter labextension list

   if ``@jupyterlab/vega3-extension`` does not appear in that list, then run
   the following to install it::

       $ jupyter labextension install @jupyterlab/vega3-extension

   Once this is installed, re-launch JupyterLab and your charts should render.


JavaScript output is disabled in JupyterLab
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are using JupyterLab and see the following ouput::

    JavaScript output is disabled in JupyterLab

it can mean one of two things is wrong

1. You are using an old version of Altair. JupyterLab only works with Altair
   version 2.0 or newer; you can check the altair version by running::

       import altair as alt
       alt.__version__

   If the version is older than 2.0, then exit JupyterLab and follow the
   installation instructions at :installation-jupyterlab:.

2. You have enabled the wrong renderer. JupyterLab works with the default
   renderer, but if you have used ``alt.renderers.enable(xxx)`` to enable
   another renderer, JupyterLab will no longer work.
   You can check which renderer is active by running::

       import altair as alt
       print(alt.renderers.active)

   JupyterLab rendering will work only if the active renderer is ``"default"``
   or ``"jupyterlab"``. You can re-enable the default renderer by running::

       import altair as alt
       alt.renderers.enable('default')

   (Note that the default renderer is enabled, well, by default, and so this
   is only necessary if you've somewhere changed the renderer explicitly).


.. _trouble-shooting-jupyterlab

Trouble-shooting Altair with Notebook
-------------------------------------

.. _notebook-vega-lite-2-object

VegaLite 2 object
~~~~~~~~~~~~~~~~~
**If you are using JupyterLab rather than the Jupyter notebook, see
:ref:`jupyterlab-vega-lite-2-object` for information on this error**

If you are using the notebook (not JupyterLab) and see the the following output::

    <Vegalite 2 object>

it usually means that you have not enabled the notebook renderer. As mentioned
in :ref:`installation-notebook`, you need to install the ``vega3`` package and
Jupyter extension, and then enable it using::

    import altair as alt
    alt.renderers.enable('notebook')

in order to render charts in the classic notebook. If the above code gives an
error::

    NoSuchEntryPoint: No 'notebook' entry point found in group 'altair.vegalite.v2.renderer'

This means that you have not installed the vega3 package. If you see this error,
please make sure to follow the standard installation instructions at
:ref:`installation-notebook`.

.. _trouble-shooting-general

General Trouble-shooting
------------------------

Plot displays, but the content is empty
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Sometimes you end up with an empty plot; for example:

.. altair-plot::

    import altair as alt

    alt.Chart('nonexistent_file.csv').mark_line().encode(
        x='x:Q',
        y='y:Q',
    )

In this case, the plot was empty because the data, ``'nonexistent_file.csv'``,
does not exist, or contains a typo in the URL.

A similar blank chart results if you refer to a field that does not exist
in the data; for example:

.. altair-plot::

   import pandas as pd

   data = pd.DataFrame({'x': [1, 2, 3],
                        'y': [3, 1, 4]})

   alt.Chart(data).mark_point().encode(
       x='x:Q',
       y='y:Q',
       color='color:Q'  # <-- this field does not exist in the data!
   )

Altair does not check whether fields are valid, because there are many avenues
by which a field can be specified within the full schema, and it is too difficult
to account for all corner cases. Improving the user experience in this is a
priority; see https://github.com/vega/vega-lite/issues/3576.

Chart does not display at all
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
For all renderers, the chart is only displayed if the **last line of the cell
evaluates to a chart object**

By analogy, consider the output of simple Python operations::

    >>> x = 4  # no output here
    >>> x      # output here, because x is evaluated
    4
    >>> x * 2  # output here, because the expression is evaluated
    8

If the last thing you type consists of an assignment operation, there will be no
output displayed. This turns out to be true of Altair charts as well:

.. altair-plot::
    :output: none

    import altair as alt
    from vega_datasets import data
    cars = data.cars.url

    chart = alt.Chart(cars).mark_point().encode(
        x='Horsepower:Q',
        y='Miles_per_Gallon:Q',
        color='Origin:N',
    )

The last statement is an assignment, so there is no output and the chart is not
shown. If you have a chart assigned to a variable, you need to end the cell with
an evaluation of that variable:

.. altair-plot::

    chart = alt.Chart(cars).mark_point().encode(
        x='Horsepower:Q',
        y='Miles_per_Gallon:Q',
        color='Origin:N',
    )

    chart

Alternatively, you can construct a chart directly, and not assign it to a varaible,
in which case the object definition itself is the final statement and will be
displayed as an output:

.. altair-plot::

    alt.Chart(cars).mark_point().encode(
        x='Horsepower:Q',
        y='Miles_per_Gallon:Q',
        color='Origin:N',
    )
