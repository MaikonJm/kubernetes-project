# Orquestração com Kubernetes

Este projeto demonstra a implementação de uma aplicação em Kubernetes com foco em orquestração. Foi configurado um cluster local utilizando Minikube/Kind e realizado o deploy de uma aplicação, expondo-a através de serviços e controladores no Kubernetes. A configuração inclui:

* Arquivos Deployment, Service e Ingress para expor a aplicação.
* Configuração de ConfigMaps, Secrets e Autoescala (Horizontal Pod Autoscaler) para gerenciamento de configurações e escalabilidade automática.
* Cluster local configurado com Minikube/Kind.


## Funcionalidades

* Deploy de Aplicações: Arquivos YAML para o deploy de uma aplicação no Kubernetes.
* Exposição de Aplicações: Utilização de Ingress e Service para expor a aplicação para acessos externos.
* Configuração de Autoscalabilidade: Utilização de Horizontal Pod Autoscaler para ajustar automaticamente o número de réplicas dos pods.
* Segurança e Configurações: Uso de Secrets e ConfigMaps para gerenciar informações sensíveis e configurações externas.
  
##Estrutura do Projeto

```bash
kubernetes-project/
│
├── k8s/
│   ├── deployment.yaml           # Arquivo de Deployment
│   ├── service.yaml              # Arquivo de Service
│   ├── ingress.yaml              # Arquivo de Ingress
│   ├── configmap.yaml            # Arquivo de ConfigMap
│   ├── secret.yaml               # Arquivo de Secret
│   ├── autoscale.yaml            # Arquivo de Horizontal Pod Autoscaler
│   ├── namespace.yaml            # Arquivo para criar namespace
│
└── README.md                     # Documentação do projeto

```
## Pré-requisitos

* Minikube ou Kind para cluster local Kubernetes.
* kubectl para interagir com o cluster.

## Instalação

1. Instalar Minikube ou Kind

2. Iniciar o Cluster:

  * Se estiver usando Minikube:
  ```bash
    minikube start
  ```
  * Se estiver usando Kind:
  ```bash
    kind create cluster
  ```

3. Aplicar os Arquivos de Configuração:

* Execute os seguintes comandos para aplicar os arquivos de configuração no seu cluster Kubernetes:
  ```bash
    kubectl apply -f k8s/namespace.yaml
    kubectl apply -f k8s/deployment.yaml
    kubectl apply -f k8s/service.yaml
    kubectl apply -f k8s/ingress.yaml
    kubectl apply -f k8s/configmap.yaml
    kubectl apply -f k8s/secret.yaml
    kubectl apply -f k8s/autoscale.yaml
  ```

4. Acessar a Aplicação:

* Após a aplicação ser implantada, você pode acessar a aplicação usando o Ingress configurado ou através do port-forwarding:
 ```bash
    kubectl port-forward svc/kubernetes-dashboard 8001:443 -n kubernetes-dashboard
 ```
5. Verificar a Escalabilidade:

* O Horizontal Pod Autoscaler foi configurado para ajustar automaticamente o número de pods com base na carga. Você pode verificar o status com:
```bash
kubectl get hpa
