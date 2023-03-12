FROM node:16-alpine

WORKDIR /app
COPY package.json .
COPY package-lock.json .
# See the differences between this and npm install here: https://docs.npmjs.com/cli/v8/commands/npm-ci
RUN npm ci
COPY . .

CMD ["npm", "start"]
