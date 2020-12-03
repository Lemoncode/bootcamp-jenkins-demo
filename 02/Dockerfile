FROM node:alpine3.12 as builder

WORKDIR /build

COPY ./src .

RUN npm install

RUN npm run build

FROM node:alpine3.12 as application

WORKDIR /opt/app

COPY ./src/package.json .

COPY ./src/package-lock.json .

COPY --from=builder /build/app .

RUN npm i --only=production

ENTRYPOINT ["node", "app.js"]