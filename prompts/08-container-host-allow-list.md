# Prompt 8 — Correct the container host allow-list

Using the previously supplied repository contract, implement the first post-audit correction: make
the shipped container start safely with its documented local command.

Read before editing:

- `Dockerfile`
- `src/server.ts`
- `README.md`
- the HTTP transport and host-validation tests

Confirm the failure mode first. The container binds the HTTP server to `0.0.0.0`, while the server
must refuse a public bind when `ALLOWED_HOSTS` is absent. Do not remove or weaken that protection.

Implement this decision:

- Keep `HOST=0.0.0.0` in the container.
- Give the image an explicit local-development default of
  `ALLOWED_HOSTS=localhost,127.0.0.1`.
- Preserve the requirement that shared deployments replace that default with their real
  client-facing hostnames.
- Keep matching port-agnostic and accept a comma-separated allow-list.
- Do not treat the allow-list as authentication or authorization.

Expand the README with:

- a working `docker build` command;
- a working local `docker run` command;
- the MCP and health URLs;
- an example overriding `ALLOWED_HOSTS` behind a gateway or reverse proxy;
- a warning that TLS, authentication, and authorization still belong at the gateway.

Add or update tests if the environment-default or validation behavior changes. Build the image when
Docker is available; otherwise state that limitation explicitly.

## Acceptance criteria

- The documented local container command starts without disabling host validation.
- Requests with an unlisted `Host` value are still rejected.
- A public bind without an explicit or image-provided allow-list still fails closed.
- `npm run check` and `git diff --check` pass.
- Generated ontology artifacts have no unexplained diff.

Classify this as a meaningful correction for Project Narrative. Prepare substantive Narrative
Context, Narrative Decision, and Narrative Consequences if opening a pull request.

Commit locally with a focused correction message. Do not push.
