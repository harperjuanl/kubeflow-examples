## Helmet Detection Deployment on with Kubeflow KServe

### Intruction

The InferenceService custom resource is the primary interface that is used for deploying models on KServe. Inside an InferenceService, users can specify multiple components that are used for handling inference requests. These components are the predictor, transformer, and explainer. 
For more detailed documentation on Kubeflow KServe, refer to `KServe <https://kserve.github.io/website/0.7/modelserving/data_plane/>`.



### Get Started
#### Prepare model and configuration files

First, you can create a notebook refer to [Kubeflow Notebook](https://elements-of-ai.github.io/kubeflow-docs/user-guide/notebooks.html#user-guide-notebooks). Then, clone model package have prepared in this notebook server.

```bash
$ git clone https://github.com/harperjuanl/helmet_yolov5_torchserve.git
$ cd kserve
```

#### Upload to MinIO

If you already have the MinIO storage, you can directly skip the MinIO deployment step, and follow the next steps to upload data to MinIO. If not, we also provide a standalone MinIO deployment guide on the kubernetes clusters, you can refer to the YAML files from [MinIO deployment files](https://github.com/vmware/ml-ops-platform-for-vsphere/tree/main/website/content/en/docs/kubeflow-tutorial/lab4_minio_deploy).

```bash
# create pvc
$ kubectl apply -f minio-standalone-pvc.yml

# create service
$ kubectl apply -f minio-standalone-service.yml

# create deployment
$ kubectl apply -f minio-standalone-deployment.yml
```

This step uploads v1/torchserve/model-store, v1/torchserve/config to MinIO buckets. You need to find the MinIO endpoint_url, accesskey, secretkey before upload using the following commands in your terminal.

```bash

# get the endpoint url for MinIO
$ kubectl get svc minio-service -n kubeflow -o jsonpath='{.spec.clusterIP}'

# get the secret name for Minio. your-namespace is admin for this Kubernetes cluster.
$ kubectl get secret -n <your-namespace> | grep minio

# get the access key for MinIO
$ kubectl get secret <minio-secret-name> -n <your-namespace> -o jsonpath='{.data.accesskey}' | base64 -d

# get the secret key for MinIO
$ kubectl get secret <minio-secret-name> -n <your-namespace> -o jsonpath='{.data.secretkey}' | base64 -d
```


#### Detection & Result

```bash
# Detect in the ternimal
$ curl -T test_1.jpg 'http://localhost:8080/predictions/helmet_detection' 
Output: 
[
  {
    "x1": 0.16830310225486755,
    "y1": 0.36698096990585327,
    "x2": 0.3356267809867859,
    "y2": 0.5662754774093628,
    "confidence": 0.9418923854827881,
    "class": "person"
  },
  {
    "x1": -0.0003846943436656147,
    "y1": 0.2697369456291199,
    "x2": 0.11975767463445663,
    "y2": 0.5021408796310425,
    "confidence": 0.9287041425704956,
    "class": "person"
  },
  {
    "x1": 0.31550225615501404,
    "y1": 0.27130556106567383,
    "x2": 0.4195330739021301,
    "y2": 0.4244980812072754,
    "confidence": 0.9224411249160767,
    "class": "person"
  },
  {
    "x1": 0.8000054359436035,
    "y1": 0.36035841703414917,
    "x2": 0.8742903470993042,
    "y2": 0.4628569483757019,
    "confidence": 0.9012498259544373,
    "class": "person"
  },
  {
    "x1": 0.44192060828208923,
    "y1": 0.3977605700492859,
    "x2": 0.5190550088882446,
    "y2": 0.4892307221889496,
    "confidence": 0.8915991187095642,
    "class": "person"
  },
  {
    "x1": 0.9677120447158813,
    "y1": 0.4071219563484192,
    "x2": 0.9998529553413391,
    "y2": 0.5111279487609863,
    "confidence": 0.875587522983551,
    "class": "person"
  },
  {
    "x1": 0.5246236324310303,
    "y1": 0.39841872453689575,
    "x2": 0.5718141794204712,
    "y2": 0.4656790792942047,
    "confidence": 0.8437989950180054,
    "class": "hat"
  },
  {
    "x1": 0.6443458795547485,
    "y1": 0.2609959542751312,
    "x2": 0.7564457654953003,
    "y2": 0.443324476480484,
    "confidence": 0.84254390001297,
    "class": "person"
  },
  {
    "x1": 0.6181862950325012,
    "y1": 0.3655022084712982,
    "x2": 0.6777603030204773,
    "y2": 0.452880322933197,
    "confidence": 0.7514644861221313,
    "class": "person"
  }
]%


# You can also choose to detect with the jupyter notebook
$ python3 -m pip install virtualenv
$ python3 -m virtualenv helmet-env
$ source yolov5-env/bin/activate

$ pip3 install -r requirements.txt
$ python3 -m pip install jupyter      # only need to install once if you don't have jupyter 
$ jupyter notebook 
# open the /helmet_yolov5_torchserve/resource/pytorch-yolov5-helmet-detection-inference.ipynb and run the code cell


# Stop the model serving
$ docker container ls
$ docker stop [the helmet container CONTAINER ID]
```


![Image text](./result.jpg)