FROM ${docker.from.id}:${docker.from.version}

RUN wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | apt-key add - \
   && add-apt-repository --yes https://adoptopenjdk.jfrog.io/adoptopenjdk/deb/ \
   && apt-get update \
   && install_clean adoptopenjdk-11-openj9

LABEL vendor="${docker.labels.vendor}" \
   ${docker.labels.namespace}.version="${project.version.majorVersion}.${project.version.minorVersion}.${project.version.incrementalVersion}" \
   ${docker.labels.namespace}.is-beta="" \
   ${docker.labels.namespace}.is-production="" \
   ${docker.labels.namespace}.release-date="${docker.labels.release-date}" \
   ${docker.labels.namespace}.maintainer="${docker.labels.maintainer}"