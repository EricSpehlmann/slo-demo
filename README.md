# SLO Demo Instructions
Brief introduction to creating a helm chart for deploying SLOs in monitoring namespace.

Confluence doc on components: https://autonomic-ai.atlassian.net/wiki/spaces/OPLAT/pages/2753036316/SLO+Framework 
## Setting up a slo chart

This chart was created by running `helm create slo-demo`
Helm will scaffold a chart for you. I removed the auto generated templates folder because we will be using the `slo` chart 
created by observability team and hosted in au helm repository. Speaking of which, that is configured in the `requirements.yaml`. 
```
dependencies:
  - name: slo
    version: "*"
    repository: "@au"
    alias: slo-demo
```
Here we are specifying that this chart is dependent on [the `slo` chart](https://github.com/autonomic-ai/slo-helm-chart). Therefore we must run `helm dep up` || `helm dependency update` to fetch
the dependent chart. More info [here]( https://helm.sh/docs/helm/helm_dependency_update/ )
## Using helm to template a PrometheusServiceLevel CRD spec

```bash
helm dep up
helm template slo-demo  . --set environment=dev > my-psl.yaml
```
This will template out a psl crd which defines the slo spec for the sloth controller to convert the PSL into Prometheus Rules CRDS. 
## applying to cluster

```bash
kubectx gke-d-tmc-01
kubens monitoring
kubectl apply -f my-psl.yaml
```
Switch to gcp-dev cluster and monitoring namespace. Using kubectl we will deploy our psl. 

## confirm psl deployment and transformation to prometheus rules crd

```bash
kubectl get psl | grep slo-demo
kubectl get prometheusrule | grep slo-demo
```
Sloth controller https://sloth.dev/examples/kubernetes/getting-started/ will have converted the PSL into a PrometheusRules CRD

This Prometheus Rules CRD gets converted by `prometheus-rules-controller` side car running on prometheus pod, into a prometheus recording rule in the `etc/config/controller-rules` directory. Prometheus will reload its config and update recording rules.


## prometheus-server for recording rule

```
kubectl port-forward <prometheus-server-#######-#####> 9090

```
browse to http://localhost:9090/rules and your recording rule should be defined. 
## delete slo-demo
```
kubectl delete psl slo-demo
```
prometheus rules CRD will be deleted and  will be ereased from prometheus rules on prometheus-server.