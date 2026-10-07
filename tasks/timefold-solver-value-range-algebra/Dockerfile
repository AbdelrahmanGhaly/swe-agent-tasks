FROM public.ecr.aws/d3j8x8q7/olympus-base-jvm:latest

ENV JAVA_HOME=/opt/jdk-25
ENV PATH=/opt/jdk-25/bin:${PATH}
ENV MVN_FLAGS="-B -Dmaven.repo.local=/opt/m2 -Dspotless.skip=true -Denforcer.skip=true -Dmaven.gitcommitid.skip=true"

WORKDIR /app
COPY . .

RUN mvn $MVN_FLAGS -DskipTests -pl build/ide-config -am install \
    && mvn $MVN_FLAGS -DskipTests -pl core -am install \
    && mvn $MVN_FLAGS -pl core dependency:resolve-plugins \
    && mvn $MVN_FLAGS dependency:get -Dartifact=org.apache.maven.surefire:surefire-junit-platform:3.5.6 \
    && mvn $MVN_FLAGS dependency:get -Dartifact=org.apache.maven.surefire:surefire-booter:3.5.6 \
    && mvn $MVN_FLAGS dependency:get -Dartifact=org.apache.maven.surefire:maven-surefire-common:3.5.6 \
    && find /app -type d -name target -prune -exec rm -rf {} + \
    && chmod -R a+rwX /opt/m2 /app

RUN groupadd -g 1000 model && useradd -u 1000 -g model -m model && chmod -R a+rwX /app /opt/m2

CMD ["/bin/bash"]
