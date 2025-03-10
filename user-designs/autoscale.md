## Autoscaling in the cloud-native ecosystem

The following are the most common autoscaling mechanisms available in Kubernetes. 

**Kubernetes**
1. **Horizontal Pod Autoscaler (HPA)** - Kubernetes native autoscaling mechanism that scales the number of pods in a deployment based on pod metrics such as CPU utilization, Memory and other custom metrics. This is the most common autoscaling mechanism used in the Kubernetes ecosystem.
2. **Vertical Pod Autoscaler (VPA)** - Kubernetes native autoscaling mechanism that automatically adjusts the CPU and memory requests of the pods. This is the least common autoscaling mechanism used in the Kubernetes ecosystem as it requires restarting the pods to apply the new resource requests.
3. **KEDA** - Kubernetes Event-driven Autoscaling (KEDA) is an open-source component that enables autoscaling of Kubernetes workloads based on external metrics. KEDA operates on top of the HPA and triggers scaling based on metrics from various sources, such as message queues, databases, or observability platforms.

## Autoscaling Policy Configuration

Autoscaling policy consists of the following components:

1. **Autoscaling behavior** 
   - Scale Up/Down - How the application should scale up or down when the load increases or decreases. 
   - Scheduled scaling - How the application should scale based on a schedule. For example, scale up during business hours and scale down during non-business hours.

2. **Autoscaling metrics**
    - Infrastructure metrics - These are metrics that are related to the infrastructure on which the application is running. Examples include CPU utilization, Memory utilization, Disk I/O, Network I/O, etc. These metrics are typically collected by the Kubernetes metrics server or from external sources like Prometheus and can be used to scale the number of pods in a deployment
    - Application metrics - These are metrics that are related to the application itself. Examples include the number of messages in a queue, the number of events triggered by an application, etc. These metrics are typically collected by the application.

## Use Story 1 : Autoscaling by Infra Metrics such as CPU, Memory Limit.

Let's consider a simple use case where we want to autoscale applications using HPA by standard resource metrics. Since these metrics are infrastructure metrics, platform engineers would typically define the autoscaling configuration in the environment resource.

```bicep
resource environment 'Applications.Core/environments@2023-10-01-preview' = {
  name: 'myenv'
  properties: {
    compute: {
      kind: 'kubernetes'
      namespace: 'default' 
      identity: {          
      }
    autoscaling: {
        minReplicas: 1
        maxReplicas: 10
        cpuUtilization: 50
        memoryLimit: 50
      }
    }
}
```
In this example, the platform engineer is configuring the autoscaling behavior for the application to scale between 1 and 10 replicas and based on some infrastructure metrics such as CPU utilization and memory limit for the pods.

Pros:
- Separates the concerns where platform engineers can configure the autoscaling policy behavior by standard pod metrics in the environment resource.
- Platform engineers can enforce the autoscaling policy at the environment level. Dev, Test environments can have different autoscaling policies than prod environments.
- Application developers are abstracted from the autoscaling behavior and infrastructure metrics. 

Cons:
- All applications in the environment will have the same autoscaling configuration.
- Developer cannot configure the autoscaling policy for their application.

### Questions

1. Does this example make sense ? Does the platform engineer decide only on the autoscaling behavior or on the infra metrics as well? Do developers have a say?

2. Do you have multiple applications in the same environment that need different autoscaling behaviors? 

3. If yes, can these applications have metadata that can be used to group them together? For example, web applications, batch applications, event-driven applications, etc.

## Use Story 2 : Autoscaling by Application Metrics

Now let's consider a use case where we want to autoscale applications using some application metrics. The application has a rabbitmq queue, and we want to scale the application based on the number of messages in the queue. In this case, the application developer would define the autoscaling configuration in the application definition. 

```bicep
resource container 'Applications.Containers@2023-10-01-preview' = {
  name: 'myapp'
  properties: {
    image: 'myregistry.azurecr.io/myapp:latest'
    autoscaling: {
      queue: rabbitmq.QueueName
      queueLength: 10
    }
  }
}

resource rabbitmq 'Applications.Messaging/rabbitmqQueues@2023-10-01-preview' = {
  name: 'rabbitmq'
  properties: {
    queueName: 'myqueue'
    ....
  }
}
```

In this example, the application developer is configuring the metrics to scale based on the number of messages in the rabbitmq queue and the platform engineer has already configured the scaling behavior 

Pros:
- Flexibility for Application developers can configure or override or add additional autoscale metrics for their application.

Cons:
- Only select metrics that are application specific will be available for the developer 

Questions:

1. Does this make sense. Do you want your developers to override or add additional metrics to the autoscaling policy for their applications?

1. If yes, do you want to only allow certain metrics to be overridden? For example, CPU and Memory metrics should be defined by the platform engineer, but application metrics can be defined by the developer.

1. Are there other personas who would be interested in configuring the autoscaling policy for their applications? For example, operations engineers, SREs, etc.
