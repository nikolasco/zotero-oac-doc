**************
Fragment Model
**************

Since OAC doesn't currently propose a model for segments (aka
fragments or regions), I suggest the following. I believe it's
reasonable in both representation and implementation (e.g. data types
map nicely into JavaScript and other languages).

Objects in the Model
====================

Points and Regions in Text Documents
------------------------------------

These rely on the W3C XPointer recommendation, which comes in four
parts:

* `Framework <http://www.w3.org/TR/xptr-framework/>`_
* `element() Scheme <http://www.w3.org/TR/xptr-element/>`_
* `xmlns() Scheme <http://www.w3.org/TR/xptr-xmlns/>`_
* `xpointer() Scheme <http://www.w3.org/TR/xptr-xpointer/>`_

.. ctype:: XPointer

  A string containing a valid XPointer description.

  Inspired by Zotero annotations.

.. class:: RangeXML

  Inspired by Zotero highlights

  .. cmember:: XPointer start

  .. cmember:: XPointer end

2D Regions
----------

Images and video can have regions/areas described by
a `simple closed curve
<http://www.mathwords.com/s/simple_closed_curve.htm>`_. The
representation chosen is primarily based on the `Basic Shapes section
<http://www.w3.org/TR/SVG11/shapes.html>`_ of the W3C `SVG 1.1
Specification <http://www.w3.org/TR/SVG11/>`_.

.. ctype:: Array

  An ordered list

.. ctype:: NonNegFloat

  A non-negative binary64 IEEE 754 floating point number, i.e. 0 or greater.

.. ctype:: PosFloat

  A positive binary64 IEEE 754 floating point number, i.e. 1 or
  greater.

.. class:: Fragment2D

  An empty class to group together descriptions of segments in
  two-dimensional space.

.. class:: Region2D : Fragment2D

  An empty class to group together descriptions of regions in
  two-dimensional space.

.. class:: Point2D : Fragment2D

  .. cmember:: NonNegFloat x

    The x-axis coordinate, in pixels.

  .. cmember:: NonNegFloat y

    The y-axis coordinate, in pixels.

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



Time
----

.. class:: Time

  .. cmember:: NonNegFloat t

    Number of seconds from the beginning of the work.

.. class:: Duration

    .. cmember:: Time begin

    .. cmember:: Time end


Combining Time and Space
------------------------

This representation is inspired by `BasicAnimation Module
 <http://www.w3.org/TR/SMIL3/smil-animation.html#animationNS-OverviewBasic>`_
 of the W3C `SMIL 3 Recommendation <http://www.w3.org/TR/SMIL3/>`_.

.. ctype:: Float

  A binary64 IEEE 754 floating point number.

.. class:: TemporaryFragment2D

  A region that is only present for part of a work.

  .. cmember:: Duration lifetime

  .. cmember:: Fragment2D region

.. class:: MovingFragment2D

  Describes a region that moves a certain amount over its
  lifetime. Intermediate positions are calculated by linear interpolation.

  .. cmember:: TemporaryFragment2D tempRegion

    The region to move over its lifetime.

  .. cmember:: Float changeX

  .. cmember:: Float changeY


Applicability of Segment Types
==============================

The :class:`PointXML` and :class:`RangeXML` must only be applied to
XML or HTML documents.

The :class:`Fragment2D` types must only be applied to images or
videos. :class:`Time` and :class:`Duration` must be only be applied to
audio or videos. :class:`TemporaryFragment2D` and
:class:`MovingFragment2D` must only be applied to videos.
