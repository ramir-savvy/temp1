FROM node:18-bullseye as node_builder
WORKDIR /build
COPY ["./", "./"]
RUN addgroup --system unbiased && adduser unbiased --shell /bin/false --system --ingroup unbiased
RUN chown unbiased:unbiased -R /build
RUN DEBIAN_FRONTEND=noninteractive apt update && apt install -y git-lfs && git lfs install
USER unbiased
ARG INFRA_TOKEN
RUN git config --global url."https://${INFRA_TOKEN}:@github.com".insteadOf "ssh://git@github.com"
RUN echo "@unbiased-security:registry=https://npm.pkg.github.com\n//npm.pkg.github.com/:_authToken=${INFRA_TOKEN}" > .npmrc
RUN npm install
RUN npm run build

FROM node:21-bullseye as modules
WORKDIR /modules
COPY ["./", "./"]
RUN addgroup --system unbiased && adduser unbiased --shell /bin/false --system --ingroup unbiased
RUN chown unbiased:unbiased -R /modules
RUN DEBIAN_FRONTEND=noninteractive apt update && apt install -y git-lfs && git lfs install
USER unbiased
ARG INFRA_TOKEN
RUN echo "@unbiased-security:registry=https://npm.pkg.github.com\n//npm.pkg.github.com/:_authToken=${INFRA_TOKEN}" > .npmrc
RUN git config --global url."https://${INFRA_TOKEN}:@github.com".insteadOf "ssh://git@github.com"
RUN npm install --production

FROM node:21-bullseye as prod_container
ENV NODE_ENV=production
RUN addgroup --system unbiased && adduser unbiased --shell /bin/false --system --ingroup unbiased

WORKDIR /app
COPY ["./*.json", "./"]

COPY --chown=unbiased:unbiased deployments/docker-entrypoint.sh /
COPY --chown=unbiased:unbiased --from=modules /modules/node_modules ./node_modules
COPY --chown=unbiased:unbiased --from=node_builder /build/out ./out/

ENTRYPOINT ["/docker-entrypoint.sh"]
