_Date:2018/02/27_<br/>
_Author:Jiamiao.Xie(xiejiamiao0326@hotmail.com)_

***

### 1.备份模型
``` shell
cp -r dishIngredient_checkpoint_inception_resnet_v2 models_old/dishIngredient_checkpoint_inception_resnet_v2
cp -r dish_classes1000_checkpoint_inception_v3/ models_old/dish_classes1000_checkpoint_inception_v3/
cd models_old
zip -r deploy_models_0227_back.zip dishIngredient_checkpoint_inception_resnet_v2 dish_classes1000_checkpoint_inception_v3
rm -rf dishIngredient_checkpoint_inception_resnet_v2
rm -rf dish_classes1000_checkpoint_inception_v3
```

### 2.删除旧模型
``` shell
cd ../
rm -rf dishIngredient_checkpoint_inception_resnet_v2
rm -rf dish_classes1000_checkpoint_inception_v3
```

### 3.解压新模型
``` shell
unzip dish_models.zip
```

### 4.检查代码配置

检查 web_image_classifiler_600*.py 里的NetWork里面的模型文件/label文件目录是否正确，如果错误则修改，正确则继续

### 5.杀死进程
查询端口占用的PID(6006代表端口)
``` shell
lsof -i:6006
```
关闭进程(8989代表查询到的PID)
```shell
kill -9 8989
```

### 6.重启服务
```shell
screen -L -t webapi1
source /etc/profile
CUDA_VISIBLE_DEVICES='0' python web_image_classifiler_6006.py
```
待启动成功，用node测试通过之后，按 ctrl + a +d 进行页面


### 7.备注
> 共有 6005、6006、6007、6008、6009 五个端口需要进行维护
其中5、8、9跑在CPU上，无需传参数，即：CUDA_VISIBLE_DEVICES=''
其中6、7跑在GPU上，需要传参数，即：CUDA_VISIBLE_DEVICES='0'或CUDA_VISIBLE_DEVICES='1'