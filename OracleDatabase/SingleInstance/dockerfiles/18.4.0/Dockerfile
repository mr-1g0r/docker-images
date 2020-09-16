# LICENSE UPL 1.0
#
# Copyright (c) 2018, 2020 Oracle and/or its affiliates.
#
# ORACLE DOCKERFILES PROJECT
# --------------------------
# This is the Dockerfile for Oracle Database 18c Express Edition
# 
# REQUIRED FILES TO BUILD THIS IMAGE
# ----------------------------------
# None
#
# HOW TO BUILD THIS IMAGE
# -----------------------
# Run: 
#      $ docker build -t oracle/database:18.4.0-xe -f Dockerfile.xe .
#
#
# Pull base image
# ---------------
# FROM oraclelinux:7-slim
FROM registry.access.redhat.com/rhel7/rhel

# Labels
# ------
LABEL "provider"="Oracle"                                               \
      "issues"="https://github.com/oracle/docker-images/issues"         \
      "volume.data"="/opt/oracle/oradata"                               \
      "volume.setup.location1"="/opt/oracle/scripts/setup"              \
      "volume.setup.location2"="/docker-entrypoint-initdb.d/setup"      \
      "volume.startup.location1"="/opt/oracle/scripts/startup"          \
      "volume.startup.location2"="/docker-entrypoint-initdb.d/startup"  \
      "port.listener"="1521"                                            \
      "port.oemexpress"="5500"                                          \
      "port.apex"="8080"                                                \
      "io.k8s.description"="Customized Oracle XE 18c image for OpenShift (Created by Igor Petukhov)"  \
      "io.k8s.display-name"="Oracle XE 18c"                             \
      "io.openshift.expose-services"="1521:1521"                        \
      "io.openshift.tags"="oracle, xe, 18c, db"

# Environment variables required for this build (do NOT change)
# -------------------------------------------------------------
ENV ORACLE_BASE=/opt/oracle \
    ORACLE_HOME=/opt/oracle/product/18c/dbhomeXE \
    ORACLE_SID=XE \
    INSTALL_FILE_1="https://download.oracle.com/otn-pub/otn_software/db-express/oracle-database-xe-18c-1.0-1.x86_64.rpm" \
    # INSTALL_FILE_1="oracle-database-xe-18c-1.0-1.x86_64.rpm" \
    RUN_FILE="runOracle.sh" \
    PWD_FILE="setPassword.sh" \
    CONF_FILE="oracle-xe-18c.conf" \
    CHECK_SPACE_FILE="checkSpace.sh" \
    CHECK_DB_FILE="checkDBStatus.sh" \
    INSTALL_DIR="$HOME/install" \
    ORACLE_DOCKER_INSTALL="true" \
    INSTALL_FILE_0="https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm"

# Use second ENV so that variable get substituted
ENV PATH=$ORACLE_HOME/bin:$PATH \
    ORA_DATA=/opt/oracle/oradata

# Copy binaries
# -------------
COPY $CHECK_SPACE_FILE $RUN_FILE $PWD_FILE $CHECK_DB_FILE $CONF_FILE $INSTALL_DIR/

RUN chgrp -R 0 $INSTALL_DIR && \
    chmod -R g=u $INSTALL_DIR && \
    sync && \
    $INSTALL_DIR/$CHECK_SPACE_FILE && \
    cd $INSTALL_DIR && \
    # Manually installing Oracle-XE pre-requisites
    curl -o oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm $INSTALL_FILE_0 && \
    yum -y localinstall oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm --skip-broken && \
    # yum-config-manager --enable registry.access.redhat.com && \
    # yum -y install openssl oracle-database-preinstall-18c && \
    yum -y localinstall $INSTALL_FILE_1 --skip-broken && \
    # yum -y localinstall $INSTALL_DIR/$INSTALL_FILE_1 && \
    rm -rf /var/cache/yum && \
    rm -rf /var/tmp/yum-* && \
    mkdir -p $ORACLE_BASE/scripts/setup && \
    mkdir $ORACLE_BASE/scripts/startup && \
    ln -s $ORACLE_BASE/scripts /docker-entrypoint-initdb.d && \
    mkdir -p $ORACLE_BASE/oradata /home/oracle && \
    # chown -R oracle $ORACLE_BASE /home/oracle && \
    chgrp -R 0 $INSTALL_DIR/* && \
    chmod -R g=u $INSTALL_DIR/* && \
    mv $INSTALL_DIR/$RUN_FILE $ORACLE_BASE/ && \
    mv $INSTALL_DIR/$PWD_FILE $ORACLE_BASE/ && \
    mv $INSTALL_DIR/$CHECK_DB_FILE $ORACLE_BASE/ && \
    mv $INSTALL_DIR/$CONF_FILE /etc/sysconfig/ && \
    ln -s $ORACLE_BASE/$PWD_FILE / && \
    cd $HOME && \
    rm -rf $INSTALL_DIR && \
    chgrp -R 0 $ORACLE_BASE/* /home/oracle && \
    chmod -R g=u $ORACLE_BASE/* /home/oracle 

# HEALTHCHECK --interval=1m --start-period=5m \
#    CMD "$ORACLE_BASE/$CHECK_DB_FILE" >/dev/null || exit 1

VOLUME ${ORA_DATA}

EXPOSE 1521
USER 1001

CMD exec $ORACLE_BASE/$RUN_FILE
