FROM alpine/git as git

WORKDIR /home/node/
RUN git clone https://github.com/americanrepublicwiki-source/quartz.git

FROM node:22-slim AS builder
COPY --from=git /home/node/ /home/node/

WORKDIR /home/node/quartz
RUN npm ci

FROM node:22-slim
EXPOSE 8080
WORKDIR /home/node/
COPY --from=builder /home/node/ /home/node/
RUN rm -rf /home/node/quartz/content 
RUN mkdir -p /tmp/quartz
WORKDIR /home/node/quartz