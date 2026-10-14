# ==========================================
# Stage 1: Build application with Gradle
# ==========================================
FROM eclipse-temurin:21-jdk AS builder

WORKDIR /app

# Copy Gradle wrapper and configuration files first to optimize Docker layer caching
COPY gradlew settings.gradle build.gradle ./
COPY gradle ./gradle

# Fix line endings (in case repository was cloned on Windows) and make gradlew executable
RUN sed -i 's/\r$//' ./gradlew && chmod +x ./gradlew

# Copy source code
COPY src ./src

# Build production Spring Boot executable JAR without running tests
# (Unit & integration tests are executed via './gradlew test')
RUN ./gradlew bootJar -x test --no-daemon

# ==========================================
# Stage 2: Lightweight runtime image
# ==========================================
FROM eclipse-temurin:21-jre

WORKDIR /app

# Best practice: create a dedicated non-root user and group
RUN groupadd -r campushub && useradd -r -g campushub -s /bin/false campushub

# Copy the built JAR artifact from builder stage
COPY --from=builder /app/build/libs/*.jar app.jar
RUN chown -R campushub:campushub /app

# Switch to non-root user
USER campushub

# Expose application port
EXPOSE 8080

# Set default JVM runtime options
ENV SERVER_PORT=8080 \
    JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

# Actuator healthcheck
HEALTHCHECK --interval=15s --timeout=5s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Launch application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
