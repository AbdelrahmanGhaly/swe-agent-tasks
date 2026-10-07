FROM public.ecr.aws/d3j8x8q7/olympus-base-jvm:latest

WORKDIR /app
COPY . .

RUN sed -i 's/\r$//' gradlew && chmod +x gradlew \
    && GRADLE_OPTS=-Dorg.gradle.daemon.registry.base=/tmp/gradle-daemon ./gradlew --no-daemon assemble compileTestJava \
    && rm -rf /opt/gradle-cache/daemon \
    && chmod -R a+rwX /app /opt/gradle-cache

CMD ["/bin/bash"]
