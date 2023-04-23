# Helmet Detection Model Deployment with Kubeflow Pipeline

## Instruction

Kubeflow Pipelines is a platform for building and deploying portable, scalable machine learning (ML) workflows based on Docker containers. 
Each pipeline represents an ML workflow, and includes the specifications of all inputs needed to run the pipeline, as well the outputs of all components.
If you are not familar with the Kubeflow Pipeline, you can refer to [kubeflow_pipelines](https://elements-of-ai.github.io/kubeflow-docs/user-guide/kfp.html).

In this tutorial, we would guide you through the Helmet Detction example (mentioned as the [helmet-notebook](https://github.com/harperjuanl/kubeflow-examples/tree/main/helmet_detection/notebook)) to build up and run Kubeflow pipelines


## Build Pipeline

### Import Pipeline Packages

First, install the Pipeline SDK using the following command. If you run this command in a Jupyter notebook, restart the kernel after installing the SDK.

```bash 
$ pip install kfp --upgrade --user --quiet
$ pip show kfp
```

```bash
import kfp
import kfp.components as comp
import kfp.dsl as dsl
from kfp.components import OutputPath
from typing import NamedTuple
from kubernetes import client
```

### Containernize Pipeline Components
We use Docker to build images. Basically, Docker can build images automatically by reading the instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image.

In this example, we provide you with following Dockerfile for Train component and Evaluate component.

```bash 
    FROM ubuntu:20.04

    # Downloads to user config dir
    ADD https://ultralytics.com/assets/Arial.ttf https://ultralytics.com/assets/Arial.Unicode.ttf /root/.config/Ultralytics/

    # Install linux packages
    RUN apt update
    RUN DEBIAN_FRONTEND=noninteractive TZ=Etc/UTC apt install -y tzdata
    RUN apt install --no-install-recommends -y python3-pip git zip curl htop libgl1-mesa-glx libglib2.0-0 libpython3.8-dev
    # RUN alias python=python3

    # Install pip packages
    COPY requirements.txt .
    RUN python3 -m pip install --upgrade pip wheel
    RUN pip install -r requirements.txt -i https://pypi.douban.com/simple/

    COPY . /
```