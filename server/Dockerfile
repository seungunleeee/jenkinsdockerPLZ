FROM node:latest
# set working directory
WORKDIR /

copy package*.json ./


# update apt packages   
# ARG env
# ENV APP_ENV=$env 
CMD ["echo", 'we re in build docker file']
RUN npm install
# RUN npm install -g typescript
COPY . ./
RUN npm run build

EXPOSE 80

CMD [ "node","./dist/index.js" ]
