**************************************
Uncertainty Metadata Conventions (UNC)
**************************************

Authors
============

Lead Authors
------------

* Sam Hunt, NPL
* Pieter De Vis, NPL

Introduction
============

Goals
-----

Measurement datasets are becoming larger, more complex, and are increasingly used to support critical applications such as manufacturing, health, and environmental monitoring. Reliable interpretation of these measurements requires accompanying uncertainty and error-covariance information - however, this is often overlooked. Where available, such information lacks standardisation and could, in principle, be highly complex and large.

The goal of this specification is to provide a standardised metadata format for storing the accompanying uncertainty/error-covariance information with measurement datasets. This format is intended to support fully capturing the content of the error-covariance matrices associated with measurement data in a compact structure, by parameterising error-covariance with a simple set of metadata.

Philosophy
----------

This specification is intended to contribute to and build upon an existing ecosystem of standards and best practices. In particular the following are adhered to, to the extent possible:

* The understanding of uncertainty concepts defined in the JGCM `GUM`_ (Guide to the expression of uncertainty in measurement) suite of documents.
* The definition of uncertainty-related terminology defined in the JGCM `VIM`_ (International Vocabulary for Metrology).
* The `NetCDF`_ data model for creating self-describing, array-oriented scientific datasets.
* Alignment with the `Climate and Forecast (CF) conventions <cf>`_ on metadata for weather and climate data.

The work builds on previous work on the standardisation uncertainty information for climate data developed within the H2020 `FIDUCEO`_ project.

Within this context, this specification also attempts to adhere to the following principles:

* **Scalability of Complexity**

  The complexity of the metadata should align with the use case:

  - *Simple use cases* should be achievable with a *simple implementation*.
  - *Complex use cases* should be *possible without unnecessary restrictions*.

* **Minimisation of Redundancy**

  The metadata specification should *avoid duplication* of information to prevent potential inconsistencies.

* **Human and Machine Readability**

  Metadata must be:

  - *Comprehensible* for humans.
  - *Parsable* by machines.


Terminology
-----------

For terminology related to measurements and associated uncertainties, definitions within the `VIM`_ (International Vocabulary for Metrology) are adopted to the extent possible. This includes the following important terms:

* `Error`_
* `Uncertainty`_
* `Coverage factor`_

The following terms are derived from X:

* Error-correlation
* Error-covariance
* fractional uncertainty

Format for Examples
-------------------

CDL syntax

The ascii format used to describe the contents of a netCDF file is called CDL (network Common Data form Language). This format represents arrays using the indexing conventions of the C programming language, i.e., index values start at 0, and in multidimensional arrays, when indexing over the elements of the array, it is the last declared dimension that is the fastest varying in terms of file storage order. The netCDF utilities ncdump and ncgen use this format (see NUG section on CDL syntax). All examples in this document use CDL syntax.


Measurement Dataset Structures
==============================

The core object is a dataset.

The components of a netCDF file are described in section 2 of the NUG [NUG]. In this section we describe conventions associated with filenames and the basic components of a netCDF file. We also introduce new attributes for describing the contents of a file.

Variables
---------

Datasets are composed of variables, which are multidimensional data arrays.

These variables will be consider *observation variables* and have *uncertainty variables* associated with them.

A dataset may also contain variables that are neither *observation variables* or *uncertainty variables*.

**Observation Variables**

*Observation variables* represent a multidimensional array of measurements.


**Uncertainty Variables**

*Uncertainty variables* represent a component of uncertainty associated with an *observation variable*. An *observation variable* may have multiple *uncertainty variables* associated with them.

*Uncertainty variables* must have the same shape as the *observation variable* they are associated with.


Dimensions
----------

A variable may have any number of named dimensions, including zero -- e.g., `"x"`, `"y"`, `"time"`. Dimensions may be of any size, including unity.

Data Types
----------

*observation variables* and have *uncertainty variables*  must be floats.

Note: these variables may be encoded as e.g. integers for efficient storage on disc.

Attributes
----------

Global/variable attributes.

This standard defines a set of attributes to:

* link *observation variables* with their associated *uncertainty variables*
* define the error-correlation properties of a given *uncertainty variables* in a compact way.

A file may also contain non-standard attributes.

Uncertainty Attributes
======================

Assigning Uncertainty Components
--------------------------------

*Uncertainty variables* are associated with an *observation variable* through the *observation variable*'s `"unc_comps"` attribute. The attribute should contain a list of the names of the *uncertainty variables* associated with an *observation variable*..

The following example of a dataset, in CDL syntax, shows a `"temperature"` variable defined along 3 dimensions - `time`, `lat`, and `lon`. `"temperature"` has two uncertainty components associated with it - `"u_calibration"` and `"u_noise"`.

.. code-block::

    variables:
      float temperature(time, lat, lon) ;
        temperature:unc_comps=["u_calibration", "u_noise"];
      float u_calibration(time, lat, lon);
      float u_noise(time, lat, lon);

Units
-----

The variable attribute `"units"` is required for variables that are dimensional. `"units"` should be defined as a string.

*Observation variables* are assumed dimensionless if the variable attribute `"units"` is not defined.

*uncertainty variables* must have the same `"units"` as the *observation variables* they are associated with. If `"units"` is not defined, the *uncertainty variable* is assumed fractional.

The following example of a dataset again shows a `"temperature"` variable associated with two uncertainty components - `"u_calibration"` and `"u_noise"`. Here, `"u_calibration"` is defined with units `K`, matching `"temperature"`. `"u_noise"` has no defined units and so is a fractional uncertainty

.. code-block::

    variables:
      float temperature(time, lat, lon);
        temperature:unc_comps=["u_calibration", "u_noise"];
        temperature:units="K"
      float u_calibration(time, lat, lon);
        u_calibration:units="K"
      float u_noise(time, lat, lon);

Uncertainty PDF Shape
---------------------

The probability density function (PDF) shape associated with the uncertainty estimate values in an *uncertainty variables* is defined with variable attribute `"pdf_shape"`.

`"pdf_shape"` can have one of the following values:

* `"gaussian"` - for uncertainties represented by a Gaussian PDF
* `"rectangular"` - for uncertainties represented by a uniform PDF
* ...

If `"pdf_shape"` is not defined for an *uncertainty variable* it is assumed to be `"gaussian"`.

The following example of a dataset again shows a `"temperature"` variable associated with two uncertainty components - `"u_calibration"` and `"u_noise"`. Here, `"u_calibration"` is defined to be represented by a rectangular PDF. `"u_noise"` has no defined `"pdf_shape"` and so is assumed Gaussian.

.. code-block::

    variables:
      float temperature(time, lat, lon);
        temperature:unc_comps=["u_calibration", "u_noise"];
        temperature:units="K"
      float u_calibration(time, lat, lon);
        u_calibration:units="K"
        u_calibration:pdf_shape="rectangular"
      float u_noise(time, lat, lon);


Error-Correlation Structure
---------------------------

For *observation variables* with N elements, the associated error-covariance matrix per uncertainty component has $N^2$ elements. Where the *observation variables* are large, it an quickly become impractical to store this data.

However, in many cases the associated error-correlation matrix can simply be parameterised in a compact form. With this data and

This standard defines a set of attributes to achieve this.

Effectively for each dimension in a *uncertainty variable*, `dim_i`, or set of dimensions, [`dim_i`, `dim_j`, ...], a error-correlation paramaterisation is defined.

.. list-table:: Error-correlation attributes
   :widths: 15 15 50 30
   :header-rows: 1

   * - Attribute name
     - Type
     - Description
     - Example
   * - err_corr_dimi_name
     - str
     - Dimension name
     - err_corr_dim1_name="time"
   * - err_corr_dimi_form
     - str
     - Parameterisation name
     - err_corr_dim1_form="random"
   * - err_corr_dimi_params
     - list[Any]
     - Parameterisation parameters
     - err_corr_dim1_params=[1,2,3]
   * - err_corr_dimi_units
     - list[str]
     - Parameterisation parameter units
     - err_corr_dim1_params=["second", "m", "K"]


Existing parmaterisations:

.. list-table:: Error-correlation parameterisations
   :widths: 25 25 50
   :header-rows: 1

   * - Parameterisation Form
     - Parameters
     - Description
   * - random
     - []
     - No error-correlation between elements in observation variable.
   * - systematic
     - []
     - Full error-correlation between elements in observation variable.


The following example of a dataset again shows a `"temperature"` variable associated with two uncertainty components - `"u_calibration"` and `"u_noise"`.

Here, `"u_calibration"` is defined to have a systematic error-correlation in the `lat` and `lon` dimensions, and random in `time` dimension (perhaps, there is a recalibration between the measurements at each time step!).

`"u_noise"` have a error-correlation defined random in all dimensions.

.. code-block::

    variables:
      float temperature(time, lat, lon);
        temperature:unc_comps=["u_calibration", "u_noise"];
        temperature:units="K"
      float u_calibration(time, lat, lon);
        u_calibration:units="K";
        u_calibration:pdf_shape="rectangular";
        u_calibration:err_corr_dim1_name=["lat", "lon"];
        u_calibration:err_corr_dim1_form="systematic";
        u_calibration:err_corr_dim1_params=[];
        u_calibration:err_corr_dim1_units=[];
        u_calibration:err_corr_dim2_name="time";
        u_calibration:err_corr_dim2_form="random";
        u_calibration:err_corr_dim2_params=[];
        u_calibration:err_corr_dim2_units=[];
      float u_noise(time, lat, lon);
        u_calibration:err_corr_dim1_name=["time", "lat", "lon"];
        u_calibration:err_corr_dim1_form="random";
        u_calibration:err_corr_dim1_params=[];
        u_calibration:err_corr_dim1_units=[];



# Links

.. _VIM: https://jcgm.bipm.org/vim/en/index.html
.. _Uncertainty: https://jcgm.bipm.org/vim/en/2.26.html
.. _Error: https://jcgm.bipm.org/vim/en/2.16.html
.. _Coverage factor: https://jcgm.bipm.org/vim/en/2.38.html
.. _GUM: https://www.bipm.org/en/committees/jc/jcgm/publications
.. _NetCDF: https://www.unidata.ucar.edu/software/netcdf/
.. _Climate and Forecast (CF) conventions: https://cfconventions.org
.. _FIDUCEO: https://research.reading.ac.uk/fiduceo/