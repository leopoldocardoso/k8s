# Estudos k8s <p align="center"><img src="http://img.shields.io/static/v1?label=STATUS&message=EM%20DESENVOLVIMENTO&color=GREEN&style=for-the-badge"/></p>

## Este repositório tem como objetivo compartilhar o conhecimento que venho adquirindo nos estudos de Kubernetes

### Nota: os arquivos citados nos comandos estão dentro de pastas dentro deste repositório

# Conceito de Kubernetes

## Camadas de Gerenciamento

- Tomam decisões globais sobre o cluster
- Detectam e respondem aos eventos do cluster
- Podem ser executadas em qualquer máquina, porém para simplificar são iniciados em uma única máquina.
- Contêineres com cargas de trabalho não rodam nesta máquina

### Arquitetura

- Adicionar imagem

### etcd

- Armazenamento do tipo chave valor
- Consistente e de alta disponibilidade
- Usado como armazenamento de apoio do kubernetes para todos os dados do cluster
- Confiável, rápido e seguro
- Por padrão escuta na porta 2379
- etcdtl ferramenta de linha de comando do etcd que pode ser usada para armazenar e recuperar pares de chave e valor
- No etcd são armazenados:
  - nodes
  - pods
  - configs
  - secrets
  - accounts
  - roles
  - bindings
  - others
- Cada mudança feita no cluster, como adição de nodes, implantação de pods ou réplicas sets é atualizado no servidor etcd

#### Comandos etcd

- ETCDCTL versão 2 suporta os seguintes comandos:

```sh
  etcdctl backup
  etcdctl cluster-health
  etcdctl mk
  etcdctl mkdir
  etcdctl set
```

- Os comandos são diferentes na versão 3:

```sh
  etcdctl snapshot save
  etcdctl endpoint health
  etcdctl get
  etcdctl put
```

- Para definir a versão correta do api set a variável de ambiente com comando  ETCDCTL_API

```sh
  export ETCDCTL_API=3
```

- Além dos comandos do etcd precisamos configurar o caminho do certificado para podermos nos autenticar no ETCD API Server

```sh
  --cacert /etc/kubernetes/pki/etcd/ca.crt
  --cert /etc/kubernetes/pki/etcd/server.crt
  --key /etc/kubernetes/pki/etcd/server.key
```

- Formato final do comando para

```sh
  kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"
```

### kube apiserver

- Principal componente do Kubernetes
- Componente da camada de gerenciamento
- Expõe a API do kubernetes
- O servidor do API é o front-end da camada de gerenciamento do kubernetes
- Projetado para ser escalado horizontalmente
- É possível executar várias instâncias do kube apiserver  e distribui o tráfego entre as instâncias
- Responsável por autenticação e validação dos pedidos dos dados armazenados o etcd
- Único componente que interage diretamente com o armazenamento de dados do etcd
- Os demais componentes como scheduler, kube-controller-manager e kubelet interagem com o apiserver para realizar atualizações em suas respectivas áreas.

### kube controller-manager

- Gerencia vários controllers do kubernetes
- Processo que monitora continuamente o estado de vários componentes dentro do sistema e trabalha para levar o sistema inteiro ao estado de funcionamento desejado

### kube-scheduler

- Responsável pelo agendamento dos PODs nos nodes
- Define qual POD vai em qual node
- Não coloca o POD no node, isso é um papel do kubelet

### Kubelet

- Registra o node
- Cria os PODs
- Monitora nodes e PODs

### kube-proxy

- Proporciona comunicação de rede entre os PODs

## Arquivo Yaml no Kubernetes

- O arquivo yaml ou yml segue uma estrutura básica com quatro campos de nível superior:
  - apiVersion: versão do objeto que estamos utilizando
  - kind: tipo do objeto que estamos utilizando
  - metadata: dados do objeto, como labels, nome, etc. sempre chave:valor
  - spec: campo com as especificações do container


## Manifesto YAML de um Pod no Kubernetes

Este é um exemplo comentado de um arquivo YAML que define um **Pod** no Kubernetes. Os comentários estão incluídos diretamente no YAML para explicar cada campo, mantendo tudo claro e organizado.


## Estrutura do YAML com Comentários

```yaml
apiVersion: v1              # Versão utilizada para Pod
kind: Pod                   # Tipo do objeto
metadata:                   # Dados sobre o objeto
  name: myapp-pod           # Nome da aplicação
  labels:                   # Pode ter qualquer valor para identificar a aplicação. Podemos ter várias labels
    app: myapp
    type: front-end
spec:                       # Neste bloco será informado as especificações do pod/deployment como as imagens de containers
  containers:
    - name: nginx-container # Nome da imagem. O travessão (-) indica que é o primeiro item da lista
      image: nginx          # Imagem do nginx no Docker Hub
```

### Tipos de Objeto:

| Tipo de Objeto         | apiVersion                     | Descrição Breve                              |
|------------------------|--------------------------------|----------------------------------------------|
| Pod                    | v1                             | Unidade básica de execução de contêineres    |
| Deployment             | apps/v1                        | Gerencia réplicas de pods com atualizações   |
| ReplicaSet             | apps/v1                        | Garante número de réplicas de pods           |
| StatefulSet            | apps/v1                        | Gerencia pods com estado persistente         |
| DaemonSet              | apps/v1                        | Executa um pod por nó                        |
| Service                | v1                             | Expõe pods como um serviço de rede           |
| Ingress                | networking.k8s.io/v1           | Gerencia tráfego HTTP/HTTPS externo          |
| ConfigMap              | v1                             | Armazena dados de configuração               |
| Secret                 | v1                             | Armazena dados sensíveis                     |
| PersistentVolume       | v1                             | Recurso de armazenamento persistente         |
| PersistentVolumeClaim  | v1                             | Solicitação de armazenamento por pods        |
| Namespace              | v1                             | Separa recursos em ambientes lógicos         |
| Node                   | v1                             | Representa um nó no cluster                  |
| Job                    | batch/v1                       | Executa tarefa única com conclusão           |
| CronJob                | batch/v1                       | Agenda execução de Jobs                      |
| HorizontalPodAutoscaler| autoscaling/v2                 | Escala pods automaticamente                  |
| Role                   | rbac.authorization.k8s.io/v1   | Define permissões em um namespace            |
| ClusterRole            | rbac.authorization.k8s.io/v1   | Define permissões no cluster                 |
| ServiceAccount         | v1                             | Identidade para processos em pods            |

## kubernetes controller

- Controller são o cérebro por trás do kubernetes
- São processos que monitoram objetos nativos do k8s

### Conceito de ReplicaSet

- O ReplicaSet é um desses controllers. Garante alta disponibilidade dos PODs, garantindo que um número mínimo de PODs estará funcionando.
- O propósito de um ReplicaSet é gerenciar um conjunto de réplicas de Pods em execução a qualquer momento.
- A função do conjunto de Replicas é monitorar os PODs, e se algum deles falhar, implantar novos. É o processo que monitora os PODs.
- É geralmente utilizado para garantir a disponibilidade de um certo número de Pods idênticos.
 - Se tivermos vários PODs podemos compartilhar a carga entre eles, utilizando o Load Balancing.
- Para criar um replicaset a partir do arquivo yaml basta executar o comando: kubectl create -f < nome do arquivo.yaml >

``````sh
  kubectl create -f replicaset-definition.yaml
``````

#### Escalando ReplicaSet

É possível escalar o ReplicaSet de 3 maneiras:

1 - Editando o arquivo yaml, por exemplo, replicaset-definition.yaml, alterando o número de réplicas. Após salvar o arquivo, executar o comando:

   ``````sh
     kubectl replace -f repicaset-definition.yaml
   ``````

2 - Escalando através do comando kubectl scale --replicas=< nro de pods > -f < nome do arquivo.yaml >

   ``````sh
    kubectl scale --replicas=3 -f replicaset-definition.yaml
  ``````

3 - Escalando através do comando kubectl scale --replicas=< nro de pods > replicaset < nome do arquivo.yaml >

   ``````sh
    kubectl scale --replicas=8 replicaset replicaset-definition.yaml
  ``````

### Daemon Sets

- Garante que uma cópia do POD esteja sempre presente em todos os nodes do cluster
- A criação do daemon set é semelhante a criação do réplica set
- Veja o yaml de criação  [aqui](./daemonset-definition\daemonset-definition.yaml/)

  #### Quando usar Daemon Sets

  - Solução de monitoramento do cluster
  - O kube-proxy pode ser implantado como daemon sets

## Conceito de POD

- O POD é o menor objeto que se pode criar no kubernetes
- Se precisarmos escalar a aplicação, adicionamos uma nova instância de um POD com a mesma aplicação
- Geralmente a relação é de um POD para um Container
- Com multicontainers, um mesmo POD pode ter multiplos containers que geralmente não são do mesmo tipo
- É possível criar um pod com o seguinte comando: kubectl run < nome do pod> < nome da image >

```sh
  kubectl run webserver nginx
```

## Conceito de Deployment

- O Deployment implementa PODs e ReplicaSets
- Fornece atualizações declarativas para PODs e ReplicaSets
- As atualizações podem ser contínuas
- O arquivo do Deployment é praticamente o mesmo arquivo do ReplicaSet, modificando apenas o kind

## Criando um Deployment

É possível criar um Deployment de 2 formas:

1 - Executando o comando kubectl create deployment --image=< nome da imagem> < nome do deployment >
    Obs: poder ser deployment ou deploy

  ``````sh
   kubectl create deploy --image=nginx nginx-deploy
  ``````

2 - Gerando um manifesto yaml executando a linha de comando kubectl create deploy --image=< nome da imagem > < nome do deploy > --dry-run=client -o yaml > deploy-definition.yaml

  ``````sh
    kubectl create deploy --image=redis redis-deployment --dry-run=client -o yaml > deployment-definition.yaml
  ``````

- Após gerar o arquivo yaml é necessário executar o comando kubectl create -f < nome do arquivo.yaml >

   ``````sh
    kubectl create -f deployment-definition.yaml
   ``````

- Algumas informações sobre o comando:

 - --dry-run=client -o yaml > deployment-definition.yaml: não executa a criação do deployment. Gera o manifesto yaml e salva este manifesto no arquivo deployment-definition.yaml.
 - Partes do comando:
   - --dry-run=client: não executa a criação do deploy
   - -o yaml > deployment-definition.yaml: gera o manifesto yaml e salva este manifesto no arquivo deployment-definition.yaml

## Conceito de Services

- Permite a comunicação entre vários componentes dentro e fora da aplicação
- O serviço kubernetes é um objeto, assim como um POD, Deployment ou ReplicaSet
- No kubernetes temos três tipos de Services:
  - NodePort
  - ClusterIP
  - LoadBalancer
- Neste primeiro momento iremos falar e demonstrar o NodePort
  - O serviço NodePort funciona mapeando portas, sendo uma no node e outra na aplicação.
  - Neste serviço há três portas mapeadas:
    - Porta onde o servidor web está rodando, geralmente a porta 80 e é sempre referenciada como TargetPort
    - Porta do próprio serviço, simplesmente referenciada apenas como porta
    - Porta do node, que usamos para acessar o servidor externalmente, NodePort. As portas do node só podem ser ajustadas entre 30000 e 32767. Se não declararmos essa porta no arquivo yaml, o kubernetes define de forma automática
- Para criar o service você executa o seguinte comando:  kubectl create -f <nome do arquivo.yaml>

  ```sh
  kubectl create -f service-definition.yaml
  ```

- Após a criação do service é possível executar o comando kubectl get service ou kubectl get svc para verificarmos se o serviço foi criado. Neste caso rodei com o grep para pegar apenas o serviço criado com o comando acima

 ``````sh
  kubectl get svc
``````

## Conceito de Namespace

- Todos os objetos do Kubernetes são criados dentro de um espaço com um nome, o namespace
- O kubernetes cria três namespaces:
  - Kube-system: Neste namespace é criado um conjunto de PODs e serviços para finalidade interna do próprio kubernetes. Pode ser visualizado com o comando:

   ``````sh
    kubectl get pods --namespace=kube-system
   ``````

  - kube-public: Neste namespace são criados recursos disponibilizados a todos usuários.
  - default: Neste namespace pode ser criados vários objetos pelo usuário.
- Em um ambiente corporativo recomenda-se utilizar namespaces separados para cada aplicação e ambiente caso seja utilizado apenas um cluster.

## Comandos Namespaces

1 - Criando um namespace

- kubectl create namespace < nome do namespace >

  ```sh
    kubectl create namespace prod
    kubectl create namespace hml
  ```

2 - É possivel criar um Namespace com o arquivo yaml

- kubectl create namespace -f < nome do arquivo.yaml >

   ```sh
    kubectl create -f namespace-definition.yaml
  ```

3 - Listar os Namespasces criados

  ```sh
    kubectl get namespace
  ```

4 - Listar os pods em determinado namespace

- kubectl get pods --namespace=< nome do namespace >

```sh
  kubectl get pods --namespace=dev
```

5 - Listar todos os pods em todos os namespasces

```sh
   kubectl get pods --all -namespaces
```

6 - Troca de contexto entre namespaces

- Este comando é bem importante e um dos mais dificeis para ser lembrado e executado no dia a dia em minha opinião. Para explicar o comando vamos usar o seguinte exemplo:

  - Considerando a imagem abaixo, vamos supor que estou trabalhando no namespace-dev e por algum motivo preciso mudar para o namespace-prod. O comando a ser executado é:

  - kubectl config set-context $(kubectl config current-context) --namespace=< nome do namespace > onde no exemplo acima seria:

```sh
     kubectl config set-context $(kubectl config current-context) --namespace=namespace-prod
```


- Exemplo de 3 namespasces em um único cluster

##  <p align="center">    ![Exemplo de namespaces](https://github.com/leopoldocardoso/k8s/blob/main/namespace-definition/imagem/namespaces.png) ##

## Comandos Imperativos VS Comandos Declarativos ##

- Comandos declarativos: quando utilizamos arquivos yamls (manifestos) para criação dos objetos em um cluster kubernetes. 
  - Exemplos
      ``````
        kubectl create -f deploy.definition.yaml

        kubectl create -f ns.definition.yaml
      ``````

- Comandos imperativos: quando usamos os comandos para criação e/ou edição dos objetos em um cluster kubernetes sem a necessidade de um arquivo yaml.
  - Exemplo:
      ``````
        kubectl run nginx --image=ngingx -> criando pod nginx
        kubectl edit deployment nginx - editando deployment nginx
        kubectl run redis --image redis:alpine -> criando o pod do redis
        kubectl expose pod redis --port=6379 --name redis-service -> criando um serviço para o redis e expondo a porta 6379
        kubectl create deployment webapp --image=nginx --replicas=3 -> criando um deployment com 3 réplicas

      ``````
  - Observação: Sempre importante utilizar o comando kubectl run --help

  ## Schedule ##
- Scheduling (agendador) é a configuração onde o POD será criado, ou seja, em que node o POD será criado.
- Geralmente o schedule é definido de forma automática, mas pode ser definido de forma manual no arquivo yaml como no exemplo do arquivo scheduling.yaml
- Caso não haja schedule o POD ficará com status de pending
- Não é possível mover os PODs entre nodes, é necessário excluir e recriá-lo em novo node com o comando abaixo:

  ``````
    kubectl replace --force -f scheduling.yaml
  ``````

## Labels e Selectors ##
- Labels e selectors são um método padrão para agrupar os objetos
- Labels são propriedades anexadas a cada item
- Os selectors ajudam a filtrar estes itens
- No momento da criação de um POD é possível colocar várias labels
- É possível selecionar um POD através da label
- Verifique o arquivo label-definition.yaml para exemplo de utilização de labels
 Exemplo:

   ``````
   kubectl get pods --selector = app01
   kubectl get all --selector env=prod --no-headers | wc -l
   kubectl get all --selector env=prod,bu=finance,tier=frontend
   ``````
## Annotations ##
- Usadas para registrar detalhes a fins de informação
- Detalhes de ferramentas, versão, e-mail entre outras.
- Verifique o arquivo annotations-definition.yaml para exemplo de utilização de labels

## Taints e Tolerations ##
- Taints são usados par definir restrições do que pode ser programado em um node.
- Permitem que um node repudie um conjunto de PODs.
- Para adicionar um taint há um node utiliza-se o seguinte comando: kubectl taint nodes < nome do node > key=value:taint-effect
- Os taint-effects são:
  - NoSchedule: Os PODs não serão criados no node
  - PreferNoSchedule: O sistema evitará criar PODs neste node
  - NoExecute: Novos PODs não serão criados no node e os já criados serão despejados com o status de evicted.

## Comandos ##
- Verificando a sintaxe e do taint
   ``````
   kubectl taint --help
  ``````

- Adicionando taint no node: kubectl taint nodes < nome do node > < chave=valor >:taint-effect
  ``````
   kubectl taint nodes node01 key=app:NoSchedule
  ``````
- Visualizando taint no node: kubectl describe node < nome do node > ou kubectl describe node < nome do node > | grep Taints

  ``````
   kubectl describe node node01
   kubectl describe node node01 | grep Taints
  ``````
- Removendo o taint do node: kubectl taint nodes < nome do node > < chave=valor >:taint-effect-
   ``````
    kubectl taint nodes node01 key=app:NoSchedule-
    kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
  ``````
 ## Node Selector ##
 - Utilizado para selecionar em que nó o POD será criado
 - Necessário colocar labels no node para dar match com a label do POD
 - Para colocar uma label em um node usa-se o comando: kubectl label nodes < nome do node >  <label-key>=<values>

   ``````
	  kubectl label nodes node01 size=large
   ``````
 - Após colocar o label no node, devemos colocar o node selector no POD no bloco spec conforme o arquivo nodeSelector.yaml
 - Caso não haja o match entre o label no node e o nodeSelector o POD fica com o status Pending
 - Para remover uma label usar o seguinte comando: kubectl label nodes < nome do node >  < label>-
 - Para mais detalhes visualize o arquivo node-selector-definition.yaml

   ``````
    kubectl label nodes worker size-
   ``````

- Para visualizar labels nos nodes executar o seguinte comando:

  ``````
    kubectl get nodes --show-labels
  ``````

## Node Affinity ##

- Proporciona capacidades avançadas para limitar a colocação de PODs em nodes específicos.
- É configurado com operators que podem ser:

  - operator: IN
    values: "valores desejados"

  - operator: NotIn
    values: "valores desejados"

  - operator: Exists # Não tem values, apenas verifica se a label existe no node.

- Para exemplos de como aplicar consulte o arquivo node-affinity-definition.yaml

### Node Affinity Types ###

- O tipo de afinidade dos nodes defini o comportamento do schedule com a afinidade dos nodes e as etapas do ciclo de vida do POD.
- Os tipos são:

   - RequiredDuringShchedulingIgnoreDuringExecution: o POD só será executado no node que tenha a mesma label do POD.

     **Importante:** Se houver uma mudança de label do node após a criação do POD o mesmo permanecerá criado.

   - preferredDuringSchedulingIgnoredDuringExecution: o POD será executado de preferência no node que tenha a mesma label.

   - RequiredDuringSchedulingRequired: Requer a label correta tanto na criação do POD quanto na execução, caso haja alguma alteração na label o POD não permancerá criado.

## Resources Requirements and Limits ##

- Cada node tem um conjunto de recursos de CPU e memória disponíveis.
- Cada POD requer um conjunto de memória e CPU para ser executado.
- O Kube-Schedule determina em que node o POD será criado. Esse node será escolhido, além de outra opções, também por recursos de CPU e memória disponíveis. Caso não haja recursos o POD ficará com o status pendente (Pending).
- É possível colocar limits de CPU e memória para utilização do POD além dos valores de requests.
- Quando colocamos os valores de requests de CPU e memória no arquivo yaml garantimos o pod receberá aquela "quantidade de recurso", desde que esteja disponível no node.
- Além do request podemos utilizar os limits para limitar o uso de um recurso do node utilizado pelo POD.
- Caso um pode tente usar CPU além de do limite estabelecido, o sistema "estrangula" (Throttle) esse POD.
- Um POD não pode utilizar mais CPU do que o definido no limit.
- Quanto a memória, um POD pode utilizar mais memória que o definido nos limits, porém se for uma constante o POD é encerrado com o status OOM killed (Out Of Memory)
- O Kubernetes não tem um padrão de utilização de memória e CPU, então caso não seja definido um limit, um POD pode utilizar todo o recurso de memória e cpu existente, sufocando o node.
- Para verificar a conguração de limits e requestes de cpu e memória acesse o arquivo limits-pod-definition.yaml.

### Comportamento CPU ###

 - No requests / No limits: POD utiliza todo recurso de CPU até ocorrer Throttle.
 - No requests / Limits: Kubernetes iguala automaticamente os requests aos Limits.
 - Requests / Limits: Cada POD recebe a quantidade de CPUS solicitadas, podendo utilizar até o limit definido.
 - Requests / No limits: Neste cenário, considerando dois PODs, os PODs receberão os recursos informados no request, porém se o POD-1, por exemplo, precisar de mais CPU do que o POD-2 ele terá, pois não existe limits, caso o POD-2 precisar de mais recurso o POD-1 cederá recurso para o POD-2.

### Comportamento Memória

- No requests / No limits: Em um cenário com dois PODs um dos PODs pode utiliza toda memória e isso não é ideal.
- No requests / Limits: Mesmo comportamento de CPU.
- Requests / Limits: Mesmo comportamento CPU
- Requests / No limits: Se o POD-1 consumir toda memória, e o POD-2 solicitar memória, a única forma de liberar memória é excluir o POD-1.
- Exemplo de limits de memória: limits-memory.yaml

### LimitRange

- Objeto que garante que cada POD tenha um padrão de utilização de memória e CPU.
- Define limites de cpu e memória.
- É criado no nível de POD e não afeta PODs já criados.

### Resources Quotas

- Objeto de nível de namespace.
- Limita a utilização de memória e CPU por namespace
- Segue yaml para exemplificar a criação de resources quotas: resources-quotas.yaml

## Daemon Sets

- Garante que uma cópia do POD esteja sempre presente em todos os nós do cluster.
- Quando usar daemon sets:
  - Solução de monitramento do cluster.
  - Na implantação do kube-proxy.
- A criação de um daemo-set é semelhante a criação de um replica-set.

## Static PODs

- Kubelet é responsável pela criação dos PODs.
- kubelet verifica o diretório /etc/kubernetes/manifests. Caso haja manifestos dentro deste diretório ele realiza a criação dos PODs.
- Qualquer alteração neste diretório o kubelet recriará o POD.
- Esses PODs criados pelo kubelet por conta própria sem a intervenção do api server ou do restante dos componentes do cluster são chamados de PODs estáticos.
- O kubelet funciona a nível de POD, só entende PODs e é por isso que só pode criar PODs.
- O diretório pode ser diferente do do /etc/kubernetes/manifests e é definido na configuração do kubelet service no path abaixo:

```
  --pod-manifest-path=/etc/kubernetes/manifests
```

- Outra opção de configuração é definir o caminho para outro arquivo de configuração.
- Use a opção config=kubeconfig.yaml
- No arquivo kubeconfig.yaml defina o staticPodPath = /etc/kubernetes/manifests

### Verificando o caminho de configuração

- Investigue o kubelet.service:
  - --pod-manifest-path = <caminho>
  - config=kubeconfig.yaml
    - verificar neste arquivo staticPodPath

- Toda essa situação se encaixa apenas se o cluster não iver o kube api server.

## Priority Classes

## <p align="center"> ![Priorities](https://github.com/leopoldocardoso/k8s/blob/main/priority-class/imagens/priorities.png)

- As prioridades podem ser altas com o número de 1 bilhão e baixa como 2 bilhões negativos.
- Número maior, indica prioridade mais alta.
- O número de 1 bilhão a 2 bilhões negativos é para os aplicativos ou cargas de trabalho implantadas como aplicativos no cluster.
- Há um intervalo separado que é dedicado a partes críticas do sistema interno como os próprios componentes do control plane.
- Eles sempre recebem a prioridade máxima e tem prioridade de até 2 bilhões.
- Podemos verificar as classes com o comando:

```sh
kubectl get priorityclasses
```

- PODs com prioridade alta são colocadso primeiro no cluster, caso haja recursos, os PODs com menor prioridade são alocados.
- Podemos criar uma nova Priority Class e adicionar a um POD.
- Verifique o arquivo priority-class.yaml.

## Multiple Schedulers

- Além do scheduler padrão, podemos ter outros schedulers.
- Para criar um POD utilzando o novo agendador, é necessário passar a informação no yaml do do POD, nas especificações do container.
- Se o nome for passado de forma incorreta o POD ficará com o status Pending.
- Podemos executar o comando kubectl get pods -o wide para verificar qual agendador pegou.
- Você pode visualizar estas configurações no arquivo multpli-schedulers.yaml.

### Configuring Scheduler Profiles

- Quando criamos um POD ele entra na fila de programação (Scheduling Queue).
- Os PODs são ordenados por prioridades, definidas no próprio POD.
- Existem plugins no kubernetes para programação dos PODs.
- É possível configurar vários perfis dentro de um único agendador.

## Admission Controllers

- Admission controllers ajudam a implementar melhores medidas de segurança para reforçar a forma de como o cluster é usado.
- Alguns Admission controllers:
  - AlwaysPullImage
  - DefaultStorageClass
  - EventRateLimit: taxa de evento que pode ajudar a definir um limit de solicitações.
  - NamespacesExists: admission controller padrão.
  - NamespaceAutoProvision: quando habilitado permite que um namespace seja criado automaticamente no momento da criação de um POD, por exemplo, caso ele não exista.
- Para ver a lista de admission controllers padrão execute o comando:

```sh
kube-apiserver -h | grep enable-admission-plugins
```

- Se tiver executando o comando em uma configuração baseada no kubeadm o comando é:

```sh
kubectl exec kube-apiserver-controlplane -n kube-system --kube-apiserver -h | grep enable-admission-plugins
```

- Para adicionar um admission controller, atualize a linha --enable-admission-plugins com o controller que você deseja adicionar.
- Os plugins NamespaceExists e NamespaceAutoProision estão obsletos e foram substituidos pelo Controller NamespaceLifecycle.
- O NamespaceLifecycle garante que as solicitações feitas para um namespace inexistente sejam rejeitas e os namespaces padrão Kube-System e kubepublic não possam ser execluídos.
- É possível checar enable-admission-plugins em de duas formas:
  - /etc/kubernetes/manifests/kube-apiserver.yaml
  - com o comando abaixo:

  ```sh
  ps -ef | grep kube-apiserver | grep admission-plugins
  ```

## Monitorando Cluster Kubernetes

- O Kubernetes não tem uma ferramenta nativa para monitoramente de recursos
- Alguns comandos podem ser utilizados após a instalação de metric servers
- O metric-servers pode ser instalado a partir da url abaixo utilizando o wget. Importante: Após baixar para testes em ambiente sem tls adicionar no arquivo yaml no campo args a linha: --kubelet-insecure-tls

	wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

- Alguns comandos:
		- kubectl top node: visualizar o consumo de memória e cpu dos nodes
		- kubectl top pod: visualizar métrica de desempenho dos PODs
		- kubectl logs -f < nome do pod >: verificação dos logs


### Rolling Updates and Rollbacks (Rollout Versioning)

- Quando criamos um rollout(implementação) é criado uma revisão (versão) deste rollout
- Quando é criado um novo rollout, uma nova versão é criada, chamada de revisao-2, por exemplo
- Isso permite acompanhar as atualizações de versão e permite realizar Rollback, ou seja, voltar a versão se necessário

- Alguns comandos:
  - Visualizar status de um deploy:

    ````
     kubectl status deploy/< nome do deploy >
    ``````
  - Visualizar revisões/versões do deploy:

     ````
     kubectl rollout history deployment/< nome do deploy >
     ``````

  - Realizando Rollback de um deploy para ultima versão antes da alteração:

     ``````
    kubectl rollout undo deployment < nome do deploy >
    ``````

  - Realizando Rollback de um deploy para uma versão específica:

    ``````
     kubectl rollout undo deploy < nomedodeploy > --to-revision=< númerodarevisão >
    ``````

### Deployment Strategy ###

- Há duas formas de executar novos deploys:

	- Recreate: Todos os PODs antigos são destruídos e após isso novos PODs são criados.
	- Rolling Update: Os PODs são recriados um a um enquantos PODs antigos são destruídos


### ENV: Variáveis no Kubernetes ###

- As variáveis podem ser utilizadas para armazenar dados não sensíveis.
- A variável é um array
- É declarada através da propriedade ENV, com syntaxe de chave:valor
- Cada item sobre a propriedade ENV começa com um travessão ( - )
- Elas pode ser declaradas de três formas num deploy/pod:
		- Diretamente no Deploy/POD: utilizado para dados não sensíveis.
		- ConfigMaps: utilizado para dados não sensíveis
		- Secrets: utilizado para dados sensíveis
- Para verificar como as ENVS são declaradas de forma direta em um pod verificar o arquivo envs-simple-pod.yaml

### ConfigMaps ###

- Uma outra forma de configuração de variáveis que facilita o gerenciamento de várias variáveis em um único POD
- É possível criar um ConfigMap de forma imperativa ou de forma declarativa
	- Criando configmap de forma imperativa: kubectl create configmap < config-name > --from-literal=< chave >=< valor >

    ``````
			 kubectl create configmap app-config --from-literal=user=leopoldo
    ``````
	- Criando configmap de forma declarativa: kubectl create -f < nome do arquivo.yaml >
  ``````
	  kubectl create -f config-map-definition.yaml
- Para visualizar o configmap criado executar o comando:
  ``````
   kubectl get configmaps
- Para obter uma descrição e visualizar os dados do configmap executar o comando:
  ``````
     kubectl describe configmap < configmap-name >
  ``````
 -  Após criar o configmap, seja de forma declarativa ou imperativa, é possível declarar o configmap no pod, como no arquivo config-map-simple-pod.yaml

### Secrets ###

- Utilizada para armazenar dados sensíveis usados por uma aplicação
- Como o configmaps, as secrets podem ser criadas de modo imperativo e declarativo
		- Criando secrets no modo imperativo: kubectl create secret generic < secret-name > --from-literal=< chave >=< valor >
    ``````
	   kubectl create secret generic secret-app --from-literal=pass=12234
    ``````
	- Criando secrets no modo declarativo: kubectl create -f < nome-do-arquivo.yaml >
    ``````
	   kubectl create -f secrets-definition.yaml
- Para declarar as secrets no pod configurar o arquivo yaml de acordo com o secret-simple-pod.yaml
- O problema é que desta forma passamos os dados em texto claro, sem nenhuma criptografia e qualquer pessoa com acesso ao arquivo yaml terá acesso aos dados
- Uma opção seria codificar os dados usando, em sistemas unix, o comando echo -n < valor > | base64
- Ainda assim qualquer pessoa com acesso ao POD teria acesso aos dados, inclusive decodificando com o comando echo -n < valor > | base64 --decode
- Para usarmos secrets com segurança o ideal é utilizar ferramentas de terceiros como vault da aws, azure, hashicorp entre outros

#### Outros comandos referente a secret: ####
- Visualiza as secrets criadas:
  ``````
    kubectl get secrets
- Descreve as secrets:kubectl describe secrets:
  ``````
    kubectl describe secret <secret-name>
- Visualiza os valores das secrets:
  ``````
    kubectl get secrets <secret-name> -o yaml
  ``````
