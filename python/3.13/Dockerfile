FROM alpine:3.20

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH
ENV GPG_KEY 7169605F62C751356D054A26A821E680E5FA6305
ENV PYTHON_VERSION 3.13.0

# Set shell
SHELL ["/bin/ash", "-o", "pipefail", "-c"]

RUN set -ex \
    && export PYTHON_VERSION=${PYTHON_VERSION} \
    && apk add --no-cache --virtual .fetch-deps \
        gnupg \
        openssl \
        tar \
        xz \
        curl \
        ca-certificates \
        tzdata \
    \
    && curl -L -o python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
    && curl -L -o python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && echo "disable-ipv6" >> "$GNUPGHOME/dirmngr.conf" \
    && gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys "${GPG_KEY}" \
    && gpg --batch --verify python.tar.xz.asc python.tar.xz \
    && { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
    && rm -rf "$GNUPGHOME" python.tar.xz.asc \
    && mkdir -p /usr/src/python \
    && tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
    && rm python.tar.xz \
    \
    && apk add --no-cache --virtual .build-deps  \
        patch \
        bzip2-dev \
        coreutils \
        dpkg-dev dpkg \
        expat-dev \
        findutils \
        build-base \
        gdbm-dev \
        libc-dev \
        libffi-dev \
        libnsl-dev \
        openssl \
        openssl-dev \
        libtirpc-dev \
        linux-headers \
        make \
        mpdecimal-dev \
        ncurses-dev \
        pax-utils \
        readline-dev \
        sqlite-dev \
        tcl-dev \
        tk \
        tk-dev \
        xz-dev \
        zlib-dev \
        bluez-dev \
    # add build deps before removing fetch deps in case there's overlap
    && apk del .fetch-deps \
    ;

RUN set -eux \
    # && for i in /usr/src/*.patch; do \
    #     patch -d /usr/src/python -p 1 < "${i}"; done \
    && cd /usr/src/python \
    && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
    && ./configure \
        --build="$gnuArch" \
        --enable-loadable-sqlite-extensions \
        --enable-optimizations \
        --enable-option-checking=fatal \
        --enable-shared \
        --with-lto \
        --with-system-libmpdec \
        --with-system-expat \
        --without-ensurepip \
        --without-static-libpython \
	; \
	nproc="$(nproc)"; \
	EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000"; \
	LDFLAGS="${LDFLAGS:--Wl},--strip-all"; \
	make -j "$nproc" \
		"EXTRA_CFLAGS=${EXTRA_CFLAGS:-}" \
		"LDFLAGS=${LDFLAGS:-}" \
		"PROFILE_TASK=${PROFILE_TASK:-}" \
	;
    # make -j "$(nproc)" \
        # LDFLAGS="-Wl,--strip-all" \
#         CFLAGS="-fno-semantic-interposition -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free" \
# set thread stack size to 1MB so we don't segfault before we hit sys.getrecursionlimit()
# https://github.com/alpinelinux/aports/commit/2026e1259422d4e0cf92391ca2d3844356c649d0
        # EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000"

CMD ["python3"]
