# Estudos k8s <p align="center"><img src="http://img.shields.io/static/v1?label=STATUS&message=EM%20DESENVOLVIMENTO&color=GREEN&style=for-the-badge"/></p>

## Este repositório tem como objetivo compartilhar o conhecimento que venho adquirindo nos estudos de Kubernetes

#### Nota: os arquivos citados nos comandos estão dentro de pastas dentro deste repositório

## Conceito de POD ##

 - O POD é o menor objeto que se pode criar no kubernetes
 - Se precisarmos escalar a aplicação, adicionamos uma nova instância de um POD com a mesma aplicação
 - Geralmente a relação é de um POD para um Container
 - Com multicontainers, um mesmo POD pode ter multiplos containers que geralmente não são do mesmo tipo 
 - É possível criar um pod com o seguinte comando: kubectl run < nome do pod> < nome da image >

```
  kubectl run webserver nginx
```

 ## Conceito de ReplicaSet ##

- O propósito de um ReplicaSet é gerenciar um conjunto de réplicas de Pods em execução a qualquer momento 
- É geralmente utilizado para garantir a disponibilidade de um certo número de Pods idênticos
- Para criar um replicaset a partir do arquivo yaml basta executar o comando: kubectl create -f < nome do arquivo.yaml >

``````
  kubectl create -f replicaset-definition.yaml
``````
    
- O número de réplicas no arquivo yaml é quantidade de pods que serão criados

## Escalando ReplicaSet ##

É possível escalar o ReplicaSet de 3 maneiras:

1 - Editando o arquivo yaml, por exemplo, replicaset-definition.yaml, alterando o número de réplicas. Após salvar o arquivo, executar o comando:

   ``````
     kubectl replace -f repicaset-definition.yaml
   ``````

2 - Escalando através do comando kubectl scale --replicas=< nro de pods > -f < nome do arquivo.yaml >

   ``````
    kubectl scale --replicas=3 -f replicaset-definition.yaml
  ``````

3 - Escalando através do comando kubectl scale --replicas=< nro de pods > replicaset < nome do arquivo.yaml >

   ``````
    kubectl scale --replicas=8 replicaset replicaset-definition.yaml
  ``````

## Conceito de Deployment ##

- O Deployment implementa PODs e ReplicaSets
- Fornece atualizações declarativas para PODs e ReplicaSets
- As atualizações podem ser contínuas
- O arquivo do Deployment é praticamente o mesmo arquivo do ReplicaSet, modificando apenas o kind

## Criando um Deployment ##

É possível criar um Deployment de 2 formas:

1 - Executando o comando kubectl create deployment --image=< nome da imagem> < nome do deployment >
    Obs: poder ser deployment ou deploy

  ``````  
   kubectl create deploy --image=nginx nginx-deploy
  ``````
2 - Gerando um manifesto yaml executando a linha de comando kubectl create deploy --image=< nome da imagem > < nome do deploy > --dry-run=client -o yaml > deploy-definition.yaml

  ``````
    kubectl create deploy --image=redis redis-deployment --dry-run=client -o yaml > deployment-definition.yaml
  ``````
   - Após gerar o arquivo yaml é necessário executar o comando kubectl create -f < nome do arquivo.yaml >

   ``````
    kubectl create -f deployment-definition.yaml
   ``````
Algumas informações sobre o comando:
 - --dry-run=client -o yaml > deployment-definition.yaml: não executa a criação do deployment. Gera o manifesto yaml e salva este manifesto no arquivo deployment-definition.yaml
 - Partes do comando:
   - --dry-run=client: não executa a criação do deploy
   - -o yaml > deployment-definition.yaml: gera o manifesto yaml e salva este manifesto no arquivo deployment-definition.yaml

## Conceito de Services ##

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

  ``````
  kubectl create -f service-definition.yaml
  ``````
- Após a criação do service é possível executar o comando kubectl get service ou kubectl get svc para verificarmos se o serviço foi criado. Neste caso rodei com o grep para pegar apenas o serviço criado com o comando acima

 ``````
  kubectl get svc
``````
## Conceito de Namespace ##

- Todos os objetos do Kubernetes são criados dentro de um espaço com um nome, o namespace
- O kubernetes cria três namespaces:
  - Kube-system: Neste namespace é criado um conjunto de PODs e serviços para finalidade interna do próprio kubernetes. Pode ser visualizado com o comando:

   ``````
    kubectl get pods --namespace=kube-system
   ``````
  - kube-public: Neste namespace são criados recursos disponibilizados a todos usuários.
  - default: Neste namespace pode ser criados vários objetos pelo usuário.
- Em um ambiente corporativo recomenda-se utilizar namespaces separados para cada aplicação e ambiente caso seja utilizado apenas um cluster.

## Comandos Namespaces ##

1 - Criando um namespace
  - kubectl create namespace < nome do namespace >

  ``````
    kubectl create namespace prod
    kubectl create namespace hml
  ``````

2 - É possivel criar um Namespace com o arquivo yaml
  - kubectl create namespace -f < nome do arquivo.yaml >

   ``````
    kubectl create -f namespace-definition.yaml
  ``````

3 - Listar os Namespasces criados

  ``````
    kubectl get namespace
  ``````

4 - Listar os pods em determinado namespace
  - kubectl get pods --namespace=< nome do namespace >

``````
  kubectl get pods --namespace=dev
``````

5 - Listar todos os pods em todos os namespasces

``````
   kubectl get pods --all -namespaces
``````

6 - Troca de contexto entre namespaces

   - Este comando é bem importante e um dos mais dificeis para ser lembrado e executado no dia a dia em minha opinião. Para explicar o comando vamos usar o seguinte exemplo:

     - Considerando a imagem abaixo, vamos supor que estou trabalhando no namespace-dev e por algum motivo preciso mudar para o namespace-prod. O comando a ser executado é:

     - kubectl config set-context $(kubectl config current-context) --namespace=< nome do namespace > onde no exemplo acima seria:

``````
     kubectl config set-context $(kubectl config current-context) --namespace=namespace-prod
``````


- Exemplo de 3 namespasces em um único cluster

##  <p align="center">    ![Exemplo de namespaces](https://github.com/leopoldocardoso/k8s/blob/main/namespace-definition/imagem/namespaces.png) ##

## Comandos Imperativos VS Comandos Declarativos ##

- Comandos declarativos: quando utilizamos arquivos yamls (manifestos) para criação dos objetos em um cluster kubernetes. 
  - Exemplos
      ``````
        kubectl create -f deploy.definition.yaml

        kubectl create -f ns.definition.yaml
      ``````

- Comandos imperativos: quando usamos os comandos para criação dos objetos em um cluster kubernetes sem a necessidade de um arquivo yaml.
  - Exemplo:
      ``````
        kubectl run nginx --image=ngingx
      ``````
  
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
- Principal função é garantir que o POD será criado nos nodes específicos
-	O Node Affinity permite colocar expressões mais avançadas do que o node selector no momento de selecionar o node
-	Segue algumas das expressões mais utilizadas:
	-	Operator: pode ter os valores:
			- In
			- Not In
			- Exists
	-	Node Affinity Types
			 - requiredDuringSchedulingIgnoredDuringExecution: o POD só será executado no node que tenha a mesma label do POD. Importante: Se houver uma mudança label do node após a criação do POD o mesmo permanecerá criado.
       - preferredDuringSchedulingIgnoredDuringExecution: o POD será executado de preferência no node que tenha a mesma label
- Para mais detalhes visualize o arquivo node-affinity-definition.yaml

## Resources Requirements and Limits ##

- Cada node tem um conjunto de recursos de CPU e memória disponíveis.
- Cada POD requer um conjunto de memória e CPU para ser executado
- O Kube-Schedule determina em que node o POD será criado. Esse node será escolhido, além de outra opções, também por recursos de CPU e memória disponíveis. Caso não haja recursos o POD ficará com o status pendente (Pending)
- É possível colocar limits de CPU e memória para utilização do POD além dos valores de requests
- Quando colocamos os valores de requests de CPU e memória no arquivo yaml garantimos o pod receberá aquela "quantidade de recurso", desde que esteja disponível no node.
- Além do request podemos utilizar os limits para limitar o uso de um recurso do node utilizado pelo POD.
- Para verificar a conguração de limits e requestes de cpu e memória acesse o arquivo limits-pod-definition.yaml
- Caso um pode tente usar CPU além de do limite estabelecido, o sistema "estrangula" (Throttle) esse POD.
- Um POD não pode utilizar mais CPU do que o definido no limit
- Quanto a memória, um POD pode utilizar mais memória que o definido nos limits, porém se for uma constante o POD é encerrado com o status OOM killed (Out Of Memory)
- O Kubernetes não tem um padrão de utilização de memória e CPU, então caso não seja definido um limit, um POD pode utilizar todo o recurso de memória e cpu existente, sufocando o node.
	
### Comportamento CPU ###

 - No requests / No limits: POD utiliza todo recurso de CPU até ocorrer Throttle
 - No requests / Limits: Kubernetes igual automaticamente os requests aos Limits
 - Requests / Limits: Cada POD recebe a quantidade de CPUS solicitadas, podendo utilizar até o limit definido
 - Requests / No limits: Neste cenário, considerando dois PODs, os PODs receberão os recursos informados no request, porém se o POD-1, por exemplo, precisar de mais CPU do que o POD-2 ele terá, pois não existe limits, caso o POD-2 precisar de mais recurso o POD-1 cederá recurso para o POD-2.
