# ==============================================================================
# Build Stage (Compile & Package)
# ==============================================================================
FROM maven:3.9-eclipse-temurin-17-alpine AS builder

WORKDIR /app

# Copy and build config-server service
COPY config-server ./config-server
RUN cd config-server && mvn clean package -DskipTests

# ==============================================================================
# Production Runtime Stage (Lightweight & Secure)
# ==============================================================================
FROM eclipse-temurin:17-jre-alpine

RUN apk add --no-cache curl && \
    addgroup -S appgroup && \
    adduser -S appuser -G appgroup

WORKDIR /app

COPY --from=builder /app/config-server/target/*.jar app.jar

RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 6969

HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:6969/actuator/health || exit 1

ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:+UseG1GC", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "-jar", "app.jar"]
