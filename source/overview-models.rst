********************************
Overview of Data Models Involved
********************************

I've spent some time outlining what I know about Zotero and connecting
it with the OAC model. The results are below.


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
* accessDate

Importantly for OAC, no attachment (even notes) can be associated with
more than one item and items can not be connected to each other
(except via the many-to-many "related" relation). Highlights and
annotations address points and ranges (respectively) in HTML
documents, but can not be related to other items.

Notes and snapshots are the only things that have more than plain text
stored by Zotero. Notes can have HTML stored in the SQLite
database. Snapshots are stored as files.


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
work done in describing them in the URI (`Temporal fragments
<http://annodex.net/TR/draft-pfeiffer-temporal-fragments-03.html>`_ and
`W3C Media Fragments WG <http://www.w3.org/2008/WebVideo/Fragments/>`_.


Mapping between OAC and Zotero
==============================

Given the current state of OAC's model, I don't think it makes sense
to follow it closely.


Existing data in Zotero
-----------------------

For OAC annotation sources and targets, I think we only need to
support a subset of the possiblities within Zotero. As sources, Zotero
annotations (plain text) and notes (HTML) could be represented using
`UUID URNs <http://tools.ietf.org/html/rfc4122>`_ and using the
`Content in RDF <http://www.w3.org/TR/Content-in-RDF/>`_ vocabulary
with some MIME type association added on. All Zotero normal attachment items
can be supported as targets, using either the attachments URL field or
a file URL, depending on whether the attachment is a link or stored
file.

Note that my proposed representations as URIs do not include any
metadata. I believe that they are best represented as properties of
the OAC context. Some Zotero-specific vocabulary may need to be
created for properties that don't map to an existing term in OAC or a
related vocabulary.

Zotero annotations and highlights can be represented by a segment in
the context, with nothing (highlights) or plain text (annotations)
associated with them.


Data to be generated by AXE
---------------------------

AXE will primarily be used to generate fragment descriptions and
display media along with their fragments. OAC segments are currently
undefined, but I believe that the following model is reasonable, both
as a representation and as something to implement.
