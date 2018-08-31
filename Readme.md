## Tensorflow Serving using Docker

### Steps to configure :

#### Step 1 : Download and install Docker

Docker is an open source software platform to create, deploy and manage virtualised application containers on a common operating system (OS), with an ecosystem of allied tools. With Docker, you have the solution that helps you manage the diverse applications, clouds and infrastructure you have today while providing your business a path forward to future applications.

To install docker on your system [click here](https://docs.docker.com/docker-for-mac/install/) .

#### Step 2 : Clone TensorFlow Serving repository
For original Tensorflow Serving Repo [click here](https://github.com/tensorflow/serving) .

There are dependency issues on managed machine when we try to build the tensorflow serving images from original source. Hence i've used this repo as the dependencies are pre-compiled here, and it offers some pre-trained model to test our architecture.
```
git clone https://e074312@fusion.mastercard.int/stash/scm/~e074312/tensorflow_serving.git
```
The Dockerfile here is modified to bypasss the dependency issue.


#### Step 3 : Build TensorFlow Serving Docker image
Move to tensorflow_serving repo and do a docker build for the image. This will use the Dockerfile present in the repo to build image and manage dependencies.
```
cd tensorflow_serving
docker build --pull -t tensorflow-serving .
```
#### Step 4 : Run TensorFlow Serving Docker image

Once the image is built, we'll run it. Use the absolute path of the repo and replace it with my path `/Users/e074312/Desktop/tensorflow_serving/model_volume/` in the command below.
```
docker run -it -p 8500:8500 -p 8501:8501 -v /Users/e074312/Desktop/tensorflow_serving/model_volume/:/home/ --name tensorflow-serving tensorflow-serving
```

On successful run your shell will look like :
```
root@15954c4d0666:/#
```
#### Step 5 : Run TensorFlow Serving API
This step will start the server and serves the rest api on port 8501. It uses the configuration from this directory on the virtual machine /home/configs/models.conf

```
tensorflow_model_server --port=8500 --rest_api_port=8501 --model_config_file=/home/configs/models.conf
```

This `model.conf` file  contains config information for the server to be able to serve the models in the repo in the right way. The structure will be like :
```
model_config_list: {
  config: {
    name:  "xor",
    base_path:  "/home/models/xor",
    model_platform: "tensorflow",
    model_version_policy: {
        all: {}
    }
  },
  config: {
    name:  "iris",
    base_path:  "/home/models/iris",
    model_platform: "tensorflow",
    model_version_policy: {
        all: {}
    }
  }
}
```
This is pretty self explanatory, the part that is important is if you want to serve all versions of your model this part is necessary, if you do not have it than it will only serve the latest model:

```
model_version_policy: {
   all: {}
}
```

#### Step 6 : Test the above architecture
Once the architecture is ready and server is up and running, we can test it using POST request

##### Classification Request
```
curl -X POST \
  http://localhost:8501/v1/models/iris:classify \
  -H 'cache-control: no-cache' \
  -H 'postman-token: f7fb6e3f-26ba-a742-4ab3-03c953cefaf5' \
  -d '{
 "examples":[
   {"x": [5.1, 3.5, 1.4, 0.2]}
  ]
}'
```

Response
```
{
    "results": [
        [
            [
                "Iris-setosa",
                0.872397
            ],
            [
                "Iris-versicolor",
                0.108623
            ],
            [
                "Iris-virginica",
                0.0189799
            ]
        ]
    ]
}
```


##### Prediction Request
```
curl -X POST \
  http://localhost:8501/v1/models/iris/versions/2:predict \
  -d '{
 "signature_name": "predict-iris",
 "instances":[
  [5.1, 3.5, 1.4, 0.2]
 ]
}'
```

Response
```
{
    "predictions": [
        [
            0.872397,
            0.108623,
            0.0189799
        ]
    ]
}
```
