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
  - Chris Auld : TE Manager / Batch AI, Deep Learning, CNTK etc
  - Krit kamuto : TE / Batch AI, Deep Learning, CNTK etc

## Hackfest period

Envisioning Workshop : Dec 1 2017  
Hackfest : Dec 18 ~ 22 2017 / 5 days  

# TBU