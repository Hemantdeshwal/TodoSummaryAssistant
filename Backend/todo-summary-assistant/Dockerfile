# Build stage
FROM maven:3.9.6-eclipse-temurin-17 AS build
WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline -B

COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Create non-root user
RUN addgroup -S app && adduser -S appuser -G app
USER appuser

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8000


ENTRYPOINT ["java","-jar","/app/app.jar"]
