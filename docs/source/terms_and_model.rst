Terminology & Information Model
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

When biologists define terms in order to describe phenomena and
observations, they rely on a background of human experience and
intelligence for interpretation. Definitions may be abstract, perhaps
correctly reflecting uncertainty of our understanding at the
time. Unfortunately, such terms are not readily translatable into an
unambiguous representation of knowledge.

For example, "allele" might refer to "an alternative form of a gene or
locus" [`Wikipedia`_], "one of two or more forms of the DNA sequence
of a particular gene" [`ISOGG`_], or "one of a set of coexisting
sequence alleles of a gene" [`Sequence Ontology`_]. Even for human
interpretation, these definitions are inconsistent: does the
definition precisely describe a specific change on a specific
sequence, or, rather, a more general change on an undefined sequence?
In addition, all three definitions are inconsistent with the practical
need for a way to describe sequence changes outside regions associated
with genes.

**The computational representation of biological concepts requires
translating precise biological definitions into data structures that
can be used by implementers.** This translation should result in a
representation of information that is consistent with conventional
biological understanding and, ideally, be able to accommodate future
data as well. The resulting *computational representation* of
information should also be cognizant of computational performance, the
minimization of opportunities for misunderstanding, and ease of
manipulating and transforming data.

Accordingly, for each term we define below, we begin by describing the
term as used by biologists (**biological definition**) as
available. When a term has multiple biological definitions, we
explicitly choose one of them for the purposes of this
specification. We then provide a computer modelling definition
(**computational definition**) that reformulates the biological
definition in terms of information content. We then translate each of
these computational definitions into precise specifications for the
(**logical model**). Terms are ordered "bottom-up" so that definitions
depend only on previously-defined terms.

.. note:: The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
          NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
          "OPTIONAL" in this document are to be interpreted as
          described in `RFC 2119`_.


Primitive Concepts
@@@@@@@@@@@@@@@@@@

.. _id:

Id
##

**Biological definition**

None.

**Computational definition**

A `CURIE-formatted <curie-spec>`_ string that uniquely identifies a
specific instance of an object within a document.

**Implementation guidance**

* This specification RECOMMENDS using a :ref:`computed-identifier` for each id.
* When an appropriate namespace exists at identifiers.org, that
  namespace MUST be used verbatim (case sensitive).


.. _residue:

Residue
#######

**Biological definition**

A residue refers to a specific `monomer`_ within the `polymeric
chain`_ of a `protein`_ or `nucleic acid`_ (Source: `Wikipedia Residue
page`_).

**Computational definition**

A character representing a specific residue (i.e., molecular species)
or groupings of these ("ambiguity codes"), using `one-letter IUPAC
abbreviations <https://www.genome.jp/kegg/catalog/codes1.html>`_ for
nucleic acids and amino acids.


.. _interbase-coordinates:

Interbase Coordinates
#####################

**Biological definition**

None.

**Computational definition**

Interbase coordinates refer to the zero-width points before and after :ref:`residues <Residue>`. An interval of interbase coordinates permits referring to any span, including an empty span, before, within, or after a sequence. See :ref:`interbase-coordinates-design` for more details on this design choice.


.. _sequence:

Sequence
########

**Biological definition**

A contiguous, linear polymer of nucleic acid or amino acid residues.

**Computational definition**

A character string of :ref:`Residues <Residue>` that represents a
biological sequence using the conventional sequence order (5'-to-3'
for nucleic acid sequences, and amino-to-carboxyl for amino acid
sequences). IUPAC ambiguity codes are permitted in Sequences.

**Information model**

A Sequence is a string, constrained to contain only characters representing IUPAC nucleic acid or
amino acid codes.

**Implementation guidance**

* Sequences MAY be empty (zero-length) strings. Empty sequences are used as the replacement Sequence
  for deletion Alleles.
* Sequences MUST consist of only uppercase IUPAC abbreviations, including ambiguity codes.

**Notes**

* A Sequence provides a stable coordinate system by which an :ref:`Allele` may be located and
  interpreted.
* A Sequence may have several roles. A “reference sequence” is any Sequence used to define an
  :ref:`Allele`. A Sequence that replaces another Sequence is called a “replacement sequence”.
* In some contexts outside the VR specification, “reference sequence” may refer to a member of set
  of sequences that comprise a genome assembly. In the VR specification, any sequence may be a
  “reference sequence”, including those in a genome assembly.
* For the purposes of representing sequence variation, it is not
  necessary that Sequences be explicitly “typed” (i.e., DNA, RNA, or
  AA).

Composite Concepts
@@@@@@@@@@@@@@@@@@

.. _interval:

Interval
########

**Biological definition**

None.

**Computational definition**

The *Interval* abstract class defines a range on a :ref:`sequence`,
possibly with length zero, and specified using
:ref:`interbase-coordinates`. An Interval may be a
:ref:`SimpleInterval` with a single start and end coordinate.
:ref:`Future Location and Interval types <planned-locations>` will
enable other methods for describing where :ref:`variation` occurs. Any
of these may be used as the Interval for Location.

.. _SimpleInterval:

SimpleInterval
$$$$$$$$$$$$$$

**Computational definition**

An :ref:`Interval` with a single start and end coordinate.

**Information model**

.. csv-table::
   :header: Field, Type, Label, Description
   :align: left

   type, string, required, Interval type; must be set to 'SimpleInterval'
   start, uint64, required, start position
   end, uint64, required, end position

**Implementation guidance**

* Implementations MUST require that 0 ≤ start ≤ end. In the case of double-stranded DNA, this constraint holds even when a feature is on the complementary strand.

**Notes**

* VR uses Interbase coordinates because they provide conceptual
  consistency that is not possible with residue-based systems (see
  :ref:`rationale <interbase-coordinates-design>`). Implementations
  will need to convert between interbase and 1-based inclusive
  residue coordinates familiar to most human users.
* Interbase coordinates start at 0 (zero).
* The length of an interval is *end - start*.
* An interval in which start == end is a zero width point between two residues.
* An interval of length == 1 may be colloquially referred to as a position.
* Two intervals are *equal* if the their start and end coordinates are equal.
* Two intervals *intersect* if the start or end coordinate of one is
  strictly between the start and end coordinates of the other. That
  is, if:

   * b.start < a.start < b.end OR
   * b.start < a.end < b.end OR
   * a.start < b.start < a.end OR
   * a.start < b.end < a.end
* Two intervals a and b *coincide* if they intersect or if they are
  equal (the equality condition is required to handle the case of two
  identical zero-width Intervals).

**Examples**

* <start, end>=<*0,0*> refers to the point with width zero before the first residue.
* <start, end>=<*i,i+1*> refers to the *i+1th* (1-based) residue.
* <start, end>=<*N,N*> refers to the position after the last residue for Sequence of length *N*.
* See example notebooks in the :ref:`reference implementation documentation <impl-vr-python>`.


.. _state:

State
#####

**Biological definition**

None.

**Computational definition**

*State* objects are one of two primary components specifying a VR
:ref:`Allele` (in addition to :ref:`Location`), and the designated
components for representing change (or non-change) of the features
indicated by the Allele Location. As an abstract class, State may
encompass concrete :ref:`sequence` changes (see :ref:`SequenceState
<sequence-state>`), complex translocations, copy number changes,
expression variation, rule-based variation, and more (see
:ref:`planned-states`).

.. _sequence-state:

SequenceState
$$$$$$$$$$$$$

**Biological definition**

None.

**Computational definition**

The *SequenceState* class specifically captures a :ref:`sequence` as a
:ref:`State`. This is the State class to use for representing
"ref-alt" style variation, including SNVs, MNVs, del, ins, and delins.

**Information model**

.. csv-table::
   :header: Field, Type, Label, Description
   :align: left
   :widths: 12, 9, 10, 30

   id, :ref:`Id`, optional, State Id; must be unique within document
   type, string, required, State type; must be set to 'SequenceState'
   sequence, :ref:`Sequence`, required, The sequence that is to be used as the state for other types.

**Examples**

* See example usage in the `reference implementation documentation <impl-vr-python>`.

.. _location:

Location
########

**Biological definition**

As used by biologists, the precision of “location” (or “locus”) varies
widely, ranging from precise start and end numerical coordinates
defining a Location, to bounded regions of a sequence, to conceptual
references to named genomic features (e.g., chromosomal bands, genes,
exons) as proxies for the Locations on an implied reference sequence.

**Computational definition**

The `Location` abstract class refers to position of a contiguous
segment of a biological sequence.  The most common and concrete
Location is a :ref:`sequence-location`, i.e., a Location based on a
named sequence and an Interval on that sequence. Additional
:ref:`planned-locations` may also be conceptual or symbolic locations,
such as a cytoband region or a gene. Any of these may be used as the
Location for Variation.

**Implementation Guidance**

* Location refers to a position.  Although it may imply a sequence,
  the two concepts are not interchangable, especially when the
  location is non-specific (e.g., a range) or symbolic (a gene).


.. _sequence-location:

SequenceLocation
$$$$$$$$$$$$$$$$

**Biological definition**

None

**Computational definition**

A Location subclass for describing a defined :ref:`Interval` over a
named :ref:`Sequence`.

**Information model**

.. csv-table::
   :header: Field, Type, Label, Description
   :align: left
   :widths: 12, 9, 10, 30

   id, :ref:`Id`, optional, Location Id; must be unique within document
   type, string, required, Location type; must be set to 'SequenceLocation'
   sequence_id, :ref:`Id`, required, An id mapping to the Identifier of the external database Sequence
   interval, :ref:`Interval`, required, Position of feature on reference sequence specified by sequence_id.

**Implementation guidance**

* For a :ref:`Sequence` of length *n*:
   * 0 ≤ *interval.start* ≤ *interval.end* ≤ *n*
   * interbase coordinate 0 refers to the point before the start of the Sequence
   * interbase coordinate n refers to the point after the end of the Sequence.
* Coordinates MUST refer to a valid Sequence. VR does not support
  referring to intronic positions within a transcript sequence,
  extrapolations beyond the ends of sequences, or other implied
  sequence.

.. important:: HGVS permits variants that refer to non-existent
               sequence. Examples include coordinates extrapolated
               beyond the bounds of a transcript and intronic
               sequence. Such variants are not representable using VR
               and must be projected to a genomic reference in order
               to be represented.

.. _variation:

Variation
#########

**Biological definition**

In biology, variation is often used to mean `genetic variation`_,
describing the differences observed in DNA among individuals.

**Computational definition**

The *Variation* abstract class is the top-level object in the
:ref:`vr-schema-diagram` and represents the concept of a molecular
state. The representation and types of molecular states are widely
varied, and there are several :ref:`planned-variation` currently under
consideration to capture this diversity. The primary Variation
subclass defined by the VR |version| specification is the
:ref:`Allele`, with the :ref:`text` subclass for capturing other
Variations that are not yet covered.

.. _allele:

Allele
$$$$$$

**Biological definition**

One of a number of alternative forms of the same gene or same genetic
locus. In the context of biological sequences, “allele” refers to one
of a set of specific changes within a :ref:`Sequence`. In the context
of VR, Allele refers to a Sequence or Sequence change with respect to
a reference sequence, without regard to genes or other features.

**Computational definition**

An Allele is a specific, single, and contiguous :ref:`Sequence` at a
:ref:`Location`. Each alternative Sequence may be empty, shorter,
longer, or the same length as the interval (e.g., due to one or more
indels).

**Information model**

.. csv-table::
   :header: Field, Type, Label, Description
   :align: left
   :widths: 12, 9, 10, 30

   id, :ref:`Id`, optional, Variation Id; must be unique within document
   type, string, required, Variation type; must be set to 'Allele'
   location, :ref:`Location`, required, Where Allele is located
   state, :ref:`State`, required, State at location

**Implementation guidance**

* Implementations MUST require that interval.end ≤ sequence_length
  when the Sequence length is known.
* Implementations MAY choose to provide a mechanism for ensuring that
  the type of sequence and the content of the state are compatible, but
  such behavior is not provided by the specification.
* Alleles are equal only if the component fields are equal: at the
  same location and with the same state.
* Alleles may have multiple related representations on the same
  Sequence type due to normalization differences.
* Implementations SHOULD normalize Alleles using :ref:`"justified"
  normalization <normalization>` whenever possible to facilitate
  comparisons of variation in regions of representational ambiguity.
* Implementations MUST normalize Alleles using :ref:`"justified"
  normalization <normalization>` when generating a
  :ref:`computed-identifier`.

**Notes**

* When the alternate Sequence is the same length as the interval, the
  lengths of the reference Sequence and imputed Sequence are the
  same. (Here, imputed sequence means the sequence derived by applying
  the Allele to the reference sequence.) When the replacement Sequence
  is shorter than the length of the interval, the imputed Sequence is
  shorter than the reference Sequence, and conversely for replacements
  that are larger than the interval.
* When the replacement is “” (the empty string), the Allele refers to
  a deletion at this location.
* The Allele entity is based on Sequence and is intended to be used
  for intragenic and extragenic variation. Alleles are not explicitly
  associated with genes or other features.
* Biologically, referring to Alleles is typically meaningful only in
  the context of empirical alternatives. For modelling purposes,
  Alleles may exist as a result of biological observation or
  computational simulation, i.e., virtual Alleles.
* “Single, contiguous” refers the representation of the Allele, not
  the biological mechanism by which it was created. For instance, two
  non-adjacent single residue Alleles could be represented by a single
  contiguous multi-residue Allele.
* The terms "allele" and "variant" are often used interchangeably,
  although this use may mask subtle distinctions made by some users.

   * In the genetics community, "allele" may also refer to a
     haplotype.
   * "Allele" connotes a state whereas "variant" connotes a change
     between states. This distinction makes it awkward to use variant
     to refer to the concept of an unchanged position in a Sequence
     and was one of the factors that influenced the preference of
     “Allele” over “Variant” as the primary subject of annotations.
   * See :ref:`Use “Allele” rather than “Variant” <use-allele>` for
     further details.
* When a trait has a known genetic basis, it is typically represented
  computationally as an association with an Allele.
* This specification's definition of Allele applies to all Sequence
  types (DNA, RNA, AA).


.. _text:

Text
$$$$

**Biological definition**

None

**Computational definition**

The *Text* subclass of :ref:`Variation` is intended to capture textual
descriptions of variation that cannot be parsed by other Variation
subclasses, but are still treated as variation.

**Information model**

.. csv-table::
   :header: Field, Type, Label, Description
   :align: left
   :widths: 12, 9, 10, 30

   id, :ref:`Id`, optional, Variation Id; must be unique within document
   type, string, required, Variation type; must be set to 'Text'
   definition, string, required, The textual variation representation not parsable by other subclasses of Variation.

**Implementation guidance**

* An implementation MUST represent Variation with subclasses other
  than Text if possible.
* An implementation SHOULD define or adopt conventions for defining
  the strings stored in Text.definition.
* If a future version of VR-Spec is adopted by an implementation and
  the new version enables defining existing Text objects under a
  different Variation subclass, the implementation MUST construct a
  new object under the other Variation subclass. In such a case, an
  implementation SHOULD persist the original Text object and respond
  to queries matching the Text object with the new object.

**Notes**

* Additional Variation subclasses are continually under
  consideration. Please open a `GitHub issue`_ if you would like to
  propose a Variation subclass to cover a needed variation
  representation.

.. _GitHub issue: https://github.com/ga4gh/vr-spec/issues
.. _genetic variation: https://en.wikipedia.org/wiki/Genetic_variation
