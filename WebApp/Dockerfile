FROM public.ecr.aws/nginx/nginx:latest

EXPOSE 80

RUN apt-get update -y && \
    apt-get upgrade -y && \
    apt-get install -y curl awscli && \
    rm -rf /var/lib/apt/lists/*

COPY ./default.conf /etc/nginx/conf.d/default.conf
COPY ./index.html /usr/share/nginx/html/index.html
COPY ./images /usr/share/nginx/html/images

CMD ["nginx", "-g", "daemon off;"]
