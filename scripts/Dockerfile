FROM alpine:3.16 as builder
ARG THREADS="4"
ARG SHA=""

# build minimized
WORKDIR /
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories && apk update && \
    apk add --no-cache --virtual .build-tools git g++ build-base linux-headers cmake python3 && \
    apk add --no-cache --virtual .build-deps curl-dev rapidjson-dev libevent-dev pcre2-dev yaml-cpp-dev
RUN git clone https://github.com/ftk/quickjspp --depth=1 && \
    pushd quickjspp && \
    git submodule update --init && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make quickjs -j $THREADS && \
    install -d /usr/lib/quickjs/ && \
    install -m644 quickjs/libquickjs.a /usr/lib/quickjs/ && \
    install -d /usr/include/quickjs/ && \
    install -m644 quickjs/quickjs.h quickjs/quickjs-libc.h /usr/include/quickjs/ && \
    install -m644 quickjspp.hpp /usr/include && \
    popd
RUN git clone https://github.com/PerMalmberg/libcron --depth=1 && \
    pushd libcron && \
    git submodule update --init && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make libcron -j $THREADS && \
    install -m644 libcron/out/Release/liblibcron.a /usr/lib/ && \
    install -d /usr/include/libcron/ && \
    install -m644 libcron/include/libcron/* /usr/include/libcron/ && \
    install -d /usr/include/date/ && \
    install -m644 libcron/externals/date/include/date/* /usr/include/date/ && \
    popd
RUN git clone https://github.com/ToruNiina/toml11 --depth=1 && \
    pushd toml11 && \
    cmake -DCMAKE_CXX_STANDARD=11 . && \
    make install -j $THREADS && \
    popd

COPY ../ /subconverter
RUN pushd subconverter && \
    [ -n "$SHA" ] && sed -i 's/\(v[0-9]\.[0-9]\.[0-9]\)/\1-'"$SHA"'/' src/version.h;\
    python3 -m ensurepip && \
    python3 -m pip install gitpython && \
    python3 scripts/update_rules.py -c scripts/rules_config.conf && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make -j $THREADS && \
    mv subconverter /usr/bin && \
    mv base ../

FROM alpine
COPY --from=builder /base /
COPY --from=builder /usr/bin/subconverter /use/bin

WORKDIR /base
CMD subconverter
