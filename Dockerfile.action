FROM maven:3.8.2-jdk-8-slim

# Install Autorest
RUN apt-get update && apt-get -qq -y install libunwind8 libicu67 libssl1.0 liblttng-ust0 libcurl4 libuuid1 libkrb5-3 zlib1g
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt-get update && apt-get -y install \
    nodejs \
    libunwind8-dev \
    && rm -rf /var/lib/apt/lists/*

# Install preprocessing script requirements
RUN apt-get update && apt-get -y install git python3-pip && pip install urllib3==1.24.2

RUN curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.asc.gpg
RUN mv microsoft.asc.gpg /etc/apt/trusted.gpg.d/
RUN curl https://packages.microsoft.com/config/debian/9/prod.list > prod.list
RUN mv prod.list /etc/apt/sources.list.d/microsoft-prod.list
RUN chown root:root /etc/apt/trusted.gpg.d/microsoft.asc.gpg
RUN chown root:root /etc/apt/sources.list.d/microsoft-prod.list

RUN apt-get update
RUN apt-get install -yy -q dotnet-sdk-5.0

ARG OPENAPI_GENERATOR_COMMIT=v5.1.0
ARG GENERATION_XML_FILE=java.xml
ARG OPENAPI_GENERATOR_USER_ORG=OpenAPITools

# Check out specific commit of openapi-generator
RUN mkdir /source && \
    cd /source && \
    git clone -n https://github.com/${OPENAPI_GENERATOR_USER_ORG}/openapi-generator.git && \
    cd openapi-generator && \
    git checkout $OPENAPI_GENERATOR_COMMIT

# Build it and persist local repository
RUN mkdir /.dotnet && chmod -R go+rwx /.dotnet && mkdir /.nuget && chmod -R go+rwx /.nuget && chmod -R go+rwx /root && umask 0 && cd /source/openapi-generator && \
    mvn install -DskipTests -Dmaven.test.skip=true -pl modules/openapi-generator-maven-plugin -am && \
    cp -r /root/.m2/* /usr/share/maven/ref

# Copy required files
COPY gen/openapi/openapi-generator/generate_client_in_container.sh /generate_client.sh
COPY gen/openapi/preprocess_spec.py /
COPY gen/openapi/custom_objects_spec.json /
COPY gen/openapi/${GENERATION_XML_FILE} /generation_params.xml

ENTRYPOINT ["mvn-entrypoint.sh", "/generate_client.sh"]
