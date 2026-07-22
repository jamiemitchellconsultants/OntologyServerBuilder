# Prompt 2 — Heterogeneous source ingestion

Using the previously supplied repository contract, implement Stage 2: deterministic adapters for
heterogeneous system definitions.

Support checked-in snapshots in these formats:

- OpenAPI, JSON or YAML;
- JSON Schema;
- XSD;
- WSDL/SOAP;
- CSV data dictionary;
- XLSX data dictionary, first worksheet only;
- exported MCP catalog containing tools, resources, schemas, or explicit entities;
- normalized JSON entity definitions.

Each system must have an `ontology/systems/<system>.system.json` manifest containing:

- stable system ID;
- name and description;
- source format;
- source path relative to the manifest;
- source version.

Every extracted entity and attribute must retain provenance: system, source path, format, and
version.

## Security and determinism requirements

- Never fetch remote references.
- Resolve only local checked-in files.
- Impose sensible source and XLSX decompression limits.
- Reject invalid manifests, duplicate IDs, and unsupported types.
- Sort extracted output deterministically.
- Never ingest credentials, live responses, or transaction data.

Add representative fixtures using different formats:

- Procurement OpenAPI with PurchaseOrder.
- Billing exported MCP catalog with Invoice.
- Treasury CSV with PaymentInstruction.
- ERP WSDL with a payment-related entity.
- Normalized JSON for the canonical ontology.

Add parser tests for each format and malformed or oversized input.

Run the build and tests, then commit this stage locally. Do not push.
