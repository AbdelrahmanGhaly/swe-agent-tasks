FROM public.ecr.aws/d3j8x8q7/olympus-base-cpp:latest

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        cimg-dev libjpeg-dev libpng-dev patchelf pkg-config fakeroot \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

RUN set -e; \
    fetch() { rm -rf "$1"; git init -q "$1"; git -C "$1" fetch -q --depth 1 "$2" "$3"; git -C "$1" checkout -q FETCH_HEAD; rm -rf "$1/.git"; }; \
    fetch lib/args https://github.com/Taywee/args.git 3de44ec671db452cc0c4ef86399b108939768abb; \
    fetch lib/cmake-scripts https://github.com/linuxdeploy/cmake-scripts.git df329871d82b9bbce76419aea59f595078dee024; \
    fetch lib/linuxdeploy-desktopfile https://github.com/linuxdeploy/linuxdeploy-desktopfile.git bea49b10f7ce1992b096209447e5d4afd7ee51c5; \
    fetch lib/linuxdeploy-desktopfile/lib/cmake-scripts https://github.com/linuxdeploy/cmake-scripts.git df329871d82b9bbce76419aea59f595078dee024

# configuring linuxdeploy downloads the excludelist to generate excludelist.h, which fails offline; a copy generated
# from a pinned revision on the default include path lets builds without network access find the header
RUN mkdir /tmp/excludelist \
    && cd /tmp/excludelist \
    && sed 's#/AppImages/master/excludelist#/AppImages/15a64c20dc23a0154622ba25829364323903b6b5/excludelist#' \
        /app/src/core/generate-excludelist.sh > generate-excludelist.sh \
    && bash generate-excludelist.sh \
    && install -m 644 excludelist.h /usr/local/include/excludelist.h \
    && rm -rf /tmp/excludelist

RUN cmake -S . -B /tmp/warmup -DBUILD_TESTING=ON -DUSE_CCACHE=OFF -DCMAKE_POLICY_VERSION_MINIMUM=3.5 -DCMAKE_BUILD_TYPE=Release \
    && cmake --build /tmp/warmup -j"$(nproc)" \
    && rm -rf /tmp/warmup \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
