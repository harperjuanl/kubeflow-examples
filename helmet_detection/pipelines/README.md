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

### Containernize Pipeline Components¶
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

### Compile Data Ingest Component
First, we need to create and specify the persistent volume (PVC) for data storage, creating a VolumeOP instance.

```bash
    vop = dsl.VolumeOp(name="create_helmet_data_storage_volume",
                        resource_name="helmet_data_storage_volume", size='10Gi', 
                        modes=dsl.VOLUME_MODE_RWO)
```
We then create a ContainerOp instance, which would be understood and used as "a step" in our pipeline, and return this "step".
```bash
    return dsl.ContainerOp(
        name = \'Download Data\', 
        image = \'harbor-repo.vmware.com/juanl/helmet_pipeline:v1\',
        command = [\'python3\', \'ingest_pipeline.py\'],
        arguments=[
            \'--dataurl\', dataurl,
            \'--datapath\', datapath
        ],
        pvolumes={
            \'/VOCdevkit\': vop.volume
        }
    )
```

We need to specify the inputs `dataurl`, `datapath` in arguments, container image in image, and volume for data storage in pvolumes. Note that here in image, we provide you with our built images, containing both train folder and evaluate folder, stored on our projects.registry repo. If you want to use your own image, please remember to change this value.

We also need to specify command. In this provided case, as we containernize the image at root directory, in command we need python3 ingest_pipeline.py. (If you containernize Train component and Evaluate component one by one in each own folder, you may need to change this value to python3 ingest_pipeline.py.)

### Declare Data Processing Component

```bash
def data_process(comp1):
    return dsl.ContainerOp(
        name = 'Process Data', 
        image = 'harbor-repo.vmware.com/juanl/helmet_ingest_data:v1',
        command = ['python3', 'prepare.py'],
        pvolumes={
            '/VOCdevkit': comp1.pvolumes['/VOCdevkit']
        }
    )
```

### Declare Model Training Component

```bash
def model_train(comp2, epoch, device, workers_num, model_export):
    return dsl.ContainerOp(
        name = 'Model Training',
        image = 'harpersweet/helmet_pipeline:v2',
        pvolumes={
            '/VOCdevkit': comp2.pvolumes['/VOCdevkit']
        },
        # command=['sh', '-c'],
        # arguments=['nvidia-smi'],
        command = ['python3', 'train_pipeline.py'],
        arguments=[
            '--epoch', epoch,
            '--device', device,
            '--workers', workers_num,
            '--output_dir', model_export
        ],
    ).set_gpu_limit(1).set_cpu_request('2').set_memory_request('8G')
```

### Compile pipeline

Execute below function to compile the YAML file:

```bash
def model_train(comp2, epoch, device, workers_num, model_export):
    return dsl.ContainerOp(
        name = 'Model Training',
        image = 'harpersweet/helmet_pipeline:v2',
        pvolumes={
            '/VOCdevkit': comp2.pvolumes['/VOCdevkit']
        },
        # command=['sh', '-c'],
        # arguments=['nvidia-smi'],
        command = ['python3', 'train_pipeline.py'],
        arguments=[
            '--epoch', epoch,
            '--device', device,
            '--workers', workers_num,
            '--output_dir', model_export
        ],
    ).set_gpu_limit(1).set_cpu_request('2').set_memory_request('8G')
```

## Execute the Pipeline

In the example, we compiled the pipeline as a YAML file. So here we provide you with a brief guide on how to run a pipeline.

### Upload the pipeline to Kubeflow UI 

Following our notebook, you should be able to see a file called helmet_pipeline_demo.yaml. 
we provide you with a already-compiled pipeline YAML files for quick-test purpose. If you prefer that, feel free to skip to pipeline running part and use them.
Upload the yaml file to Pipelines on Kubeflow UI.

![Image text](https://github.com/harperjuanl/kubeflow-examples/blob/main/helmet_detection/pipelines/imgs/helmet-pipeline-01.png)
![Image text](https://github.com/harperjuanl/kubeflow-examples/blob/main/helmet_detection/pipelines/imgs/helmet-pipeline-02.png)

### Create experiment and run

Create an experiment for this pipeline, and create a run. This time, you need to provide two inputs, dataset and data_path, exactly the ones for our first step Data Download. If you do not intend to make any personalization on datasets and data path, enter following values

![Image text](https://github.com/harperjuanl/kubeflow-examples/blob/main/helmet_detection/pipelines/imgs/helmet-pipeline-03.png)
![Image text](https://github.com/harperjuanl/kubeflow-examples/blob/main/helmet_detection/pipelines/imgs/helmet-pipeline-04.png)

### Check logs and outputs 
![Image text](https://github.com/harperjuanl/kubeflow-examples/blob/main/helmet_detection/pipelines/imgs/helmet-pipeline-05.png)