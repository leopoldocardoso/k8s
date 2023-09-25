# k8s

## Nota: os arquivos citados nos comandos estão dentro de pastas dentro deste repositório

## Conceito de POD ##

 - O POD é o menor objeto que se pode criar no kubernetes
 - Se precisarmos escalar a aplicação, adicionamos uma nova instância de um POD com a mesma aplicação
 - Geralmente a relação é de um POD para um Container
 - Com multicontainers, um mesmo POD pode ter multiplos containers que geralmente não são do mesmo tipo 
 - É possível criar um pod com o seguinte comando: kubectl run < nome do pod> < nome da image >

    - kubectl run webserver nginx

 ## Conceito de ReplicaSet ##

- O propósito de um ReplicaSet é gerenciar um conjunto de réplicas de Pods em execução a qualquer momento 
- É geralmente utilizado para garantir a disponibilidade de um certo número de Pods idênticos
- Para criar um replicaset a partir do arquivo yaml basta executar o comando: kubectl create -f < nome do arquivo.yaml >

   - kubectl create -f replicaset-definition.yaml
    
- O número de réplicas no arquivo yaml é quantidade de pods que serão criados

## Escalando ReplicaSet ##

É possível escalar o ReplicaSet de 3 maneiras:

1 - Editando o arquivo yaml, por exemplo, replicaset-definition.yaml, alterando o número de réplicas. Após salvar o arquivo, executar o comando:

    - kubectl replace -f repicaset-definition.yaml

2 - Escalando através do comando kubectl scale --replicas=< nro de pods > -f < nome do arquivo.yaml >

    - kubectl scale --replicas=3 -f replicaset-definition.yaml

3 - Escalando através do comando kubectl scale --replicas=< nro de pods > replicaset < nome do arquivo.yaml >

    - kubectl scale --replicas=8 replicaset replicaset-definition.yaml

## Conceito de Deployment ##

- O Deployment implementa PODs e ReplicaSets
- Fornece atualizações declarativas para PODs e ReplicaSets
- As atualizações podem ser contínuas
- O arquivo do Deployment é praticamente o mesmo arquivo do ReplicaSet, modificando apenas o kind

## Criando um Deployment ##

É possível criar um Deployment de 2 formas:

1 - Executando o comando kubectl create deployment --image=< nome da imagem> < nome do deployment >
    Obs: poder ser deployment ou deploy

     - kubectl create deploy --image=nginx nginx-deploy

2 - Gerando um manifesto yaml executando a linha de comando kubectl create deploy --image=< nome da imagem > < nome do deploy > --dry-run=client -o yaml > deploy-definition.yaml

     - kubectl create deploy --image=redis redis-deployment --dry-run=client -o yaml > deployment-definition.yaml

   - Após gerar o arquivo yaml é necessário executar o comando kubectl create -f < nome do arquivo.yaml >

    - kubectl create -f deployment-definition.yaml

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

      - kubectl create -f service-definition.yaml

- Após a criação do service é possível executar o comando kubectl get service ou kubectl get svc para verificarmos se o serviço foi criado. Neste caso rodei com o grep para pegar apenas o serviço criado com o comando acima

      - kubectl get svc

## Conceito de Namespace ##

- Todos os objetos do Kubernetes são criados dentro de um espaço com um nome, o namespace
- O kubernetes cria três namespaces:
  - Kube-system: Neste namespace é criado um conjunto de PODs e serviços para finalidade interna do próprio kubernetes. Pode ser visualizado com o comando:
      - kubectl get pods --namespace=kube-system
  - kube-public: Neste namespace são criados recursos disponibilizados a todos usuários.
  - default: Neste namespace pode ser criados vários objetos pelo usuário.
- Em um ambiente corporativo recomenda-se utilizar namespaces separados para cada aplicação e ambiente caso seja utilizado apenas um cluster.

## Comandos Namespaces ##

1 - Criando um namespace
  - kubectl create namespace < nome do namespace >

      - kubectl create namespace prod

2 - É possivel criar um Namespace com o arquivo yaml
  - kubectl create namespace < arquivo.yaml >

      - kubectl create -f namespace-definition.yaml



![Exemplo de namespaces](https://github.com/leopoldocardoso/k8s/blob/main/namespace-definition/imagem/namespaces.png)