FROM nginxinc/nginx-unprivileged:1.31-alpine

LABEL org.opencontainers.image.source="https://github.com/Osman-SK/home-cluster"
LABEL org.opencontainers.image.description="K8s cluster status dashboard"
LABEL org.opencontainers.image.licenses="MIT"

COPY index.html /usr/share/nginx/html/index.html
EXPOSE 8080
