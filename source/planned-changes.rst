I've spent some time outlining what I know about Zotero and connecting
it with the OAC model. The results are below.

===========================
Zotero's current data model
===========================

My understanding of the relevant portions to Zotero's data model is:

Normal Standalone Items
  web page, article, video, whatever. I think of these as "works" (in
  the sense of copyright, a fixed, tangible representation). There's
  many potentially many key-value pairs associated with them (fields
  w/ data values).

Attachment Items
  snapshot, link (URL), stored copy. They must be connect to *exactly*
  one normal standalone item.

Note Items
  a special kind of attachment that can be detached. That is, they
  must be connected to *at most* one normal standalone item.

Highlights
  consist of start pointer and end pointer into the HTML. Connected to
  exactly one normal standalone item.


Annotation
  consist of an x,y position (where to display, always top left of
  placeholder),  row and column count (for size of text area), and an
  offset (pointer into HTML, for displaying placeholder when
  collapsed), and some text. Connected to exactly one normal
  standalone item.

All items have only have title, related (item), and tags. Notes have a
title, but when they're edited its set to the contents of the
note. Attachments have:
* title
* url (perhaps)

Importantly for OAC, no attachment (even notes) can be associated with
more than one item and items can not be connected to each other
(except via the many-to-many "related" relation). Highlights and
annotations address points and ranges (respectively) in HTML
documents, but can not be related to other items.

Notes and snapshots are the only things that have more than plain text
stored by Zotero. Notes can have HTML stored in the SQLite
database. Snapshots are stored as files.

================
OAC's data model
================

The OAC model remains poorly defined and is likely to
change. Currently, it represents annotations as a connection between
two fragments of works (with the entire work being the largest
fragment): a source (the content of the annotation) and a target
(the work that the annotation is about/annotating). The works must be
addressable by a URI; whether or not  there needs to be a resource
associated with that URI remains unclear.

OAC contexts consist of zero or more fragment descriptions (OAC
segment) and some metadata about the source/target URI; I suspect the
name "context" was chosen because URIs already have a fragment portion
(the part after '#'). In context of HTTP URIs, user agents don't send
to the fragment portion to the server.

.. note:: I believe that an OAC context with no OAC segments should be
  interpreted as referring to no particular part of the work, i.e. the
  whole thing.

In defining OAC segments, it appears that little (or no) consideration
has been given to other efforts to address fragments. The most
prominent of these is `XPointer
<http://www.w3.org/TR/2002/PR-xptr-framework-20021113/>`_, which is a
full-fledged W3C recommendation. There's also at least several
`existing RDF vocabularies <http://esw.w3.org/topic/W3PhotoVocabs>`_
for describing fragments of images and multimedia; there's also some
work done in describing them in the URI (`Temporal fragements
<http://annodex.net/TR/draft-pfeiffer-temporal-fragments-03.html>`_ and
`W3C Media Fragments WG <http://www.w3.org/2008/WebVideo/Fragments/>`_.

==============================
Implementing OAC within Zotero
==============================

Given the current state of OAC's model, I don't think it makes sense
to follow it closely.

-----------------------
Existing data in Zotero
-----------------------

For OAC annotation sources and targets, I think we only need to
support a subset of things within Zotero. As sources, Zotero
annotations (plain text) and notes (HTML) are relatively easy to
display and could be encoded as a data URI
(e.g. ``data:text/html:<b>hi</b>``); All Zotero normal attachment items
can be supported as targets, using either the attachments URL field or
a file URL, depending on whether the attachment is a link or stored
file.

Note that my proposed representations as URIs do not include any
metadata. I believe that they are best represented as properties of
the OAC context. Some Zotero-specific vocabularly may need to be
created for properties that don't map to an existing term in OAC or a
related vocabulary.

Zotero annotations can be represented by an segment in the context (as
an XPointer) and a source URI of type ``text/plain``. Zotero highlights can be
represented as a pair of XPointers (start and end) and a source URI of
``about:blank``.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Points and Regions in Documents
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. ctype:: XPointer

  A string containing a valid XPointer description.

  Used to represent Zotero annotations.

.. class:: RangeXML

  Used to represent Zotero highlights

  .. cmember:: XPointer start

  .. cmember:: XPointer end

---------------------------
Data to be generated by AXE
---------------------------

AXE will primarily be used to generate fragment descriptions and
display media along with their fragments. OAC segments are currently
undefined, but I believe that the following model is reasonable, both
as a representation and as something to implement.

^^^^^^^^^^
2D Regions
^^^^^^^^^^

Images and video can have regions/areas described by
a `simple closed curve
<http://www.mathwords.com/s/simple_closed_curve.htm>`_. Types of curves
to support are:

.. ctype:: Array

  An ordered list

.. ctype:: NonNegFloat

  A non-negative binary64 IEEE 754 floating point number, i.e. 0 or greater.

.. ctype:: PosFloat

  A positive binary64 IEEE 754 floating point number, i.e. 1 or
  greater.

.. class:: Region2D

  An empty class to group together descriptions of regions in
  two-dimensional space.

.. class:: Point2D

  .. cmember:: NonNegFloat x

    The x-axis coordinate, in pixels.

  .. cmember:: NonNegFloat y

    The y-axis coordinate, in pixels.

.. note:: :class:`Point2D` must not be used as an OAC segment.

.. class:: Rectangle : Region2D

  .. cmember:: Point2D topLeft

    The point with the top-left coordinate of the square,
    i.e. smallest x-axis and y-axis coordinates.

  .. cmember:: PosFloat width, in pixels.

  .. cmember:: PosFloat height, in pixels.

.. note:: Squares are just rectangles with equal width and height.

.. class:: Circle : Region2D

  .. cmember:: Point2D center

    The center of the circle.

  .. cmember:: PosFloat rx

    The radius of the circle, in pixels.

.. class:: Ellipse : Region2D

  .. cmember:: Point2D center

    The center of the ellipse.

  .. cmember:: PosFloat rx

    The x-axis radius of the ellipse, in pixels.

  .. cmember:: PosFloat ry

    The y-axis radius of the ellipse, in pixels.

.. note:: Circles could just ellipses with equal x-axis radius and
  y-axis radius.

.. class:: Polygon : Region2D

  .. cmember:: Array points

    The points of the polygon's vertices. Must have at least three
    elements. All elements must be distinct and of type :ctype:`Point2D`.

    The sides of the polygon are formed by adjacent points in the
    array. A pair of points  are adjacent to one another if they are
    at positions x and x+1 in the array, or they are the first and
    last elements. No side should intersect any other side.


^^^^^
Time
^^^^^

.. class:: Time

  .. cmember:: NonNegFloat t

    Number of seconds from the beginning of the work.

.. class:: Duration

    .. cmember:: Time begin

    .. cmember:: Time end

^^^^^^^^^^^^^^^^^^^^^^^^
Combining Time and Space
^^^^^^^^^^^^^^^^^^^^^^^^

.. ctype:: Float

  A binary64 IEEE 754 floating point number.

.. class:: TemporaryRegion2D

  A region that is only present for part of a work.

  .. cmember:: Duration lifetime

  .. cmember:: Region2D region

.. class:: MovingRegion2D

  Describes a region that moves a certain amount over its
  lifetime. Intermediate positions are calculated by linear interpolation.

  .. cmember:: TemporaryRegion2D tempRegion

    The region to move over its lifetime.

  .. cmember:: Float changeX

  .. cmember:: Float changeY

----------------------
Fragments, old and new
----------------------

^^^^^^^^^^^^^^^^^^^^^^
Internal Serialization
^^^^^^^^^^^^^^^^^^^^^^
I believe that fragment descriptions should be stored in a serialized
JSON form. In addition to the members above, each object will contain
the name of its class.

(would version numbers make sense?)

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Valid Applications of Fragment Descriptions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The :class:`PointXML` and :class:`RangeXML` must only be applied to
XML or HTML documents.

The :class:`Region2D` types must only be applied to images or
videos. :class:`Time` and :class:`Duration` must be only be applied to
audio or videos. :class:`TemporaryRegion2D` and
:class:`MovingRegion2D` must only be applied to videos.

^^^^^^^
Display
^^^^^^^

Each fragment class will implement the following interface:

.. class:: Fragment

  .. method:: show()

  .. method:: hide()

  .. method:: display(source)