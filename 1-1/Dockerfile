FROM nginx:latest

LABEL maintainer="iyeonjae"
LABEL description="E1-1 과제용 커스텀 NGINX 웹 서버"

ENV AUTHOR="iyeonjae" \
    COURSE="DevOps E1-1"

COPY src/index.html /usr/share/nginx/html/index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
