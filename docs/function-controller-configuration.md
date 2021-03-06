# Controller configurations for Functions

### Using ConfigMap

Configurations for functions can be done in `ConfigMap`: `kubeless-config` which is a part of `Kubeless` deployment manifests.

Deployments for function can be configured in `data` inside the `ConfigMap`, using key `deployment`, which takes a string in the form of `yaml/json` and is driven by the structure of [v1beta2.Deployment](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#deployment-v1beta2-apps):

E.g. In the below configuration, new **annotations** are added globally to all function deployments and podTemplates and **replicas** for each function pod will be `2`.

```yaml
apiVersion: v1
data:
  deployment: |-
    {
      "metadata": {
          "annotations":{
            "annotation-to-deployment": "value"
          }
      },
      "spec": {
        "replicas": 2,
        "template": {
          "annotations": {
            "annotations-to-pod": "value"
          }
        }
      }
    }
  ingress-enabled: "false"
  service-type: ClusterIP
kind: ConfigMap
metadata:
  name: kubeless-config
  namespace: kubeless
```

It is **recommended** to have controlled custom configurations on the following **items** (*but is not limited to just these*):

> Warning: You should know what you are doing.

- v1beta2.Deployment.ObjectMeta.Annotations
- v1beta2.Deployment.Spec.replicas
- v1beta2.Deployment.Spec.Strategy
- v1beta2.Deployment.Spec.Template.ObjectMeta.Annotations
- v1beta2.Deployment.Spec.Template.Spec.NodeSelector
- v1beta2.Deployment.Spec.Template.Spec.NodeName

Having said all that, if one wants to override configurations from the `ConfigMap` then in `Function` manifest one needs to provide the details as follows:

```yaml
apiVersion: kubeless.io/v1beta1
kind: Function
metadata:
  name: testfunc
spec:
  deployment:  ### Definition as per v1beta2.Deployment
    metadata:
      annotations:
        "annotation-to-deploy": "final-value-in-deployment"
    spec:
      replicas: 2  ### Final deployment gets Replicas as 2
      template:
        metadata:
          annotations:
            "annotation-to-pod": "value"
  deps: ""
  function: |
    module.exports = {
      foo: function (req, res) {
            res.end('hello world updated!!!')
      }
    }
  function-content-type: text
  handler: hello.foo
  runtime: nodejs8
  service:
    ports:
    - name: http-function-port
      port: 8080
      protocol: TCP
      targetPort: 8080
    type: ClusterIP
```
