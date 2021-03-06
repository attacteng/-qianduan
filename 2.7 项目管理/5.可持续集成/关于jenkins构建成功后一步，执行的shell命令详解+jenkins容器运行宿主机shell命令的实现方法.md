- [关于jenkins构建成功后一步，执行的shell命令详解+jenkins容器运行宿主机shell命令的实现方法](https://www.cnblogs.com/sxdcgaq8080/p/10599166.html)

## 1.展示这段shell命令 +详解

```bash
#=====================================================================================
#=================================定义初始化变量======================================
#=====================================================================================

#操作/项目路径(Dockerfile存放的路径)
BASE_PATH=/apps/swapping


# jenkins构建好的源jar路径，jenkins的workspace下，jenkins服务内地址为：/var/jenkins_home/workspace
#因为docker启动的jenkins，目录进行了宿主机的目录挂载，则使用宿主机目录：  /apps/Devops/jenkins/workspace
#完整地址应为：/apps/Devops/jenkins/workspace/项目名称/target/  后面会进行拼接
SOURCE_PATH=/apps/Devops/jenkins/workspace


#【docker 镜像】【docker容器】【Dockerfile同目录下的jar名字[用它build生成image的jar]】【jenkins的workspace下的项目名称】
#这里都以这个命名[微服务的话，每个服务都以ms-swapping这种格式命名]
#注意统一名称！！！！！
SERVER_NAME=swapping

#容器id  [grep -w 全量匹配容器名] [awk 获取信息行的第一列，即容器ID]  [无论容器启动与否，都获取到]
CID=$(docker ps -a | grep -w "$SERVER_NAME" | awk '{print $1}')

#镜像id  [grep -w 全量匹配镜像名] [awk 获取信息行的第三列，即镜像ID]
IID=$(docker images | grep -w "$SERVER_NAME" | awk '{print $3}')


#源jar完整地址  [jenkins构建成功后，会在自己的workspace/项目/target 下生成maven构建成功的jar包，获取jar包名的完整路径]
#例如：/apps/Devops/jenkins/workspace/swapping/target/swapping-0.0.1-SNAPSHOT.jar
SOURCE_JAR_PATH=$(find "$SOURCE_PATH/$SERVER_NAME/target/"  -name "*$SERVER_NAME*.jar" )

DATE=`date +%Y%m%d%H%M%S`


#=====================================================================================
#============================对原本已存在的jar进行备份================================
#=====================================================================================



# 备份
function backup(){
    if [ -f "$BASE_PATH/$SERVER_NAME.jar" ]; then
        echo "=========================>>>>>>>$SERVER_NAME.jar 备份..."
            mv $BASE_PATH/$SERVER_NAME.jar $BASE_PATH/backup/$SERVER_NAME-$DATE.jar
        echo "=========================>>>>>>>备份老的 $SERVER_NAME.jar 完成"

    else
        echo "=========================>>>>>>>老的$BASE_PATH/$SERVER_NAME.jar不存在，跳过备份"
    fi
}



#=====================================================================================
#=========================移动最新源jar包到Dockerfile所在目录=========================
#=====================================================================================


 
# 查找源jar文件名，进行重命名，最后将源文件移动到Dockerfile文件所在目录
function transfer(){
       
         
    echo "=========================>>>>>>>源文件完整地址为 $SOURCE_JAR_PATH"

        
    echo "=========================>>>>>>>重命名源文件"
        mv $SOURCE_JAR_PATH  $SOURCE_PATH/$SERVER_NAME/target/$SERVER_NAME.jar

    echo "=========================>>>>>>>最新构建代码 $SOURCE_PATH/$SERVER_NAME/target/$SERVER_NAME.jar 迁移至 $BASE_PATH"
        cp $SOURCE_PATH/$SERVER_NAME/target/$SERVER_NAME.jar $BASE_PATH 

    echo "=========================>>>>>>>迁移完成Success"

}
 


#=====================================================================================
#==================================构建最新镜像=======================================
#=====================================================================================


 
# 构建docker镜像
function build(){
    
    #无论镜像存在与否，都停止原容器服务，并移除原容器服务
    echo "=========================>>>>>>>停止$SERVER_NAME容器，CID=$CID"
    docker stop $CID

    echo "=========================>>>>>>>移除$SERVER_NAME容器，CID=$CID"
    docker rm $CID

    #无论如何，都去构建新的镜像
    #构建新的镜像之前，移除旧的镜像
    if [ -n "$IID" ]; then
        echo "=========================>>>>>>>存在$SERVER_NAME镜像，IID=$IID"


        echo "=========================>>>>>>>移除老的$SERVER_NAME镜像，IID=$IID"
        docker rmi $IID

        echo "=========================>>>>>>>构建新的$SERVER_NAME镜像，开始---->"
        cd $BASE_PATH
        docker build -t $SERVER_NAME .
        echo "=========================>>>>>>>构建新的$SERVER_NAME镜像，完成---->"

    else
        echo "=========================>>>>>>>不存在$SERVER_NAME镜像，构建新的镜像，开始--->"


        cd $BASE_PATH
        docker build -t $SERVER_NAME .
        echo "=========================>>>>>>>构建新的$SERVER_NAME镜像，结束--->"
    fi
}
 

#=====================================================================================
#==============================运行docker容器，启动服务===============================
#=====================================================================================




# 运行docker容器
# 先备份老的jar包
# 再移动新的jar包到Dockerfile文件所在目录
# 接着，构建新的镜像
# 最后运行最新容器，启动服务
function run(){
    backup
    transfer
    build

    docker run --name $SERVER_NAME -itd --net=host -v /etc/localtime:/etc/localtime:ro  -v /etc/timezone:/etc/timezone:ro  $SERVER_NAME 

}
 
#入口
run
```

## 2.声明

```
这段shell脚本是在宿主机/服务器上运行，不是在jenkins容器内运行的
```

## 3.实现方式

```
实现jenkins容器可以运行宿主机上的shell命令，是通过给jenkins安装Push Over SSH插件来完成的
```

具体步骤参见：https://www.cnblogs.com/sxdcgaq8080/p/10489369.html最后一点！！