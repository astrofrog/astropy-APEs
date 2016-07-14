Roadmap for Python 3-only support
---------------------------------

author: Tom Robitaille

date-created: 2016 July 13

date-last-revised: 2016 July 13

type: Process

status: Draft

Abstract
--------

The purpose of this APE is to set out a roadmap for future releases of the
Astropy core package, in particular to identify the process for switching to
releases that support only Python 3.

Detailed description
--------------------

A number of scientific Python projects, including IPython, Matplotlib, Sympy,
and others have now signed the `Python 3 statement
<https://python3statement.github.io>`_ which states that scientific Python
packages will drop support for Python 2 no longer than 2020. In fact, the
IPython developers have now released `IPython 5.0
<http://blog.jupyter.org/2016/07/08/ipython-5-0-released/>`_, which will be the
last major release to support Python 2 - since it is a long term support (LTS)
release, there will be 5.x bug fix releases until some time in 2019, but future
major releases that add new features, starting with 6.0 (in 2017), will support
only Python 3. This APE proposes that the Astropy project should also sign the Python 3 statement
statement, and provides a target roadmap for this to happen.

Roadmap
-------

The proposed roadmap for releases, including past releases for reference, is the
following::

    v1.0 - LTS - February 19, 2015 (supported with bug fixes until June 2017)
    v1.1 - December 11, 2015
    v1.2 - June 23, 2016
    v1.3 - December 2016
    v2.0 - LTS - June 2017 (supported with bug fixes until end of 2019)
    v3.0 - December 2017 - first release to support only Python 3+
    v3.1 - June 2018
    v3.2 - December 2018
    v3.3 - June 2019
    v4.0 - LTS - December 2019 (supported with bug fixes for two years)

Note that v1.0, v2.0, and v4.0 are marked as long-term support releases (LTS),
which means that they are typically supported with bug fixes for at least two
years. The above roadmap proposes that the last Python 2-compatible release will
be v2.0, which we will support with bug fixes until the end of 2019. In the mean
time, v3.0, to be released in December 2017, will be the first release that does
not support Python 2. Note that v3.0 would not be an LTS release, since we would
otherwise need to maintain two LTS releases in parallel.

Implementation
--------------

In practice, dropping Python 2.7 support will involve:

* Removing the bundled `six <https://pythonhosted.org/six/>`_ package, and
  update all the code that used ``astropy.extern.six`` to use the Python 3
  syntax.
* Removing Python 2.7 from the configuration for the continuous integration
  services (such as Travis and AppVeyor at the time of writing)
* Making sure that the documentation on ReadTheDocs is built using Python 3.x
* Removing any backports that may exist for Python 2.7 compatibility
* Updating the documentation to remove any mention of Python 2.7, as well as
  making sure that Python 3.x is not presented as a special case.
* Remove Python 2 and 2.7 from the PyPI classifiers

Challenges
----------

There are two main challenges with the plan outlined above:

* Once we make the switch to a Python 3-only code base, between v2.0 and v3.0,
  people will no longer be able to contribute code to Astropy without using
  Python 3. This means that even though users can in theory wait until the end
  of 2019 to switch to Python 3, developers and contributors will need to make
  the switch (at least for their Astropy development environment) as soon as
  2017.

* If we upload Astropy v3.0 to PyPI, and it does not support Python 2, then if
  Python 2.7 try and pip install Astropy, the install or import will fail. This
  is because PyPI does not check the current Python version against the PyPI
  meta-data for the package (which may indicate for example whether the package
  is Python 2-compatible), and will download Astropy v3.0 regardless of the
  active Python version. However, there is a solution, which is …

Benefits
--------

There are several benefits to following the plan proposed above:

* Since we will need to keep adding Python 3.x releases to the continuous
  integration over the coming years, we will at least be able to remove Python
  2.7, making sure that the number of builds does not grow out of control.

* Since developers/contributors will need to switch to using Python 3 for
  Astropy development, we will be training more people to do this transition,
  who will then be able to help their colleagues also make the transition.

* We will be able to start using Python 3-only features internally, including
  for example function annotations (e.g., for units), matrix multiplication
  (e.g., for coordinates).

Alternatives
------------

An alternative plan would be to continue making major releases that support
Python 2 until 2019, for example:

    v1.0 - LTS - February 19, 2015 (supported with bug fixes until June 2017)
    v1.1 - December 11, 2015
    v1.2 - June 23, 2016
    v1.3 - December 2016
    v2.0 - LTS - June 2017 (supported with bug fixes until end of 2019)
    v2.1 - December 2017
    v2.2 - June 2018
    v2.3 - December 2018
    v2.4 - June 2019 - last release to support Python 2.7
    v3.0 - LTS - December 2019 (supported with bug fixes for two years)

This would allow more time for the PyPI limitations mentioned above to be
resolved and more time for developers to make the transition to Python 3.
However, if we actually want users to virtually all be using Python 3 by 2020,
then it does not make sense to delay the proposed release plan in this way,
since the developers and infrastructure need to be ready for Python 3-only
releases before the users are.

Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
