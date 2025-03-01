FROM tomcat:10-jdk21
RUN rm -rf /usr/local/tomcat/webapps/*
RUN mv /target/*.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
