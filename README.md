# kfp-fun

apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: spark-pipeline-
spec:
  entrypoint: spark-pipeline
  templates:
  - name: spark-pipeline
    dag:
      tasks:
      - name: create-spark-app
        template: spark-operator
        arguments:
          parameters:
          - name: action
            value: create
          - name: namespace
            value: my-namespace
          - name: image
            value: docker.io/my-spark-app:latest
          - name: main-class
            value: com.example.sparkapp.MyApp
          - name: arguments
            value: "arg1,arg2"
          - name: spark-version
            value: "3.1.1"
          - name: driver-cores
            value: "1"
          - name: driver-memory
            value: "512m"
          - name: executor-cores
            value: "1"
          - name: executor-instances
            value: "3"
          - name: executor-memory
            value: "512m"
    inputs:
      parameters:
      - name: action
      - name: namespace
      - name: image
      - name: main-class
      - name: arguments
      - name: spark-version
      - name: driver-cores
      - name: driver-memory
      - name: executor-cores
      - name: executor-instances
      - name: executor-memory
    outputs: {}
  - name: spark-operator
    inputs:
      parameters:
      - name: action
      - name: namespace
      - name: image
      - name: main-class
      - name: arguments
      - name: spark-version
      - name: driver-cores
      - name: driver-memory
      - name: executor-cores
      - name: executor-instances
      - name: executor-memory
    container:
      image: sparkop:v1.0
      command: ["/spark/bin/spark-submit"]
      args:
      - "--master"
      - "k8s://https://kubernetes.default.svc"
      - "--deploy-mode"
      - "cluster"
      - "--name"
      - "{{workflow.name}}-{{pod.name}}-{{inputs.parameters.main-class}}"
      - "--class"
      - "{{inputs.parameters.main-class}}"
      - "--conf"
      - "spark.kubernetes.namespace={{inputs.parameters.namespace}}"
      - "--conf"
      - "spark.kubernetes.container.image={{inputs.parameters.image}}"
      - "--conf"
      - "spark.kubernetes.driver.request.cores={{inputs.parameters.driver-cores}}"
      - "--conf"
      - "spark.kubernetes.driver.limit.cores={{inputs.parameters.driver-cores}}"
      - "--conf"
      - "spark.kubernetes.driver.request.memory={{inputs.parameters.driver-memory}}"
      - "--conf"
      - "spark.kubernetes.driver.limit.memory={{inputs.parameters.driver-memory}}"
      - "--conf"
      - "spark.executor.instances={{inputs.parameters.executor-instances}}"
      - "--conf"
      - "spark.executor.cores={{inputs.parameters.executor-cores}}"
      - "--conf"
      - "spark.executor.memory={{inputs.parameters.executor-memory}}"
      - "local:///opt/spark/work-dir/executor.jar"
      - "{{inputs.parameters.arguments}}"
