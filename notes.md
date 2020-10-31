The philosophy of Kustomize is to supply a minimal set of working manifests that can be composed
with other manifests, and modified using Kustomize. If a user wishes to remove some part of the
manifest, there's a good chance either
1. the application will no longer function
2. your manifest was not simple enough

This results in more control over artifacts for users, _and_ in a lower maintenance burden for the
developers of the artifacts.

- Intended that all bases should be deployable without kustomizations. Therefore, treated the label
  `app.kubernetes.io/name` as reserved, and set it manually for all bases.

### Pros:
- Lower maintenance burden for artifact developers
- More control over deployment for users
- Better control of redeployment of pods dependent on secrets/configmaps
- Less abstraction
  - when something goes wrong there are fewer components in the system- you don't need to know (1)
    what helm is doing (2) what k8s is doing (3) what $layer is doing.. (Obviously, there is only
    one layer fewer).
  - when reasoning about compatibility and versioning, one less element in the compatibility matrix
- Fewer tools to use- services can be deployed with _only_ kubectl, if desired
- Easier to compose systems for testing- similarly to docker-compose

### Cons:
- Maybe a less cohesive ownership of resources in the cluster. How does helm 3 manage this? (If
  kustomize can't duplicate this easily, this will be a downside thereof).

### Questions:
- How does kubectl/kustomize identify deployed artifacts when upgrading?
- How does kubectl/kustomize manage failed upgrades?
- How does kubectl/kustomize manage shared resources? Does it expect to "own" the cluster? How can
  a user handle whatever behaviour it exhibits here?
- When using commonLabels it's possible to overwrite those used by subcharts. Is it just
  best-practice to not use `commonLabels` such as `kubernetes.io/name`?
- Do commonLabels get applied to selectors?
- How does kustomize handle deletions during upgrades?
- How to manage immutable fields? For example, try to change a Service .spec.clusterIP and apply
  the changed configuration.
- Is it possible for users of the ML charts to make a smooth transition from Helm?
- How does Kustomize handle the diamond problem? (How does Helm handle the diamond problem?)
  - https://github.com/kubernetes-sigs/kustomize/issues/1251
- Can Kustomize catch problems before helm would?
- What advantages are gained by a `--dry-run` kubectl install?
- Can Kustomize deploy to multiple namespaces simultaneously?
- How to handle ingresses? Should that be left to the user, or put in a different base, or put in
  the service base?
- If I want to compose a configmap with a sidecar and then apply that composition to a service, how
  do I do that?


#### Onboarding
- Use Postman
- Port-forward central ledger
  ```sh
  kubectl port-forward -n cl deploy/centralledger-service 3001
  ```
- Modify postman environment
  `HOST_CENTRAL_LEDGER=localhost:3001`
  `BASE_CENTRAL_LEDGER_ADMIN=''`
- Find and replace everywhere `participants/hub` (case-sensitive!) with `participants/Hub`
- Using the Postman _Runner_, run `New-Deployment-FSP_Setup_MojaloopSims/FSP Onboarding/`
  onboarding folder

#### Transfer
- Use Postman
- Port-forward ml-api-adapter
  ```sh
  kubectl port-forward -n cl deploy/ml-api-adapter-service 3000
  ```
- Modify environment:
  `HOST_SWITCH=localhost:3000`
  `BASE_PATH_SWITCH=''`


Objective:

ALS
quoting service
ml-api-adapter
central ledger

Therefore, Kafka and MySQL

All services needed to run P2P for clearing

TODO:
- Review all liveness/readiness probes. Perhaps remove them from all node-only services and apply
  those from the kustomization. This is because the liveness/readiness probe configurations
  probably depends on the individual cluster compute resources available.
- Is there any use to the various sidecars?
- Remove all `progressDeadlineSeconds: 600`- this is the default value
- Remove all:
  ```
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  ```
  Those are the defaults.
- Remove all `service.type: ClusterIP` (the default)
- Remove all `revisionHistoryLimit` values- this is probably something for a user to decide
- Move all wait-for-kafka and wait-for-mysql init containers into kustomizations- this way the
- service names they connect to can be controlled
- Look through TODOs in the code
- Collapse all mysql access configmaps into one. Perhaps supplied by the mysql base.
- Prettify the config json
- Use SecretGenerators and ConfigMapGenerators instead of secrets and configmaps
- It's possible to have kustomize produce all manifests as individual files with
  ```sh
  mkdir out
  kustomize build -o out
  ```
  Could split our single files into multiple files like this.
- Treated `app.kubernetes.io/name` as reserved. For this to work, it might be the case that
  consumers have to respect this also. Is this sensible? Presumably it's possible to apply a
  transformer to change this value?
- Look at the notes from the previous attempt
- Move all scripts out of configmaps and into files - see the upstream charts for the source of
  these
- Move all json config out of configmaps and into files - see the upstream charts for the source of
  these
- Split central ledger components into multiple bases? Probably not..
- Ingresses! Put them in Mojaloop, or in the individual services?
