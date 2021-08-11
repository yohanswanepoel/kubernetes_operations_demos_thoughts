# Parko's Stuff - a collection

## Current state
* Show petclinic running in prod
  * Green theme
  * Show + explain ArgoCD/GitOps
    * https://argocd-server-parko-petclinic-prod.violet-cluster-new-2761a99850dd8c23002378ac6ce7f9ad-0000.au-syd.containers.appdomain.cloud
    * https://github.com/mattparko/petclinic-argocd
* Show empty/clean test environment
* Show newly developed app in Dev

## Tomcat and S2I in Dev
* Illustrate the concept of S2I
* Actual build takes around 10 minutes

```
oc project parko-petclinic-dev

oc new-app --name=petclinic registry.access.redhat.com/ubi8/openjdk-11~https://github.com/mattparko/spring-petclinic.git#dev
```

## Maven Build Pipeline
* Illustrate concept of a Maven build inside a pipeline
* Show `maven-build-and-upload` pipeline (in `parko-pipelines` project)
* Show Nexus: http://nexus-parko-nexus.violet-cluster-new-2761a99850dd8c23002378ac6ce7f9ad-0000.au-syd.containers.appdomain.cloud/#browse/browse:maven-releases:com

## Container customisation
* Briefly highlight example container image
  * Will be used in a later pipeline to optimise a build for Liberty
* Quay scans: https://quay.io/repository/mparkins/openliberty?tab=tags
* Podman builds
```
FROM openliberty/open-liberty:springBoot2-ubi-min

USER root

COPY scripts/ubi7-minimal /security-scripts/
COPY banners/issue /etc/

RUN echo Update packages and install security fixes && \
    microdnf update -y && \
    /security-scripts/xccdf_org.ssgproject.content_rule_accounts_max_concurrent_login_sessions.sh && \
    /security-scripts/xccdf_org.ssgproject.content_rule_accounts_tmout.sh && \
    /security-scripts/xccdf_org.ssgproject.content_rule_rpm_verify_permissions.sh && \
    microdnf clean all && \
    rm -rf /security-scripts/ /var/cache/dnf/ /var/tmp/* /tmp/* /var/tmp/.???* /tmp/.???*

USER 1001
```
* Talk through potential to use a pipeline (touch on this soon)

## Tomcat to Liberty Pipeline
* A mock up of a potential, but fictitious workflow
* Briefly talk through workflow
* Show potential triggers
* Kick off PipelineRun
* Could be via GUI (either bouncing ball or previous run), or:
```
oc create -f petclinic-spring-tomcat-to-liberty-pipelinerun.yaml
```
* See deployment into test directly from pipeline
* See deployment into prod via GitOps
* Check Quay for latest container security scans
