### How to deploy 'Kafka on etcd' on AWS with Banzai Cloud's Pipline 

In this tutorial we will guide you through how to deploy a 'Kafta on etcd' application from the ground on AWS.

At first you have to deploy our Pipeline Control Plane which takes care all the hard work.

[Pipeline Control Plane](https://github.com/banzaicloud/pipeline-cp-launcher/tree/0.2.0)


### Prerequisites of hosting the control plane on AWS

Hosting `Pipeline Control Plane` and creating Kubernetes clusters on **`AWS`**

   1. [AWS](https://portal.aws.amazon.com/billing/signup?type=enterprise#/start) account
   1. AWS [EC2 key pair](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

### Launch Pipeline Control Plane on `AWS`

   The easiest way for running a Pipeline Control Plane is to use a [Cloudformation](https://aws.amazon.com/cloudformation/) template.

   * Navigate to: https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new

   * Select `Specify an Amazon S3 template URL` and add the URL to our template:

    `https://s3-eu-west-1.amazonaws.com/cf-templates-grr4ysncvcdl-eu-west-1/2018026em9-new.templatee93ate9mob7`

     <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/ControlPlaneFromTemplate.png" target="_blank"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/ControlPlaneFromTemplate.png" height="230"></a>

   * Fill in the following fields on the form:

     * **Stack name**
       * specify a name for the Control Plane deployment

         <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/StackName.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/StackName.png" height="130"></a>

     * **AWS Credentials**
        * Amazon access key id - specify your [access key id](http://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys)
        * Amazon secret access key - specify your [secret access key](http://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys)

        <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/AwsCred.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/AwsCred.png" height="150"></a>

     * **Azure Credentials and Information** - _needed only for creating Kubernetes clusters on `Azure`_
        * AzureClientId - see how to get Azure Client Id above
        * AzureClientSecret - see how to get Azure Client Secret above
        * AzureSubscriptionId - your Azure Subscription Id
        * AzureTenantId - see how to get Azure Client Tenant Id above

        <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/AzureCred.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/AzureCred.png" height="200"></a>

     * **Control Plane Instance Config**
        * InstanceName - name of the EC2 instance that will host the Control Plane
        * ImageId - pick the image id from the  [README](https://github.com/banzaicloud/pipeline-cp-launcher/blob/0.2.0/README.md)
        * KeyName - specify your AWS [EC2 key pair](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

        <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/ControlPlaneInstanceConfig.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/ControlPlaneInstanceConfig.png" height="180"></a>

     * **Banzai Pipeline Credentials**
        * Pipeline API Password - specify the password for accessing the Pipeline REST [API](https://github.com/banzaicloud/pipeline/blob/0.2.0/docs/create.md) exposed by the Pipeline PaaS. **_Take note of the user name and password as those will be required when setting the [secrets](#repository-secrets) for the GitHub repositories in the CI/CD workflow._**

         <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/PipelineCred.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/PipelineCred.png" height="150"></a>

     * **Banzai-Ci Credentials**
        * Orgs - comma-separated list of Github organizations whose members to grant access to use Banzai Cloud Pipeline's CI/CD workflow
        * Github Client - GitHub OAuth `Client Id`
        * Github Secret - Github OAuth `Client Secret`

         <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/CloudFormulationDetails3.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/CloudFormulationDetails3.png" height="150"></a>

     * **Grafana Dashboard**
        * Grafana Dashboard Password - specify password for accessing Grafana dashboard with defaults specific to the application

        <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/GrafanaCred.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/GrafanaCred.png" height="100"></a>

     * **Prometheus Dashboard**
        * Prometheus Password - specify password for accessing Prometheus that collects cluster metrics

         <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/PrometheusCred.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/PrometheusCred.png" height="100"></a>

     * **Advanced Pipeline Options**
        * PipelineImageTag - specify `0.2.0` for using current stable Pipeline release.

        <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/AdvencedPipOpt.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/AdvencedPipOpt.png" height="150"></a>

     * **Slack Credentials**
          * this section is optional. Complete this section to receive  cluster related alerts through a [Slack](https://slack.com) push notification channel.

     * **Alert SMTP Credentials**
        * this section is optional. Fill this section to receive cluster related alerts through email.

   * Finish the wizard to create a `Control Plane` instance.
   * Take note of the PublicIP of the created Stack. We refer to this as the PublicIP of `Control Plane`.

       <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/CloudFormulationDetails5.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/CloudFormulationDetails5.png" height="250"></a>

   * Go back to the earlier created GitHub OAuth application and modify it. Set the `Authorization callback URL` field to `http://{control_plane_public_ip}/authorize`

     <a href="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/OAuthCallback.png"><img src="https://raw.githubusercontent.com/banzaicloud/pipeline/master/docs/images/howto/OAuthCallback.png" height="70"></a>

### Deploying Kafka on etcd using Pipeline

Now our Pipeline Control Plane is ready to help us to deploy applications in a cloud native way.

We will communicate with the CP through it's REST interface. In the near future you will be able manage these steps on a fancy UI.

Let's start the process:

At first we have to create a Kubernetes cluster where our Kafka app will be deployed to:

```
curl --request POST \
  --url 'http://{PublicIPOfYourCP}/pipeline/api/v1/clusters' \
  --header 'Authorization: Basic {Base64EncodedStringofYourCPCredetial(username:password)}' \
  --header 'Content-Type: application/json' \
  --data '{
  "name":"awscluster-tutorial-1",
    "location": "eu-west-1",
    "cloud": "amazon",
    "nodeInstanceType": "m4.xlarge",
    "properties": {
        "amazon": {
            "node": {
                "spotPrice": "0.2",
                "minCount": 1,
                "maxCount": 2,
                "image": "ami-06d1667f"
            },
            "master": {
                "instanceType": "m4.xlarge",
                "image": "ami-06d1667f"
            }
        }
    }
}'

```

As you can see we are going to create our cluster in the `"eu-west-1"` location, the master and the worker nodes will also be deployed to an `"m4.xlarge"` instance. We are trying to run our worker nodes on a `spot instance` and our offer is `USD 0.2` for them.

The output looks like this:

```
{"Ip":"x.x.x.x","message":"Cluster created successfully!","name":"awscluster-tutorial-1","resourceId":2,"status":201}
```

In the output you can find the IP address of the master node, the name and the id of the created cluster.

Now our Kubernetes cluster is being created and will be ready after couple of minutes.

You can check the Cluster status by the following API call:

```
curl --head \
  --url 'http://x.x.x.x/pipeline/api/v1/clusters/2' \
  --header 'Authorization: Basic YWRtaW46UGFzczEyMzQ=' \
  --header 'Content-Type: application/json'

```

 The result should look like this:
```
 HTTP/1.1 200 OK

 ```
To deploy our Kafka on etcd solution to our newly created cluster use the following call:


```
curl --request POST \
   --url 'http://x.x.x.x/pipeline/api/v1/clusters/2/deployments' \
   --header 'Authorization: Basic YWRtaW46UGFzczEyMzQ=' \
   --header 'Content-Type: application/json' \
   --data '{"name": "banzaicloud-stable/kafka"}'

```

And the result:

```   
{"notes":"","release_name":"wrinkled-aardwolf","status":201}
```

The following call help us to check our deployments on the given cluster:

```
 curl --head --url 'http://x.x.x.x/pipeline/api/v1/clusters/2/deployments'  \
  --header 'Authorization: Basic YWRtaW46UGFzczEyMzQ=' \
  --header 'Content-Type: application/json'

```
If everything is fine then we should see this result:

```
 HTTP/1.1 200 OK
```
