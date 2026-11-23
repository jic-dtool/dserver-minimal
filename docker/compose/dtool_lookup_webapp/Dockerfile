FROM node:20-bookworm-slim

# Track the upstream default branches; override with a commit SHA to pin.
ARG CLIENT_JS_REF=main
ARG WEBAPP_REF=master

RUN apt-get update \
    && apt-get install --yes --no-install-recommends \
        git ca-certificates build-essential python3 \
    && rm -rf /var/lib/apt/lists/*

# The webapp has a local `file:` dependency on dserver-client-js, so that
# companion library is built first and linked in.
RUN git clone --depth 1 --branch ${CLIENT_JS_REF} \
        https://github.com/jic-dtool/dserver-client-js.git /dserver-client-js
WORKDIR /dserver-client-js
RUN npm install && npm run build

# Authentication is switched off at runtime via VUE_APP_AUTH_ENABLED=false
# (see envs/dtool_lookup_webapp). Rewrite the local dependency path and drop the
# committed lockfile, which pins dserver-client to the original relative path.
RUN git clone --depth 1 --branch ${WEBAPP_REF} \
        https://github.com/jic-dtool/dtool-lookup-webapp.git /app \
    && sed -i 's|file:../../dserver-client-js|file:/dserver-client-js|g' \
        /app/dtool-lookup-webapp/package.json \
    && rm -f /app/dtool-lookup-webapp/package-lock.json

WORKDIR /app/dtool-lookup-webapp
COPY compose/dtool_lookup_webapp/build-node-app.sh /build-node-app.sh
RUN chmod 775 /build-node-app.sh
# The script uses bash features (process substitution), so run it with bash.
ENTRYPOINT ["bash", "/build-node-app.sh"]
