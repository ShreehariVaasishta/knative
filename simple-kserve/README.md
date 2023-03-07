# Simple KServe Inference Logger

Logging the prediction input(request data) and prediction response.

## Instructions

### Message Dumper

We will create a simple service called message-dumper, which receives the data in form of a HTTP POST request from KServe for each request and response(individually). KServe includes a unique identifier(uuid) to know for which request the prediction response was returned.

1. Setup Message Dumper

- Create a file named _*message-dumper.yaml*_ and paste the below contents in it.
  ```yaml
  apiVersion: serving.knative.dev/v1
  kind: Service
  metadata:
  name: message-dumper
  spec:
  template:
    spec:
    containers:
      - image: gcr.io/knative-releases/knative.dev/eventing-contrib/cmd/event_display
  ```
- Create the service now on kubernetes
  ```bash
  kubectl create -f message-dumper.yaml
  ```

2. Inference Service with logger

- Create an inference service with logger enabled. Let's create a simple sklearn inference with logger enabled.

```yaml
# sklearn-basic-logger.yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: sklearn-iris
spec:
  predictor:
    logger:
      mode: all
      url: http://message-dumper.default/
    sklearn:
      storageUri: gs://kfserving-examples/models/sklearn/1.0/model
```

Important thing to note here is the part of logger as mentioned below. To enable logger in your inference service, you can add the below lines to any inference service.

```yaml
logger:
  mode: all
  url: http://message-dumper.default/
```

**NOTE:** You can add the url of your external service which handles the data. Here, as an example im adding the url of a kubernetes service - message-dumper, which we created above.

- Create the service

```yaml
kubectl create -f sklearn-basic-logger.yaml
```

3. Now make a request to the inference service and check the logs of the message-dumper pod(container: user-container). Here you should be able to see the request and response logs.

#### **NOTE:** This logger is basically an implementation based on KNative's eventing.
