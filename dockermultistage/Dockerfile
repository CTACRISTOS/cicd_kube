FROM openjdk:8 AS BUILD_IMAGE
RUN apt update && apt install maven -y
RUN git clone -b main https://github.com/CTACRISTOS/cicd_kube.git
RUN cd cicd_kube && mvn install

FROM tomcat:8-jre11

RUN rm -rf /usr/local/tomcat/webapps/*

COPY --from=BUILD_IMAGE cicd_kube/target/vprofile-0.0.1.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]