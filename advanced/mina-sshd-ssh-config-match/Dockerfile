FROM public.ecr.aws/d3j8x8q7/olympus-base-jvm:latest

# ssh_config(5) for the ssh the base image ships: the same openssh-client version is reinstalled, letting only that
# page through the image's man-page exclusion
RUN apt-get -o Acquire::Retries=10 update \
    && apt-get -o Acquire::Retries=10 install -y --no-install-recommends --reinstall \
        -o Dpkg::Options::=--path-include=/usr/share/man/man5/ssh_config.5.gz \
        openssh-client=1:9.2p1-2+deb12u9 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . .
ENV MAVEN_OPTS="-Dmaven.repo.local=/opt/m2repo"

# sshd-common and its parent are built and its tests compiled (none run); the surefire JUnit provider for the
# project's junit-jupiter 5.14.2 is fetched so test.sh runs offline
RUN mvn -B -ntp -DskipTests -Dformatter.skip=true -Dimpsort.skip=true -Dcheckstyle.skip=true -Dpmd.skip=true \
        -Drat.skip=true -Danimal.sniffer.skip=true -Denforcer.skip=true -Dassembly.skipAssembly=true \
        -pl sshd-common -am install \
    && mvn -B -ntp dependency:get -Dartifact=org.apache.maven.surefire:surefire-junit-platform:3.5.4 \
    && mvn -B -ntp dependency:get -Dartifact=org.junit.platform:junit-platform-launcher:1.14.2 \
    && rm -rf /app/target /app/*/target \
    && chmod -R a+rwX /opt/m2repo /app

CMD ["/bin/bash"]
