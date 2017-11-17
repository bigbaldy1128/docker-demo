# Docker安装使用
* 安装docker（或者安装docker-ce）
```sh
yum install -y docker
```
* 安装docker-ce（上面的安装方式有可能版本不是最新的，导致无法使用docker-compose）
```sh
yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce
```
* 启动
```sh
systemctl start docker
```
* 编写Dockerfile
```docker
# 基于哪个镜像
FROM java:8 
# 将本地文件夹挂载到当前容器
VOLUME /tmp
# 复制文件到容器
ADD docker-demo-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
# 声明需要暴露的端口
EXPOSE 8888
# 配置容器启动后的命令
ENTRYPOINT ["java","-jar","/app.jar"]
```
* 使用docker build命令构建镜像，注意后面的"."别丢了
```sh
docker build -t bigbaldy/docker-demo .
```
* 启动镜像
```sh
docker run -d -p 8888:8888 bigbaldy/docker-demo
```
* 访问http://127.0.0.1:8888/hello 返回hello kubernetes
* 推送镜像
```sh
docker push bigbaldy/docker-demo //需要输入你在DockerHub上注册的用户密码
```
* 推送镜像到私有库
    - docker run -d -p 5000:5000 registry:2 //使用docker registry2.0搭建私有仓库
    - docker tag bigbaldy/docker-demo 127.0.0.1:5000/docker-demo //修改镜像标签，否则会提示找不到镜像的
    - docker push 127.0.0.1:5000/docker-demo
* 获取私有仓库镜像列表
```python
import requests
import json
import traceback

repo_ip = '127.0.0.1'
repo_port = 5000

def getImagesNames(repo_ip,repo_port):
    docker_images = []
    try:
        url = "http://" + repo_ip + ":" +str(repo_port) + "/v2/_catalog"
        res =requests.get(url).content.strip()
        res_dic = json.loads(res)
        images_type = res_dic['repositories']
        for i in images_type:
            url2 = "http://" + repo_ip + ":" +str(repo_port) +"/v2/" + str(i) + "/tags/list"
            res2 =requests.get(url2).content.strip()
            res_dic2 = json.loads(res2)
            name = res_dic2['name']
            tags = res_dic2['tags']
            for tag in tags:
                docker_name = str(repo_ip) + ":" + str(repo_port) + "/" + name + ":" + tag
                docker_images.append(docker_name)
                print docker_name
    except:
        traceback.print_exc()
    return docker_images

print getImagesNames(repo_ip, repo_port)
```
执行就可以看到[u'127.0.0.1:5000/bigbaldy/docker-demo:latest']
## Docker-Compose（一条命令启动多个镜像）
* 安装docker-compose(https://github.com/docker/compose/releases)
* 配置启动文件，例如docker-demo.yml
```yaml
version: '3'
services:
  docker-demo:
    image: bigbaldy/docker-demo
    restart: always
    ports:
      - 8888:8888
```
* 使用docker-compose启动镜像
```sh
docker-compose -f docker-demo.yml up
```
## Maven插件构建Docker镜像
http://blog.csdn.net/qq_22841811/article/details/67369530 //运行会报各种错误！！文章后面有各种解决方式，可以尝试，个人倾向于手动写Dockerfile，然后通过jenkins调用docker命令进行镜像push
# Kubernetes安装使用
## minikube
* [官网](https://kubernetes.io/docs/tasks/tools/install-minikube/README.md)
### 1. 安装virtualbox
* [下载virtualbox](https://www.virtualbox.org/wiki/Linux_Downloads)
* yum install SDL
* yum install xxxx.rpm
* yum install kernel-devel //注意版本要与系统内核版本一致，若不一致请下载相应的kernel-devel安装包或者升级系统内核（http://blog.csdn.net/u010250863/article/details/70169985）
* 如果安装过KVM，请停止服务
    - systemctl stop libvirtd
    - ps -ef|grep kvm|grep -v grep|cut -c 9-15|xargs kill -9
    - modprobe -r kvm_intel
    - modprobe -r kvm
### 2. 安装kubectl 
* 查看最新稳定版：https://storage.googleapis.com/kubernetes-release/release/stable.txt
* 下载，修改版本号即可下载相应版本，例如：https://storage.googleapis.com/kubernetes-release/release/v1.8.3/bin/linux/amd64/kubectl
* chmod +x ./kubectl
* mv ./kubectl /usr/local/bin/kubectl
* [官网](https://kubernetes.io/docs/tasks/tools/install-kubectl/README.md)
### 3. 安装minikube
* [下载](https://github.com/kubernetes/minikube/releases)
### 4. 启动minikube
* minikube start //注意启动过程中会下载localkube，需要翻墙(export http_proxy=http://10.16.13.18:8080)
### 5. 运行container
```sh
kubectl run docker-demo --image=127.0.0.1:5555/bigbaldy/docker-demo --port=8888
```
查看pod:
```sh
kubectl get pods
```
会一直卡在creating，没有成功
### 5. Web UI (Dashboard)
* [官网](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
* 部署Dashboard UI
```sh
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
* 访问Dashboard UI （未实现，远程机器权限问题还在研究）