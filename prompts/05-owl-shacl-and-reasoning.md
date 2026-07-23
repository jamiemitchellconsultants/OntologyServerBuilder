# Prompt 5 — OWL, SHACL, and build-time reasoning

Using the previously supplied repository contract, implement Stage 5: standards-based RDF
artifacts and bounded build-time reasoning.

Use N3 for RDF construction and Turtle serialization.

Add `ontology/owl-axioms.json` with governed support for:

- `rdfs:subClassOf`;
- `owl:equivalentClass`;
- `rdfs:subPropertyOf`;
- `owl:equivalentProperty`;
- `owl:inverseOf`;
- `owl:TransitiveProperty`.

Implement a fixed-point materializer inspired by OWL 2 RL, but do not claim complete OWL
compliance.

Rules must include:

- transitive subclass closure;
- equivalent classes as subclass rules in both directions;
- transitive subproperty closure;
- equivalent properties in both directions;
- inverse properties;
- transitive property paths;
- accepted mappings as source-to-canonical subclass premises.

Every inferred JSON edge must record:

- `source: "owl-inference"`;
- rule used;
- premise IDs;
- inherited confidence, using the minimum premise confidence.

Keep inferred edges in a separate `ontology.json` `inferences` collection. Make them available to
RDF and SPARQL, but do not add superclass inferences to ordinary relationship-path adjacency. Shared
superclass membership must not create misleading business paths.

Generate deterministic:

- `ontology/compiled/ontology.owl.ttl`
- `ontology/compiled/shapes.ttl`

OWL must represent systems, classes, datatype properties, object properties, mappings, governed
axioms, relationships, confidence, provenance, and inference evidence.

SHACL must represent entity node shapes, attribute datatypes, required attributes, and governed
relationship constraints.

Extend `ontology:check` to verify all four generated artifacts.

Document the exact supported profile and deliberate exclusions in `docs/owl-profile.md`.

Add tests that parse generated Turtle, exercise every supported rule, prove fixed-point behavior,
and reject axioms referencing unknown classes.

Run `npm run check` and commit the stage locally.
