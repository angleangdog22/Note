### pod应用



```#
## k8s管理应用的最小单位
docker ——————> container(image)   

k8s ----> pod (images)


# docker  
docker run -it  --name test01 <image_name> /bin/bash 

# k8s 
kubectl run <name> --image=<imagename>  --port <port_number>


## 一般是apply -f 启动yaml 来生成pod
#### --dry-run 尝试运行，但不运行
### -oyaml 以yaml格式输出

生成模板 ## 有模板生成模板，没模板用explain查询创建
kubectl run   test01 --image nginx:latest --dry-run -oyaml >> test.yaml 

```

