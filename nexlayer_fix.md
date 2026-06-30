# Pinned fix — airbyte (#151)

Two failures fixed:
1. nexlayer.yaml used airbyte/webapp:latest which does NOT exist on Docker Hub
   (airbyte publishes only versioned tags) -> image pull failure -> 503. Pinned 1.7.8.
2. The webapp nginx template (/etc/nginx/templates/default.conf.template) declares three
   upstreams that nginx resolves AT STARTUP and hard-fails ("host not found in upstream")
   if unset:
     upstream api-server            { server $AIRBYTE_SERVER_HOST; }
     upstream connector-builder-server { server $CONNECTOR_BUILDER_API_HOST; }
     upstream keycloak              { server $KEYCLOAK_INTERNAL_HOST; }
   We point all three at the webapp's own pod (app.pod:8080) so nginx starts and serves
   the static React UI at "/" (HTTP 200). /api/ and /auth/ then proxy back to this same
   static nginx (harmless 404s) — fine for the UI shell.

Keep image pinned to a real tag and keep the three *_HOST vars resolvable. Do NOT use :latest.

SCOPE: serves the airbyte WEB UI shell only (the task's "get the webapp serving" goal).
The full airbyte control plane (airbyte-server + worker + temporal + bootloader + db +
connector-builder + keycloak) is a 7+ pod stack that cannot run single-pod; in-browser
API/login calls will not function until that backend is deployed separately. Pointing the
three upstreams at a real airbyte-server/keycloak/connector-builder set would light up the
API, but that backend is out of scope for a single-pod UI deploy.
