# Kubernetes administration: practical study companion

Use the scenario tests to decide what to investigate, then use a disposable cluster to practise the work. Reading a correct answer is not the same as configuring, observing and recovering a workload yourself.

## Set up a safe workspace

- Use a cluster you own and can discard. Do not use an employer's production cluster, shared customer environment or an exam environment for these investigations.
- Confirm the selected context before every session. Context names are not evidence of what cluster they point to: inspect the cluster server and user as well.
- Choose a new, uniquely named study namespace. If that namespace already exists, do not assume you own its contents; choose another name.
- Keep a record of every object you create. Cleanup must target those objects only, not arbitrary resources that happen to have similar labels.
- Use approved container images that your lab can pull. Record the resolved image digest when you want a repeatable exercise. Registry limits, admission rules and runtime security settings may require adaptations.
- Check the current Linux Foundation exam instructions separately. Match version-sensitive practical work to the actual environment you will use; this workbook is not a terminal exam simulator.

Start with read-only checks:

```sh
kubectl config current-context
kubectl config view --minify
kubectl version
kubectl get nodes
```

Do not share the raw kubeconfig or credentials in screenshots, notes or a public repository. Record only the non-secret details needed to identify your lab.

## 1. Follow the authorization path

In your study namespace, create a ServiceAccount, a Role allowing get/list/watch on Pods, and a RoleBinding connecting them. Use an administrator identity that is authorized to impersonate the study account to check `kubectl auth can-i` for Pod reads and deletes.

Then create a second, narrower read grant. Does it subtract an existing permission? Explain the difference between changing a grant and adding another grant. Finally, inspect the namespace of the binding, the namespace of the subject, and the referenced role independently.

For the ClusterRole-reference variation, create only a uniquely named study ClusterRole and record it for explicit cleanup. A RoleBinding can apply its namespaced permissions in one namespace; a ClusterRoleBinding has a different scope. Do not grant cluster-admin just to make a failed test pass.

Evidence to retain: the exact manifests, the impersonated identity, the requested verb/resource/namespace and your actual authorization result.

Reading: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

## 2. Separate context from namespace

With two disposable contexts available, compare a one-request `--context` override with `kubectl config use-context`. Inspect the saved current context after each. Compare `--namespace` with `--context`; they are not interchangeable.

Do not run a write command simply to prove which cluster was selected. A read-only namespace or Pod listing is enough to investigate the selection path.

Reading: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

## 3. Change a binding reference deliberately

Create two study Roles and a binding referencing the first. Try the proposed roleRef change and retain the real API response. Plan a replacement binding that preserves only the intended subjects. Compare this with changing the rules of a Role: the referenced identity and its contents are different things.

Consider the brief authorization impact of deleting and recreating a binding before using that approach in a real change plan.

Reading: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

## 4. Observe initialization before probing

Use `scenario-lab.yaml` as an optional starting point after reviewing it. Its regular init container finishes before the web container starts. Inspect the init status, events and actual logs. Change the init step in your own copy so that it does not complete, then explain why the application has not started.

Keep regular init containers separate from restartable sidecars. A readiness probe cannot skip unfinished regular initialization.

Reading: https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

## 5. Diagnose a Service in layers

Inspect the Deployment's Pod labels, the Service selector, the eligible EndpointSlices, the Service port and the backend target port. In a copy of your study Service, introduce exactly one mismatch. Capture the actual evidence before correcting it.

For a named-target-port extension, run two backends that genuinely listen on different numeric ports but declare the same port name. Do not change only `containerPort` and assume that this changes what the application listens on.

Reading: https://kubernetes.io/docs/concepts/services-networking/service/

Reading: https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/

## 6. Test both sides of a network policy

Use a lab network plugin that actually enforces NetworkPolicy. Start with known working connectivity and record source/destination addresses and namespaces. Add one policy at a time.

Compare a single peer entry containing both namespaceSelector and podSelector with two separate peer entries. Explain which traffic each version selects. If both endpoints are isolated in the relevant direction, inspect source egress as well as destination ingress. Preserve DNS access if your test depends on name resolution; a DNS failure is not proof of an application-port rule failure.

Reading: https://kubernetes.io/docs/concepts/services-networking/network-policies/

## 7. Compare normal and headless discovery

Create a normal selector-based Service and a separate headless Service for the same ready study Pods. Inspect actual DNS answers and EndpointSlices. Explain why a virtual Service address and direct backend addresses support different client behavior.

Keep `publishNotReadyAddresses` and Pod readiness explicit. Do not infer readiness or ordering guarantees from a headless Service alone.

Reading: https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

Reading: https://kubernetes.io/docs/concepts/services-networking/service/

## 8. Separate startup, readiness and liveness

In a copy of the study workload, design distinct checks for initialization, ability to receive traffic and a condition that justifies restart. Observe real probe events, restart counts and endpoint eligibility.

Do not deliberately restart unrelated workloads. A readiness failure does not, by itself, request a container restart. A startup probe can defer the other checks until initialization succeeds; it is not a permanent waiver of health checks.

Reading: https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/

## 9. Distinguish requests, limits and utilization

Inspect the admitted Pod specification, not only the submitted YAML: admission defaults can supply values. Compare scheduling based on requests with measured CPU usage and limit enforcement.

If your lab already has working resource metrics and an HPA, investigate a CPU percentage target with and without appropriate CPU requests. Do not install an unreviewed metrics add-on just to follow this worksheet. Compare a percentage-of-request target with an absolute usage target before drawing conclusions.

Reading: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

Reading: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

## 10. Investigate configuration delivery

Provide a harmless ConfigMap value to a study container through an environment variable. Change the ConfigMap and inspect the running process's value before and after replacing the Pod. Contrast environment-variable delivery with a mounted ConfigMap; account for mount type and propagation behavior rather than assuming every update is immediate.

Never put a real credential into a ConfigMap to make the exercise more realistic.

Reading: https://kubernetes.io/docs/concepts/configuration/configmap/

## 11. Trace Job completion and failure

The optional study manifest includes a deliberately long-running Job with a short active deadline. Inspect the Job condition, Pod events and actual elapsed behavior. Compare this with a Job whose process exits in error and retries under a backoff limit.

Record whether you are using an ordinary or indexed Job and whether custom failure/success policies apply. Do not treat starting a Pod as successful Job completion.

Reading: https://kubernetes.io/docs/concepts/workloads/controllers/job/

## 12. Inspect scheduling constraints and quota separately

In your own copy of a study Deployment, add a required node-affinity rule that matches no lab node. Inspect the actual scheduling event. Changing a preferred weight cannot override that required rule.

In a different uniquely named study namespace, use a small Pod-count ResourceQuota and observe an attempted additional Pod creation. Compare a rejected creation with an admitted but unscheduled Pod. Do not confuse every Forbidden response with an RBAC denial: read the supplied reason.

Reading: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

Reading: https://kubernetes.io/docs/concepts/policy/resource-quotas/

## 13. Treat storage as real data

First identify your lab provisioner, supported access modes, topology and reclaim behavior. Do not assume a default StorageClass exists. Inspect a claim, its selected class, binding events and eventual PV before writing test data.

For WaitForFirstConsumer, compare an unused claim with one referenced by a schedulable Pod. For access modes, distinguish node-scoped ReadWriteOnce from single-Pod ReadWriteOncePod and check the actual driver support. A Kubernetes access-mode label is not a substitute for application-safe shared writes.

Investigate expansion only with disposable data and a supported driver. Successful expansion is not a promise of shrinking. A smaller replacement volume requires a separately planned and verified migration. Check reclaim policy before deleting any claim; Retain does not mean Kubernetes has made a backup.

Reading: https://kubernetes.io/docs/concepts/storage/persistent-volumes/

Reading: https://kubernetes.io/docs/concepts/storage/storage-classes/

## 14. Build a maintenance plan before disrupting a node

Use a disposable multi-node cluster with enough spare capacity if you practise cordon, drain, taints or disruption budgets. Inspect current workloads, DaemonSets, local data, budgets and controller ownership first.

Predict which existing Pods remain under NoSchedule and which may be evicted under NoExecute. For eviction-based drain, explain why `--ignore-daemonsets` does not delete the DaemonSet Pods and why a strict budget can block an eviction. Do not use force deletion as the default answer to a blocked maintenance window.

Certificate renewal, etcd restore and control-plane upgrades need their own complete recovery plans. Read the current kubeadm and version-skew instructions before attempting them; do not infer an upgrade or restore sequence from a multiple-choice answer alone.

Reading: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

Reading: https://kubernetes.io/docs/tasks/run-application/configure-pdb/

Reading: https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

Reading: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/

Reading: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

Reading: https://kubernetes.io/releases/version-skew-policy/

## 15. Keep a small evidence notebook

For every investigation, record:

| Field | Your notes |
| --- | --- |
| Context, namespace and non-secret resource identifiers | |
| Requirement and symptoms | |
| Read-only checks and actual results | |
| Hypothesis and evidence that would disprove it | |
| Smallest authorized change | |
| Verification after the change | |
| Recovery and cleanup of objects you created | |

For multi-container Pods, name the container when collecting logs. Distinguish current logs from the previous terminated instance. Preserve the actual event and error text rather than writing down the result you expected to see.

Reading: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/

Reading: https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/

## Optional manifest use and cleanup

Read `scenario-lab.yaml` before applying it. Replace every occurrence of `cka-study-unique` with a new namespace name that does not already exist. The manifest includes that Namespace, two small web replicas, a Service and a short-lived deadline exercise. It does not install a driver, controller, metrics service or network plugin. Admission controls or image-access rules in your lab can require changes.

Apply only to your verified disposable context. Record what your cluster actually returns, including any errors, rather than assuming a particular result. When finished, delete only the namespace and resources that you created for this exercise. Do not delete a namespace that predated your study work. Review for any cluster-scoped resources you created separately and clean up those exact objects explicitly.
