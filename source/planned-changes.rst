***************
Planned Changes
***************

Database Schema
===============

We'll need to add a 'uuid' column to itemNotes. Also, the following
will be new tables:

simpleTextItems
  itemID

  content
    the plain text content

  uuid
    UUID to be used in exporting

.. note:: should UUIDs be stored in columns or as a field (in
  itemDataValues)? I dunno.

oacAnnotations
  id
    for attaching contexts

  targetID
    points to an itemID

  targetCtxID
    points to a oacContext's ID

  sourceID
    points to an itemID

  sourceCtxID
    points to a oacContexts's ID

oacContexts
  id
    for attaching segments/fragments

  fixity
    perhaps a hash of the target?

oacSegments
  id

  ctxID
    points at a oacContexts's ID
    


Segment JSON
============
I believe that fragment descriptions should be stored in a serialized
`JSON <http://tools.ietf.org/html/rfc4627>`_ form. In addition to the
members above, each object will contain the name of its class.

(would version numbers make sense?)

Display
=======

Each segment class will implement something like the following interface:

.. class:: Segment

  Constructor gets the item it will be working with and source
  items to display arround it (e.g. plain text and/or HTML notes).

  .. method:: show()

    Show this segment in some way (outline of area, markers on timeline, etc.)

  .. method:: hide()

    Hides this segment

  .. method:: getSources()

    Lists any sources the user might have added

A side-effect is that new segment types can be added without modifying
the database schema.
