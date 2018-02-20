# SSG AI Chatbot with Deep Learning Hack

TBU

## Key technologies

The technologies outlined and included in this solution are:

- [Azure Batch and Batch AI](https://azure.microsoft.com/en-us/services/batch-ai/) to train deep learning model
- [Azure Logic App](https://azure.microsoft.com/en-us/services/logic-apps/) to notify changes of trained model
- [Web App for Container on Linux](https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-custom-docker-image) to host AI inference Service
- Private container registry using [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/).
- [Continuous Deployment](https://docs.microsoft.com/en-us/azure/app-service/app-service-continuous-deployment) using [Container WebHook](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-webhook) from Azure Container Registry to Web App for Container  
- [Deployment Slot](https://docs.microsoft.com/en-us/azure/app-service/web-sites-staged-publishing) for staging/production environments swap in Web App for Container
- [Azure Functions](https://azure.microsoft.com/en-us/services/functions/) to host API Gateway(customer's Chatbot) with Serverless Architecture
- Continuous Deployment with [Github Private Repository](https://github.com/)
- [Cosmos DB (SQL)](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction) for managing state of end users
- [Azure Storage Queue](https://azure.microsoft.com/en-us/services/storage/queues/) for Reactive communication between Azure Functions
- [Application Insight](https://azure.microsoft.com/en-us/services/application-insights/) to monitor performance and usage of API gateway

## Core Team

The team was comprised of members from SSG AI team and Microsoft CSE:

- SSG.COM
  - [Hoondong Kim](http://hoondongkim.blogspot.jp/) : Chief Lead / Deep Learnig, Batch AI
  - Sung Ryu : Developer / Deep Learnig, Serverless
  - Donghoon Seo : Developer / Docker, Container
  - Younggyu Jeon : Developer / Python, Docker, Container
  - Insuk Seo : Developer / PHP, Web App
- Microsoft 
  - [Taeyoung Kim](https://github.com/taeyo) : TE / Architecture, Serverless, WebApp, Container, Code Migration etc
  - Hun Choi : PM / Project Management
  - [Chris Auld](https://github.com/cauldnz) : TE Manager / Batch AI, Deep Learning, CNTK etc
  - Krit kamuto : TE / Batch AI, Deep Learning, CNTK etc

##Technical Delivery
1. AI Model Training Layer: This layer will support training of model at scale. We use Keras over TensorFlow with Horovod. Training uses Azure GPU machines orchestrated via Azure BatchAI.

## Layer 1: AI Model Training
Shinsegae came to the hack with an existing Long-Short-Term-Model (LSTM) based Recurrent neural network model. This model was effective but training efficiency and time was becoming a challenge. They wanted to be able to retrain the model more frequently as well as to experiment with dfferent network architectures and other hyperparameters. Our goal during the hack was to build out a pipeline to improve the training process while leaving the actual model untouched.

Key things that we wanted to achieve in the new pipeline were;
1. Be able to take advanatage of the [Azure Low Priority VMs] (https://azure.microsoft.com/en-gb/blog/announcing-public-preview-of-azure-batch-low-priority-vms/).
2. Be able to train the model across multiple GPUs and ideally multiple machines containing multiple GPUs.
3. Leave the model as unchanged as possible. It was important for Shinsegae that they were able to continue to run the model using TensorFlow and to build it using [Keras](https://keras.io/).

Microsoft Azure provides a range of [GPU equipped machine types](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-gpu). We worked primarily with the NC series machines which contain Kepler generation K80 GPUs. Now that they have become generally available we think that it is most appropriate to use the ND series machines. These provide the Pascal generation P40 GPU unit. These are ideally suited to training of deep neural networks thanks to their large memory capacity (24GB) and support for extremely high performance half precision (16 bit) FLOPs . There is a trend towards the use of half precision for training and inferencing deep neural nets and this is supported by the major frameworks. See *Courbariaux et al. Training deep neral networks with low precision multiplications. 2015.* [arXiv:1412.7024](https://arxiv.org/abs/1412.7024)

Azure N series instances are equpped with up to four GPUs. In order to support training across multiple nodes a high performance networking interconnect is required. The N*24r instances are equipped with a secondary high perormance [RDMA capable networking](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances). A high performance network is required because a key limitation to scale in DNN training is the transfer of weight updates between the computation nodes (and indeed between each GPU).

### Architecutres Considered
We considered a number of architechtures for the pipeline build. This included;

- Hadoop + Spark + BigDL
- Hadoop + Spark + TensorFlowOnSpark
- Azure Batch AI + TensorFlow + Keras + Horovod

For a detailed discussion on these options and the evaluated pros and cons please see: [Deep Learning Multi Host & Multi GPU Architecture # 1 - Review and configuration with tensorflow, cntk, keras, horovod, Azure Batch AI](http://hoondongkim.blogspot.jp/2018/01/deep-learning-multi-host-multi-gpu.html). The post is in Korean language so please use *translate* via the Chrome browser.

The Shinsegae team have also evaluated the rleative performance of a multi-GPU scale-up approach via keras vs the Horovod scale out approach. [To summarize](http://hoondongkim.blogspot.jp/2018/01/deep-learning-multi-host-multi-gpu_11.html), the Keras shared weights approach is more efficient for single node multi-GPU training but will be limited to 8 GPUs. The delta between the two approaches narrows significantly at 4 GPUs which is the maximum available in an Azure N series machine.  

### Multi-GPU and Multi-node training using Horovod
While TensorFlow does provide support for [distributed training](https://www.tensorflow.org/deploy/distributed), this often involves significant modifications to the model training program. Early on in the hack we concluded that we'd like to experiment with the [Horovod](https://github.com/uber/horovod) framework from Uber. 

Horovod applies an MPI based [work distribution pattern](https://github.com/uber/horovod/blob/master/docs/concepts.md) to TenorFlow training. From our initial investigations it appeared to do so in a fairly transparent way. It provides [support for Keras](https://github.com/uber/horovod/blob/master/examples/keras_mnist.py) which was critical to success for us.

### Simple GPU cluster management with Azure Batch AI
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

# TBU