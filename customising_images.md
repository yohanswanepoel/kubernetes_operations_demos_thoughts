# Publishing Container Images Internally
## Liberty Images
* The IBM Liberty team constantly release updated versions of the Liberty container images
* They maintain multiple architectures, as well as multiple versions of the various components.
* Do you want your developers pulling directly from these, and other, container images?
* Need to ensure stability, security, consistency
* Also need to meet developer's needs, in both dev and prod environments
* Automated ingestion of Liberty images (CI/CD, DevSecOps, Secure software supply chain)

## Why customise images?
* Additional security controls (eg. US Dept of Defence Ironbank Containers)
* Internal security and configuration requirements (SOE)
* Security patching
* Missing components for common, internal development requirements

Example Container file with customisations:
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

## To S2I or not to S2I?
* S2I images are shipped with package building tools
* Provides a way to compile or package applications from source and then run the resulting container
* S2I images can also often take pre-built binaries
* The alternative to S2I is to build your application yourself, build a container image containing that application and the correct entrypoint, and push the resulting image to a container registry
* This is no less valid (and arguably far more popular in the wild)
* The non-S2I approach can still be automated end-to-end
* You may choose to provide one or both to application developers

## Show and Tell
* Show difference in security scan results: https://quay.io/repository/mparkins/openliberty?tab=tags
* Show (or talk through) running a container image build in a Pipeline
