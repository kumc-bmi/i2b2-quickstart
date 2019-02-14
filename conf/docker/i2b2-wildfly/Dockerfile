FROM jboss/wildfly:10.0.0.Final
LABEL maintainer=" <Kavishwar (Kavi)  Wagholikar waghsk@gmail.com>"

ENV DS_IP ${DS_IP:-i2b2-pg}
ENV DS_PORT ${DS_PORT:-5432}
ENV DS_TYPE ${DS_TYPE:-pg}
ENV DS_PASS ${DS_PASS:-demouser}

ENV ADMIN_USER ${ADMIN_USER:-admin}
ENV ADMIN_PASS ${ADMIN_USER:-demoadmin}

# Create management users 
RUN /opt/jboss/wildfly/bin/add-user.sh ${ADMIN_USER} ${ADMIN_PASS} --silent 

# "Because image size matters, using ADD to fetch packages from remote
#  URLs is strongly discouraged; you should use curl or wget instead."
#  -- https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy

# ant
WORKDIR /tmp
RUN curl -L -o apache-ant-1.9.13-bin.zip https://www-us.apache.org/dist//ant/binaries/apache-ant-1.9.13-bin.zip && \
    sha256sum apache-ant-1.9.13-bin.zip | grep 1b52929a4f5137faa95d9f05dd335c893a02a7569ff4ffcf89b55139d8779c39 && \
    unzip -q apache-ant-1.9.13-bin.zip && \
    rm apache-ant-1.9.13-bin.zip
ENV PATH="/tmp/apache-ant-1.9.13/bin:${PATH}"

# I2b2 WAR
RUN curl -L -o i2b2.tar.gz https://github.com/i2b2/i2b2-core-server/archive/v1.7.10.0001.tar.gz && \
    sha256sum i2b2.tar.gz | grep 9db4f95a35974e7f3aa959c8821b3ac35c277f159f93d7827e7788fff90a6b14 && \
    tar zxf i2b2.tar.gz && \
    rm i2b2.tar.gz
WORKDIR /tmp/i2b2-core-server-1.7.10.0001

RUN ( \
   cd edu.harvard.i2b2.server-common && \
   ant dist deploy jboss_pre_deployment_setup \
     -Djboss.home=/opt/jboss/wildfly/standalone)
RUN ( \
   cd edu.harvard.i2b2.pm && \
   ant -f master_build.xml build-all deploy \
     -Djboss.home=/opt/jboss/wildfly/standalone)
RUN ( \
  cd edu.harvard.i2b2.ontology && \
  ant -f master_build.xml clean build-all deploy \
     -Dedu.harvard.i2b2.ontology.applicationdir=/tmp \
     -Djboss.home=/opt/jboss/wildfly/standalone)
RUN ( \
  cd edu.harvard.i2b2.crc && \
  ant -f master_build.xml clean build-all deploy \
     -Dedu.harvard.i2b2.crc.applicationdir=/tmp \
     -Djboss.home=/opt/jboss/wildfly/standalone)
RUN ( \
  cd edu.harvard.i2b2.workplace && \
  ant -f master_build.xml clean build-all deploy \
     -Dedu.harvard.i2b2.workplace.applicationdir=/tmp \
     -Djboss.home=/opt/jboss/wildfly/standalone)

# App related configs
COPY --chown=jboss:jboss standalone/configuration /opt/jboss/wildfly/standalone/configuration/

# Wildfly configs
COPY --chown=jboss:jboss wfconfig /opt/jboss/wildfly/standalone/configuration/ 

# JDBC driver
COPY --chown=jboss:jboss dsconfig/${DS_TYPE}/driver  /opt/jboss/wildfly/modules/

# Expose the ports we're interested in
EXPOSE 9090 

# This will boot WildFly in the standalone mode and bind to all interface
CMD ["sh","/opt/jboss/wildfly/bin/standalone.sh","-c","standalone.xml","-b","0.0.0.0","-bmanagement", "0.0.0.0"]
