# ==========================================
# Stage 1: Build the Spring Boot Application
# ==========================================
FROM maven:3.9.6-eclipse-temurin-21-alpine AS builder
WORKDIR /build

# Copy the entire backend source code
COPY . .

# Build JAR without running unit tests
RUN mvn clean package -DskipTests -B

# Extract JAR layers for optimized container startup & caching
WORKDIR /build/target
RUN java -Djarmode=layertools -jar *.jar extract

# ==========================================
# Stage 2: Hardened Production Runtime
# ==========================================
FROM eclipse-temurin:21-jre-alpine AS runtime
WORKDIR /app

# Security Hardening: Update OS packages & install curl for healthchecks
RUN apk update && apk upgrade --no-cache && \
    apk add --no-cache curl && \
    rm -rf /var/cache/apk/*

# Security Hardening: Create non-root group and user (UID/GID 10001)
RUN addgroup -g 10001 -S appgroup && \
    adduser -u 10001 -S appuser -G appgroup -H -s /sbin/nologin

# Copy extracted JAR layers from builder with non-root ownership
COPY --from=builder --chown=appuser:appgroup /build/target/dependencies/ ./
COPY --from=builder --chown=appuser:appgroup /build/target/spring-boot-loader/ ./
COPY --from=builder --chown=appuser:appgroup /build/target/snapshot-dependencies/ ./
COPY --from=builder --chown=appuser:appgroup /build/target/application/ ./

# Switch to non-root user
USER 10001:10001

EXPOSE 8080

ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/./urandom"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS org.springframework.boot.loader.launch.JarLauncher"]