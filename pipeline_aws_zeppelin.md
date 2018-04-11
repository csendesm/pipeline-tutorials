### How to deploy a CI/CD flow for Zeppelin notebooks on AWS with Banzai Cloud's Pipeline

In this tutorial we will guide you through how to deploy a CI/CD flow for Zeppelin notebooks from the ground on AWS.

At first you have to deploy our Pipeline Control Plane which takes care all the hard work.

[Pipeline Control Plane](https://github.com/banzaicloud/pipeline-cp-launcher/tree/0.2.0)


### Prerequisites of hosting the control plane on AWS

# Hosting Pipeline Control Plane on AWS

Follow the steps below for hosting `Pipeline Control Plane` on `AWS`.
On `AWS` we use a [Cloudformation](https://aws.amazon.com/cloudformation/) template in order to provision a Pipeline control plane.

The control plane image (AMI) is currently published to one region, `eu-west-1` aka Ireland. When launching the control plane please pass the following ImageId `ami-ece5b095`.

## Pre-requisites

1. [AWS](https://portal.aws.amazon.com/billing/signup?type=enterprise#/start) account
1. AWS [EC2 key pair](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

## Command line

For creating the control plane launcher through command line take a look at `.env.example` as a start to learn what environment variables are required by the `Makefile`. _Note_ the makefile uses `aws` cli which needs to be installed first if not available on the machine.

* deploy - `make create-aws`
* delete - `make terminate-aws`

## Amazon Web Console

* Initiate the creation of a new stack: [![Launch stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new)

* Select `Specify an Amazon S3 template URL` and add the URL to our template `https://s3-eu-west-1.amazonaws.com/cf-templates-grr4ysncvcdl-eu-west-1/2018079qfE-new.templateejo9oubl16`

<a href="images/ControlPlaneFromTemplate.png" target="_blank"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/ControlPlaneFromTemplate.png" height="230"></a>

* Fill in the following fields on the form:

  * **Stack name**
    * specify a name for the Control Plane deployment

      <a href="images/StackName.png"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/StackName.png" height="130"></a>

  * **AWS Credentials**
     * Amazon access key id - specify your [access key id](http://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys)
     * Amazon secret access key - specify your [secret access key](http://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys)

     <a href="images/AwsCred.png"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/AwsCred.png" height="150"></a>

  * **Control Plane Instance Config**
     * InstanceName - name of the EC2 instance that will host the Control Plane
     * ImageId - `ami-ece5b095`
     * KeyName - specify your AWS [EC2 key pair](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

     <a href="images/ControlPlaneInstanceConfig.png"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/ControlPlaneInstanceConfig.png" height="180"></a>

  * **Pipeline Credentials**
     * Github Client - GitHub OAuth `Client Id`
     * Github Secret - Github OAuth `Client Secret`

      <a href="images/CloudFormationPipelineCred.png"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/CloudFormationPipelineCred.png" height="110"></a>

  * **Banzai-Ci**
     * Orgs - comma-separated list of Github organizations whose members to grant access to use Banzai Cloud Pipeline's CI/CD workflow

      <a href="images/CloudFormationCiOrgs.png"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/CloudFormationCiOrgs.png" height="70"></a>

  * **Grafana Dashboard**
     * Grafana Dashboard Password - specify password for accessing Grafana dashboard with defaults specific to the application

     <a href="images/CloudFormationGrafanaCred.png"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/CloudFormationGrafanaCred.png" height="90"></a>

  * **Prometheus Dashboard**
     * Prometheus Password - specify password for accessing Prometheus that collects cluster metrics

      <a href="images/CloudFormationPrometheusCred.png"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/CloudFormationPrometheusCred.png" height="100"></a>

  * **Advanced Pipeline Options**
     * PipelineImageTag - specify `0.3.0` for using current stable Pipeline release.

     <a href="images/CloudFormationAdvancedPipOpt.png"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/CloudFormationAdvancedPipOpt.png" height="130"></a>

  * **Slack Credentials**
       * this section is optional. Complete this section to receive  cluster related alerts through a [Slack](https://slack.com) push notification channel.

  * **Alert SMTP Credentials**
     * this section is optional. Fill this section to receive cluster related alerts through email.

* Finish the wizard to create a `Control Plane` instance.

## Deployment end points

Check the output section of the deployed cloud formation template for the endpoints where the deployed services can be reached:

* PublicIP - the IP of the host where Pipeline is running
* Pipeline - the endpoint for the Pipelne REST API
* Grafana - the endpoint for Grafana
* PrometheusServer - the endpoint for [federated](https://banzaicloud.com/blog/prometheus-federation/) Prometheus server.

<a href="images/CloudFormationOutput.png"><img src="https://github.com/banzaicloud/pipeline-cp-launcher/raw/0.3.0/docs/images/CloudFormationOutput.png" height="250"></a>


 ###  Zeppelin CI/CD

  Let's start with our example [Zeppelin project](https://github.com/banzaicloud/zeppelin-pdi-example)

   Fork the repository into your GitHub account. You'll find a couple of *Banzai Cloud Pipeline CI/CD flow descriptor templates* for the released cloud providers (Amazon, Azure). Make a copy of the template corresponding to your chosen cloud provider and name it **.pipeline.yml**. This is the Banzai Cloud Pipeline **CI/CD flow descriptor** which is one of the **spotguides** associated with the project.

   On the Drone UI running on the Banzai Cloud control plane enable the build for your fork. On the project's build details section add the secrets needed (Pipeline endpoint, credentials). Check the descriptor for any placeholders and substitute them with your corresponding values.

   >Note: there is a video for our Spark CI/CD example [available](https://banzaicloud.com/blog/pipeline-howto/) driving through the CI/CD UI

   That's all, your project is now configured for the Banzai Cloud Pipeline CI/CD flow! The flow will be triggered whenever a new change is pushed to the repository (configurable on the UI).

   ## Understanding the Banzai Cloud CI/CD flow descriptor

   The Banzai Cloud Pipeline CI/CD flow descriptor has to be named *.pipeline.yml*; it contains the steps of the flow that are executed sequentially. (The Banzai Cloud Pipeline CI/CD flow is using Drone in the background, however the CI/CD flow descriptor is an abstraction that is not directly tied to any particular product or implementation. It can also be wired to use CircleCI or Travis).

   The example descriptor has the following steps:

   1. ``create_cluster`` - creates or reuses a (managed) Kubernetes cluster supported by [Pipeline](https://github.com/banzaicloud/pipeline) like EC2, AKS, GKE
   2. install tooling for the cluster (using helm charts):
         - ``install_monitoring`` cluster monitoring (Prometheus)
         - ``install_spark_history_server`` Spark History Server
         - ``install_zeppelin`` Zeppelin

   > Note: these steps related to the infrastructure are only done once, and reused after the first run if the cluster is not deleted as the last step

   3. ``remote_checkout`` check out the code from the git repository
   4. ``run`` run the Notebook

   You can name the steps as you wish, they are only used for delimiting the phases of the flow. Steps are implemented as Docker containers that use the configurations items passed in the flow descriptor (step section).

   Note that compared to [manually setting up History Server and event logging](https://banzaicloud.com/blog/spark-history-server/) using the Banzai Cloud Pipeline CI/CD flow you only have to replace the S3 bucket / Blob container name in ``install_spark_history_server.logDirectory`` and ``install_zeppelin.deployment_values.zeppelin.sparkSubmitOptions.eventLogDirectory`` properties in *.pipeline.yml* with your own.

   ## Checking the results

   As of the version 0.2.0 of Banzai Cloud Pipeline supports the deployments and use of the Spark History Server.
   The details of the Spark jobs can be checked as usual on the History Server UI. Information provided by the Spark jobs are saved to a persistent storage (s3, wasb) and the Spark History Server reads and displays it. This way the details of the execution can be kept even after the Kubernetes cluster is destroyed.

   If you don't destroy the infrastructure as part of the Banzai Cloud Pipeline CI/CD flow, you can query the available endpoints (zeppelin, monitoring, spark history server) by issuing a GET request:

   ```
   curl --request GET --url 'http://[control-plane]/pipeline/api/v1/clusters/{{cluster_id}}/endpoints'
   ```


   ![Spark history Server UI](https://www.banzaicloud.com/img/blog/zeppelin/shs.png)

   > __Warning__! Be aware that clusters created with the flow on the cloud provider costs you money. It's advised to destroy your environment when the development is finished (or at the end of the day). If you are running on AWS you might consider using spot instances and our watchguard to safely run spot clusters in production with [Hollowtrees](https://banzaicloud.com/blog/hollowtrees/)
