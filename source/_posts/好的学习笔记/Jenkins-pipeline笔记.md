---
title: 笔记-Jenkins
categories: 好的学习笔记
tags: Jenkins
---

* Jenkins pipeline 语法
* Jenkins 应用



# Jenkins pipeline 语法

# 一些问题

【问题】Jenkins pipeline 和 groovy 的关系？

![image-20230526115403123](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526115403123.png)



【问题】也就是说 Jenkinsfile 其实是 Groovy 文件？

![image-20230526115444975](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526115444975.png)



【问题】那 Jenkinsfile 和普通的 Groovy 文件有什么区别呢，它是增加了什么，它增加的这些东西是怎么兼容Groovy语法的？



![image-20230526115558478](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526115558478.png)



【问题】我没有在 Jenkinsfile 中看到任何引入，pipeline、stage、steps、agent 这些关键字是怎么被引入的？



![image-20230526115741124](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526115741124.png)



![image-20230526115833707](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526115833707.png)



【问题】jenkins 使用 bash 脚本和使用 pipeline 脚本的比较

![image-20230529220737533](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529220737533.png)

# Jenkins 语法

### pipeline 

所有有效的声明式流水线必须包含在一个 `pipeline` 块中，如下：

![image-20230526133735362](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526133735362.png)



### agent

该部分必须在 `pipeline` 块的顶层被定义。用于指定 Jenkins Pipeline 的运行环境或代理节点。如下，配置了一个 Kubernetes 代理节点来执行 Pipeline：

![image-20230526134352805](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526134352805.png)



### parameters

在 Jenkins Pipeline 中的作用是允许用户输入和配置参数，以实现灵活性、可配置性、自动化流程控制和交互性。

![image-20230526134656564](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526134656564.png)



### stages

定义和组织 Pipeline 的多个阶段，实现流水线建模、可视化、并行执行、异常处理和可重用性。

![image-20230526140144528](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526140144528.png)





### steps

它是 Pipeline 脚本的核心部分，决定了 Pipeline 的具体执行逻辑和行为。通过在 `steps` 块中编写和组织任务，可以构建出灵活、可扩展的流水线，用于实现持续集成和持续交付的自动化流程。



![image-20230526140613483](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526140613483.png)

![image-20230526140547879](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526140547879.png)



### containers

 用于定义在构建过程中要使用的容器环境。它通常与 `agent` 配置一起使用，以指定构建过程中要运行的容器



![image-20230526140516983](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526140516983.png)



### script

执行自定义的 Groovy 脚本代码，用于实现灵活的逻辑处理和定制化的操作



![image-20230526140443752](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526140443752.png)

![image-20230526140413028](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230526140413028.png)



# Jenkins 全局变量获取

### 一些问题

【问题】Jenkins Pipeline 还有哪些全局变量？

![image-20230529144619912](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529144619912.png)



# Jenkins 应用

关键的就两步：

* loadJson
* runSteps

如下所示。

![image-20230529204012240](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529204012240.png)



# LoadJson

1. 获取环境变量
2. 获取 json 文件路径
3. 根据  json  文件创建对应的 StepFactory 对象，并且获取根据环境变量赋值 json 文件中的变量值



![image-20230529163530838](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529163530838.png)



### 一、获取环境变量 getEnvironment

如下图所示，【getEnvironment】方法需要获取 【env、build、Cause、workspace】等 Jenkins 全局变量。

![image-20230529144841121](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529144841121.png)



### 二、获取 json 文件路径

略.....................



### 三、根据  json  文件创建对应的 StepFactory 对象（重要）

1. 根据 Json 文件路径设置项目目录路径，PROJECT_PATH
2. 根据 Json 文件路径设置项目目录，PROJECT_DIR
3. 获取运行时变量的名称和值并存入环境变量中
4. 整体加载 json 配置文档
5. 初始化构建需要的对象



如下所示，【readJSON】是读取路径下的 json 文件的内容。json 表示的是这个文件的路径。

![image-20230529164323199](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529164323199.png)



【根据 Json 文件路径设置项目目录路径】，项目目录路径就是当前 json 文件的路径，如下所示，每个项目 module 的根目录下都有自己的 jenkins-project.json。

![image-20230529165422600](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529165422600.png)



![image-20230529165727494](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529165727494.png)



【根据 Json 文件路径设置项目目录】，项目目录名称可以通过截取项目目录路径的最后一部分获得，前面一部分其实就是 jenkins workspace 的路径，将该部分替换为空就可以 了，如下所示：



![image-20230529170238543](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529170238543.png)



【获取运行时变量的名称和值并存入环境变量中】，在 json 文件中定义了一个模块名称为 【运行时变量】，需要将这些变量设置到 jenkins 的环境变量中，以便后续可以直接使用。如下所示，【RuntimeVariable】中定义了 【PROJECT_VERSION】这个变量，并且后续是以获取环境变量的方式获取该值。



![image-20230529170637940](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529170637940.png)



![image-20230529170746481](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529170746481.png)



但是需要注意的是，这个变量里面有引用其它环境变量的值，所以需要先补充完整，再将这个变量存入环境变量。如下所示：



![image-20230529190411937](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529190411937.png)





【整体加载 json 配置文档】，前面已经将所有变量都设置为环境变量，接下来就是将 json 文件中的环境变量替换为真实的值。

![image-20230529190636955](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529190636955.png)



【初始化构建需要的对象】，将完整的 json 文件解析为 Step 对象。 

![image-20230529191035109](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529191035109.png)



# RunSteps

### runStep

通过【loadJson】后，json 文件的所有参数已经补充完整，而且已经将文件解析为 StepFactory 对象。接下来就是执行 StepFactory 对象里的 【Script】 里的命令。通过 jenkins pipeline 的【runStdoutScript】执行脚本并获取其标准输出，然后将输出与 StepFactory 对象里的【Success-IndexOf】比较，看是否符合。



![image-20230529204338298](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529204338298.png)



![image-20230529204858981](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529204858981.png)



### Json 文件脚本解释

``` yml
{
    "部署":
    {
        "基于基础镜像进行构建":
        {
            "Type": "COMMAND_STATUS",
            "Script":
            {
                "创建configmap": "cd ${PROJECT_PATH};sed -i 's/^/    /g' src/main/resources/application.yml;cat ../build_script/k8s-configmap-template.yaml src/main/resources/application.yml > ./configmap/${PROJECT_DIR}.yaml;",
                "替换configmap配置": "cd ${PROJECT_PATH};sed -i 's/deploy-pod-name/${PROJECT_DIR}/g' ./configmap/${PROJECT_DIR}.yaml",
                "替换配置文件": "cd ${PROJECT_PATH}; sed -i 's/enabled: false/enabled: true/g' src/main/resources/bootstrap.yml;sed -i 's/deploy-pod-name/${PROJECT_DIR}/g' src/main/resources/bootstrap.yml;sed -i 's/deploy-name-space/${JOB_NAME}/g' src/main/resources/bootstrap.yml",
                "复制文件": "cd ${PROJECT_PATH}; cp -rf ../build_script  ./build_script/",
                "替换images配置": "cd ${PROJECT_PATH};sed -i 's/deploy-pod-name/${PROJECT_DIR}/g' build_script/Dockerfile-thin-cm",
                "替换镜像版本": "cd ${PROJECT_PATH};export BASEVERSION=$(md5sum pom.xml|cut -d\" \" -f1);sed -i \"s/IMAGESVERSION/jar-base\\/${JOB_NAME}\\/${PROJECT_DIR}:$BASEVERSION/g\" build_script/Dockerfile-thin-cm;",
                "构建镜像": "cd ${PROJECT_PATH}; docker build -t repo.gydev.cn:8082/${JOB_NAME}/${PROJECT_DIR}:${PROJECT_VERSION} -f build_script/Dockerfile-thin-cm .;docker push repo.gydev.cn:8082/${JOB_NAME}/${PROJECT_DIR}:${PROJECT_VERSION};docker rmi repo.gydev.cn:8082/${JOB_NAME}/${PROJECT_DIR}:${PROJECT_VERSION}",
                "切换POD用户": "if [ ${build_outer_net} == 'true' ]; then echo '构建外网，不执行此命令'; else kubectl config use-context jenkins@default-cluster; fi",
                "更新configmap": "if [ ${build_outer_net} == 'true' ]; then echo '构建外网，不执行此命令'; else cd ${PROJECT_PATH};sed -i 's/deploy-name-space/${JOB_NAME}/g' configmap/${PROJECT_DIR}.yaml;kubectl apply -f configmap/${PROJECT_DIR}.yaml; fi",
                "更新k8s里的镜像": "if [ ${build_outer_net} == 'true' ]; then echo '构建外网，不执行此命令'; else cd ${PROJECT_PATH};sed -i 's/image-version/${PROJECT_VERSION}/g' build_script/k8s-script-common-cm-probe.yaml;sed -i 's/deploy-pod-name/${PROJECT_DIR}/g' build_script/k8s-script-common-cm-probe.yaml;sed -i 's/deploy-name-space/${JOB_NAME}/g' build_script/k8s-script-common-cm-probe.yaml;kubectl apply -f build_script/k8s-script-common-cm-probe.yaml; fi"
            }
        },
        "构建基础镜像":
        {
            "Type": "COMMAND_STATUS",
            "Script":
            {
                "复制dockfile": "cd ${PROJECT_PATH}; cp -rf ../build_script  ./build_script/",
                "下载jar包": "cd ${PROJECT_PATH};java -Dthin.dryrun=true -Dthin.root=m2 -jar target/*.jar;ls m2;",
                "生成基础镜像": "cd ${PROJECT_PATH};docker build -f build_script/Dockerfile-base -t repo.gydev.cn:8082/jar-base/${JOB_NAME}/${PROJECT_DIR}:$(md5sum pom.xml|cut -d\" \" -f1) .;docker push repo.gydev.cn:8082/jar-base/${JOB_NAME}/${PROJECT_DIR}:$(md5sum pom.xml|cut -d\" \" -f1);"
            }
        }
    }
}
```



#### 1、创建 configmap 文件

``` bash
cd ${PROJECT_PATH};
sed -i 's/^/    /g' src/main/resources/application.yml;
cat ../build_script/k8s-configmap-template.yaml src/main/resources/application.yml > ./configmap/${PROJECT_DIR}.yaml;
```



将 【application.yml】追加到【k8s-configmap-template.yaml】文件后，输出到新的 configmap 文件中。

![image-20230601182146291](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230601182146291.png)



#### 2、替换 configmap 配置文件

```bash
cd ${PROJECT_PATH};
sed -i 's/deploy-pod-name/${PROJECT_DIR}/g' ./configmap/${PROJECT_DIR}.yaml;
```



前面生成的 configmap 文件中存在一些环境变量，需要替换获得完整的 configmap 文件，如下所示。

![image-20230601182446229](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230601182446229.png)



#### 3、替换 bootstrap 配置文件

```bash
cd ${PROJECT_PATH};
sed -i 's/enabled: false/enabled: true/g' src/main/resources/bootstrap.yml;
sed -i 's/deploy-pod-name/${PROJECT_DIR}/g' src/main/resources/bootstrap.yml;
sed -i 's/deploy-name-space/${JOB_NAME}/g' src/main/resources/bootstrap.yml;
```



SpringCloud 的 bootstrap  文件中存在一些环境变量，需要替换获得完整的 bootstrap 文件。如下所示。

![image-20230601182543644](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230601182543644.png)



#### 4、复制脚本文件

```bash
cd ${PROJECT_PATH};
cp -rf ../build_script  ./build_script/;
```



复制父项目中的通用构建脚本文件到当前项目中。每个项目都需要自己的构建脚本文件。



#### 5、替换 images 配置

```bash
cd ${PROJECT_PATH};
sed -i 's/deploy-pod-name/${PROJECT_DIR}/g' build_script/Dockerfile-thin-cm;
```



前面复制了脚本文件到当前项目中，需要替换其中的【Dockerfile-thin-cm】文件的环境变量值。

![image-20230530172245171](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230530172245171.png)



#### 6、替换镜像版本

``` bash
cd ${PROJECT_PATH}
export BASEVERSION=$(md5sum pom.xml | cut -d\" \" -f1)
sed -i \"s/IMAGESVERSION/jar-base\\/${JOB_NAME}\\/${PROJECT_DIR}:$BASEVERSION/g\" build_script/Dockerfile-thin-cm
```



![image-20230530172532880](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230530172532880.png)



#### 7、构建镜像

```bash
cd ${PROJECT_PATH}
docker build -t repo.gydev.cn:8082/${JOB_NAME}/${PROJECT_DIR}:${PROJECT_VERSION} -f build_script/Dockerfile-thin-cm .
docker push repo.gydev.cn:8082/${JOB_NAME}/${PROJECT_DIR}:${PROJECT_VERSION}
docker rmi repo.gydev.cn:8082/${JOB_NAME}/${PROJECT_DIR}:${PROJECT_VERSION}
```



前面已经获得完整的 Dockerfile 文件，可以进行镜像构建了。



#### 8、切换POD用户

``` bash
if [ ${build_outer_net} == 'true' ]; then
    echo '构建外网，不执行此命令'
else
    kubectl config use-context jenkins@default-cluster
fi
```



#### 9、更新 configmap

``` bash
if [ ${build_outer_net} == 'true' ]; then
    echo '构建外网，不执行此命令'
else
    cd ${PROJECT_PATH}
    sed -i 's/deploy-name-space/${JOB_NAME}/g' configmap/${PROJECT_DIR}.yaml
    kubectl apply -f configmap/${PROJECT_DIR}.yaml
fi
```



【configmap】文件中还有 deploy-name-space 这个环境变量没有替换，这这里替换之后执行该文件。

【问题】问什么不在前面第二步替换 configmap 配置文件出就把全部环境变量替换呢？



#### 10、更新 k8s 里的镜像

```bash
if [ ${build_outer_net} == 'true' ]; then
    echo '构建外网，不执行此命令'
else
    cd ${PROJECT_PATH}
    sed -i 's/image-version/${PROJECT_VERSION}/g' build_script/k8s-script-common-cm-probe.yaml
    sed -i 's/deploy-pod-name/${PROJECT_DIR}/g' build_script/k8s-script-common-cm-probe.yaml
    sed -i 's/deploy-name-space/${JOB_NAME}/g' build_script/k8s-script-common-cm-probe.yaml
    kubectl apply -f build_script/k8s-script-common-cm-probe.yaml
fi
```



当前项目中的【k8s-script-common-cm-probe.yaml】中存在环境变量值，需要替换得到完整文件。然后执行。如下图所示，该文件定义了 service 和 deployment。【probe】是探针的意思。

![image-20230530175705493](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230530175705493.png)



【问题】Deployment 创建 pod 是在 service 之后创建的，那 service 的 selector 是怎么能匹配得到还没创建的 pod？

![image-20230530185238834](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230530185238834.png)



# Jenkins 的流水线配置

如下，Jenkins 中流水线配置的项目是【cos-power-csms】，然后脚本文件是项目根路径下的 【Jenkinsfile】。但是我们在项目中的 Jenkinsfile 可以看到，它调用了方法【k8sCluster】，但是这个方法不在【cos-power-csms】这个项目中，那它是怎么调用到的呢？

其实是因为 Jenkins 的全局配置中配置了 【Global Pipeline Libraries】，这就表示里面的项目是【共享库】，通过注解【@Library('shared-library')】Jenkins 可以调用共享库中的 `k8sCluster.groovy` 实现，而无需在当前代码片段中显式引入该文件或函数。

![image-20230529215506183](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529215506183.png)



![image-20230529215524968](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529215524968.png)



![image-20230529220040020](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529220040020.png)



![image-20230529215417950](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230529215417950.png)