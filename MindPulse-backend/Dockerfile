# Docker 镜像构建

FROM maven:3.8.1-jdk-8-slim as builder

#解决容器时期与真实时间相差8小时的问题
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/shanghai > /etc/timezone

# Copy local code to the container image.
WORKDIR /app
COPY pom.xml .
COPY src ./src

# Build a release artifact.
RUN mvn package -DskipTests

EXPOSE 8101

# Run the web service on container startup.
CMD ["java","-jar","/app/target/MindPulse-backend-0.0.1-SNAPSHOT.jar","--spring.profiles.active=prod"]