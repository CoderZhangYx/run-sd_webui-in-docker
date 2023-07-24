# stabel-diffusion-webui的docker环境搭建指北
1.克隆代码
```
git clone https://ghproxy.com/https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui
```

2.创建Dockerfile
```
mkdir docker
cd docker
touch Dockerfile
```
打开Dockerfile粘贴如下
```
# pull docker imgage with specified cuda version
FROM nvidia/cuda:11.6.2-cudnn8-devel-ubuntu18.04
ENV TORCH_CUDA_ARCH_LIST="6.0 6.1 7.0 7.5 8.0 8.6+PTX" \
	TORCH_NVCC_FLAGS="-Xfatbin -compress-all" \
	CMAKE_PREFIX_PATH="$(dirname $(which conda))/../" \
	FORCE_CUDA="1"

# nvidia key settings
RUN rm /etc/apt/sources.list.d/cuda.list && \
	apt-key del 7fa2af80 && \
	apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/3bf863cc.pub && \
	apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/7fa2af80.pub

# (Optional)specify apt source mirror
RUN sed -i 's/http:\/\/archive.ubuntu.com\/ubuntu\//http:\/\/mirrors.aliyun.com\/ubuntu\//g' /etc/apt/sources.list

# Install the required packages
RUN apt-get update
RUN apt-get install -y ffmpeg libsm6 libxext6 git ninja-build libglib2.0-0 libsm6 libxrender-dev libxext6 
RUN apt-get install -y vim tmux wget

# (Optional)install support for chinese language 
# RUN apt-get install -y language-pack-zh-hans
# RUN locale-gen zh_CN.UTF-8 && echo "export LC_ALL=zh_CN.UTF-8">> /etc/profile && /bin/bash -c "source /etc/profile"

RUN wget -c https://repo.anaconda.com/miniconda/Miniconda3-py310_23.1.0-1-Linux-x86_64.sh -O miniconda.sh
RUN bash miniconda.sh -b
RUN rm miniconda.sh
RUN /root/miniconda3/bin/conda init
RUN /bin/bash -c "source ~/.bashrc"
RUN /root/miniconda3/bin/pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

CMD source ~/.bashrc
```

3.创建镜像
```
docker build -t webui:v0 .
```

4.创建容器并进入
```
docker run -it --name webui --gpus all -v /path/to/your/code/:/workspace webui:v0 bash
cd /workspace
```

8.构建虚拟环境
```
cd stable-diffusion-webui
python -m venv venv
source venv/bin/activate
```

9.先手动装torch，不然大概率报错
```
pip install torch==1.13.1+cu116 torchvision==0.14.1+cu116 torchaudio==0.13.1 --extra-index-url https://download.pytorch.org/whl/cu116
```

10.手动安装cython
```
pip install cython
```

11.安装其余部分  
**(Optional)** 将launch.py中的所有 *https://github* 替换为 *https://ghproxy.com/https://github*   
**将webui.sh中can_run_as_root改为1**
```
bash webui.sh
```


