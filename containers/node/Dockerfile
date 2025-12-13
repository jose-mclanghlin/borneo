FROM node:20.18.0-alpine3.20

RUN addgroup -S company && adduser -S company -G company
USER company

RUN mkdir "/home/company/.npm-packages" && \
    npm config set prefix "/home/company/.npm-packages"

ENV NPM_PACKAGES="/home/company/.npm-packages"
ENV NODE_PATH="/home/company/.npm-packages/lib/node_modules"
ENV PATH="$PATH:$NPM_PACKAGES/bin"

RUN npm install -g @opentelemetry/api@1.9.0  && \
    npm install -g @opentelemetry/auto-instrumentations-node@0.52.0 \
    npm install -g @opentelemetry/winston-transport@0.7.0
