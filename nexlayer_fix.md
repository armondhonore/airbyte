# Pinned fix — airbyte (#151)

The deploy failed because nexlayer.yaml used airbyte/webapp:latest, which does NOT
exist on Docker Hub (airbyte publishes only versioned tags) -> image pull failure -> 503.

In airbyte 1.x the webapp image is a PURE STATIC nginx serving the React UI from
/usr/share/nginx/html on :8080 with the STOCK nginx default.conf. It has NO proxy
template and requires NO INTERNAL_API_HOST / server pod to start. A single pinned
webapp pod serves the UI at 2xx.

Pin a real tag (1.7.8). Do NOT revert to :latest. INTERNAL_API_HOST is unused in 1.x.

SCOPE: serves the airbyte WEB UI shell only. The full airbyte control plane
(airbyte-server + worker + temporal + bootloader + db + connector-builder) is a 6+ pod
stack that cannot run single-pod; in-browser API calls fail until that backend is
deployed separately. The web UI itself is 2xx per the "get the webapp serving" boundary.
