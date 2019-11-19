# IBM Cloud Functions and Apache OpenWhisk - Lab 2

## Introduction

This lab walks you through the steps required to create, build, and run a Serverless application using IBM Cloud Functions. Serverless computing refers to a model where the existence of servers is entirely abstracted away. Even though servers exist, developers are relieved from the need to care about their operation. They are relieved from the need to worry about low-level infrastructural and operational details such as scalability, high-availability, infrastructure-security, and other details. Serverless computing is essentially about reducing maintenance efforts to allow developers to quickly focus on developing code that adds value.

Serverless computing simplifies developing cloud-native applications, especially microservice-oriented solutions that decompose complex applications into small and independent modules that can be easily exchanged. Some promising solutions like Apache OpenWhisk have recently emerged that ease development approaches used in the serverless model. IBM Cloud Functions is a Function-as-a-Service (FaaS) platform on IBM Cloud, built using the Apache OpenWhisk open source project, that allows you to execute code in response to an event.
It provides a serverless deployment and operations model. With a granular pricing model that works at any scale, you get exactly the resources you need – not more, not less – and you are charged for code that is really running.

IBM Cloud Functions provides:

- Support for multiple languages, including JavaScript, Python, Swift, and Java
- Support for running custom logic through Docker containers
- The ability to focus more on value-adding business logic, and less on low-level infrastructural and operational details.
- The ability to easily chain together microservices to form workflows via composition.

## Prerequisites
1. You will need an IBM Cloud Account. Either use your existing account, or create a new account by using the link provided in the workshop.
2. Install IBM Cloud CLI - https://cloud.ibm.com/docs/cli/reference/bluemix_cli?topic=cloud-cli-install-ibmcloud-cli#shell_install
3. Install the Functions plugin - https://cloud.ibm.com/functions/learn/cli

## Create an action
### Step 1 - Log into your IBM account using the CLI
1. Use the CLI to log into your account:
```
» ibmcloud login                                                         
API endpoint: https://cloud.ibm.com
Region: us-south

Email> upkar.ibm.watson.5@gmail.com

Password>
Authenticating...
OK

Targeted account Upkar Lidder's Account (a086ce7d00df4423ab024b123b587e76)


API endpoint:      https://cloud.ibm.com
Region:            us-south
User:              upkar.ibm.watson.5@gmail.com
Account:           Upkar Lidder's Account (a086ce7d00df4423ab024b123b587e76)
Resource group:    No resource group targeted, use 'ibmcloud target -g RESOURCE_GROUP'
CF API endpoint:
Org:
Space:
```

2. Target your Cloud Foundary space using the following command
```
» ibmcloud target --cf                                                    
Targeted Cloud Foundry (https://api.us-south.cf.cloud.ibm.com)

Targeted org upkar.ibm.watson.5@gmail.com

Targeted space dev



API endpoint:      https://cloud.ibm.com
Region:            us-south
User:              upkar.ibm.watson.5@gmail.com
Account:           Upkar Lidder's Account (a086ce7d00df4423ab024b123b587e76)
Resource group:    No resource group targeted, use 'ibmcloud target -g RESOURCE_GROUP'
CF API endpoint:   https://api.us-south.cf.cloud.ibm.com (API version: 2.142.0)
Org:               upkar.ibm.watson.5@gmail.com
Space:             dev
```

3. Ensure that you installed the [Cloud Functions plugin](https://cloud.ibm.com/functions/learn/cli) correctly. You might see an empty list if you don't have any actions deployed. You should not get any error at this point.

```
» ibmcloud fn action list                                                
actions
/upkar.ibm.watson.5@gmail.com_dev/att-hello                            private nodejs:10
/upkar.ibm.watson.5@gmail.com_dev/hello-debug                          private nodejs:10
/upkar.ibm.watson.5@gmail.com_dev/MYACTION                             private nodejs:10
/upkar.ibm.watson.5@gmail.com_dev/upkar-hello-debug                    private nodejs:10
/upkar.ibm.watson.5@gmail.com_dev/my-service-dev-hello                 private nodejs:10
/upkar.ibm.watson.5@gmail.com_dev/paypal-async                         private nodejs:10
/upkar.ibm.watson.5@gmail.com_dev/paypal-hello                         private nodejs:10
```

### Step 2 - Create a visual recognition service and get the credentials in the UI.
1. Log into the IBM Cloud Dashboard.
2. Click on `catalog` to create a new `Visual Recognition` service. If you already have one created, look it up in `Resources`.
3. Get the credentials from the service page.
**link to image**
### Step 2 (alternatively) - Create a visual recognition service and get the credentials in the CLI.
1. Check if you already have a visual recgonition service from the cli. If not, create a new one.
```
» ibmcloud resource service-instances                                     [9:35:37]
Retrieving instances with type service_instance in all resource groups at all locations under account Upkar Lidder's Account as upkar.ibm.watson.5@gmail.com...
OK
Name                                Location   State    Type
Watson Studio-35                    us-south   active   service_instance
cloud-object-storage-gv             global     active   service_instance
Tone Analyzer-jm                    us-south   active   service_instance
Visual Recognition-8t               us-south   active   service_instance
Machine Learning-6z                 us-south   active   service_instance
Watson OpenScale-nf                 us-south   active   service_instance
Cloudant-dn                         us-south   active   service_instance
Speech to Text-uq                   us-south   active   service_instance
IBM Cognos Dashboard Embedded-wr    us-south   active   service_instance
Voice Agent with Watson-pa          us-south   active   service_instance
Watson Assistant-8k                 us-south   active   service_instance
Natural Language Understanding-hu   us-south   active   service_instance
Watson Assistant-a5                 us-south   active   service_instance
Language Translator-8w              us-south   active   service_instance
```

As you can see, I already have a visual recognition service named `Visual Recognition-8t` created. If you don't see any visual recognition service, create one with the following command

```
» ibmcloud resource service-instance-create my-visual-recognition watson_vision_combined lite us-south
Creating service instance my-visual-recognition in resource group Default of account Upkar Lidder's Account as upkar.ibm.watson.5@gmail.com...
```
If CLI complains about empty resource group, use the following command to target the `Default` resource group.

```
» ibmcloud target -g Default
Targeted resource group Default

API endpoint:      https://cloud.ibm.com
Region:            us-south
User:              upkar.ibm.watson.5@gmail.com
Account:           Upkar Lidder's Account (a086ce7d00df4423ab024b123b587e76)
Resource group:    Default
CF API endpoint:   https://api.us-south.cf.cloud.ibm.com (API version: 2.142.0)
Org:               upkar.ibm.watson.5@gmail.com
Space:             dev
```

3. Get the credentials from the CLI

```
» ibmcloud resource service-keys --instance-name 'Visual Recognition-8t' --output json
[
    {
        "guid": "f39ee417-dfd1-44a3-a997-687a63da2d4c",
        "id": "crn:v1:bluemix:public:watson-vision-combined:us-south:a/a086ce7d00df4423ab024b123b587e76:5e94eb87-f85c-45cf-96f6-ba4cbfae35eb:resource-key:f39ee417-dfd1-44a3-a997-687a63da2d4c",
        "url": "/v2/resource_keys/f39ee417-dfd1-44a3-a997-687a63da2d4c",
        "created_at": "2019-07-23T22:16:30.516812598Z",
        "updated_at": "2019-07-23T22:16:30.516812598Z",
        "deleted_at": null,
        "name": "Auto-generated service credentials",
        "account_id": "a086ce7d00df4423ab024b123b587e76",
        "resource_group_id": "91a2b0eb7a884016a2cc8e8aca66dcd1",
        "source_crn": "crn:v1:bluemix:public:watson-vision-combined:us-south:a/a086ce7d00df4423ab024b123b587e76:5e94eb87-f85c-45cf-96f6-ba4cbfae35eb::",
        "state": "active",
        "credentials": {
            "apikey": "Afg7cT4f4x4feU6-uzT7Ufgadbnai69b7zIv8cXZKdlm",
            "iam_apikey_description": "Auto-generated for key f39ee417-dfd1-44a3-a997-687a63da2d4c",
            "iam_apikey_name": "Auto-generated service credentials",
            "iam_role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Manager",
            "iam_serviceid_crn": "crn:v1:bluemix:public:iam-identity::a/a086ce7d00df4423ab024b123b587e76::serviceid:ServiceId-eb1e2b7f-a1c4-428c-adb2-9b78cf4e2746",
            "url": "https://gateway.watsonplatform.net/visual-recognition/api"
        },
        "iam_compatible": true,
        "resource_instance_url": "/v2/resource_instances/5e94eb87-f85c-45cf-96f6-ba4cbfae35eb",
        "crn": "crn:v1:bluemix:public:watson-vision-combined:us-south:a/a086ce7d00df4423ab024b123b587e76:5e94eb87-f85c-45cf-96f6-ba4cbfae35eb:resource-key:f39ee417-dfd1-44a3-a997-687a63da2d4c"
    },
```
4. You need the `credentials` object from the json above.

```
"credentials": {
            "apikey": "Afg7cT4f4x4feU6-uzT7Ufgadbnai69b7zIv8cXZKdlm",
            "iam_apikey_description": "Auto-generated for key f39ee417-dfd1-44a3-a997-687a63da2d4c",
            "iam_apikey_name": "Auto-generated service credentials",
            "iam_role_crn": "crn:v1:bluemix:public:iam::::serviceRole:Manager",
            "iam_serviceid_crn": "crn:v1:bluemix:public:iam-identity::a/a086ce7d00df4423ab024b123b587e76::serviceid:ServiceId-eb1e2b7f-a1c4-428c-adb2-9b78cf4e2746",
            "url": "https://gateway.watsonplatform.net/visual-recognition/api"
        }
```

### Step 3 - create the action
## Command line labs

## Intermediate command line labs
If you want to practice more with the cli, you can follow the following labs
1. [Setting up the environment.](https://github.com/IBM-Cloud/openwhisk-workshops/tree/master/bootcamp/ex0%20-%20setting%20up%20development%20environment)
2. [Bundling NPM modules](https://github.com/IBM-Cloud/openwhisk-workshops/tree/master/bootcamp/ex1.3%20-%20bundling%20NPM%20modules)
