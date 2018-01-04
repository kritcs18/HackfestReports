# SSG AI Chatbot with Deep Learning Hack

## Key technologies

The technologies outlined and included in this solution are:

- [Azure Batch and Batch AI](https://azure.microsoft.com/en-us/services/batch-ai/) to train deep learning training
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

## Core Team (내용 수정해야 함)

The team was comprised of members from SSG AI team and Microsoft CSE:

- SSG.COM
  - Hoondong Kim : Chief lead, Deep Learnig
  - Sung Ryu : Deep Learnig, Serverless
  - Donghoon Seo : Container
  - Younggyu Jeon : Python, Container
  - Insuk Seo : PHP, Web App
- Microsoft 
  - Taeyoung Kim : Design, Serverless, WebApp, Container, Code Migration etc
  - Hun Choi : PM, Project Management
  - Chris Auld : Batch, Deep Learning, CNTK etc
  - Krit kamuto : Batch, Deep Learning, CNTK etc

## Partner profile (내용 수정해야 함)

Shinsegae is The largest retailer in South Korea with E-Mart(160 stores across the country), Shinsegae department stores, TRADERS as their subsidiary. Also, SSG.COM, integrated e-commerce service for their subsidiaries’ e-commerce site, growing fast ($800M revenue, 37.6% YoY growth in CY16)  …

## Problem statement (내용 수정해야 함)

고객은 현재 Deep Learning 기반의 AI Chatbot 서비스를 개발하고 있다. 현재는 시범적으로 모든 서비스를 Azure의 VM 상에서 테스트를 하고 있다. 테스트되고 있는 서비스 영역은 총 4개의 영역으로서, 그들은 각각 AI Model Traing Layer와 AI Serving(Inference) Layer, API Gateway(Chatbot App) Layer, Admin Management Web Layer로 나누어져 있다. 모든 서비스들은 현재 Azure VM에서 테스트되고 있다.

IaaS 기반의 아키텍처는 추후 운영 규모가 커질 경우, 자연스럽게 관리해야 할 VM의 수가 늘어나게 되면서 Infra 관리 및 유지보수에 추가적인 리소스를 투입해야 하기에 효율적이지 않다고 판단했다. 그렇기에 가급적이면 IaaS를 사용하지 않는 아키텍처를 원했다. 또한, AI Model Training Layer의 경우 상시 운영해야 할 필요도 없는데 GPU가 지원되는 VM을 계속해서 사용해야만 하기에 비용적인 측면에서도 부담스러워했다. 해서, 고객은 가급적 모든 서비스를 Managed Service 환경, 즉 PaaS 환경을 활용하는 아키텍처로 변경하고 싶어했다.

또한, AI Inference Layer의 경우는 고객사 자체의 사전 테스트를 통해서 굳이 GPU가 필요하지 않다는 결론을 얻었으며, 이번 핵페스트에서는 가급적 Azure App Service를 적용하면서 Web App의 수많은 훌륭한 기능들(자동 배포, 배포 슬롯 등)을 활용하고 싶어했다. 

더불어, Chatbot App에 해당하는 API Gateway 영역 역시 마이크로서비스나 서버리스 아키텍처를 도입해서 개발자들이 개별 로직에만 집중할 수 있도록 개선하고 싶어했다.

그리고, 모든 서비스에 대해서 전체적으로 배포 자동화를 도입하길 원했다. 기존에는 모든 서비스에 애플리케이션을 배포하는 부분이 수동으로 이루어지고 있었으며 이러한 반복 배포 작업이 비효율적이라고 판단했기에 이번 핵페스트를 통해서 가능한 한 많은 부분에 Continuous Deployment를 적용하기 원했다. 기존의 고객사 아키텍처는 대략 다음과 같다.

![images/as-is-arch.png](images/as-is-arch.png)

## Requirements and goals (내용 수정해야 함)

For this technical engagement, we defined requirements:

- 각각의 서비스들은 가능하다면 PaaS 환경에서 운영하도록 설계한다.
- 유연하고(flexible) 확장가능한(scalable) Architecture로 설계한다.
- 배포가 빈번하게 발생하는 서비스 Layer에 대해서는 자동 배포(Continuous Deployment) 전략을 적용한다.
- 서버리스 아키텍처가 적용 가능한 서비스 Layer에는 그를 적용한다.
- 모든 요청/응답 데이터는 로그로 남겨서 차트 등을 통해서 가시적으로 살펴볼 수 있어야 한다.

## Source Repos

고객사의 요청에 의해서, 이번 핵페스트에서는 GitHub Private Repo를 사용하도록 한다.
각각의 서비스 Layer별로 전용 Repo를 구성하였으며, 구성된 Repository의 모습은 다음과 같다.

- total 4 Private Repos, one for each service layer 

    ![Github repos](images/Repos.png)

## Technical Delivery

이번 핵페스트에서는 전체 Production 서비스를 이루고 있는 총 4개의 서비스 영역에 대해서 기존의 아키텍처를 분석하고 최적화된 to-be 아키텍처를 정의하였으며 또한, 각 영역의 담당 개발자들과 함께 각각의 서비스 영역에 대해 기술 검증 및 마이그레이션을 수행하였다. 이번 핵페스트에서 다루어진 서비스 Layer들은 다음과 같다.

1. AI Model Training Layer : Tenserflow와 keras를 사용하여 Deep Learning을 수행하는 레이어
2. AI Inference(& Serving) Layer : Python, Flask, keras를 사용하여 모델 데이터를 Web API로 서빙하는 레이어(이는 LUIS 즉, [Language Understanding Intelligent Service](https://www.luis.ai/)처럼 Intent 분석을 수행하는 서비스이다).
3. API Gateway(Chatbot App) Layer: Python, Flask 기반으로 ChatBot Application 역할을 수행하는 Web API 서비스. 채널 및 방 관리는 SendBird를 활용한다.
4. Admin Management WebSite Layer : PHP로 개발된 관리자용 웹 사이트. Intent나 Entity 관리 및 Realtime Training 호출 등의 작업을 수행할 수 있다.

각각의 서비스에 대해서 기술적인 부분과 지역적인 고려, 고객의 요구사항들을 반영하여 정의된 최종 아키텍처는 다음과 같다.

![Main Architecture](images/main_arch1.png)

다만, 이러한 아키텍처를 위해서 사용해야 할 Azure Resource 중에는 아직 Korea Region에서 가용하지 않은 것들도 있기에, 고객 입장에서 성능적으로 가장 효과적이도록 지역적으로 배분하여 설계한 모습은 다음과 같다.

![Main Architecture by region](images/main_arch_region.png)



## Architectural Decisions & Solutions

각각의 서비스 Layer에 대해서 고객과 MS 엔지니어들은 다음과 같이 설계하고 구현하였다.

----------------------

### **Layer 1 : AI Model Traininig Layer**

----------------------
need to fill

#### will be filled out by Chris and Krit

----------------------

#### Customer Feedback

고객사의 기술 리더인 김훈동 Chief 리드는 전공이 Deep Learning이기에 이번 핵페스트에서 다루어본 CNTK와 Batch AI 기술에 대해 만족도가 대단히 높았다. Facebook과 Slack을 통해서 이번 핵페스트가 큰 도움이 되었음을 여러 차례 피력하였으며 또한, 자신이 경험한 기술적인 가치를 다른 데이터 개발자들에게 알리고 싶어했다. 해서, 그는 수 차례 블로그에 이번 핵페스트에서 다루었던 실험 내용들을 정리해서 다른 개발자들에게 공유하고 있다. 

Title : Deep Learning Multi Host & Multi GPU Architecture with tensorflow, cntk, keras, horovod, **Azure Batch AI**    
Link : http://hoondongkim.blogspot.kr/2018/01/deep-learning-multi-host-multi-gpu.html

심지어는, 이러한 내용을 단계별로 정리하여 다음과 같은 제목으로 개발자 커뮤니티와 함께 Hands-On Lab을 진행하기도 하였다.

Title : Deep Learning with distributed GPU based on Azure Batch AI  
Link : https://onoffmix.com/event/123844

----------------------

### **Layer 2 : AI Inference(& Serving) Layer**

----------------------

#### As-Is

AI Serving Layer는 AI Inference 역할을 수행하는 웹 서비스로, 기존에는 Python 언어로 개발되어 있었으며, [Data Science Virtual Machine](https://azure.microsoft.com/en-us/services/virtual-machines/data-science-virtual-machines/)(OS는 Ubuntu) 머신 내 Flask Web server 상에서 운영 중에 있다(개발환경은 jupyter 노트북을 사용). 사용하는 라이브러리는 tensorflow(1.3.0), keras(2.0.8), python-twitter 등이다.

이 서비스는 역할적으로는 LUIS 즉, [Language Understanding Intelligent Service](https://www.luis.ai/)와 유사한 서비스이다. SSG.COM 데이터 플랫폼 팀에서 자체 개발한 AI Model Training Layer에서 trained된 Model(.H5) 바이너리 파일을 h5py 라이브러리를 사용하여 Flask 웹서버의 메모리에 통채로 올려서 LUIS 관련 서비스를 제공하다. 현재 주된 역할은 챗봇 사용자의 메시지에 대한 Intent와 Entity 등을 파악하는 것이지만, 이후로는 더 많은 역할을 수행하도록 계속해서 발전될 예정이다.

#### What customer want

기존 서비스는 기본적으로 구동시키는 데에 메모리를 최소 2G(trained model의 size가 약 1.7G) 이상 소비하는 구조로 되어 있다. 또한, GPU를 활용하는 것이 성능적으로 나을 것이라는 기대때문에 Data Science VM에서 운영하도록 설계가 되어 있다(일종의 검증 상황이었기에 Availability Set 없이 Single VM으로 운영). 하지만, 이는 최적의 운영 환경은 아닐 것이라고 생각하고 있으며, 더 나은 환경이 무엇인지 검증하고 싶어했다.

사전 기술 미팅 중에 우리는 층이 그리 깊지 않은 Deep Learning 모델은 CPU만으로도 만족할만한 성능을 얻을 수 있을 것 같다는 의견을 제시하였고, 고객은 이러한 의견에 동의하여 핵페스트 이전에 사전 성능 테스트를 통해서 그에 대한 결과를 확인하기로 하였다. 

테스트 결과, 그들이 개발한 AI Serving Layer는 GPU보다는 CPU 환경에서 더욱 빠르게 동작한다는 테스트 결과를 얻었다(하단의 Refer 참고). 그에 따라 고객은 GPU 기반의 VM을 사용하는 IaaS 시나리오에서 벗어나, PaaS(Azure App Service) 환경을 도입하길 원했으며, 이를 통해 안정성, 관리성 뿐만 아니라 유연한 개발 및 배포 환경을 확보하고자 하였다.

> Refer to [Experiment and Performance Test for Deep Learning Inference & Serving : > GPU vs CPU (Korean)](http://hoondongkim.blogspot.jp/2017/12/deep-learning-inference-serving.html) 
>   - blog post by Hoondong Kim(SSG Chief Dev, AI Data platform team)  
>   - Topic : Is GPU really better than CPU in AI Inference? 
>   - Conclusion : CPU is better than GPU in their cases. 

#### Analysis and Design 

Envisioning Plan 미팅 후에 검토되었던 방안은 Web App(Windows)의 KUDU(SCM) 환경에서 필요한 python 라이브러리들을 미리 Web App에 설치하고 Web App 환경이 주는 수많은 혜택들(Continuous Deployment, Auto Scale, Deployment Slot, App Service Environment etc)을 활용하고자 하였다. 하지만, 개발에 사용된 라이브러리들이 모두 Linux 기반의 라이브러리들이기에 가급적이면 OS로 Linux를 사용하는 것이 바람직하다는 의견이 제시되었고, TWG - Compute 그룹과의 기술 미팅에서도 이 부분의 아키텍처는 Container 기반으로 가져가는 것이 나아보인다는 의견이 제시되어 최종 아키텍처는 Web App for Container로 진행하기로 하였다(Web App for Container는 아직 Web App의 모든 features들을 제공하지는 않지만 고객사에서 원하는 배포 슬롯과 Continuous Deployment는 사용 가능하기에 적합한 시나리오로 채택되었음). 이를 통해 설계된 아키텍처는 다음과 같다.

![images/AIServ-arch01.png](images/AIServ-arch01.png)

다만, 이번 핵페스트에서는 VSTS(Visual Studio Team Service)를 활용한 Docker build, Push의 자동화는 배제하기로 하였다. 이는 고객사의 QA 정책과 관련된 이슈들(Test 및 QA 관련 이슈)이 발생할 가능성이 있어서이다. 해서, 별도의 전용 Docker Build 머신(VM)을 활용하여 로컬 테스트를 수행한 다음에 수동으로 Azure Container Registry에 Push 하는 것으로 협의하였다.

#### Continuous Deployment Scenario

핵페스트에 앞서, TE는 약 2-3일에 걸쳐 배포 시나리오를 여러 방향으로 사전 검토하였다. 실제 핵페스트 기간 중에는 2일간 고객과 함께 기존 시스템을 Container 기반으로 재구축하여 Web App for Container에서 구동시키는 데 성공했다. Docker Images를 위한 Private Registry로는 Web App for Container와 같은 Region에 있는 Azure Container Registry를 사용했으며, Registry Webhook을 설정하여 새로운 Docker Image가 Push되면, 자동으로 Web App의 Stage Slot에 배포되도록 하였다. 배포가 완료되면 고객의 담당자가 각 Slot의 상태를 체크한 뒤에 Swap을 수동으로 진행한다. 이렇게 정의된 Continuous Deployment 시나리오는 다음과 같다.

![images/AIServ-arch02.png](images/AIServ-arch02.png)

다음의 스냅샷은 Dockerfile의 코드와 Web App의 Azure Container Registry 설정 및 Webhook을 통해 자동 배포가 수행된 이력을 보여준다.

Azure Container Registry Settings

![Azure Container Registry Settings](images/AiServ_Docker_Container_s.png)

Dockerfile Example

![Dockerfile](images/dockerfile.png)

Push History via Container WebHook

![Push history via WebHook](images/AiServ_ACR_s.png)

#### Considering the Region

게이트웨이의 역할이자, Chatbot App에 해당하는 Web API Gateway(chatbot) Layer는 한국에서 서비스되고 있는 반면, AI Serving/Inference Layer(AI service like LUIS)는 Japan region에서 Web app for Container로 서비스하고 있다. 그 이유는 한국에 아직 Azure Container Registry이 지원되지 않기 때문이다(Azure Container Registry과 Web App for Container는 동일 Region에 있어야 비용 및 배포 비용이 절약된다). 만일, ACR(Azure Container Registry)을 한국에 생성할 수 있다면 더욱 빠른 Latency를 얻을 수 있을 것이다. 고객은 추후 한국에서 ACR이 서비스 된다면, 현재 AI Serving Layer와 관계된 모든 서비스(ACR, Web App for Container, Blob Storage for H5 model files etc)를 한국으로 옮길 계획이다.

#### Issues and Workaround

기존 모듈을 Managed Service 환경으로 옮기면서 겪었던 몇 가지 문제 중 하나는 로딩 타임 제약과 관계된 것이다. 현재 고객이 Python으로 개발한 AI Inference 서비스의 동작 방식은 약 1.7G의 모델 파일을 Flask 웹 서버가 시작하는 시점에 모두 메모리로 로드하여 서비스하는 구조이다. 물론, 이러한 구조는 사실 개선 혹은 재설계가 필요한 부분이지만 그는 고객사에서 자체 개발 일정에 따라 추후 개선해야 하는 부분이며, 지금 변경할 수는 있는 부분이 아니기에 현재로서는 기존 개발된 상태의 모듈 그대로 서비스가 동작하게 해야 한다. 문제가 되는 부분은 기존에 개발된 Web App이 처음 시작하는데 약 4분~5분 정도의 초기 로딩타임이 걸린다는 것이다. 그런데, Web App for Container는 기본적으로 초기 시작 시간의 Limit이 230 seconds이기에 그 이상의 로딩 time이 걸리는 Web App은 로딩이 불가능하다. 그렇기에 고객사의 기존 서비스가 제대로 시작할 수 없는 문제가 발생했으며, 다음과 같은 Container Log가 계속해서 남는 것을 확인할 수 있었다. 

```python
ERROR - Container site *****  did not start within expected time limit
```

이는 고객사의 모듈이 최소 240~300 sec 이상의 로딩 시간을 요구하기에 발생하는 문제이다. 이를 해결하기 위해서는 Web App의 App Settings에 **WEBSITES_CONTAINER_START_TIME_LIMIT**을 최대치인 **600** 초로 설정해야 하며, 그 결과 올바로 로딩을 완료할 수 있었다.

Application Setting for Stage

![Application Setting for Stage](images/AiServ_Stage_App_Settings_s.png)

#### What customer consider for better performance

그러나, 이러한 모델 데이터는 앞으로도 계속해서 커질 가능성이 있으며, 이는 결국 언젠가는 올바로 로딩할 수 없는 상황을 초래하게 될 가능성이 높다. 더불어, 현재에도 배포와 SWAP 시 초기 로딩에 너무 많은 시간을 요구하는 것은 큰 문제이다. 해서, 이 이슈는 애플리케이션을 튜닝(혹은 개발 변경)하여 기술적으로 개선할 필요가 있다. 고객도 이러한 문제에 대해서 동의하였으며, 해결을 위한 기술적 고민을 시작할 것임을 시사했다. 

덧붙여, 현재 작성된 Dockerfile은 이미지 빌드 시에 필요한 Model 파일들을 모두 wget을 사용하여 로컬 경로로 복사한 뒤에, Model 파일을 포함한 채 이미지로 빌드하고 있다. 이는 고객이 사용하는 h5py 라이브러리가 로컬 경로의 파일만 로드할 수 있다는 제약이 있기 때문이다. 이 부분을 개선하여 Python에서 직접 Remote Location(Blob Storage)에 있는 h5 파일을 로드할 수 있도록 변경한다면, 혹은 다른 라이브러리를 사용해서 유사한 기능을 제공하도록 개선할 수 있다면 Docker 이미지 안에 모델 바이너리 파일을 포함시키지 않아도 되기에 빌드 이미지 크기와 Push에 소모되는 데이터 트래픽도 현져하게 줄일 수 있을 것이다. 

#### Customer Feedback

현재의 Web App for Container 플랫폼은 Deployment Slot 간에 Swap을 수행하는 데 너무 많은 시간이 걸린다. Production과 Stage 모두가 올바로 동작하는 것을 확인하고 Swap을 수행하는 경우에도(이미 두 Web App이 모두 운영 상황에 있음에도 불구하고) 새로운 이미지를 다시 로드하는 만큼의 시간이 걸린 뒤에 Swap이 수행된다. Swap 시간과 관련된 부분은 Microsoft의 담당 개발팀에서 개선해 줄 필요가 있다.

----------------------

### **Layer 3 : API Gateway(Chatbot App) Layer**

----------------------

#### As-Is

API Gateway는 사실상 Chatbot App의 역할을 수행하며, SenBird의 WebHook을 통해서 봇 사용자가 물어오는 모든 채팅 내용을 수신하는 API Gateway의 역할을 수행한다. 기존에는 Python/Flask 기반으로 개발되었으며, 별도의 VM 상에서 운영 중에 있다(개발환경은 jupyter 노트북을 사용). 

#### What they want

고객은 이번 핵페스트를 통해서 챗봇의 서비스 환경을 IaaS가 아닌 Managed Service 환경으로 바꾸기를 희망했고, 가능하다면 기존의 서비스 로직을 분석해서 깔끔하게 로직을 재 정리하면서 동시에 서버리스 아키텍처가 적용되기를 희망했습니다. 그를 통해, 좀 더 효율적이고 효과적으로 챗봇의 개별 로직에만 집중하면서 서비스를 지속적으로 개선해 나가고 싶어했습니다.

#### Analysis and Design

기존의 서비스 로직을 분석해 본 결과 다음과 같은 문제점들이 도출되었습니다.

문제점

- API 호출이 하나의 큰 동기적인 호출로 구성되어 있다.
- 모든 요청은 전체 메서드를 모두 수행한다.
- 내부적으로 외부 API에 대한 호출이 수 차례 발생하고 그에 따라 분기되는 구조를 갖고 있다
- 세션 정보를 보관하기 위해서 Redis를 사용하고 있으며, 그를 지원하기 위한 코드 또한 직접 개발 및 관리하고 있다

기존의 로직을 개별적으로 모두 분석해 본 결과, 기존 Python 코드는 요청과 응답이 하나의 단일 Sync 호출로 이루어지는 구조였으며, 내부에서 다양한 condition에 따라 분기하는 로직을 가지고 있었습니다. 모든 사용자의 입력은 intent 분석을 위해서 AI Serving Layer을 호출하며, 호출 후에 얻어진 Intent에 따라 다양한 조건에 따라 로직이 분기되는 구조입니다. 그렇기에, 상황에 따라 로직이 Skip되는 경우도 있고, 상황에 따라서는 추가적으로 다른 API로의 외부 호출이 다시 발생하는 경우도 있었습니다. 

더욱 안 좋은 점은 그러한 모든 처리가 논리적으로는 하나의 큰 단일 메서드 안에서 단일 쓰레드로 수행되는 비효율적인 구조였다는 것입니다. 해서, 이러한 아키텍처 구조를 Async하게 변경하였으며, 그와 동시에 각각의 로직을 분리하여 서버리스 아키텍처로 혁신하였습니다. 그 결과, 각 로직 간의 통신은 큐를 통해서 진행하고 최종 응답 메세지는 비동기적으로 SendBird에 Proactive 방식의 메세지로 전송하도록 아키텍처를 변경하였습니다.

그렇게 변경된 솔루션은 다음과 같습니다.

솔루션
- 가능한 한 모든 분기 로직은 개별 Function으로 분리한다. 
- 모든 호출은 비동기로 처리한다. 
- 사용자에 대한 응답도 비동기적으로 Proactive Massage를 통해서 응답한다. 
- 목적에 따라 다양한 큐를 활용하고 큐를 통해서 로직이 분기하도록 한다. 
- 사용자의 상태는 Finction의 Input/output Binding을 사용하여 Cosmos DB에 저장하도록 한다.	

그 결과 구성된 아키텍처는 다음과 같습니다.

![APIgw-arch01.png](images/APIgw-arch01.png)

#### Re-Design Control Flow and Code Blocks

정의된 아키텍처에 따라 기존의 코드들을 분석하고, 재작성 가능 여부를 점검하는 과정을 수행합니다. 고객은 가능하다면 Python을 사용하고 싶어했지만, 아직 Azure Function에서는 Python이 Experimental로만 지원되는 상황이기에 좀 더 안정적인 언어를 선택해야 했습니다. 결과적으로 고객은 모든 기존 코드를 C#으로 마이그레이션 하는 데에 동의하였고, 개발 언어와 플랫폼이 변경됨에 따라 기존 Python 코드와 제어 흐름을 모두 분석할 필요가 있었습니다. 

고객사의 API Gateway 개발 담당자와 1일 동안 모든 로직을 분석하면서 Function 수준의 분리 작업을 수행하였고, 기존에 Redis로 보관하던 사용자의 상태 정보는 Cosmos DB를 사용하여 안정성과 가용성을 높이도록 하였습니다. 로직을 분리하고 정리하면서 아키텍처는 더욱 간소화되었고 불필요한 기존 코드도 깔끔하게 정리되었습니다.

![APIgw-arch02.png](images/APIgw-arch02.png)

이 과정으로 통해서 기존에 개발된 API Gateway 로직들을 Function으로 분리한 아키텍처는 다음과 같습니다.

![APIgw-arch03.png](images/APIgw-arch03.png)

case에 따라 서로 다른 큐로 메세지를 전달하고, 각각의 분기되는 로직은 해당 큐에 메세지가 오는 경우에만 수행하도록 변경하였습니다. 이는 기존의 로직에서 수 많은 if, else if를 남발했던 경우보다 직관적이며, 각각의 로직에만 집중해서 코드를 개선할 수 있다는 이점이 있습니다. 이러한 작업을 통해서 기존의 Flask, Python으로 작성되어 있던 로직은 C#, Function app으로 100% migration 되었습니다. 기존의 코드 예는 다음과 같습니다.

![APIgw-code1.png](images/APIgw-code1.png)

고객이 C#에 익숙하기 않기에 초기에는 약간의 걱정이 있었으나, Function에서는 Trigger, Input, Output이 모두 Attribute로 간단하게 설정이 가능하기에 상당 부분의 어려운 코드 영역은 단순히 어트리뷰트 설정만으로 완료할 수 있었고, 실제 로직만을 C#으로 변경하면 되었기에 크게 어렵지 않게 이러한 작업을 수행할 수 있었습니다. 마이그레이션 후 고객은 오히려 C#의 언어적인 멋진 기능들과 Visual Studio의 편리함에 만족스러워하였습니다. 실제로 사용된 코드의 예는 다음과 같습니다.

![APIgw-code3.png](images/APIgw-code3.png)

#### Performance

모든 코드를 서버리스 아키텍처 기반으로 분리한 뒤, 모의 테스트를 수행했을 때 결과는 만족스럽게도 2초 이내에 응답을 얻을 수 있었습니다. 사실상 가장 많은 시간이 소요되는 API Serving Layer를 호출하는 시간이 포함되었음에도 총 소요시간이 2초 이내로 나온 것은 만족스러운 결과였습니다. 

또한, 각 Function 들의 수행시간을 살펴보기 위해서 Function App에는  Application Insight도 적용하였습니다. 그렇기에, Function App은 Application Insight를 통해서 모든 호출과 호출 빈도, 각 API의 실행 시간들을 실시간 모니터링 할 수 있습니다. 어떤 Function이 가장 많이 호출되고, 수행시간은 얼마나 걸리는지, 예외가 발생하는 경우 그 원인은 무엇인지 등을 각각의 Function 별로, 또한 실시간으로 살펴볼 수 있기에 디버깅 시에도 도움이 되며, 추후 개선해야 할 부분을 파악하는 데에도 도움이 됩니다.

<모니터링 화면 삽입>

현재 한국에서는 Consumption Plan이 지원되지 않고, 또한 고객도 초기에는 App Service Plan을 직접 조정하여 적절한 비용 플랜을 검증하고자 하기에 현재는 S1 크기의 플랜 하에서 Function App이 동작하고 있습니다. 하지만, 나중에 실제로 운영을 시작할 경우 및 사용자의 요청이 몰릴 경우에는 좀 더 높은 Plan을 사용할 것입니다.

#### Issues and Workaround

모든 개발 소스는 GitHub의 Private Repository와 통합되어야 하고 Web App의 Deployment Option 기능을 사용해서 자동 배포가 되어야 하는데, 현재 Web App의 Deployment Option에서 제공하는 GitHub 통합 기능은 Public Repository만 지원하고 Private Repository는 지원하지 않습니다. 그렇기에 이를 해결하기 위한 별도의 workaround가 필요했습니다. 이를 해결하려면 다음의 링크에서 제안하는 방법대로 약간의 메뉴얼적인 작업을 수행해 줄 필요가 있습니다. 

Refer to https://nsamteladze.wordpress.com/2015/07/19/continuous-deployment-from-github-enterprise-repository-to-azure-web-app/. 

#### Customer Feedback

핵페스트 기간 내에 과연 Migration이 가능할 지에 대해 우려했으나, 기간 내에 모든 로직을 정리하고 완료한 부분에 매우 깊은 감명을 받았고 Azure에서 제공하는 서버리스 기능들에 매우 흡족해 하였습니다. 서버리스를 활용하니, 인프라적인 코드나 플랫폼 기반 코드를 더이상 신경쓰지 않고 오로지 Business Logic에만 집중할 수 있어서 만족스럽다는 의견이었습니다. 

>> 류성의 피드백이 있으면 좋을 듯. 바로 윗 ^ 구문 참고해서 작성하면 좋을 듯.

### Admin Management WebSite Layer

#### As-is 
Admin Web Site는 관리자 전용 웹 사이트입니다. 관리자는 Admin 웹사이트에서 다양한 Utterance를 입력하고, 각종 질의 관련 Rule을 등록하는 등 Intent, Entity와 연계된 다양한 관리 기능을 사용할 수 있습니다. 또한, 관리자는 On-Demand 방식으로 Model Training을 즉각 수행하도록 명령을 실행할 수도 있습니다. 관리자 웹 사이트는 PHP로 개발되었으며, 다양한 CSS 및 Bootstrap UI들을 사용하여 화려한 UI를 제공하고 있습니다. 현재는 Azure VM에서 호스팅되고 있으며, 수동으로 소스 배포 및 관리되고 있습니다.

#### What customer want

고객은 다른 Layer들과 마찬가지로 Admin WebSite Layer도 별도의 VM에서 이를 관리하기 보다는 언제든지 Scale in/out이 가능한 PaaS 플랫폼을 적용하고자 했습니다.  다만, 현재 일부 라이브러리들이 다소 복잡하게 흩어져 있어서 Azure App Service로 이전하려면 기존의 구조를 단일 폴더의 하위로 통합하는 작업이 필요합니다. 또한, 기존의 파일 업로드 기능도 기능 개선이 필요합니다.  정리하자면, IaaS 환경에 맞춰 개발된 기존 소스 구조를 PaaS에서 매끄럽게 운영이 가능하도록 마이그레이션을 하길 바랬습니다. 또한, Github private에 커밋한 소스들이 즉각 Web App에 반영되기를 원했습니다. 

#### Analysis and Design 

현재는 일부 클라이언트 라이브러리들이 VM 로컬에 다소 복잡하게 흩어져 있고, 상대적인 경로로 매핑이 되어져 있기 때문에 Azure App Service로 이전하려면 기존의 흩어져있는 폴더 구조를 단일 폴더의 하위로 통합하는 작업이 필요합니다. 그렇기에, 다운로드 되어 있는 수십 개의 클라언트 라이브러리 중에서 사용되는 것과 사용되지 않는 것들을 파악해서 사용하는 것들만 따로 모아서 정리하는 작업을 수행했습니다.

또한, 기존의 파일 업로드 기능도 App Service에서 올바로 동작하도록 코드 레벨에서의 수정이 필요했습니다.  기존에는 사용자들이 파일을 업로드하면 로컬 서버의 Temp 드라이브에 임시적으로 저장한 뒤, 서버 내 특정 위치로 이동시키는 방식으로 코드가 작성되어 있는데, 이를 Azure Storage에 통합되어 저장되도록 변경해야 합니다. 



그리고, 그 밖에 발생가능한 이슈들에 대해서는 Migration 후에 메뉴얼 테스트를 통해서 문제점을 확인하고 점검해야 합니다.

## Business impact
이하 내용 추가

## Partner technical engagement feedback
이하 내용 추가

## Conclusion
이하 내용 추가

## Future possibilities
이하 내용 추가
