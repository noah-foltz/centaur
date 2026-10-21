# syntax=docker/dockerfile:1.7

FROM ironsh/iron-proxy:0.49.0@sha256:c4628019c24f4cc8d77564a26b7c9cedb00accee6f93d06270e85fb8f9c6a7da

USER root
# iron-proxy verifies upstream TLS against the system bundle
# (/etc/ssl/certs/ca-certificates.crt). ca-certificates >= 20261020-r0 distrusts
# the eMudhra-chained roots whose intermediate CA keys were disclosed on
# 2026-10-20; fail the build on an older bundle.
RUN --mount=type=cache,target=/var/cache/apk,sharing=locked \
    apk add --update-cache --cache-dir /var/cache/apk --upgrade curl jq ca-certificates ca-certificates-bundle \
    && apk info -e "ca-certificates>=20261020-r0" "ca-certificates-bundle>=20261020-r0"

COPY services/iron-proxy/iron-proxy.yaml /usr/local/share/iron-proxy/proxy.yaml.default
COPY --chmod=755 services/iron-proxy/entrypoint.sh /entrypoint.sh

EXPOSE 8080

ENTRYPOINT ["/entrypoint.sh"]
