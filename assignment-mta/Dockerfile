FROM node:8-alpine

EXPOSE 3000

RUN apk --update add graphicsmagick

WORKDIR /app

COPY package*.json ./

RUN npm install && npm cache clean --force

COPY . .

ENV CHARCOAL_FACTOR=0.1

CMD ["node", "index.js"]