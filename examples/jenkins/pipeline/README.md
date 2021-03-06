# Using Jenkins Pipelines with OpenShift

This set of files will allow you to deploy a Jenkins server that is capable of executing Jenkins pipelines and
utilize pods run on OpenShift as Jenkins slaves.

To walk through the example:

0. If using `oc cluster up`, be sure to grab the [latest oc command](https://github.com/openshift/origin/releases/latest)

1. Stand up an openshift cluster from origin master, installing the standard imagestreams to the openshift namespace:

        $ oc cluster up

    If you do not use oc cluster up, ensure the imagestreams are registered in the openshift namespace, as well as the
jenkins template represented by jenkinstemplate.json by running these commands as a cluster admin:

        $ oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/image-streams/image-streams-centos7.json -n openshift
        $ oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/jenkins/pipeline/jenkinstemplate.json -n openshift

2. Login as a normal user (any username is fine)

        $ oc login

3. Create a project for your user named "pipelineproject"

        $ oc new-project pipelineproject

4. Run this command to instantiate the template which will create a pipeline buildconfig and some other resources in your project:

        If you used cluster up:
        $ oc new-app jenkins-pipeline-example

        Otherwise:
        $ oc new-app -f https://raw.githubusercontent.com/openshift/origin/master/examples/jenkins/pipeline/samplepipeline.json

    At this point if you run `oc get pods` you should see a jenkins pod, or at least a jenkins-deploy pod. (along with other items in your project)  This pod was created as a result of the new pipeline buildconfig being defined by the sample-pipeline template.

5. View/Manage Jenkins (optional)

    You should not need to access the jenkins console for anything, but if you want to configure settings or watch the execution,
    here are the steps to do so:

    First you must determine the jenkins password which was randomly generated for you:

        $ oc env dc/jenkins --list | grep JENKINS_PASSWORD

    The value will appear as:

        JENKINS_PASSWORD=faUXMOFUhRYXO2Gx

    If you have a router running (`oc cluster up` provides one), run:

        $ oc get route

    and access the host for the Jenkins route.

    If you do not have a router, you can access jenkins directly via the service ip.  Determine the jenkins service ip ("oc get svc") and go to it in your browser on port 80.  Do not confuse it with the jenkins-jnlp service.

    Login with the user name is `admin` and the password as determined above.

6. Launch a new build

        $ oc start-build sample-pipeline

    Jenkins will: create an instance of the sample-pipeline job, launch a slave, trigger a build in openshift, trigger a
deployment in openshift, and tear the slave down.

    If you monitor the pods in your default project, you will also see the slave pod get created and deleted.
