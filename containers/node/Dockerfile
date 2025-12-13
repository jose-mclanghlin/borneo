FROM node:20.18.0-alpine3.20

RUN addgroup -S plub && adduser -S plub -G plub
USER plub

RUN mkdir "/home/plub/.npm-packages" && \
    npm config set prefix "/home/plub/.npm-packages"

ENV NPM_PACKAGES="/home/plub/.npm-packages"
ENV NODE_PATH="/home/plub/.npm-packages/lib/node_modules"
ENV PATH="$PATH:$NPM_PACKAGES/bin"

RUN npm install -g @opentelemetry/api@1.9.0  && \
    npm install -g @opentelemetry/auto-instrumentations-node@0.52.0 \
    npm install -g @opentelemetry/winston-transport@0.7.0
