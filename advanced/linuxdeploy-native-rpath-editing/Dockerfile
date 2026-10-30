FROM public.ecr.aws/d3j8x8q7/olympus-base-cpp:latest

# package versions follow Debian bookworm's mirrors, which drop superseded versions at every point release
RUN apt-get update -o Acquire::Retries=5 \
    && apt-get install -y --no-install-recommends -o Acquire::Retries=5 \
        cimg-dev libjpeg-dev libpng-dev patchelf pkg-config fakeroot \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .

# the submodules at the commits the base commit records, and the AppImage excludelist at a pinned revision; every
# download is retried and time-limited
RUN set -e; \
    fetch() { \
        for attempt in 1 2 3 4 5; do \
            rm -rf "$1"; \
            if git init -q "$1" && timeout 300 git -C "$1" fetch -q --depth 1 "$2" "$3" && git -C "$1" checkout -q FETCH_HEAD; then \
                rm -rf "$1/.git"; \
                return 0; \
            fi; \
            sleep $((attempt * 10)); \
        done; \
        return 1; \
    }; \
    fetch lib/args https://github.com/Taywee/args.git 3de44ec671db452cc0c4ef86399b108939768abb; \
    fetch lib/cmake-scripts https://github.com/linuxdeploy/cmake-scripts.git df329871d82b9bbce76419aea59f595078dee024; \
    fetch lib/linuxdeploy-desktopfile https://github.com/linuxdeploy/linuxdeploy-desktopfile.git bea49b10f7ce1992b096209447e5d4afd7ee51c5; \
    fetch lib/linuxdeploy-desktopfile/lib/cmake-scripts https://github.com/linuxdeploy/cmake-scripts.git df329871d82b9bbce76419aea59f595078dee024; \
    fetch /tmp/appimages https://github.com/probonopd/AppImages.git 15a64c20dc23a0154622ba25829364323903b6b5

# configuring linuxdeploy downloads the excludelist to generate excludelist.h, which fails offline; a copy generated
# from the pinned revision on the default include path lets builds without network access find the header
RUN mkdir /tmp/excludelist \
    && cd /tmp/excludelist \
    && sed 's@wget --quiet "$url" -O -@cat /tmp/appimages/excludelist@' /app/src/core/generate-excludelist.sh > generate-excludelist.sh \
    && bash generate-excludelist.sh \
    && test -s excludelist.h \
    && install -m 644 excludelist.h /usr/local/include/excludelist.h \
    && rm -rf /tmp/excludelist /tmp/appimages \
    && chmod -R a+rwX /app

CMD ["/bin/bash"]
