FROM ${docker.from.id}:${docker.from.version}

# Latest Tomcat8 in Ubuntu 16.04 repositories is 8.0.32 which has an annoying bug
# https://bz.apache.org/bugzilla/show_bug.cgi?id=58999
# Download and use newer 8.5 instead, and create structure identical to default package for simpler switchover
# Also build libtcnative
COPY tomcat8-defaults policy.d /tmp/
RUN wget -O /tmp/tomcat.tar.gz http://www-eu.apache.org/dist/tomcat/tomcat-8/v${dep.tomcat.version}/bin/apache-tomcat-${dep.tomcat.version}.tar.gz \
   && install_clean \
      automake \
      autoconf \
      make \
      gcc \
      libapr1 \
      libaprutil1 \
      libssl1.0.0 \
      libapr1-dev \
      libaprutil1-dev \
      libssl-dev \
   && cd /tmp \
   && tar xzf tomcat.tar.gz \
   && mkdir -p /var/lib/tomcat8/lib /var/lib/tomcat8/webapps /var/log/tomcat8 /var/cache/tomcat8 /usr/share/tomcat8 /etc/tomcat8/policy.d \
   && mv /tmp/apache-tomcat-${dep.tomcat.version}/bin /tmp/apache-tomcat-${dep.tomcat.version}/lib /usr/share/tomcat8 \
   && mv /tmp/apache-tomcat-${dep.tomcat.version}/conf/* /etc/tomcat8/ \
   && mv /tmp/*.policy /etc/tomcat8/policy.d/ \
   && rm -rf /tmp/apache-tomcat-${dep.tomcat.version} /tmp/tomcat.tar.gz \
   && ln -s /etc/tomcat8 /var/lib/tomcat8/conf  \
   && ln -s /var/log/tomcat8 /var/lib/tomcat8/logs \
   && ln -s /var/cache/tomcat8 /var/lib/tomcat8/work \
   && nativeBuildDir="$(mktemp -d)" \
   && tar -xvf /usr/share/tomcat8/bin/tomcat-native.tar.gz -C "$nativeBuildDir" --strip-components=1 \
   && cd "$nativeBuildDir/native" \
   && gnuArch="$(arch)" \
   && ./configure --build="$gnuArch" --libdir=/usr/share/tomcat8/native-jni-lib --prefix=/usr/share/tomcat8 --with-apr="$(which apr-1-config)" --with-ssl=yes --with-java-home="$(find / -iname javac | grep /usr/lib | sed -r 's/\/bin\/javac//')" \
   && make -j "$(nproc)" \
   && make install \
   && ln -sv /usr/share/tomcat8/native-jni-lib/libtcnative-1.so /usr/lib/ \
   && cd / \
   && rm -rf "$nativeBuildDir" \
   && rm /usr/share/tomcat8/bin/tomcat-native.tar.gz \
   && groupadd -g 12000 tomcat8 \
   && useradd -s /bin/false -b /var/lib/tomcat8 -g tomcat8 -u 12000 -M tomcat8 \
   && chown -R tomcat8:tomcat8 /var/lib/tomcat8 /var/log/tomcat8 /var/cache/tomcat8 /usr/share/tomcat8 /etc/tomcat8 \
   && mv /tmp/tomcat8-defaults /etc/default/tomcat8 \
   && apt-get autoremove -q -y \
      automake \
      autoconf \
      make \
      gcc \
      libapr1-dev \
      libaprutil1-dev \
      libssl-dev

# add prepared files that would be too awkward to handle via RUN / sed
COPY server.xml initTomcat.sh startTomcat.sh tomcat-ng.conf /tmp/

# apply our tomcat default configurations
RUN cp /tmp/*.xml /etc/tomcat8/ \
   && chown root:tomcat8 /etc/tomcat8/server.xml \
   && chown -R tomcat8:tomcat8 /var/lib/tomcat8 /var/log/tomcat8 \
   && chmod 640 /etc/tomcat8/server.xml \
   && mkdir -p /usr/share/tomcat8/common/classes /usr/share/tomcat8/server/classes /usr/share/tomcat8/shared/classes \
   && sed -i 's/^handlers = .*/handlers = java.util.logging.ConsoleHandler/' /etc/tomcat8/logging.properties \
   && sed -i 's/^JAVA_OPTS=.*/JAVA_OPTS="-Djava.awt.headless=true %JAVA_OPTS%"/' /etc/default/tomcat8 \
   && sed -i 's/^#LOGFILE_DAYS=.*/LOGFILE_DAYS=2/' /etc/default/tomcat8 \
   && sed -i 's/^#LOGFILE_COMPRESS=.*/LOGFILE_COMPRESS=1/' /etc/default/tomcat8 \
   && mkdir -p /etc/my_init.d \
   && mv /tmp/tomcat-ng.conf /etc/syslog-ng/conf.d/tomcat-ng.conf \
   && mv /tmp/initTomcat.sh /etc/my_init.d/20_initTomcat.sh \
   && chmod +x /etc/my_init.d/20_initTomcat.sh \
   && mkdir /etc/service/tomcat8 \
   && mv /tmp/startTomcat.sh /etc/service/tomcat8/run \
   && chmod +x /etc/service/tomcat8/run

ENV JMX_RMI_PORT=5000

EXPOSE 8080 8081 5000

LABEL vendor="${docker.labels.vendor}" \
   ${docker.labels.namespace}.version="${project.version.majorVersion}.${project.version.minorVersion}.${project.version.incrementalVersion}" \
   ${docker.labels.namespace}.is-beta="" \
   ${docker.labels.namespace}.is-production="" \
   ${docker.labels.namespace}.release-date="${docker.labels.release-date}" \
   ${docker.labels.namespace}.maintainer="${docker.labels.maintainer}"