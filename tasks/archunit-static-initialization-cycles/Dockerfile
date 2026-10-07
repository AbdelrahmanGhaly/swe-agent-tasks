FROM public.ecr.aws/d3j8x8q7/olympus-base-jvm:latest
WORKDIR /app
COPY . .
RUN printf 'org.gradle.java.installations.paths=/opt/jdk-25\norg.gradle.java.installations.auto-download=false\n' >> /opt/gradle-cache/gradle.properties \
    && printf 'allprojects {\n    tasks.register("resolveAllClasspaths") {\n        doLast {\n            project.configurations.findAll { it.canBeResolved && it.name.endsWith("Classpath") }.each { configuration ->\n                configuration.incoming.artifactView { lenient(true) }.files.files\n            }\n        }\n    }\n}\n' > /tmp/resolve-classpaths.gradle \
    && ./gradlew --no-daemon testClasses \
    && ./gradlew --no-daemon -I /tmp/resolve-classpaths.gradle resolveAllClasspaths \
    && rm -rf /tmp/resolve-classpaths.gradle /opt/gradle-cache/daemon /opt/gradle-cache/.tmp /opt/gradle-cache/workers /opt/gradle-cache/notifications \
    && chmod -R a+rwX /opt/gradle-cache /app
CMD ["/bin/bash"]
