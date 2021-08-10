# Migrating to Liberty

## From Tomcat (SpringBoot)
Put together a bit of a workflow that:
1. Takes a SpringBoot app which has been packaged with a Tomcat server
1. Decouples the embedded libraries from the main logic using springBootUtility
1. Builds the resulting thin JAR and library cache into a Liberty container
1. Pushes the image to a container registry
1. Deploys the image using the Liberty Operator

This is likely not a scenario that directly relates to you, but we had to start somewhere. For demonstration purposes it shows off:
* Possible migration workflows
* Using OpenShift Pipelines (Tekton)
* Retrieving artefacts from Nexus
* Building and publishing an image inside a pipeline (non-S2I)
* Deploying an operator-backed Open Liberty application from a pipeline

## Show and Tell
* Talk through `maven-build-and-upload` pipeline
* Talk through and demonstrate `spring-tomcat-to-liberty` pipeline
