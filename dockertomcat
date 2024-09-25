#Base image
FROM tomcat:8.0

#COPY
ADD ./target/LoginWebApp.war /usr/local/tomcat/webapps/

WORKDIR /usr/local/tomcat/webapps/

CMD ["catalina.sh", "run"]