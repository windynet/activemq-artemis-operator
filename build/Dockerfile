FROM registry.access.redhat.com/ubi7/go-toolset:1.13.4-18 AS build-env

ENV GOPATH=/tmp/go 
ENV GOOS=linux
ENV CGO_ENABLED=0
ENV BROKER_NAME=activemq-artemis

RUN mkdir -p /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/bin
RUN mkdir -p /tmp/activemq-artemis-operator

ADD . ${GOPATH}/src/github.com/rh-messaging/activemq-artemis-operator
WORKDIR ${GOPATH}/src/github.com/rh-messaging/activemq-artemis-operator

RUN /opt/rh/go-toolset-1.13/root/usr/bin/go build -o /tmp/activemq-artemis-operator/${BROKER_NAME}-operator /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/cmd/manager


FROM registry.access.redhat.com/ubi7:7.8-311 AS base-env

ENV BROKER_NAME=activemq-artemis
ENV OPERATOR=/home/${BROKER_NAME}-operator/bin/${BROKER_NAME}-operator
ENV USER_UID=1000
ENV USER_NAME=${BROKER_NAME}-operator
ENV CGO_ENABLED=0
ENV GOPATH=/tmp/go
ENV JBOSS_IMAGE_NAME="amq7/amq-broker-rhel7-operator"
ENV JBOSS_IMAGE_VERSION="0.15"

WORKDIR /

RUN mkdir -p /manifests/0.9.1
RUN mkdir -p /manifests/0.13.0
RUN mkdir -p /manifests/0.14.0
RUN mkdir -p /manifests/0.15.0
RUN chown -R `id -u`:0 /manifests
COPY --from=build-env /tmp/activemq-artemis-operator /home/${BROKER_NAME}-operator/bin
COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/build/bin/entrypoint /home/${BROKER_NAME}-operator/bin

COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/deploy/olm-catalog/${BROKER_NAME}-operator/${BROKER_NAME}.package.yaml /manifests
COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/deploy/olm-catalog/${BROKER_NAME}-operator/0.9.1/ /manifests/0.9.1
COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/deploy/olm-catalog/${BROKER_NAME}-operator/0.13.0/ /manifests/0.13.0
COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/deploy/olm-catalog/${BROKER_NAME}-operator/0.14.0/ /manifests/0.14.0
COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/deploy/olm-catalog/${BROKER_NAME}-operator/0.15.0/ /manifests/0.15.0
COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/deploy/crds/* /manifests/0.9.1
COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/deploy/crds/* /manifests/0.13.0
COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/deploy/crds/* /manifests/0.14.0
COPY --from=build-env /tmp/go/src/github.com/rh-messaging/activemq-artemis-operator/deploy/crds/* /manifests/0.15.0

RUN useradd ${BROKER_NAME}-operator
RUN mkdir -p /home/${BROKER_NAME}-operator/bin && chown -R `id -u`:0 /home/${BROKER_NAME}-operator/bin && chmod -R 755 /home/${BROKER_NAME}-operator/bin

USER ${USER_UID}
ENTRYPOINT ["/home/${BROKER_NAME}-operator/bin/entrypoint"]

LABEL \
      com.redhat.component="amq-broker-rhel7-operator-container"  \
      com.redhat.delivery.appregistry="false" \
      description="ActiveMQ Artemis Broker Operator"  \
      io.k8s.description="An associated operator that handles broker installation, updates and scaling."  \
      io.k8s.display-name="ActiveMQ Artemis Broker Operator"  \
      io.openshift.expose-services=""  \
      io.openshift.s2i.scripts-url="image:///usr/local/s2i"  \
      io.openshift.tags="messaging,amq,integration,operator,golang"  \
      maintainer="Roddie Kieley <rkieley@redhat.com>"  \
      name="amq7/amq-broker-rhel7-operator" \
      summary="ActiveMQ Artemis Broker Operator"  \
      version="0.15"
