FROM node:current-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev && npm install express
COPY . .
EXPOSE 3000
CMD [ "node", "app.js" ]
