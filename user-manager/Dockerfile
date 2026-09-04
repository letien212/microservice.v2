# ==============================================================================
# Build Stage (Compile & Package)
# ==============================================================================
FROM maven:3.9-eclipse-temurin-17-alpine AS builder

WORKDIR /app

# Step 1: Build & Install shared library (common-lib)
COPY common-lib/common-lib ./common-lib
RUN cd common-lib && mvn clean install -DskipTests

# Step 2: Build executable user-manager service
COPY user-manager ./user-manager
RUN cd user-manager && mvn clean package -DskipTests

# ==============================================================================
# Production Runtime Stage (Lightweight & Secure)
# ==============================================================================
FROM eclipse-temurin:17-jre-alpine

RUN apk add --no-cache curl && \
    addgroup -S appgroup && \
    adduser -S appuser -G appgroup

WORKDIR /app

COPY --from=builder /app/user-manager/target/*.jar app.jar

RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 8001

HEALTHCHECK --interval=30s --timeout=5s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8001/actuator/health || exit 1

ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:+UseG1GC", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "app.jar"]
