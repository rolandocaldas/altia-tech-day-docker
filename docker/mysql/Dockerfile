FROM mysql:5.7
ARG LOCAL_UID
ARG LOCAL_GID
RUN usermod -u ${LOCAL_UID:-999} mysql && groupmod -g ${LOCAL_GID:-999} mysql