.. _input-data:

Input data
==========

OGGM needs various data files to run. To date, **we rely exclusively on
open-access data that are all downloaded automatically for the user**. This
page explains the various ways OGGM relies upon to get the the data it needs.


System settings
---------------

OGGM implements a bunch of tools to make access to the input data as painless
as possible for you, including the automated download of all the required files.
This requires that you tell OGGM where to store these data.


Calibration data and testing: the ``~/.oggm`` directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At the first import, OGGM will create a cached ``.oggm`` directory in your
``$HOME`` folder. This directory contains all data obtained from the
`oggm sample data`_ repository. It contains several files needed only for
testing, but also some important files needed for calibration and validation.
For example:

- The CRU `baseline climatology`_ (CL v2.0, obtained from
  `crudata.uea.ac.uk/ <https://crudata.uea.ac.uk/cru/data/hrg/>`_ and prepared
  for OGGM),
- The `reference mass-balance data`_ from WGMS with
  `links to the respective RGI polygons`_,
- The `reference ice thickness data`_ from WGMS (`GlaThiDa`_ database).

.. _oggm sample data: https://github.com/OGGM/oggm-sample-data
.. _baseline climatology: https://github.com/OGGM/oggm-sample-data/tree/master/cru
.. _reference mass-balance data: https://github.com/OGGM/oggm-sample-data/tree/master/wgms
.. _links to the respective RGI polygons: http://fabienmaussion.info/2017/02/19/wgms-rgi-links/
.. _reference ice thickness data: https://github.com/OGGM/oggm-sample-data/tree/master/glathida
.. _GlaThiDa: http://www.gtn-g.ch/data_catalogue_glathida/

The ``~/.oggm`` directory should be updated automatically when you update OGGM,
but if you encounter any problems simply delete it (it will be
re-downloaded automatically at the next import).


All other data: auto-downloads and the ``~/.oggm_config`` file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Unlike runtime parameters (such as physical constants or working directories),
the input data is shared accross runs and even accross computers if you want
to. Therefore, the paths to previously downloaded data are stored in a
configuration file that you'll find in your ``$HOME`` folder:
the ``~/.oggm_config`` file.

The file should look like::

    dl_cache_dir = /path/to/download_cache
    dl_cache_readonly = False
    tmp_dir = /path/to/tmp_dir
    cru_dir = /path/to/cru_dir
    rgi_dir = /path/to/rgi_dir
    test_dir = /path/to/test_dir
    has_internet = True

Some explanations:

- ``dl_cache_dir`` is a path to a directory where *all* the files you
  downloaded will be cached for later use. Most of the users won't need to
  explore this folder (it is organized as a list of urls) but you have to make
  sure to set this path to a folder with sufficient disk space available. This
  folder can be shared across computers if needed. Once an url is stored
  in this cache folder, OGGM won't download it again.
- ``dl_cache_readonly`` indicates if writing is allowed in this folder (this is
  the default). Setting this to ``True`` will prevent any further download in
  this directory (useful for cluster environments, where this data might be
  available on a readonly folder).
- ``tmp_dir`` is a path to OGGM's temporary directory. Most of the topography
  files used by OGGM are downloaded and cached in a compressed (zip) format.
  They will be extracted in ``tmp_dir`` before use. OGGM will never allow more
  than 100 ``.tif`` files to exist in this directory by deleting the oldest ones
  following the rule of the `Least Recently Used (LRU)`_ item. Nevertheless,
  this directory might still grow to quite a large size. Simply delete it
  if you want to get this space back.
- ``cru_dir`` is the location where the CRU climate files are extracted. They
  are quite large! (approx. 6Gb)
- ``rgi_dir`` is the location where the RGI shapefiles are extracted.
- ``test_dir`` is the location where OGGM will write some of its output during
  tests. It can be set to ``tmp_dir`` if you want to, but it can also be
  another directory (for example a fast SSD disk). This folder shouldn't grow
  too large but here again, delete it if you need to.

.. note::

  ``tmp_dir``, ``cru_dir`` and ``rgi_dir`` can be overridden and set to a
  specific directory by defining an environment variable ``OGGM_EXTRACT_DIR``
  to a directory path. Similarly, the environment variables
  ``OGGM_DOWNLOAD_CACHE`` and ``OGGM_DOWNLOAD_CACHE_RO`` override the
  ``dl_cache_dir`` and ``dl_cache_readonly`` settings.

.. _Least Recently Used (LRU): https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_Recently_Used_.28LRU.29


Topography data
---------------

When creating a :ref:`glacierdir` a suitable topographical data source is
chosen automatically, depending on the glacier's location. Currently we use:

- the `Shuttle Radar Topography Mission`_ (SRTM) 90m Digital Elevation Database v4.1
  for all locations in the [60°S; 60°N] range
- the `Greenland Mapping Project`_ (GIMP) Digital Elevation Model
  for mountain glaciers in Greenland (RGI region 05)
- the `Radarsat Antarctic Mapping Project`_ (RAMP) Digital Elevation Model, Version 2
  for mountain glaciers in the Antarctic continent
  (RGI region 19 with the exception of the peripheral islands)
- the `Viewfinder Panoramas DEM3`_ products
  elsewhere (most notably: North America, Russia, Iceland, Svalbard)


.. _Shuttle Radar Topography Mission: http://srtm.csi.cgiar.org/
.. _Greenland Mapping Project: https://bpcrc.osu.edu/gdg/data/gimpdem
.. _Radarsat Antarctic Mapping Project: http://nsidc.org/data/nsidc-0082
.. _Viewfinder Panoramas DEM3: http://viewfinderpanoramas.org/dem3.html

These data are downloaded only when needed and stored in the ``dl_cache_dir``
directory. The gridded topography is then reprojected and resampled to the local
glacier map. The local grid is defined on a Transverse Mercator projection centered over
the glacier, and has a spatial resolution depending on the glacier size. The
default in OGGM is to use the following rule:

.. math::

    \Delta x = d_1 \sqrt{S} + d_2

where :math:`\Delta x` is the grid spatial resolution (in m),  :math:`S` the
glacier area (in km) and :math:`d_1`, :math:`d_2` some parameters (set to 14 and 10,
respectively). If the chosen spatial resolution is larger than 200 m
(:math:`S \ge` 185 km :math:`^{2}`) we clip it to this value.


.. ipython:: python
   :suppress:

    import json
    from oggm.utils import get_demo_file
    with open(get_demo_file('dem_sources.json'), 'r') as fr:
        DEM_SOURCE_INFO = json.loads(fr.read())
    # for k, v in DEM_SOURCE_INFO.items():
    #   print(v)

**Important:** when using these data sources for your OGGM runs, please refer
to the original data provider of the data! OGGM will add a ``dem_source.txt``
file in each glacier directory specifying how to cite these data. We
reproduce this information here:


SRTM V4
    Jarvis A., H.I. Reuter, A.  Nelson, E. Guevara, 2008, Hole-filled seamless SRTM data V4,
    International  Centre for Tropical  Agriculture (CIAT),
    available  from http://srtm.csi.cgiar.org.

RAMP V2
    Liu, H., K. C. Jezek, B. Li, and Z. Zhao. 2015.
    Radarsat Antarctic Mapping Project Digital Elevation Model, Version 2.
    Boulder, Colorado USA. NASA National Snow and Ice Data Center Distributed Active Archive Center.
    doi: https://doi.org/10.5067/8JKNEW6BFRVD.

GIMP V1.1
    Howat, I., A. Negrete, and B. Smith. 2014.
    The Greenland Ice Mapping Project (GIMP) land classification and surface
    elevation data sets, The Cryosphere. 8. 1509-1518.
    https://doi.org/10.5194/tc-8-1509-2014

VIEWFINDER PANORAMAS DEMs
    There is no recommended citation for these data. Please refer to the website above in case of doubt.

.. warning::

    A number of glaciers will still suffer from poor topographic information.
    Either the errors are large or obvious (in which case the model won't run),
    or they are left unnoticed. The importance of reliable topographic data for
    global glacier modelling cannot be emphasized enough, and it is a pity
    that no consistent, global DEM is yet available for scientific use.


Climate data
------------

The MB model implemented in OGGM needs monthly time series of temperature and
precipitation. The current default is to download and use the `CRU TS`_
data provided by the Climatic Research Unit of the University of East Anglia.

.. _CRU TS: https://crudata.uea.ac.uk/cru/data/hrg/


CRU (default)
~~~~~~~~~~~~~

If not specified otherwise, OGGM will automatically download and unpack the
latest dataset from the CRU servers.

.. warning::

    While the downloaded zip files are ~370mb in size, they are ~5.6Gb large
    after decompression!

The raw, coarse (0.5°) dataset is then downscaled to a higher resolution grid
(CRU CL v2.0 at 10' resolution) following the anomaly mapping approach
described by Tim Mitchell in his `CRU faq`_ (Q25). Note that we don't expect
this downscaling to add any new information than already available at the
original resolution, but this allows us to have an elevation-dependent dataset
based on a presumably better climatology. The monthly anomalies are computed
following Harris et al., (2010): we use standard anomalies for temperature and
scaled (fractional) anomalies for precipitation. At the locations where the
monthly precipitation climatology is 0 we fall back to the standard anomalies.

**When using these data, please refer to the original provider:**

Harris, I., Jones, P. D., Osborn, T. J., & Lister, D. H. (2014). Updated
high-resolution grids of monthly climatic observations - the CRU TS3.10 Dataset.
International Journal of Climatology, 34(3), 623–642. https://doi.org/10.1002/joc.3711

.. _CRU faq: https://crudata.uea.ac.uk/~timm/grid/faq.html


User-provided dataset
~~~~~~~~~~~~~~~~~~~~~

You can provide any other dataset to OGGM by setting the ``climate_file``
parameter in ``params.cfg``. See the HISTALP data file in the `sample-data`_
folder for an example.

.. _sample-data: https://github.com/OGGM/oggm-sample-data/tree/master/test-workflow

GCM data
~~~~~~~~

OGGM can also use climate model output to drive the mass-balance model. In
this case we still rely on gridded observations (CRU) for the baseline
climatology and apply the GCM anomalies computed from a preselected reference
period. This method is sometimes called the
`delta method <http://www.ciesin.org/documents/Downscaling_CLEARED_000.pdf>`_.

Currently we can process data from the
`CESM Last Millenium Ensemble <http://www.cesm.ucar.edu/projects/community-projects/LME/>`_
project (see :py:func:`tasks.process_cesm_data`), but adding other models
should be relatively easy.


Mass-balance data
-----------------

TODO