FROM alpine:3.18.5

# Set default ports.
ENV HTTP_PORT 80
ENV HTTPS_PORT 443
ENV RTMP_PORT 1935

RUN apk add --no-cache \
  ca-certificates \
  gettext \
  openssl \
  pcre \
  curl \
  alsa-utils

# Cleanup.
RUN rm -rf /var/cache/* /tmp/*

RUN mkdir /usr/lib/pulseaudio
RUN mkdir /usr/local/nginx
RUN mkdir /etc/nginx

COPY ./build/usr_local_nginx/. /usr/local/nginx
COPY ./build/etc_nginx/. /etc/nginx
COPY ./build/ffmpeg/ffmpeg /usr/local/bin/ffmpeg
COPY ./build/ffmpeg/ffprobe /usr/local/bin/ffprobe
COPY ./build/usr_lib/. /usr/lib
COPY ./build/usr_local_lib/. /usr/local/lib
COPY ./build/usr_lib_pulseaudio/. /usr/lib/pulseaudio

## Set permissions
RUN chmod 775 /usr/local/nginx/sbin/nginx && \
    chmod 775 /usr/local/bin/ffmpeg && \
    chmod 775 /usr/local/bin/ffprobe

RUN chmod +x /usr/local/nginx/sbin/nginx && \
    chmod +x /usr/local/bin/ffmpeg && \
    chmod +x /usr/local/bin/ffprobe

## Create nginx logs
RUN touch /usr/local/nginx/logs/access.log && \
    touch /usr/local/nginx/logs/error.log

# Add NGINX path, config and static files.
ENV PATH "${PATH}:/usr/local/nginx/sbin"
COPY ./build/nginx.conf /etc/nginx/nginx.conf.template
RUN mkdir -p /opt/data && mkdir /www
COPY ./build/static /www/static

## nginx custom folder permissions for user nobody
RUN mkdir /opt/data/recordings && \
    mkdir /opt/data/hls && \
    mkdir /opt/data/logs

RUN chown nobody.65534 -R /opt/data
RUN chmod 775 -R /opt/data

EXPOSE 1935
EXPOSE 80
EXPOSE 443

CMD envsubst "$(env | sed -e 's/=.*//' -e 's/^/\$/g')" < \
  /etc/nginx/nginx.conf.template > /etc/nginx/nginx.conf && \
  nginx