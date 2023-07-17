FROM public.ecr.aws/nginx/nginx:1-alpine-perl

# Copy files and directories from the application
COPY default.conf /etc/nginx/conf.d/default.conf
COPY index.html favicon.ico Rakefile /usr/share/nginx/html/
COPY style/ /usr/share/nginx/html/style/
COPY meta/ /usr/share/nginx/html/meta/
COPY js/ /usr/share/nginx/html/js/

EXPOSE 8080
