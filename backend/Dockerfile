FROM node:22-alpine

RUN apk update && apk upgrade

RUN npm install -g npm@11.19.1

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY src ./src

EXPOSE 5000

CMD ["npm", "start"]
