# k8s
## Conceito de POD ##

 - O POD é o menor objeto que se pode criar no kubernetes
 - Se precisarmos escalar a aplicação, adicionamos uma nova instância de um POD com a mesma aplicação
 - Geralmente a relação é de um POD para um Container
 - Com multicontainers, um mesmo POD pode ter multiplos containers que geralmente não são do mesmo tipo 

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