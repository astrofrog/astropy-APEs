NDData Plan
-----------

author: Thomas Robitaille

date-created:

date-last-revised: 2014 October 2

type: Standard Track

status: Discussion

Abstract
--------

This APE is intended to provide a long-term plan for the ``astropy.nddata``
sub-package. The package has been the subject of continuous debate since the
start of the astropy project, and has changed in scope several times, so this
APE is aimed at agreeeing on the scope and future of the sub-package.

Detailed description
--------------------

Introduction
^^^^^^^^^^^^

At the first Astropy coordination meeting in 2011, it was decided that as well
as having a generic table container, it would be useful to have a generic
container for gridded data. The ``astropy.table`` package has since then seen a
large amount of development, and the API has now stabilized. The added value of
the table pacakge compared to using simple Numpy structured arrays is clear -
the ``Table`` class makes it very easy to do common operations on tables such
as adding or removing columns or rows, and reading/writing tables to common
file formats.

On the other hand, ``NDData`` development has stagnated and we have not been
able to converge on a stable API. Part of this is due to the fact that there is
in fact a huge variety of 'n-dimensional datasets' and that there is very
little in common for example between a spectrum and an image, in terms of what
can be done with them. This has prevented the ``NDData`` class from including
functionality, and in fact we have been adding functonality then removing it
after we have realized that it is not general enough. An example of this is
arithmetic - what happens when we add two n-dimensional datasets? Of course,
the data values can be added, but what happens to the mask? To the flags? To
the uncertainties? The truth is that there is no general recipe for dealing
with this, and that it may be a mistake to try and define it in such a general
way.

This APE takes the approach of re-thinking the purpose of NDData and trying to
define a scope for the future.

The 'why' of NDData
^^^^^^^^^^^^^^^^^^^

First, why do we actually need a generic data container that can be
sub-classed? What is the benefit of this versus simply defining separate base
classes for spectra, images, and other types of data? There are several
possible answers:

1. We want to provide users with a consistent experience across data objects -
   that is, the user should know that meta-data is always consistently named
   ``meta``, that a mask can always be accessed with ``mask``, and that the
   data can be accessed with ``data``.

2. By providing a common base class which can define a unified I/O interface
   which taps into the astropy I/O registry, we can seamlesstly make it that
   all data objects have ``read`` and ``write`` methods that behave
   consistently.

3. We want functions and methods in Astropy to know if an object passed to them
   is an n-dimensionall data object that can be expected to have specific
   attributes (such as ``wcs``, ``mask``, and so on).

In principle, none of these *require* a base class. We could simply agree on a
standard for data objects that defines what certain attributes should be
called. This is a valid solution, but at the same time, having a base class can
enforce this and factor out some boilerplate code.

Proposal for the ``NDData`` class
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The proposal in this APE is to simplify the ``NDData`` class to the extreme,
such that it essentially does only the following things:

* It defines properties that will be in common to all ``NDData`` sub-classes,
  but does not do any input validation. In fact, several of the 'base'
  properties can raise NotImplementedError when not implemented by a base class
  to avoid having the user think that the property can be used.

* It provides generic ``read`` and ``write`` methods that connect to the I/O
  registry, as for the ``Table`` class.

* It defines generic slicing capabilities, because this is a general operation
  that one should be able to do on a regular n-dimensional dataset.

The ``NDData`` class should **not** define any arithmetic operations, which are
impossible to generalize.

Furthermore, the ``NDData`` class would be made into an abstract base class, so
that it can never be used directly by users. The idea would then be that users
should only ever be using sub-classes, such as ``Image`` or ``Spectrum1D``. No
function in Astropy or in affiliated packages would be required to be able to
handle generic ``NDData`` objects.

The following properties should be included in the base class:

* ``data`` - the data itself. No restrictions are placed on the type of this
  data. For example, it could be a plain Numpy array, masked Numpy array, an
  Astropy Quantity, or an h5py data buffer.

* ``mask`` - the mask of the data, following the Numpy convention of `True`
  meaning masked, and `False` meaning unmasked. Sub-classes could choose to
  connect this to ``data.mask``.

* ``unit`` - the unit of the data values, which should be an Astropy Unit (this
  is one place where it makes sense to place a restriction on the type).
  Sub-classes could choose to connect this to ``data.unit``

* ``wcs`` - an object that can be used to describe the relationship between
  positions in 'pixel' space, and world coordinates. This can (but does not
  have to) be an Astropy WCS object.

* ``meta`` - a dict-like object that can be used to contain arbitrary metadata.
  This could be a plain Python dict, an ordered dict, a FITS Header object, and
  so on, provided that it offers dict-like item access and iteration.

* ``uncertainty`` - an object describing the uncertainties in the data.

If sub-classes do not support e.g. ``uncertainty``, they can simply raise a
``NotImplementedError``.

Handling of ``NDData`` in Astropy and affiliated packages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If a user has a data object such as an image, it would be nice if they can use
functions directly on this image and have them return an image object. At the
same time, we do not want to force people to use special data containers if
they have for example a Numpy array and a WCS object. This raises the quesiton
of whether we should duplicate the API for all functions, to provide one
interface for ``NDData`` subclasses, and one for separate attributes. The
proposal here is that functions should only define a single API that takes
separate keyword arguments for e.g. ``data``, ``mask``, and so on, but that we
then provide a way for users to be able to call these functions with ``NDData``
sub-classes (see `Implementation`_).

Branches and pull requests
--------------------------

https://github.com/astropy/astropy/pull/2855
N/A

Implementation
--------------

``NDData`` class
^^^^^^^^^^^^^^^^

The ``NDData`` class should be dramatically simplified to comply with the
proposal above. It should contain very little apart from the property
definitions. For example, for the WCS, it should simply contain::

    @property
    def wcs(self):
        return self._wcs

    @wcs.setter
    def wcs(self, value):
        self._wcs = wcs

The only exceptions to this are that the type of the unit should be checked (it
should be an Astropy unit), but otherwise all the properties listed above
should follow this simple template.

The ``read`` and ``write`` methods can be adapted from the ``Table`` class.

The only other functionality this APE suggests adding is slicing. This could be
done by simply having code similar to the following inside ``__getitem__``::

    def __getitem__(self, slice):

        new = self.__class__()

        if self.data is not None:
            new.data = self.data[slice]

        if self.mask is not None:
            new.mask = self.mask[slice]

        if self.wcs is not None:
            new.wcs = self.wcs[slice]
        ...

That is, the slicing is simply delegated to the objects. This requires two
things:

* An internal list of which properties should be sliced (for example ``meta``
  should not be sliced). ``NDData`` and its sub-classes could contain a
  ``_slicable`` class attribute that lists properties that should be sliced.

* Slicing capability on the objects stored inside the properties. If the WCS
  object is not slicable, then an error should be raised since the slicing
  cannot be successfully carried out. The example code above could include a
  nice error message if a property is not slicable.

Faciliating the use of ``NDData`` sub-classes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to make it possible for functions to accept ``NDData`` sub-classes and
return these, we can implement a decorator that will automatically split up an
``NDData`` object as needed. Let us consider the following function::

    def test(data, wcs=None, unit=None, n_iterations=3):
        ...

We can provide a decorator called e.g. ``nddata_support``::

    @nddata_support
    def test(data, wcs=None, unit=None, n_iterations=3):
        ...

which makes it so that if the user passes an ``NDData`` sub-class called e.g.
``nd``, the function would automatically be called with::

    test(nd.data, wcs=nd.wcs, unit=nd.unit)

That is, the decorator looks at the signature of the function and checks if any
of the arguments are also properties of the ``NDData`` object, and passes them
as individual arguments.

An error could be raised if an ``NDData`` property is set but the function does
not accept it - for example, if ``wcs`` is set, but the function cannot support
WCS objects, an error would be raised. On the other hand, if an argument in the
function does not exist in the ``NDData`` object or is not set, it is simply
left to its default value.

If the function call succeeds, then the decorator will make a new ``NDData``
object (with the correct class) and will populate the properties as needed. In
order to figure out what is returned by the function, the decorator will need
to accept a list which gives the name of the output values::

    @nddata_support(returns=['data', 'wcs'])
    def test(data, wcs=None, unit=None, n_iterations=3):
        ...

Finally, the decorator could be made to restrict input to specific ``NDData``
sub-classes (and sub-classes of those)::

    @nddata_support(accepts=Image, returns=['data', 'wcs'])
    def test(data, wcs=None, unit=None, n_iterations=3):
        ...

Backward compatibility
----------------------

This APE will require packages such as ``specutils`` and ``ccdproc`` to
completely refactor how they use the ``NDData`` class. This will also break
compatibility with users currently using ``NDData`` directly, but this is
assumed to be a very small fraction (if any) of users.


Alternatives
------------

One alternative is to remove the ``NDData`` class alltogether, but this only
defers the questions raised here to the more specific sub-classes - for example
if we create an ``Image`` class, this will still be a generic base class for
``CCDImage``, ``XRayImage``, and so on, and the same issues will arise.

Decision rationale
------------------

<To be filled in when the APE is accepted or rejected>
