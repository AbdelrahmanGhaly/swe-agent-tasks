FROM public.ecr.aws/d3j8x8q7/olympus-base-typescript:latest
WORKDIR /app
ENV NODE_ENV=development
ENV NPM_CONFIG_CACHE=/tmp/.npm
COPY . .
# The test/messageformat-wg submodule carries a nested CLDR submodule whose
# keyboard-abnf tooling ships its own vitest files and its own package.json.
# The root workspace never installs those dependencies (abnf, fast-xml-parser),
# and vitest.config.mts only excludes mf1/examples and node_modules, so `npm test`
# picks the files up and fails to resolve them. The MF2 conformance vectors this
# repo actually uses live in test/messageformat-wg/test, which is untouched.
RUN rm -rf test/messageformat-wg/publish
# build:esm is the repository's own `pretest` step: mf1/packages/core and
# mf1/packages/runtime tests import the runtime's esm output, and 16 of them fail
# without it. It is a required runtime artifact, not a convenience build.
# npm ci is reproducible here: package-lock.json is committed at the repo root.
# The base image defines no unprivileged user and the platform runs the container
# under an arbitrary uid, so /app is made group and world writable rather than
# chowned. vitest needs to write node_modules/.vite, and the repo's own core
# tests write temporary .mjs files next to their sources. Narrowing this to a
# named user is not possible at build time and locks out the solve-time uid.
RUN npm ci --include=dev --no-audit --no-fund \
    && npm run build:esm -w mf1/packages/runtime \
    && mkdir -p /app/node_modules/.vite \
    && chmod -R a+rwX /app \
    && chmod a+rwx /app
CMD ["/bin/bash"]
