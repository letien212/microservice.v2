# ==============================================================================
# Build Stage (Compile & Package)
# ==============================================================================
FROM maven:3.9-eclipse-temurin-17-alpine AS builder

WORKDIR /app

# Cache Maven dependencies layer
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code and build executable JAR
COPY src ./src
RUN mvn clean package -DskipTests

# ==============================================================================
# Production Runtime Stage (Lightweight & Secure)
# ==============================================================================
FROM eclipse-temurin:17-jre-alpine

# Security: Install curl for healthcheck & create non-root user
RUN apk add --no-cache curl && \
    addgroup -S appgroup && \
    adduser -S appuser -G appgroup

WORKDIR /app

# Copy built artifact from builder stage
COPY --from=builder /app/target/*.jar app.jar

# Adjust permissions for non-root execution
RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 8001

# Healthcheck for container orchestration
HEALTHCHECK --interval=30s --timeout=5s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8001/actuator/health || exit 1

# Production JVM Performance & Memory Tuning flags
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:+UseG1GC", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "app.jar"]
