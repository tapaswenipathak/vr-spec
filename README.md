The [GA4GH](https://www.ga4gh.org/) [Variation Representation
Specification](https://vr-spec.readthedocs.io/) provides a
comprehensive framework for the computational representation of
biological sequence variation.  VR-Spec is the result of a
collaboration among [contributors](CONTRIBUTORS.md) representing
national information resource providers, major international public
initiatives, and diagnostic testing laboratories.


# Specific goals:

* Develop common language- and protocol-neutral information models and
  nomenclature for biological sequence variation.
* From the information models, develop data schemas.  The current
  schema is defined in JSON Schema, but other formats are expected.
* Provide algorithmic guidance and conventions to minimize
  representational ambiguity.
* Define a globally unique *computed identifier* for covered data
  classes.
* Develop a [reference
  implementation](https://github.com/ga4gh/vr-python) to facilitate
  adoption and [validation
  tests](https://github.com/ga4gh/vr-spec/tests/validation) to ensure
  consistency of implementations.

The VR model is the product of the GA4GH `Variation Representation
<https://ga4gh-gks.github.io/variant_representation.html>`__ group.



# Using the schema

The schema is available in the `schema/` directory, in both yaml and
json versions.  It conforms to json schema draft-07.  For a list of
libraries that support json schema, see [JSON
Schema>Implementations](https://json-schema.org/implementations.html).



# Contributing to the schema

VR uses yaml as the source document for json schema

To convert vr.yaml to vr.json:

    make vr.json

You'll probably have to `pip install pyyaml` first.

To watch for changes and update automatically:

    make watch &

