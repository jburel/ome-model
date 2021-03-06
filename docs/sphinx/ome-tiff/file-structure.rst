OME-TIFF file structure
=======================

The following section discusses various considerations when designing
OME-TIFF filesets distributed over multiple files.

Single versus multiple files
----------------------------

Splitting a fileset across multiple files can have advantages in terms of
acquisition but also processing purposes. The OME-TIFF file format can support
any image organization. However, using one TIFF file per timepoint per channel
with the focal planes for that timepoint and channel stored sequentially
within the TIFF makes for very easy creation of :ref:`TiffData <TiffData>`
elements (see :ref:`tubhiswt_samples`).

The main downside of splitting OME-TIFF over multiple files is their
inherent fragility to common file-system operations such as file renaming or
file copying which have the potential to "break" the fileset.

See :doc:`data` for examples of single OME-TIFF file versus OME-TIFF
distributed over multiple files.

File size
---------

The TIFF file format internally uses 32-bit byte offsets. The largest offset
which can be represented is 4GB, making this value the upper limit of the file
size supported by the design.

The OME-TIFF file format supports the BigTIFF file extensions allowing the
use of 64-bit byte offsets to overcome this size limit.

The main limitation of the BigTIFF file extension is the degree of its
adoption in the community. Although OME Bio-Formats and OMERO will handle
OME-TIFF using the BigTIFF file extension, other tools might not be able to
open it.

Metadata redundancy
-------------------

Storing multiple planes per OME-TIFF file and keeping the OME-XML metadata
embedded in every file header is recommended (see the
:ref:`tubhiswt_samples`) but is not always practical.
Normally, the OME-XML metadata block is small in comparison with the binary
pixel data in the file, but in some cases, it may be disproportionately
larger.

Common reasons for this situation include:

- storing each image plane in its own TIFF file
- having a large amount of metadata such as plane-specific timestamps
- having many structured annotations in the OME-XML.

For example, if you have a dataset with 1,000 time points, with each plane
recorded at 512x512 as uint8 pixel type, storing each plane in its own file
uncompressed requires ~256KB of disk per file, and ~256MB total. But if you
have 5MB of corresponding OME-XML metadata, embedding a copy of that metadata
in every file would result in a dataset nearly 20 times larger than before,
requiring ~5.5MB of disk per file, and more than 5GB total.

One of the advantages of reproducing the OME-XML metadata across files is
redundancy. If one of the constituent files survives data loss, the metadata
survives. The space tradeoff of this duplication is acceptable when compared
to the bulk of the pixel data in most cases, provided a suitable number of
planes are stored in each TIFF.

Companion file vs master OME-TIFF file
--------------------------------------

Since the :doc:`2011-06 version</schemas/june-2011>` of the OME-XML schema, it
is possible to store partial OME-XML metadata blocks in some or all of the
TIFF files pointing to a master file containing the full OME-XML metadata (see
the :ref:`technical specification<binary_only>` for more details).

The master file can be either a master OME-TIFF file  (see the
:ref:`master-sample`) or a companion OME-XML file (see the
:ref:`companion-sample`).

Using a companion OME-XML file allows information that can only be generated
at the end of the acquisition to be easily appended without the need to
manipulate existing TIFF IFDs. It has the same drawbacks as those
highlighted above in the sense that this companion file needs to be preserved
as part of the fileset to prevent metadata loss.

.. seealso::

   :ome-devel:`Proposed tweak to µManager data files in 2.0 <2016-April/003618.html>`
      Community discussion about usage of companion file in OME-TIFF filesets
