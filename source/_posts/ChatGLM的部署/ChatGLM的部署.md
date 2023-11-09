title: ChatGLM的部署
date: 2023-11-9 8:54:46
tags:
---

## 创建任务
先大致看看我的云脑配置
![[Pasted image 20231028030935.png]]
```
镜像：mindspore2.0.0_cann6.3_notebook
计算：NPU（Ascend 910）
```

## 拉取Mindspore大模型的代码仓

用这个拉取可以避免GitHub拉取不成功的问题
`git clone https://openi.pcl.ac.cn/mindspore-courses/step_into_chatgpt.git`

## 安装Mindformers

在终端输入
```
git clone -b dev https://gitee.com/mindspore/mindformers.git
cd mindformers
bash build.sh
```

这是Mindformers官方的仓库：`https://github.com/mindspore-lab/mindformers.git`

运行的好好的，以下是正在运行的代码![[Pasted image 20231028010258.png]]

但是突然间，他提示我**uninstall**？我瞬间懵了，让我看看报错![[Pasted image 20231028010511.png]]
```
报错信息：
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
mindinsight 1.8.0 requires Flask>=2.1.0, but you have flask 2.0.1 which is incompatible.

ERROR： pip 的依赖关系解析器当前并未考虑所有已安装的软件包。这种行为导致了以下依赖冲突。
mindinsight 1.8.0 需要 Flask>=2.1.0，但您的 flask 2.0.1 与之不兼容。
```

这是Flask的版本问题，我尝试安装falsk（2.1.0），于是我输入
`pip install flask==2.1.0`
好的，出来一个新的报错![[Pasted image 20231028011033.png]]
```
报错信息：
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
modelarts-mindspore-model-server 1.0.4 requires Flask==2.0.1, but you have flask 2.1.0 which is incompatible.
modelarts-mindspore-model-server 1.0.4 requires Jinja2==3.0.1, but you have jinja2 3.1.2 which is incompatible.

ERROR： pip 的依赖关系解析器当前并未考虑所有已安装的软件包。这种行为导致了以下依赖冲突。
modelarts-mindspore-model-server 1.0.4 需要 Flask===2.0.1，但你的 flask 2.1.0 与之不兼容。
modelarts-mindspore-model-server 1.0.4 需要 Jinja2===3.0.1，但您的 jinja2 3.1.2 不兼容。
```
可以看出，还是包的问题，然后我再调试将flask调回2.0.1，将jinja2降为3.0.1，然后他告诉我跟Jupyter有冲突，那没事啊，这不是代码运行的环境，所以我又将mindformers的源代码编译了一遍，结果
![[Pasted image 20231028012300.png]]
```
mindformers is already installed with the same version as the provided wheel. Use --force-reinstall to force an installation of the wheel.
```

他告诉我mindformers已经安装过了！但是至少没有**ERROR**了吗。

## 下载ckpt和tokenizer文件

先进入我们要运行的源代码的文件夹：
`cd step_into_llm/Season2.step_into_llm/01.ChatGLM`

用wget安装：
```
安装ckpt
wget https://ascend-repo-modelzoo.obs.cn-east-2.myhuaweicloud.com/XFormer_for_mindspore/glm/glm_6b.ckpt

安装tokenizer(时间短):
wget https://ascend-repo-modelzoo.obs.cn-east-2.myhuaweicloud.com/XFormer_for_mindspore/glm/ice_text.model
```

安装tokenizer(时间短):![[Pasted image 20231028013508.png]]

安装ckpt：![[Pasted image 20231028013540.png]]
安装成功的截图
![[Pasted image 20231028022001.png]]

## 运行cli_demo_glm.py

运行 cli_demo_glm.py 文件
`python cli_demo_glm.py`

运行中戛然而止，出现以下报错![[Pasted image 20231028023149.png]]
```
[ERROR] PIPELINE(804,ffff8b57cac0,python):2023-10-28-02:21:07.048.981 [mindspore/ccsrc/pipeline/jit/init.cc:466] operator()] Failed to parse profiler data.AttributeError: module 'cv2' has no attribute 'gapi_wip_gst_GStreamerPipeline'
```

告诉我cv2 "模块没有 "gapi_wip_gst_GStreamerPipeline "属性，网上冲浪！然后找到了解决办法：
```
pip install opencv-python install "opencv-python-headless<4.3"
```

但是这一关过了，有出现一个新的报错！![[Pasted image 20231028023837.png]]
```
RuntimeError: Unsupported device target GPU. This process only supports one of the ['Ascend', 'CPU']. Please check whether the GPU environment is installed and configured correctly, and check whether current mindspore wheel package was built with "-e GPU". For details, please refer to "Device load error message".

不支持设备目标 GPU。该进程只支持['Ascend', 'CPU']中的一种。请检查 GPU 环境是否正确安装和配置，并检查当前的 mindspore wheel 包是否使用"-e GPU "构建。详情请参阅 "设备加载错误信息"。
```

什么，你怎么使用GPU，我在构建的时候采用的是NPU，于是我查看cli_demo_glm.py文件（当然一开始我肯定是将报错复制然后网上冲浪，得到的结论是可能版本不匹配，还有是没有安装cuDNN，但是当我打开cuDNN的时候，他是英伟达的，但我这是昇腾，肯定不是没有安装cuDNN，版本也是没有问题，所以我最后想到去看看我需要编译的源码）![[Pasted image 20231028025839.png]]

这里一开始是GPU，我给他改成NPU，不行，他告诉我没有这个选项![[Pasted image 20231028030015.png]]
```
ValueError: For 'context.set_context', the argument 'device_target' must be one of ['CPU', 'GPU', 'Ascend', 'Davinci'], but got NPU.
```

然后我改成Ascend，就可以运行了！![[Pasted image 20231028030123.png]]

但是他为什么不回我的话？就像上图一样，然后我写完我的疑问，回去一看：![[Pasted image 20231028030303.png]]

那我问问![[Pasted image 20231028030412.png]]

可以！我再试试问他其他东西![[Pasted image 20231028030613.png]]

代码也可以帮写，可以的，那到这里ChatGLM的部署就到这里完结了！感谢观看
![[Pasted image 20231028031216.png]]