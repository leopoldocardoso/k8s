# Guia de Troubleshooting para a Prova CKA (Certified Kubernetes Administrator)

A seção de Troubleshooting vale **30% da nota** e testa sua capacidade de diagnosticar e resolver problemas de forma sistemática. A chave é o método, não a memorização.

## 1. Troubleshooting de Aplicações (Pods)

O cenário mais comum: um Pod não inicia ou está em um estado de erro.

**Sintomas:** `Pending`, `CrashLoopBackOff`, `Error`, `ImagePullBackOff`.

### Plano de Ação Sistemático

1.  **`kubectl get pods -o wide`**
    *   Verifica o status e em qual nó o Pod está (ou deveria estar).

2.  **`kubectl describe pod <nome-do-pod>`**
    *   **Seu comando mais importante.** Role até a seção `Events` no final. Ela quase sempre revela a causa raiz.
    *   **`Pending`**: O `describe` mostrará o motivo.
        *   *Causa Comum*: Recursos insuficientes (`insufficient cpu/memory`).
        *   *Causa Comum*: O Pod não pode ser agendado devido a `taints` não tolerados ou `node selectors/affinity` que não correspondem a nenhum nó.
    *   **`ImagePullBackOff` / `ErrImagePull`**: A imagem do contêiner não pôde ser baixada.
        *   *Causa Comum*: Nome da imagem ou tag incorreta no manifesto.
        *   *Causa Comum*: Erro de autenticação com o registro de imagem. Verifique se um `ImagePullSecret` é necessário e se está configurado corretamente.
    *   **`CrashLoopBackOff` / `Error`**: O contêiner inicia, mas falha imediatamente. O problema está na aplicação, não na orquestração do Pod.
        *   *Próximo Passo*: Verificar os logs.

3.  **`kubectl logs <nome-do-pod>`**
    *   Mostra a saída padrão (stdout) da aplicação. Procure por mensagens de erro, stack traces, ou erros de configuração.

4.  **`kubectl logs <nome-do-pod> --previous` (ou `-p`)**
    *   Essencial para Pods em `CrashLoopBackOff`. Mostra os logs da *última* execução que falhou.

5.  **`kubectl exec -it <nome-do-pod> -- /bin/sh`**
    *   Use se o Pod está `Running` mas a aplicação não responde.
    *   **Dentro do Pod**: Verifique a existência de arquivos de configuração, permissões, e teste a conectividade de rede com `curl localhost:<porta>` ou `ping`.

## 2. Troubleshooting de Rede (Services & DNS)

O Pod está `Running`, mas inacessível de fora ou por outros Pods.

**Sintomas:** Conexão recusada, timeout, nome do serviço não resolve.

### Plano de Ação Sistemático

1.  **Verifique a Conexão Service <-> Pods**
    *   **`kubectl describe service <nome-do-servico>`**: Olhe a seção `Endpoints`.
    *   **Se `Endpoints` estiver vazio**: O `selector` do Service está errado. Os labels no `selector:` do Service **não correspondem** aos labels dos Pods.
    *   **Como verificar**: Compare a saída de `kubectl get service <svc> -o yaml` (olhe o `selector`) com `kubectl get pods --show-labels`.

2.  **Verifique as Portas**
    *   Ainda no `describe service`, confirme se `Port`, `TargetPort`, e `NodePort` (se aplicável) estão corretos.
    *   *Erro Comum*: `port` é a porta do Service, `targetPort` é a porta do contêiner.

3.  **Teste o DNS do Cluster**
    *   Inicie um Pod de diagnóstico: `kubectl run dns-test --rm -it --image=busybox:1.28 -- /bin/sh`
    *   Dentro do Pod, teste a resolução de nomes: `nslookup <nome-do-servico>`. Deve retornar a `ClusterIP` do serviço.
    *   Se `nslookup` falhar, o problema pode ser o CoreDNS. Verifique o status dos Pods do CoreDNS: `kubectl get pods -n kube-system`.

## 3. Troubleshooting do Cluster (Nós & Control Plane)

Problemas de infraestrutura que afetam todo o cluster.

**Sintomas:** Nó em `NotReady`, Pods não são agendados, `kubectl` lento.

### Plano de Ação Sistemático

1.  **`kubectl get nodes`**
    *   Verifique o status de todos os nós.

2.  **`kubectl describe node <nome-do-no-problematico>`**
    *   Foque nas seções `Conditions` e `Events`. Elas indicarão problemas de pressão de recursos (`memory`, `disk`), problemas de rede, ou se o Kubelet parou de reportar.

3.  **Conecte-se ao Nó via SSH (se necessário)**
    *   **Verifique o serviço `kubelet`**:
        *   `systemctl status kubelet`
        *   `journalctl -u kubelet -f` (para ver logs ao vivo)
    *   **Verifique o runtime do contêiner**:
        *   `systemctl status containerd` (ou `docker`)

4.  **Verifique os Componentes do Control Plane**
    *   Os componentes (`etcd`, `kube-apiserver`, `kube-scheduler`, `kube-controller-manager`) rodam como Pods no namespace `kube-system`.
    *   **`kubectl get pods -n kube-system`**: Verifique se todos estão `Running`.
    *   Se algum estiver falhando, use `describe` e `logs` nele para investigar.
