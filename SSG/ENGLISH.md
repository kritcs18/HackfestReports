# SSG AI Chatbot with Deep Learning Hack

The AI Data Platform of SSG.COM has developed and provided its own AI chatbot using various open sources. However, they doubted about the effectiveness of the current architecture and decided to take a journey with Microsoft to obtain a better architecture. Every necessary area was verified in this Hackfest as much as possible - from deep learning to serverless.

## Key technologies

The technologies outlined and included in this solution are:

- [Azure Batch and Batch AI](https://azure.microsoft.com/en-us/services/batch-ai/) : Operating environment for deep learning model training
- [Azure Logic App](https://azure.microsoft.com/en-us/services/logic-apps/) : Notifying the change of a training model to the administrator
- [Web App for Container on Linux](https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-custom-docker-image) : The hosting environment of the AI inference service. Besides, web services that has Python dependency are configured to be served here(The reason is that Azure functions are in the Python experimental phase as of May 2018).
- [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/) : Private container image repository
- [Container WebHook](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-webhook) was used to apply [Continuous Deployment](https://docs.microsoft.com/en-us/azure/app-service/app-service-continuous-deployment) from Azure Container Registry to Web App for Container.
- [Deployment Slot](https://docs.microsoft.com/en-us/azure/app-service/web-sites-staged-publishing) : Deployment Slot was used to swap between staging and production of Web App for Container
- [Azure Functions](https://azure.microsoft.com/en-us/services/functions/) : A serverless platform for customer’s API Gateway (Chatbot)
- [Github Private Repository](https://github.com/) was used for continuous deployment (Update: GitHub was transferred to the VSTS project in March 2018)
- [Cosmos DB (SQL)](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction) : The state repository of chatting users
- [Azure Storage Queue](https://azure.microsoft.com/en-us/services/storage/queues/) : Communication media among Azure functions
- [Application Insight](https://azure.microsoft.com/en-us/services/application-insights/) : Monitors API gateway performance and use statistics

## Core Team

The team was composed of the SSG AI Data Platform team and Microsoft CSE engineers.

- SSG.COM
  - [Hoondong Kim](http://hoondongkim.blogspot.jp/) : Senior developer/deep learning, batch AI
  - Sung Ryu : Developer/deep learning, serverless
  - Donghoon Seo : Developer/docker, container
  - Younggyu Jeon : Developer/Python, docker, container
  - Insuk Seo : Developer/PHP, web App
- Microsoft 
  - Hun Choi : PM / Project Management
  - [Taeyoung Kim](https://github.com/taeyo) : Technical Evangelist / Architect, serverless, container, code migration, etc
  - [Chris Auld](https://github.com/cauldnz) : Software Engineer lead / Batch AI, deep learning, CNTK, etc
  - [Krit Kamtuo](https://github.com/kritcs18) : Software Engineer / batch AI, deep learning, CNTK, etc

## Hackfest period
Advance workshop: December 1, 2017   
Hackfest: December 18 ~ 22, 2017 (5 days)

## Partner Profile

Shinsegae Group is the representative and largest general merchandiser in Korea and running various businesses with its subsidiary companies, such as retail, fashion, hotel business, food and beverage, infrastructure, and multi-purpose shopping mall. Shinsegae Group launched an integrated online shopping service called "SSG.COM" in 2014 that combines several online shopping malls running independently, including E-Mart, Shinsegae Department Store. Traders, and Shinsegae TV Shopping. The SSG.COM service is growing very fast by more than 30% every year and its customer service center is receiving up to 20,000 service calls every day.

SSG.COM started an AI chatbot service development project based on the open source technology in the middle of 2017 and plans to release the service officially in the first half of 2018, to saving the operating expenses of the customer service center and provide better user experience to customers.

> Update: The production version service was officially opened at the end of March 2018

In addition, SSG.COM plans to add various types of AI technologies to their services, besides the customer service using the AI chatbot, in order to respond to the market situation that competitors are aggressively integrating AI technologies with their e-Commerce services.

## Problem Statement

Currently, SSG.COM is developing an AI chatbot service based on deep learning. All their demonstration services are tested in Azure VM (GPU DSVM (Data Science VM)). A total of four service areas are tested - AI model training layer, AI serving (inference) layer, API gateway (chatbot App) layer, and admin management web layer. At present, all services are tested in the Azure VM.

It was decided that the IaaS-based architecture is not efficient because it requires additional resources for infrastructure management and maintenance if the operation production scale is increased later, which will naturally require more VMs to manage. Therefore, they wanted an architecture that doesn’t use IaaS as much as possible. In addition, the AI model training layer doesn’t have to be running at all times but the GPU-supported VM should be used continuously, which is costly to the company. As a result, SSG.COM wanted to migrate all services to the managed service environment as much as possible, that is, an architecture that uses the PaaS environment.

In addition, the pre-test result of SSG.COM showed that the AI inference layer doesn’t really need a GPU. Therefore, SSG.COM wanted to make the use of numerous excellent functions features (automatic deployment, deployment slot, etc.) of the web App by applying the Azure App service as much as possible in this Hackfest.

> Note : The chatbot belongs to the real-time online inference scenario instead of batch inference and has the distinct characteristic of one batch size. The result of experimenting the inference performance difference between the CPU and GPU in this case is blogged here.
> 
> [Comparison of miniBatch Deep Learning Realtime Inference Performance, CPU vs GPU](http://hoondongkim.blogspot.kr/2017/12/deep-learning-inference-serving.html) 

SSG.COM also wanted to make improvements in such a way that developers can concentrate on individual logic only by introducing the micro service microservices or serverless architecture in the API gateway area that belongs to the chatbot area.

In addition, SSG.COM wanted to introduce overall automated deployment for all services. Previously, the application of all services was deployed manually. As SSG.COM decided that repetitive deployment work like this is inefficient, SSG.COM wanted to apply continuous deployment to as many parts as possible in this Hackfest. The below figure shows the overall existing architecture of SSG.COM.

![images/as-is-arch.png](images/as-is-arch.png)

## Requirements and Goals

The followings describe the requirements and goals defined in advance while carrying out Hackfest on this occasion.

- Each service should be operated in the PaaS environment if possible. 
    > Update: All services were configured and opened as Serverless FaaS (Function as a Service) and BaaS (Backend as a Service, web App) when the production version was launched at the end of March 2018. When those services were opened, there was no VM at all in the production zone, except the Dev zone.
- Each service should be design with a flexible and scalable architecture.
- AI model training should be designed to use multiple hosts and multiple GPUs.
- The continuous deployment strategy should be applied to the service layer that entails frequent deployment.
- The serverless architecture should be applied to the service layer, if applicable.
- All request/response data should be logged so that it can be viewed visually using a chart or other methods.

## Sources Repository

GitHub Private Repo was used for this Hackfest according to the SSG.COM’s request. Dedicated Repo was configured for each service layer and the below figure shows the configured repository.

> Update: It is changed to VSTS when opened.

Total 4 Private Repos, one for each service layer

![Github repos](images/Repos.png)

## Project Planning & Management

It took one and half month from an initial customer meeting to the completion of Hackfest. The Game Plan review process was conducted that reviewed and discussed an architecture with Technical Pillar, the Microsoft CSE engineer community, and conference calls, based on the requirements of the Shinsegae project team and 1st To-Be architecture that were secured after holding the envisioning workshop. The plan for the final architecture and technology to use were confirmed through this process.

![Project Milestone](images/SSG_Milestone.png)

The backlog of the SSG Hackfest was composed of 3 epics, 11 user stories, and 28 sub-tasks, and VSTS (Visual Studio Team Services) was used as a backlog management tool. Even though VSTS itself is a good tool that provides the CI/CD features and improves DevOps efficiency, it was used for backlog management in a limited way because build/release automation was out of the scope of this Hackfest. Besides VSTS, Slack was used as a collaboration tool among developers participating in Hackfest.

> Update: Afterwards, the SSG development team introduced VSTS when opening the project.

![VSTS Kanban](images/SSG_VSTS.png) 

![Slack Channel](images/SSG_Slack.png)

Microsoft and Shinsegae project developers shared the project progress status and Lessons & Learn at a 15 ~ 20-minute stand-up meeting every morning during the Hackfest period. At that time, the Kanban board with Post-it was used. Even though VSTS also provides a task status panel like the Kanban board, the physical Kanban board was also used to provide a motivation effect provided by the action that each developer can move the Post-It specifying task details according to the task progress

![Kanban Board](photos/hack_07.jpg) 

![Standup Meeting](photos/hack_03.jpg)

## Technical Delivery

During the Hackfest, the existing architecture was analyzed and an optimized To-Be architecture was defined for a total of 4 service areas that compose the entire production service. In addition, technology verification and migration were conducted for each service area together with the developer in charge of each area. The followings are the service area handled in this Hackfest.

1. AI Model Training Layer: A layer that conducts deep learning using Tenserflow and Keras.

2. AI Inference (& Serving) Layer: A layer that serves the model data with a web API using Python, Flask, and Keras. (This service analyzes an intent like the LUIS (Language Understanding Intelligent Service))

3. API Gateway (chatbot App) Layer: A web API service that plays a role of the chatbot application based on Python and Flask. SendBird was used to manage channels and rooms.

4. Admin Management WebSite Layer: Administrator web site developed with PHP. This layer can manage intents or entities and call real-time training.

## Proposed architecture

The below is the final architecture that was defined by considering technical aspects and regions and reflecting customer requirements for each service.

![Main Architecture](images/main_arch1.png)

However, some Azure resources that should be used for this architecture were not available in the Korea Region yet. As a result, the below shows the design that is most effective in terms of performance from the customer perspective and is distributed geographically. 

> Update: All Azure resources except Azure Batch AI became available when the service was opened at the end of March 2018. Currently, most of the Korea Region is utilized.

![Main Architecture by region](images/main_arch_region.png)

## Architectural Decisions & Solutions

SSG.COM and MS engineers have designed and implemented each service layer as follows.

### Layer 1: AI Model Training

Shinsegae came to the hack with an existing Long-Short-Term-Model (LSTM) based Recurrent neural network model. This model was effective but training efficiency and time was becoming a challenge. They wanted to be able to retrain the model more frequently as well as to experiment with dfferent network architectures and other hyperparameters. Our goal during the hack was to build out a pipeline to improve the training process while leaving the actual model untouched.

Key things that we wanted to achieve in the new pipeline were;
1. Be able to take advanatage of the [Azure Low Priority VMs] (https://azure.microsoft.com/en-gb/blog/announcing-public-preview-of-azure-batch-low-priority-vms/).
2. Be able to train the model across multiple GPUs and ideally multiple machines containing multiple GPUs.
3. Leave the model as unchanged as possible. It was important for Shinsegae that they were able to continue to run the model using TensorFlow and to build it using [Keras](https://keras.io/).

Microsoft Azure provides a range of [GPU equipped machine types](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-gpu). We worked primarily with the NC series machines which contain Kepler generation K80 GPUs. Now that they have become generally available we think that it is most appropriate to use the ND series machines. These provide the Pascal generation P40 GPU unit. These are ideally suited to training of deep neural networks thanks to their large memory capacity (24GB) and support for extremely high performance half precision (16 bit) FLOPs . There is a trend towards the use of half precision for training and inferencing deep neural nets and this is supported by the major frameworks. See *Courbariaux et al. Training deep neral networks with low precision multiplications. 2015.* [arXiv:1412.7024](https://arxiv.org/abs/1412.7024)

Azure N series instances are equpped with up to four GPUs. In order to support training across multiple nodes a high performance networking interconnect is required. The N*24r instances are equipped with a secondary high perormance [RDMA capable networking](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances). A high performance network is required because a key limitation to scale in DNN training is the transfer of weight updates between the computation nodes (and indeed between each GPU).

#### Architecutres Considered
We considered a number of architechtures for the pipeline build. This included;

- Hadoop + Spark + BigDL
- Hadoop + Spark + TensorFlowOnSpark
- Azure Batch AI + TensorFlow + Keras + Horovod

For a detailed discussion on these options and the evaluated pros and cons please see: [Deep Learning Multi Host & Multi GPU Architecture # 1 - Review and configuration with tensorflow, cntk, keras, horovod, Azure Batch AI](http://hoondongkim.blogspot.jp/2018/01/deep-learning-multi-host-multi-gpu.html). The post is in Korean language so please use *translate* via the Chrome browser.

The Shinsegae team have also evaluated the rleative performance of a multi-GPU scale-up approach via keras vs the Horovod scale out approach. [To summarize](http://hoondongkim.blogspot.jp/2018/01/deep-learning-multi-host-multi-gpu_11.html), the Keras shared weights approach is more efficient for single node multi-GPU training but will be limited to 8 GPUs. The delta between the two approaches narrows significantly at 4 GPUs which is the maximum available in an Azure N series machine.  

#### Multi-GPU and Multi-node training using Horovod
While TensorFlow does provide support for [distributed training](https://www.tensorflow.org/deploy/distributed), this often involves significant modifications to the model training program. Early on in the hack we concluded that we'd like to experiment with the [Horovod](https://github.com/uber/horovod) framework from Uber. 

Horovod applies an MPI based [work distribution pattern](https://github.com/uber/horovod/blob/master/docs/concepts.md) to TenorFlow training. From our initial investigations it appeared to do so in a fairly transparent way. It provides [support for Keras](https://github.com/uber/horovod/blob/master/examples/keras_mnist.py) which was critical to success for us.

#### Simple GPU cluster management with Azure Batch AI
Our team had significant previous experience using the [Azure Batch](https://azure.microsoft.com/en-us/services/batch/) HPC services available in Azure. Batch also provides access to the lower cost low priority VMs. We considered the use of [Azure Batch Shipyard](https://github.com/Azure/batch-shipyard) which is a docker based layer built on top of Batch. However, we settled on using the newly released [Batch AI](https://azure.microsoft.com/en-us/blog/batch-ai-public-preview/) service. because this had only been available for a couple of months prior to the hack this was the first time using it for all of our team. We found that our background expertise in both Batch and Batch Shipyard was useful in onboarding.

Batch AI provides automated stand-up and tear down of clusters of machines in the Batch service. These machines can then use any of a variety of ML frameworks executing either in the VM itself or via a docker container on top of the VM. We used the *linuxdsvmubuntu* image for training. There may be efficiency gains that can be achieved, particularly around cluster startup, by building a lighter weight image and container combination. We also install *MPI* via *apt* and *Horovod* via *PyPI* during machine startup and these could probably be incorporated into a container build.

If you'd like to experiement with this thn we recommend the below job configuration can be used with the [Batch AI TensorFlow GPU recipe](https://github.com/Azure/BatchAI/tree/master/recipes/TensorFlow/TensorFlow-GPU).

```
parameters = models.job_create_parameters.JobCreateParameters(
     location=cfg.location,
     cluster=models.ResourceId(cluster.id),
     node_count=2,
     input_directories=input_directories,
     std_out_err_path_prefix=std_output_path_prefix,
     container_settings=models.ContainerSettings(
         models.ImageSourceRegistry(image='tensorflow/tensorflow:1.1.0-gpu')),
     job_preparation=models.JobPreparation(
         command_line="apt update; apt install mpi-default-dev mpi-default-bin -y; pip install horovod"),
     custom_toolkit_settings = models.CustomToolkitSettings(
         command_line='mpirun -mca btl_tcp_if_exclude docker0,lo --allow-run-as-root --hostfile $AZ_BATCHAI_MPI_HOST_FILE python $AZ_BATCHAI_INPUT_SCRIPTS/trainAll_simple_v2.py'))
```

#### Customer Feedback

Kim Hundong, chief lead, technology lead of SSG.COM, was quite satisfied with CNTK and Batch AI technology that were handled in the Hackfest, because he majored in deep learning.  He expressed his opinions on Facebook and Slack several times that the Hackfest was quite helpful, and he wanted to inform the technical value he has experienced to other data developers. As a result, he arranged the experiment details he has handled in the Hackfest and shared with other developers by posting his article on the blog.

Title: Deep Learning Multi Host & Multi GPU Architecture with TensorFlow, CNTK, Keras, Horovod, Azure Batch AI
Link:    
http://hoondongkim.blogspot.kr/2018/01/deep-learning-multi-host-multi-gpu.html

Title: Deep Learning Multi Host & Multi GPU performance comparison on Azure Batch AI (TensorFlow + Keras + Horovod + Azure Batch AI) Link:    http://hoondongkim.blogspot.kr/2018/01/deep-learning-multi-host-multi-gpu_11.html

He even summarized those contents in phases and conducted a hands-on lab with the developer community with the following titles.

Title: Deep Learning with distributed GPU based on Azure Batch AI    
Link: https://onoffmix.com/event/123844


### Layer 2: AI Inference (& Serving) Layer

#### As-Is

AI Serving Layer is a web service that plays a role of AI inference. It was developed using the Python language and running on the Flask web server in [Data Science Virtual Machine](https://azure.microsoft.com/en-us/services/virtual-machines/data-science-virtual-machines/) (Ubuntu OS). (The development environment is Jupyter Notebook.) The used library includes TensorFlow (1.3.0), Keras (2.0.8), Python-twitter, etc.

This service plays a role similar to [Language Understanding Intelligent Service](https://www.luis.ai/)(Language Understanding Intelligent Service). The model (.H5) binary file that has been trained in AI Model Training Layer internally developed by the SSG.COM Data Platform team was uploaded to Flask web server memory using the h5py library to provide LUIS-related services. Currently, its major role is to understand the intent and entity of the chatbot users’ message but it will be developed continuously to play more roles.

#### What the Customer Wants

Basically, the existing service uses at least 2G of memory to run (as the size of the trained model is about 1.7G). It addition, it is designed to run in the Data Science VM based on the expectation that the use of a GPU will improve performance. (Run with a single VM without Availability Set because it’s a kind of verification situation.) However, SSG.COM thought that it would not be an optimal operating environment and wanted to verify which environment is better.
We presented our opinions during the prior technical meeting that satisfactory performance can be obtained using the CPU only if the deep learning model has a few layers, and SSG.COM agreed to our opinions and decided to check the result by running a performance test before the Hackfest.

The test result showed that AI Serving Layer developed by SSG.COM runs faster in a CPU environment, rather than a GPU environment. (See the below reference.) Accordingly, SSG.COM wanted to introduce the PaaS (Azure App Service) environment, instead of the IaaS scenario that uses a GPU-based VM, to secure stability and manageability as well as flexible development and deployment environment. Contrary to their initial expectations, the CPU performance was better than that of the GPU, because the chatbot is real-time inference and its miniBatch size is 1.

Refer to [Experiment and Performance Test for Deep Learning Inference & Serving: > GPU vs CPU (Korean)](http://hoondongkim.blogspot.jp/2017/12/deep-learning-inference-serving.html)
blog post by Hoondong Kim (SSG Chief Dev, AI Data platform team)

Topic: Is GPU really better than CPU in AI Inference?    
Conclusion: CPU is better than GPU in their cases.

#### Analysis and Design

The approach reviewed after the envisioning meeting was that we can install Python libraries in the web App in advance, which are needed in the KUDU (SCM) environment of the web App (Windows), and utilize many benefits provided by the web App environment such as continuous deployment, auto scale, deployment slot, App service environment, etc.). However, there were opinions that the use of the Linux OS is more desirable, as the libraries used for development were based on the Linux OS. Also, opinions were presented during the technical meeting with the TWG - Compute group that a container-based architecture seemed better for this part. As a result, Web App for Container was finally selected as the architecture. (Even though Web App for Container didn’t provide all web App features, it was selected as an appropriate scenario because Development Slot and Continuous Deployment can be used, which was wanted by SSG.COM) The below figure shows the architecture designed in this way.

![images/AIServ-arch01.png](images/AIServ-arch01.png)

However, it was decided to exclude Docker build and Push automation using VSTS (Visual Studio Team Service) in the Hackfest, because there was a possibility that an issue related to the QA policy of SSG.COM (test and QA related issue) can be raised. Consequently, it was agreed to run a local test using a separate dedicated Docker Build machine (VM) and push to the Azure Container Registry manually.

> Update: Migrated to VSTS when opening the production version and entire automation transfer is still in progress.

#### Continuous Deployment Scenario

TE reviewed the deployment scenario from the various aspects for 2 ~ 3 days before the Hackfest. Actually, the existing system was re-developed based on Container together with SSG.COM for 2 days during the Hackfest period and was successfully executed on Web App for Container. Azure Container Registry in the same region with Web App for Container was used as the Private Registry of Docker images, and Registry Webhook was set to automatically deploy a new Docker image among the Stage Slot of the web App when it is pushed. When deployment is finished, the person in charge in SSG.COM checks the state of each slot and manually swaps. Continuous Deployment defined in this way is as follows.

![images/AIServ-arch02.png](images/AIServ-arch02.png)

The below snapshot shows Dockerfile code, Azure Container Registry of the web App, and history of executing automatic deployment using Webhook.

Azure Container Registry Settings

![Azure Container Registry Settings](images/AiServ_Docker_Container_s.png)

Dockerfile Example

![Dockerfile](images/dockerfile.png)

Push History via Container WebHook

![Push history via WebHook](images/AiServ_ACR_s.png)

#### Considering the Region

Web API Gateway (chatbot) Layer, which plays a role of a gateway and corresponds to the chatbot App, is available in Korea but AI Serving/Inference Layer (AI service like LUIS) is served in Japan as Web App for Container, because Azure Container Registry is not supported in Korea yet. (The deployment cost and speed of Azure Container Registry and Web App for Container are efficient when they are located in the same region.) If ACR (Azure Container Registry) can be created in Korea, latency can be reduced more. SSG.COM plans to move all services related to AI Serving Layer (ACR, Web App for Container, Blob Storage for H5 model files, etc.) to Korea when the ACR service is launched in Korea.

> Update: The Korea Region was available when the service was opened and the service in question was opened in the Korea region.

#### Issues and Workaround

One of the problems experienced while migrating existing modules to the managed service environment was related to the loading time limit. The AI Inference service developed by SSG.COM using Python runs in such a way that the 1.7G model file is loaded into memory for the service when the Flask web server starts. Of course, this structure actually needs improvements or re-design. However, it cannot be modified now and should be improved later according to their internal development schedule. Therefore, the service should run using the developed module as it is now. The problem is that the developed web App takes 4 ~ 5 minutes for loading when starting for the first time. However, Web App for Container sets the initial start time limit to 230 seconds by default and cannot load a web App that takes more than 230 seconds for loading. Therefore, the existing service of SSG.COM could not be started properly and it was found that the following Container log was left continuously.

```python
ERROR - Container site *****  did not start within expected time limit
```

As described above, this problem occurred because SSG.COM modules took at least 240 ~ 300 seconds for loading. To solve this problem, **WEBSITES_CONTAINER_START_TIME_LIMIT** was set to **600** seconds in the App Settings of the web App. As a result, the App was loaded successfully.

Application Setting for Stage

![Application Setting for Stage](images/AiServ_Stage_App_Settings_s.png)

#### What the customer considers for better performance

However, this model data can grow bigger and bigger continuously and there is a high possibility that the data cannot be loaded properly in the long run. Besides, it is another serious problem that initial loading currently takes too long for deployment and swapping. Therefore, this issue needs to be improved technically by tuning (or modifying development) the application. SSG.COM has also agreed to this issue and implied that they will start to think about how to solve the problem technically.

Besides, the created Dockerfile builds an image by including model files, after copying all model files needed to build an image to the local directory using wget, because the h5py library, used by SSG.COM, has a limitation that only the file in the local directory can be loaded. If it can be modified to allow Python to directly load the h5 file in the remote location (Blob Storage), or provide similar functions using other libraries, model binary files don’t have to be included in the Docker image, which will remarkably reduce the build image size and data traffic needed for push.

#### Customer Feedback

The current Web App for Container platform takes too much time in swapping among Deployment Slots. Even swapping after checking both production and stage run properly takes as much time as loading a new image again (even two web Apps are already in an operational state). The Microsoft development team needs to improve the problem that swapping takes too long.

The size of the deep learning model to deploy could be reduced by one third by separating the embedding layer file, changing the model file format, and freezing variables except the online training part. The adjusted size didn’t interfere with operation. TensorFlow Serving was not used in the inference layer to reduce the burden of converting the Keras model for TensorFlow and dependency on gRPC, create an environment that enables training as soon as online serving is available, and create a model that even CNTK Backend can be utilized.

When serving deep learning with Flask, additional performance tuning is required due to the characteristics of Flask and Python. To increase the performance of Flask Deep Learning Serving by more than 10 times in the Production environment, additional settings are needed in Docker (e.g., nginx + flask + uwsgi), which requires different combinations depending on the characteristic of the model.

### Layer 3: API Gateway (chatbot App) Layer

#### As-Is

Actually, the API Gateway is the role of the chatbot App. SendBird is used to manage channels and rooms. All chats of the bot user using WebHook of SendBird are transferred to and processed by this service layer. The API Gateway was developed based on Python/Flask and operated in a separate VM. (The development environment is Jupyter Notebook.)
What the Customer Wants

SSG.COM wanted to change the chatbot service environment to the managed service environment, not IaaS, during the Hackfest. SSG.COM also wanted to re-arrange logic by separating existing service logic, and apply the serverless architecture at the same time, because they thought that this layer would be most effective if the serverless architecture was applied. SSG.COM wanted to manage the layer more efficiently and effectively by applying the serverless architecture and enable developers to improve the service continually by concentrating on each individual chatbot logic.

#### Analysis and Design

The following problems were found when analyzing existing service logic at the source code level.

**Problems**
- The API call was composed of one big synchronous call.
- All calls executed the logic of a large method conditionally and didn’t respond until logic was completed.
- Internally, a call to an external API occurred several times and ramification was repeated each time.
- Redis was used to keep the session information and codes to support Redis were internally developed and controlled.

When all existing logic was analyzed individually, the request and response of the existing Python code was composed of a single Sync call.  In addition, logic was branched internally according to various conditions. That is, all user inputs should call AI Serving Layer to analyze the intent and logic was branched according to the intent obtained after the call and various conditions. Therefore, logic was skipped or an external call to another API occurred depending on the situation. The external call was invoked to invoke the web API developed by other departments or invoke QnA Maker developed in Azure, in order to obtain the data needed for reply additionally.

Even worse, it had an inefficient structure that all processing was executed synchronously in a single big method in terms of logic. Therefore, the architecture structure was changed to an asynchronous one and each logic was separated to migrate to the serverless architecture at the same time. The architecture was changed in such a way that communication among each logic was established through the message queue and the final response message was sent to SendBird asynchronously using a proactive method.

The followings are the solution changed in such a way.

**Solutions**

- Separated all branching logic into each individual function as much as possible.
- Changed all calls to asynchronous calls.
Responded to the user asynchronously using the proactive message.
- Utilized various queues according to the purpose and branched logic using the queue. 
    - Can be replaced with Event Hub if load increases later.
- Saved the user’s state in Cosmos DB using the input/output binding of the function.

As a result, the architecture was configured as follows.

![APIgw-arch01.png](images/APIgw-arch01.png)

#### Re-Design Control Flow and Logics

Existing codes were analyzed according to the defined architecture and it was checked whether those codes can be re-written. SSG.COM hoped to use Python if it was possible. However, a more stable programming language should have been selected because Python was supported experimentally only in Azure Function. Consequently, SSG.COM agreed to migrate all existing codes to C# and existing Python codes and control flows were needed to be disassembled according to the serverless architecture as the development language and platform were changed.

All logic was analyzed with the person in charge of API gateway development in SSG.COM for one day and separation task was performed at the function level, and the user state information that was stored in Redis was transferred to Cosmos DB for enhanced stability and availability. As logic was separated and arranged, the architecture became more simplified and unnecessary codes were removed clearly.

![APIgw-arch02.png](images/APIgw-arch02.png)

The below figure shows the architecture that separates developed API gateway logic into functions through this process.

![APIgw-arch03.png](images/APIgw-arch03.png)

Messages were transferred to different queues according to the branching condition and each branching logic is changed in such a way that it is conducted when a message is received by the pertinent queue only. The change has the advantage that logic is more intuitive than using “if” and “else” if excessively in existing logic and developers can improve codes by concentrating on each logic. Through this work, existing logic written in Flask and Python was completely migrated to C# and Function. The following figure shows the example of the existing code.

![APIgw-code1.png](images/APIgw-code1.png)

There was a little concern in the initial stages as SSG.COM developers were not familiar with C#. However, as triggers, input, and output can be easily set using attributes in Function, migration was completed easily contrary to the SSG.COM developers’ concern. SSG.COM developers were quite satisfied when the areas that they had expected difficulty (queue and Cosmos DB interface codes) were linked with simple attribute setting only. Therefore, the only thing they had to do was changing business logic with C# only and code migration was completed within 2 days. SSG.COM developers were rather satisfied with the outstanding functions of the C# language and convenience of Visual Studio after migration. The below figure shows the example of codes that were actually used.

![APIgw-code3.png](images/APIgw-code3.png)

#### Performance

When a mock test was performed after separating all codes based on the serverless architecture, the result was satisfactory (response within 2 seconds), because the response time includes the time required to call API Serving Layer, which takes the longest time.

In addition, Application Insight, which is a kind of application performance monitoring tool provided by Azure, was applied to the Function App to monitor the performing time of each Function. Therefore, the chatbot App is now able to monitor all calls, calling frequency, and execution time of each API in real time, using Application Insight. It is helpful for debugging and understanding what should be improved later, because the developer can check the function frequently called, performing time, and cause of exception by function and in real time.

Currently, Consumption Plan is not supported in Korea and SSG.COM wants to check which cost plan is appropriate by adjusting the App Service Plan directly in the initial stages. Therefore, the Function App is now operated under the S1 size plan. A higher plan will be used when actual operation is started or user requests are concentrated.

#### Issues and Workaround

All development source codes should have been integrated with the Private Repository of GitHub and configured to be deployed automatically using the ‘Deployment Option’ feature of the web App. However, the GitHub integration function provided by the Deployment Option of the web App currently supports Public Repository only (not Private Repository). Therefore, a separate workaround was needed to solve this problem. To solve the problem, some manual work should be performed using the solution proposed by the following link.
Refer to https://nsamteladze.wordpress.com/2015/07/19/continuous-deployment-from-github-enterprise-repository-to-azure-web-app/

#### Customer Feedback

SSG.COM was a little concerned about whether they can finish migration within the project period including code migration in the initial stages of the Hackfest. However, they said they are impressed because they could analyze all logic and migrate codes together with Microsoft engineers and complete the project on schedule. They were also quite satisfied with serverless functions provided by Azure. They also said that they are happy because they can focus on business logic only without paying attention to infrastructure codes or platform-based codes anymore, by introducing the serverless architecture and technology.

### Admin Management WebSite Layer

#### As-Is

The Admin Web Site is a web site exclusive for the administrator. The administrator can carry out various management functions associated with the intent and entity, such as entering various utterance and registering various query-related rules. The web site also provides the function of combing proper response messages according to the identified intent or managing the response message depending on the individual situation. In addition, the administrator can execute a command to train model immediately using an on-demand method. The administrator web site is developed with PHP and provides fancy UI using various CSS and bootstrap UI. Currently, it is hosted in the Azure VM and source codes are distributed and managed manually.

#### What the Customer Wants

Like other layers, SSG.COM wanted to apply the PaaS platform to Admin WebSite Layer that can scale in/out at any time, rather than managing it in a separate VM. However, some client libraries were scattered somewhat intricately and the existing structure was needed to be integrated with the sub-folder of a single folder to migrate to Azure App Service. Therefore, they wanted to migrate the existing source code structure developed for the IaaS environment so that it can be operated in PaaS smoothly. They also wanted that the source code committed in GitHub Private can be reflected in the web App immediately.

In addition, Maria DB was used as a data repository and they wanted to change it to MySQL on Azure. However, as MySQL on Azure was in a preview state yet and not supported in the Korea Region, it was decided to exclude tasks related to MySQL on Azure from the Hackfest.

> Update: MySQL on Azure became GA one week before opening the production version at the end of March 2018. Therefore, MySQL on Azure used with the developer version could be applied to the production version. As a result, the meta DB was also opened by applying PaaS.

#### Analysis and Design

Some client libraries were scattered among VM locals some intricately and mapped with a relative path. Therefore, the existing scatter folder structure was needed to be integrated with the sub-folder of a single folder to migrate to Azure App Service. Therefore, libraries in use and libraries not in use among tens of downloaded client libraries were checked and only those in use were arranged.

In addition, the existing file upload function required modification at the code level so that it could run properly in App Service. Previously, the code was written in such a way that when the user uploads a file, the file is saved in the Temp drive of the local server and moved to a specific location in the server. It should have been modified to be saved in Azure Storage in an integrated manner.

Besides, other matters that require attention when introducing PaaS were transferred to SSG.COM, so that it can be referred during additional development. The links of the reference document are as follows.

Considerations when migrating the web App to Azure PaaS
https://github.com/taeyo/AzurePaaS/tree/master/ConsiderationWhileMigrateWebAppToAzure

Structural example to refer when migrating to the Azure web App 
https://github.com/taeyo/AzurePaaS/tree/master/WebAppBasicArch

The following is the architecture that is created by reflecting this.

![Admin Web Site Architecture](images/admin01.png)

The followings are the part of the source that modifies the existing code to use Blob Storage.

```php
<?php
...
use MicrosoftAzure\Storage\Common\ServicesBuilder;
use MicrosoftAzure\Storage\Blob\Models\CreateContainerOptions;
use MicrosoftAzure\Storage\Blob\Models\PublicAccessType;
use MicrosoftAzure\Storage\Common\ServiceException;

/*... Omitted ... */

    $blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);
    $pic = "ICO_".microtime(TRUE)."_".$_FILES['iconf']['name'];
    $pic_loc = $_FILES['iconf']['tmpName'];

    $folder = "/home/site/wwwroot/****/";
    $arrayValue = strtolower(array_pop( explode( '.', $pic )));

    if ( $arrayValue == "****" ) {
        if(move_uploaded_file($pic_loc,$folder.$pic) ) {
            //echo $folder.$pic;
            $image = $folder.$pic;
            $content = fopen($image,"r");
            $blob_name = $pic;
            try{
                $blobRestProxy->createBlockBlob("files", $blob_name, $content);
            }
            catch(ServiceException $e){
                $code = $e->getCode();
                ...
            }
```

And, the followings show actual operation.

![관리자 UI](images/adminweb_ui01.png)

![관리자 UI](images/adminweb_ui02.png)

![관리자 UI](images/adminweb_ui03.png)

SSG.COM is supplementing the part that has a possibility of problem occurrence by running various tests after migration and still adds and improves functions. The following shows Repo that code commit is still executed recently.

![Repo](images/admin02.png)

#### Issues and Workaround

Like other layers, this layer (admin management web site) should have been integrated with the Private Repository of GitHub and configured to be deployed automatically using the Deployment Option feature of the web App. However, as mentioned before, a workaround should be used to integrate Azure App Service with Private Repository. Refer to the following link regarding this issue.

Refer to https://nsamteladze.wordpress.com/2015/07/19/continuous-deployment-from-github-enterprise-repository-to-azure-web-app/

> Update: GitHub was transferred to VSTS when opened.

## Moments of the Hackfest

Envisioning Session

![1](photos/hack_envision01.jpg)| ![2](photos/hack_envision02.jpg)
![3](photos/hack_envision03.jpg)| ![4](photos/hack_envision04.jpg)

Hackfest

![4](photos/hack_01.jpg) | ![6](photos/hack_02.jpg)
![7](photos/hack_03.jpg) | ![8](photos/hack_04.jpg)
![9](photos/hack_05.jpg) | ![10](photos/hack_08.jpg)
![9](photos/hack_07.jpg) | ![10](photos/hack_06.jpg)

Video : SSG.COM Hackfest

[![SSG.COM Hackfest](images/MPNG.PNG)](https://www.youtube.com/watch?v=Red-D5ChNzk)


## Business impact 

Many e-Commerce/online shopping mall businesses that are in competition with SSG.COM have already integrated AI technologies with their services actively. To cope with this situation, Shinsegae is also making heavy investments in the AI technology and plans to release many AI-related services, beginning with the AI bot service. Shinsegae is now equipped with a competitive AI bot platform that can reduce the time-to-market of new services dramatically, by implementing the serverless/PaaS-based platform to perfection on this occasion.

## Partner technical engagement feedback

The followings are the feedback of the SSG.COM developers who participated in the Hackfest.

> "As I had to lead the project that is an AI project, not the existing web project, I tried to solve the problem of redundancy, multi-user, and GPU machine costs, which I have not experienced before, using PaaS and concentrate on business logic only. I also tried to use GPU resources for deep learning training efficiently using PaaS, if those resources are heavily used temporarily and not always used. Besides, I tried to configure the unfamiliar AI service safely and flexibly using the auto scale-out feature or non-stop deployment feature. I could develop all solutions in the area I wanted during the Hackfest and it was a meaningful experience because I can apply the experience to the production phase."
> 
> Kim Donghun, chief lead, SSG AI Data Platform 

> "In this Hackfest, I participated in AI Model Training Layer as a sub-member and led API Gateway Layer as a main leader. In particular, the impressive part was that all Python logic of API Gateway was disassembled and re-configured completely based on serverless. I could not imagine that code migration can be completed within the Hackfest period. However, it was actually completed, to my surprise. Thanks to the Hackfest, I myself could experience how convenient the serverless technology is. If another chance is given, I’d like to participate in Hackfest again."
>
> Ryoo Seong, developer, SSG AI Data Platform

## Conclusion

### Our Impact

- The AI chatbot of SSG.COM has successfully changed the platform from IaaS to PaaS after the Hackfest and the case was shared inside of SSG.COM and evaluated as an exemplary case by other departments. In addition, the architecture affects other departments.

- Lead Kim Hundong continues to run various tests running on Azure Batch AI by combining TensorFlow, Keras, CNTK, and Horovod after the Hackfest. He is improving the service layer more efficiently by continuing the test (by separating the pre-processing and post-processing from the existing module). In addition, he is continually sharing technical value he has experienced with other data experts using his blog.

    https://www.facebook.com/kim.hoondong/posts/10214834432715314

    Deep Learning Multi Host & Multi GPU Architecture
Link: 
    http://hoondongkim.blogspot.kr/2018/01/deep-learning-multi-host-multi-gpu.html

    Deep Learning Multi Host & Multi GPU performance comparison on Azure Batch AI (TensorFlow + Keras + Horovod + Azure Batch AI) Link: 
    http://hoondongkim.blogspot.kr/2018/01/deep-learning-multi-host-multi-gpu_11.html

    He even summarized those contents in phases and conducted a hands-on lab with the developer community with the following titles.

    HOL: Deep Learning with Multi-GPU based on Azure Batch AI
FB Link: 
    https://www.facebook.com/groups/krazure/permalink/1840871035955057/  
    Link: https://onoffmix.com/event/123844

- After the Hackfest, the administrator web site is making functional improvements as changes are committed several times a day and the web App is deployed continually.

### General lessons

Running the AI Model Training service on Azure Batch AI was a great choice. In particular, an optimal architecture could be obtained that fits the AI Data Team of SSG.COM by running various tests using CNTK or Horovod.

Integrating the Container technology with AI Inference Layer was also a good attempt. We could check whether the existing method is really meaningful from a new perspective, which depends on the GPU by habit. And, we could secure an architecture that is improved on costs and technology through several verification. In particular, integrating the Container technology with Azure Web App was quite effective.

The complete migration of the chatbot App to the serverless architecture was also a great achievement. SSG.COM developers now can focus on individual logic only and can have significant effects with less efforts when using various external services (state server, database, queue, etc.). As trust in the technology was built in this way, SSG.COM considers using Microsoft Bot Framework for the next version.
