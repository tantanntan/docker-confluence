FROM java:openjdk-8-jre
MAINTAINER Atlassian Confluence

# Setup useful environment variables
ENV CONFLUENCE_HOME     /var/atlassian/application-data/confluence
ENV CONFLUENCE_INSTALL  /opt/atlassian/confluence
ENV CONFLUENCE_CATALINA_MEM  1024
ENV CONF_VERSION  5.10.8
ENV JAVA_HOME           /usr/lib/jvm/java-8-openjdk-amd64/jre

ARG CONFLUENCE_HOSTNAME="localhost"
ARG TIMEZONE="Asia/Tokyo"

LABEL Description="This image is used to start Atlassian Confluence" Vendor="Atlassian" Version="${CONF_VERSION}"

ENV CONFLUENCE_DOWNLOAD_URL http://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-${CONF_VERSION}.tar.gz

# Use the default unprivileged account. This could be considered bad practice
# on systems where multiple processes end up being executed by 'daemon' but
# here we only ever run one process anyway.
ENV RUN_USER            daemon
ENV RUN_GROUP           daemon


# Install Atlassian Confluence and helper tools and setup initial home
# directory structure.
RUN set -x \
    && apt-get update --quiet \
    && apt-get install --quiet --yes --no-install-recommends libtcnative-1 xmlstarlet fonts-ipafont-gothic\
    && apt-get clean \
    && mkdir -p                           "${CONFLUENCE_HOME}" \
    && chmod -R 700                       "${CONFLUENCE_HOME}" \
    && chown ${RUN_USER}:${RUN_GROUP}     "${CONFLUENCE_HOME}" \
    && mkdir -p                           "${CONFLUENCE_INSTALL}/conf" \
    && curl -Ls                           "${CONFLUENCE_DOWNLOAD_URL}" | tar -xz --directory "${CONFLUENCE_INSTALL}" --strip-components=1 --no-same-owner \
    && chmod -R 700                       "${CONFLUENCE_INSTALL}/conf" \
    && chmod -R 700                       "${CONFLUENCE_INSTALL}/temp" \
    && chmod -R 700                       "${CONFLUENCE_INSTALL}/logs" \
    && chmod -R 700                       "${CONFLUENCE_INSTALL}/work" \
    && chown -R ${RUN_USER}:${RUN_GROUP}  "${CONFLUENCE_INSTALL}/conf" \
    && chown -R ${RUN_USER}:${RUN_GROUP}  "${CONFLUENCE_INSTALL}/temp" \
    && chown -R ${RUN_USER}:${RUN_GROUP}  "${CONFLUENCE_INSTALL}/logs" \
    && chown -R ${RUN_USER}:${RUN_GROUP}  "${CONFLUENCE_INSTALL}/work" \
    && echo -e                            "\nconfluence.home=${CONFLUENCE_HOME}" >> "${CONFLUENCE_INSTALL}/confluence/WEB-INF/classes/confluence-init.properties" \
    && xmlstarlet                         ed --inplace \
        --delete                          "Server/@debug" \
        --delete                          "Server/Service/Connector/@debug" \
        --delete                          "Server/Service/Connector/@useURIValidationHack" \
        --delete                          "Server/Service/Connector/@minProcessors" \
        --delete                          "Server/Service/Connector/@maxProcessors" \
        --delete                          "Server/Service/Engine/@debug" \
        --delete                          "Server/Service/Engine/Host/@debug" \
        --delete                          "Server/Service/Engine/Host/Context/@debug" \
                                          "${CONFLUENCE_INSTALL}/conf/server.xml" \
    && touch -d "@0"                      "${CONFLUENCE_INSTALL}/conf/server.xml"

# set timezone
RUN sed -i -e "s|Etc/UTC|${TIMEZONE}|" /etc/timezone

# add font for macro title's font
RUN mkdir -p "${JAVA_HOME}/lib/fonts/fallback"
RUN ln -s /usr/share/fonts/opentype/ipafont-gothic/ipag.ttf "${JAVA_HOME}/lib/fonts/fallback/ipag.ttf"

# set Confluence settings
COPY ./setenv.sh  ${CONFLUENCE_INSTALL}/bin/
COPY ./server.xml ${CONFLUENCE_INSTALL}/conf/
RUN sed -i -e "4s/localhost/${CONFLUENCE_HOSTNAME}/" ${CONFLUENCE_INSTALL}/conf/server.xml

# Use the default unprivileged account. This could be considered bad practice
# on systems where multiple processes end up being executed by 'daemon' but
# here we only ever run one process anyway.
USER ${RUN_USER}:${RUN_GROUP}

# Expose default HTTP connector port.
EXPOSE 8090

# Set volume mount points for installation and home directory. Changes to the
# home directory needs to be persisted as well as parts of the installation
# directory due to eg. logs.
VOLUME ["${CONFLUENCE_INSTALL}", "${CONFLUENCE_HOME}"]

# Set the default working directory as the Confluence installation directory.
WORKDIR ${CONFLUENCE_INSTALL}

# Run Atlassian Confluence as a foreground process by default.
CMD ["./bin/catalina.sh", "run"]
