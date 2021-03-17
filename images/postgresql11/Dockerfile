FROM debian:10 as builder
RUN apt-get -y update -qq \
    && apt-get -y install \
        -o APT::Install-Recommends="false" \
        -o APT::Install-Suggests="false" \
        # Build tools from sources.
        build-essential \
        g++ \
        cmake \
        git \
        wget \
        gnupg

RUN sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt buster-pgdg main" > /etc/apt/sources.list.d/pgdg.list' \
    && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
    && apt-get update \
    && apt install -y postgresql-server-dev-12

ENV NOMINATIM_VERSION v3.6.0

RUN true \
    && git clone \
        --config advice.detachedHead=false \
        --single-branch \
        --branch $NOMINATIM_VERSION \
        --depth 1 \
        --recursive \
        https://github.com/openstreetmap/Nominatim \
        src \
    && cd src \
    && mkdir build \
    && cd build \
    && cmake .. \
    && make -j`nproc` \
    && ./utils/setup.php --setup-website \
    && mv ./module/nominatim.so website/nominatim.so \
    && chmod o=rwx .


FROM bitnami/postgresql:11
COPY --from=builder nominatim.so /opt/bitnami/postgresql/lib/nominatim.so
