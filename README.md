# Домашнее задание к занятию «Управление доступом»

### Цель задания

В тестовой среде Kubernetes нужно предоставить ограниченный доступ пользователю.

------

### Чеклист готовности к домашнему заданию

1. Установлено k8s-решение, например MicroK8S.
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым github-репозиторием.

------

### Инструменты / дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) RBAC.
2. [Пользователи и авторизация RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/470503/).
3. [RBAC with Kubernetes in Minikube](https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b).

------

### Задание 1. Создайте конфигурацию для подключения пользователя

1. Создайте и подпишите SSL-сертификат для подключения к кластеру.

```
user@k8s:/opt/hw_k8s_9$ sudo openssl genrsa -out netology.key 2048
user@k8s:/opt/hw_k8s_9$ sudo cat netology.key
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQChVwJMvMEjm9zF
p1GTgXWNRPYNmIHCpftgAfKZDqvFRAJJziSBjL+6yAjY/mdAvEUOGnAdAT3fsk4x
6GLjI6Mh5Jmk7k0YuTVy2k78DKVzTZP4AzAchtX7377NhPZtG7iNY/OxRKVtuo7/
KVqQVpyMjymOi8fzpKKSwfargHbI5EvgGjFpWoUhiHUhH5SlNcFjwoBpMnfH8czz
2Zar6g14k2jfM58GQerI8T6TFTnuGQWGOHxPvyRDOc12lgzPnQl60skDeIL1UrEu
GQH1wi1BetYpDX4gkQPt3COi9TiEOoidKWYFgndFn+xnmbNmUk+6NNZ2/qcGJDuc
6SL9p4RPAgMBAAECggEAKE9K3c1THAh3ElMJiKcRragLKb5uvSknMweJi0AlHnYt
dC8y48M8q/gKbdyyA3SGdE2asUR8JwWvj7yV6FYhDfjFgnWfgYvUlMuCbGrkc3hw
fcieHqJ5mCKA02xi/UOtynWsjx+tjUrNK2czn1hkaKHkKh82Z+M8Uxpu/M5t3yb1
EL4JT4YoVUvPzy3nSDqmX7UBHcwIsCz3MpjNy1XWQHqImzpGZkfJN7lI0jvVxZH+
SC3up+jCfknCRpXBeho5hCDGx6PRfDYqVjSb3mpe1IWAddvrpGkEcK/NYx6uohiG
.....
```

```
user@k8s:/opt/hw_k8s_9$ sudo openssl req -new -key netology.key -out netology.csr -subj "/CN=test/O=test"
user@k8s:/opt/hw_k8s_9$ cat netology.csr
-----BEGIN CERTIFICATE REQUEST-----
MIICYzCCAUsCAQAwHjENMAsGA1UEAwwEdGVzdDENMAsGA1UECgwEdGVzdDCCASIw
DQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKFXAky8wSOb3MWnUZOBdY1E9g2Y
gcKl+2AB8pkOq8VEAknOJIGMv7rICNj+Z0C8RQ4acB0BPd+yTjHoYuMjoyHkmaTu
TRi5NXLaTvwMpXNNk/gDMByG1fvfvs2E9m0buI1j87FEpW26jv8pWpBWnIyPKY6L
x/OkopLB9quAdsjkS+AaMWlahSGIdSEflKU1wWPCgGkyd8fxzPPZlqvqDXiTaN8z
nwZB6sjxPpMVOe4ZBYY4fE+/JEM5zXaWDM+dCXrSyQN4gvVSsS4ZAfXCLUF61ikN
fiCRA+3cI6L1OIQ6iJ0pZgWCd0Wf7GeZs2ZST7o01nb+pwYkO5zpIv2nhE8CAwEA
.....
```

```
user@k8s:/opt/hw_k8s_9$ sudo scp /var/snap/microk8s/current/certs/ca.crt  .
user@k8s:/opt/hw_k8s_9$ sudo scp /var/snap/microk8s/current/certs/ca.key   .
user@k8s:/opt/hw_k8s_9$ sudo openssl x509 -req -in netology.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out netology.crt -days 500
Certificate request self-signature ok
subject=CN = test, O = test
```

2. Настройте конфигурационный файл kubectl для подключения.

```
user@k8s:/opt/hw_k8s_9$ microk8s kubectl config set-credentials test --client-certificate=netology.crt --client-key=netology.key --embed-certs=true
User "test" set.
user@k8s:/opt/hw_k8s_9$ microk8s kubectl config get-clusters
NAME
microk8s-cluster
user@k8s:/opt/hw_k8s_9$ microk8s kubectl config set-context test --cluster=microk8s-cluster --user=test
Context "test" created.
user@k8s:/opt/hw_k8s_9$ microk8s kubectl config get-contexts
CURRENT   NAME       CLUSTER            AUTHINFO   NAMESPACE
*         microk8s   microk8s-cluster   admin
          test       microk8s-cluster
user@k8s:/opt/hw_k8s_9$ microk8s kubectl config use-context test
Switched to context "test".
user@k8s:/opt/hw_k8s_9$ microk8s kubectl config get-contexts
CURRENT   NAME       CLUSTER            AUTHINFO   NAMESPACE
          microk8s   microk8s-cluster   admin
*         test       microk8s-cluster
user@k8s:/opt/hw_k8s_9$ microk8s kubectl get nodes
NAME   STATUS   ROLES    AGE   VERSION
k8s    Ready    <none>   34d   v1.31.3
```

3. Создайте роли и все необходимые настройки для пользователя.

```
user@k8s:/opt/hw_k8s_9$ microk8s kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:16443
  name: microk8s-cluster
contexts:
- context:
    cluster: microk8s-cluster
    user: admin
  name: microk8s
- context:
    cluster: microk8s-cluster
    user: test
  name: test
current-context: test
kind: Config
preferences: {}
users:
- name: admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: test
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: artem-role
rules:
- apiGroups: [""]
  resources: ["pods/log", "pods/describe"]
  verbs: ["get", "list", "watch"]
```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: test_role_binding
subjects:
- kind: User
  name: test
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: test-role
  apiGroup: rbac.authorization.k8s.io
```

```
user@k8s:/opt/hw_k8s_9$ microk8s kubectl apply -f role.yml
role.rbac.authorization.k8s.io/test-role created
user@k8s:/opt/hw_k8s_9$ microk8s kubectl apply -f role-binding.yml
rolebinding.rbac.authorization.k8s.io/test_role_binding created

user@k8s:/opt/hw_k8s_9$ microk8s kubectl get rolebinding.rbac.authorization.k8s.io
NAME                ROLE             AGE
test_role_binding   Role/test-role   2m27s
user@k8s:/opt/hw_k8s_9$ microk8s kubectl get rolebinding.rbac.authorization.k8s.io test_role_binding
NAME                ROLE             AGE
test_role_binding   Role/test-role   2m36s
user@k8s:/opt/hw_k8s_9$ microk8s kubectl describe rolebinding.rbac.authorization.k8s.io test_role_binding
Name:         test_role_binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  test-role
Subjects:
  Kind  Name  Namespace
  ----  ----  ---------
  User  test
```

4. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию (`kubectl logs pod <pod_id>`, `kubectl describe pod <pod_id>`).

```
user@k8s:/opt/hw_k8s_9$ microk8s kubectl logs nginx-https-ssl-595f7c9685-9m7rw
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2024/12/24 07:46:13 [notice] 1#1: using the "epoll" event method
2024/12/24 07:46:13 [notice] 1#1: nginx/1.27.3
2024/12/24 07:46:13 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2024/12/24 07:46:13 [notice] 1#1: OS: Linux 6.8.0-51-generic
2024/12/24 07:46:13 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 65536:65536
2024/12/24 07:46:13 [notice] 1#1: start worker processes
2024/12/24 07:46:13 [notice] 1#1: start worker process 29
2024/12/24 07:46:13 [notice] 1#1: start worker process 30
```

```
user@k8s:/opt/hw_k8s_9$ microk8s kubectl describe pod nginx-https-ssl-595f7c9685-9m7rw
Name:             nginx-https-ssl-595f7c9685-9m7rw
Namespace:        default
Priority:         0
Service Account:  default
Node:             k8s/10.129.0.38
Start Time:       Mon, 23 Dec 2024 08:19:34 +0000
Labels:           app=nginx-hs
                  pod-template-hash=595f7c9685
Annotations:      cni.projectcalico.org/containerID: 7051c31655b24287e118d17d62d6a1b0207d60bc71c453584c82005f87fe860f
                  cni.projectcalico.org/podIP: 10.1.77.45/32
                  cni.projectcalico.org/podIPs: 10.1.77.45/32
Status:           Running
IP:               10.1.77.45
IPs:
  IP:           10.1.77.45
Controlled By:  ReplicaSet/nginx-https-ssl-595f7c9685
Containers:
  nginx:
    Container ID:   containerd://d06e505407bb9a3067727b3ada8122363a974ff8b05e5cec44953d3c03db7586
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:fb197595ebe76b9c0c14ab68159fd3c08bd067ec62300583543f0ebda353b5be
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 24 Dec 2024 07:46:06 +0000
    Last State:     Terminated
      Reason:       Unknown
      Exit Code:    255
      Started:      Mon, 23 Dec 2024 08:19:36 +0000
      Finished:     Tue, 24 Dec 2024 07:39:15 +0000
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from nginx-html (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9tf2q (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  nginx-html:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      myconfigmap
    Optional:  false
  kube-api-access-9tf2q:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

5. Предоставьте манифесты и скриншоты и/или вывод необходимых команд.

[role.yml](https://github.com/stepynin-georgy/hw_k8s_9/blob/main/role.yml)

[role-binding.yml](https://github.com/stepynin-georgy/hw_k8s_9/blob/main/role-binding.yml)

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
