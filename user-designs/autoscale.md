## Autoscaling in Kubernetes 

## Questions

**Autoscaling Metrics**

1. Are you using any metrics other than those built into Kubernetes like CPU and memory?

1. How often are you autoscaling based on custom application metrics? Application metrics would be something like the number of messages in a rabbitmq queue, the number of events triggered by an application, etc.

1. Are you using KEDA? 

**Autoscaling Behavior**

1. What are the autoscaling behaviors you are using today? 
    - Scale up/down/zero?
    - Scheduled scaling?
    - Any other?

**Personas and Responsibilities**

1. Who configures the HPA in your Kubernetes environment today?

1. What are the personas involved in autoscaling? Our assumptions are:
	- Developers specify which container gets autoscaled based on which metric and at the specified threshold
	- Platform engineers configure the environment with min and max replicas
	- SRE/operations engineers tune autoscaling by overriding tuning parameters such as thresholds

    Eg: env.bicep

    ```bicep
    resource environment 'Applications.Core/environments@2023-10-01-preview' = {
        name: 'myenv'
        properties: {
            compute: {
                kind: 'kubernetes'
                namespace: 'default' 
                // Platform engineer specifies the autoscaling behavior
                autoscaling: {
                    minReplicas: 1
                    maxReplicas: 10
                }
            }
        }
    }    
    ```
    app.bicep

    ```bicep
    resource container 'Applications.Containers@2023-10-01-preview' = {
        name: 'myapp'
        properties: {
            image: 'myregistry.azurecr.io/myapp:latest'
            // Developer specifies the autoscaling metrics
            autoscaling: {
                metric: 'cpu'
                threshold: 50
            }
        }
    }

6. Should the metric threshold be specified by the developer or the SRE/operations engineer? 

7. Should the developer be able to override autoscaling behavior and limits?

8. Is the ownership the same across all environments? For example, do you need the ability to delegate autoscaling behavior configuration to developers in a test environment?

    Eg: env.bicep

        ```bicep
        //Created in the non-prod-environment resource group
        //Access is limited to platform engineers
        //DO WE NEED DEVELOPERS HERE IN RBAC?
        resource environment 'Applications.Core/environments@2023-10-01-preview' = {
            name: 'myenv'
            properties: {
                compute: {
                    kind: 'kubernetes'
                    namespace: 'default' 
                    autoscaling: {
                        minReplicas: 1 
                        maxReplicas: 5
                    }
                }
            }
        }    
        ```

9. Should a container opt-in to autoscaling or should it rely on the environment configuration?

    Eg: env.bicep

    ```bicep
    resource environment 'Applications.Core/environments@2023-10-01-preview' = {
        name: 'myenv'
        properties: {
            compute: {
                kind: 'kubernetes'
                namespace: 'default' 
                autoscaling: {
                    minReplicas: 1
                    maxReplicas: 10
                    // Default autoscaling thresholds for all applications
                    metric: [
                        {
                            name: 'cpu'
                            threshold: 50
                        }
                        {
                            name: 'memory'
                            threshold: 50
                        }
                    ]
                }
            }
        }
    }    
    ```
    app.bicep

    ```bicep
    resource container 'Applications.Containers@2023-10-01-preview' = {
        name: 'myapp'
        properties: {
            image: 'myregistry.azurecr.io/myapp:latest'
            // Should the developer opt-in to autoscaling?
            autoscaling : on // opt-in or opt-out
        }
    }

10. How granular should autoscaling rules be? 
    - Only application-specific? 
    - All applications are treated the same in an environment?
    - Default for all applications with the ability to override? 

## Scenarios

**Environment Configuration**

```bicep 
 resource environment 'Applications.Core/environments@2023-10-01-preview' = {
  name: 'test'
  properties: {
    compute: {
      kind: 'kubernetes'
      namespace: 'default' 
      identity: {          
      }
    autoscaling: {
        minReplicas: 1
        maxReplicas: 5
      }
    }
}
```


1. Scenario 0 : No Autoscaling 

    ```bicep
    resource container 'Applications.Containers@2023-10-01-preview' = {
        name: 'myapp'
        properties: {
            image: 'myregistry.azurecr.io/myapp:latest'
        }
    }
    ```

1. Scenario 1 – Scaling based on system resource metrics (cAdvisor is already collecting and sending to metrics server)

    ```bicep
    resource container 'Applications.Containers@2023-10-01-preview' = {
        name: 'myapp'
        properties: {
            image: 'myregistry.azurecr.io/myapp:latest'
            autoscaling: {
                metric: 'cpu'
                threshold: 50
            }
        }
    }

1. Scenario 2 - Autoscaling based on a metric from a Radius-managed resource (a metric from a different resource other than the container which is being autoscaled)
    ```bicep
    resource container 'Applications.Containers@2023-10-01-preview' = {
        name: 'myapp'
        properties: {
            image: 'myregistry.azurecr.io/myapp:latest'
            autoscaling: {
            metric: myQueue.queueLength
            threshold: 10
            }
        }
    }

    resource rabbitmq 'MyCompany.App/rabbitmq' = {
        name: 'myQueue'
    }
   ``` 

1. Scenario 3 – Scaling based on application metric exposed by container (a custom metric exposed by the application for Prometheus to pickup)
    ```bicep
    resource container 'Applications.Containers@2023-10-01-preview' = {
        name: 'myapp'
        properties: {
            image: 'myregistry.azurecr.io/myapp:latest'
            autoscaling: {
                metric: concurrent-users
                threshold: 50
            }
            metrics: {
                name: concurrent-users
                endpoint: /metricsz
            }
        }
     }
    ```

1. Scenario 4 - SRE override
    ???
    Override threshold from 50 to 40 for example
    No change in metric configuration

    Not p0