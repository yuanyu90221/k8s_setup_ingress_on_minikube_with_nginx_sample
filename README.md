# k8s_setup_ingress_on_minikube_with_nginx_sample
## 前言

參考文件： https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/
今天要來實作 [Set up Ingress on Minikube with the NGINX Ingress Controller](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/) 這個任務

Ingress 是一個用來定義路由規則的 k8s 元件

Ingress controller 則是實作這個路由的實體

這個任務將會使用 nginx 當作 Ingress Controller

透過設定路由規則, 讓 client 端透過 HTTP URI 存取到對應的應用

## 佈署目標

1 啟動 minikube 的 Ingress Controller 功能

2 佈署一個 hello world 的應用

3 建立 Ingress 路由, 並且測試路由

4 建立第二個應用

5 更改路由導入第二個應用

## 啟動 minikube 的 Ingress Controller 功能

使用以下指令開啟 minikube 的 ingres controller 功能

```shell=
minikube addons enable ingress
```
![](https://i.imgur.com/c7uW0pk.png)

使用以下指令查看 enable 之後的結果

```shell=
kubectl get pods -n ingress-nginx
```

![](https://i.imgur.com/PGRMnm4.png)


## 佈署一個 hello world 的應用

使用以下指令佈署一個 hello world app

```shell=
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
```

![](https://i.imgur.com/pqkHa53.png)


使用 NodePort 的方式讓使用者可以透過 clusterIP 存取到 hello world 應用

執行以下指令

```shell=
kubectl expose deployment web --type=NodePort --port=8080
```
![](https://i.imgur.com/AHd9NYx.png)


使用以下指令檢查 NodePort 設定

```shell=
kubectl get service web
```

![](https://i.imgur.com/oI0RHFO.png)


使用以下指令讓 minikube 給予 service 一個對外 IP 

```shell=
minikube service web --url
```

![](https://i.imgur.com/2cYHVXD.png)


透過上面的 url 從瀏覽器存取 hello world 應用

![](https://i.imgur.com/qc3i4Y7.png)



## 建立 Ingress 路由, 並且測試路由

建立 example-ingress.yaml 如下

```yaml=
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
```

上面設定內容如下

建立一個 Ingress 

名稱設定為 example-ingress

設定路由規則如下

domain name 是 hello-world.info

對應到的 service 名稱是 web

對應到的 service port 是 8080

使用的 matchPath 為 Prefix

建立指令如下

```yaml=
kubectl apply -f example-ingress.yaml
```
![](https://i.imgur.com/ucRcXlq.png)

檢查 ingress 設定使用以下指令

```shell=
kubectl get ingress
```

![](https://i.imgur.com/tcYk2i9.png)


設定 domain resolve 到 /etc/hosts 新增一行如下

```yaml=
192.168.49.2 hello-world.info
```

要注意的是, 這邊的 IP 是根據 kubectl get ingress 查到的那個 IP, 每個實作各有不同

驗證剛剛的設定透過以下指令

```shell=
curl hello-world.info
```

![](https://i.imgur.com/nJ5PSOs.png)

## 建立第二個應用

使用以下指令佈署第2個 hello world app

```shell=
kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
```

![](https://i.imgur.com/4qEoZLb.png)



使用 NodePort 的方式讓使用者可以透過 clusterIP 存取到第2個 hello world 應用

執行以下指令

```shell=
kubectl expose deployment web2 --type=NodePort --port=8080
```
![](https://i.imgur.com/IbBVGjh.png)


## 更改路由導入第二個應用

新增以下項目到 example-ingress.yaml 

```yaml=
- path: /v2
  pathType: Prefix
  backend:
    service:
      name: web2
      port:
        number: 8080
```

更新之後的 example-ingress.yaml 如下

```yaml=
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
          - path: /v2
            pathType: Prefix
            backend:
              service:
                name: web2
                port:
                  number: 8080
```

設定說明如下：

新增路由 /v2

對應到 service 名稱 web2 

對應到 service Port 8080 

使用以下指令更新 Ingress 路由

```shell=
kubectl apply -f example-ingress.yaml
```
![](https://i.imgur.com/oBfKX2h.png)

測試設定的 Ingress 路由

測試 hello-world.info

```shell=
curl hello-world.info
```

![](https://i.imgur.com/dsl36O1.png)


測試 hello-world.info/v2

```shell=
curl hello-world.info/v2
```

![](https://i.imgur.com/rokkgBN.png)

## 清除 Ingress 佈署

```shell=
kubectl delete -f example-ingress.yaml
```
![](https://i.imgur.com/BRcRQYN.png)


## 後記

部署每個資源之前都必須理解佈署目標的架構

否則就會迷失在該怎麼處理設定

做個圖像化的架構圖是個不錯的方式