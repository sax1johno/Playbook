# Prometheus Operator in Openshift

Prometheus operator can be deployed in openshift using Openshift subscription. Create a file named `prometheus-subscription.yaml` with following content

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  generateName: prometheus-
  namespace: {NAMESPACE}
spec:
  source: rh-operators
  name: prometheus
  startingCSV: prometheusoperator.0.22.2
  channel: preview
```

Replace `{NAMESPACE}` with the name of namespace in which you want prometheus operator to be deployed.

Currently okd doesn't support prometheus operator to watch more than one namespace. If needed you can create a custom CSV of prometheus operator that uses the latest version of prometheus operator image and specify multiple namespaces to watch.

To get the current CSV, create a subscription for the prometheus operator and then go to `k8s/all-namespaces/clusterserviceversions` in the browser and select the subscription you created and copy its yaml file. Make the following changes to the CSV:

- Change version
- Remove fields like selflink, uuid etc.
- Update image of prometheus operator, minimum v0.29.0.
- Search the block that creates the prometheus operator container and update the `namespace` arg and change it to `'-namespaces=$(K8S_NAMESPACE),SECOND_NAMESPACE'` (change `SECOND_NAMESPACE` to the name of extra namespace that you want it to watch). You can add multiple namespaces as well by separating there names with `,`.

After changes your CSV should look something like this
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  annotations:
    alm-examples: >-
      [{"apiVersion":"monitoring.coreos.com/v1","kind":"Prometheus","metadata":{"name":"example","labels":{"prometheus":"k8s"}},"spec":{"replicas":2,"version":"v2.3.2","serviceAccountName":"prometheus-k8s","securityContext":
      {},
      "serviceMonitorSelector":{"matchExpressions":[{"key":"k8s-app","operator":"Exists"}]},"ruleSelector":{"matchLabels":{"role":"prometheus-rulefiles","prometheus":"k8s"}},"alerting":{"alertmanagers":[{"namespace":"monitoring","name":"alertmanager-main","port":"web"}]}}},{"apiVersion":"monitoring.coreos.com/v1","kind":"ServiceMonitor","metadata":{"name":"example","labels":{"k8s-app":"prometheus"}},"spec":{"selector":{"matchLabels":{"k8s-app":"prometheus"}},"endpoints":[{"port":"web","interval":"30s"}]}},{"apiVersion":"monitoring.coreos.com/v1","kind":"Alertmanager","metadata":{"name":"alertmanager-main"},"spec":{"replicas":3,
      "securityContext": {}}}]
  labels:
    alm-catalog: rh-operators
  name: prometheusoperator.0.29.0
  namespace: {NAMESPACE}
spec:
  apiservicedefinitions: {}
  customresourcedefinitions:
    owned:
      - description: A running Prometheus instance
        displayName: Prometheus
        kind: Prometheus
        name: prometheuses.monitoring.coreos.com
        resources:
          - kind: StatefulSet
            name: ''
            version: v1beta2
          - kind: Pod
            name: ''
            version: v1
        specDescriptors:
          - description: Desired number of Pods for the cluster
            displayName: Size
            path: replicas
            x-descriptors:
              - 'urn:alm:descriptor:com.tectonic.ui:podCount'
          - description: A selector for the ConfigMaps from which to load rule files
            displayName: Rule Config Map Selector
            path: ruleSelector
            x-descriptors:
              - 'urn:alm:descriptor:com.tectonic.ui:selector:core:v1:ConfigMap'
          - description: ServiceMonitors to be selected for target discovery
            displayName: Service Monitor Selector
            path: serviceMonitorSelector
            x-descriptors:
              - >-
                urn:alm:descriptor:com.tectonic.ui:selector:monitoring.coreos.com:v1:ServiceMonitor
          - description: The ServiceAccount to use to run the Prometheus pods
            displayName: Service Account
            path: serviceAccountName
            x-descriptors:
              - 'urn:alm:descriptor:io.kubernetes:ServiceAccount'
          - description: >-
              Limits describes the minimum/maximum amount of compute resources
              required/allowed
            displayName: Resource Requirements
            path: resources
            x-descriptors:
              - 'urn:alm:descriptor:com.tectonic.ui:resourceRequirements'
        version: v1
      - description: >-
          A Prometheus Rule configures groups of sequentially evaluated
          recording and alerting rules.
        displayName: Prometheus Rule
        kind: PrometheusRule
        name: prometheusrules.monitoring.coreos.com
        version: v1
      - description: Configures prometheus to monitor a particular k8s service
        displayName: Service Monitor
        kind: ServiceMonitor
        name: servicemonitors.monitoring.coreos.com
        resources:
          - kind: Pod
            name: ''
            version: v1
        specDescriptors:
          - description: The label to use to retrieve the job name from
            displayName: Job Label
            path: jobLabel
            x-descriptors:
              - 'urn:alm:descriptor:com.tectonic.ui:label'
          - description: A list of endpoints allowed as part of this ServiceMonitor
            displayName: Endpoints
            path: endpoints
            x-descriptors:
              - 'urn:alm:descriptor:com.tectonic.ui:endpointList'
        version: v1
      - description: Configures an Alertmanager for the namespace
        displayName: Alertmanager
        kind: Alertmanager
        name: alertmanagers.monitoring.coreos.com
        resources:
          - kind: StatefulSet
            name: ''
            version: v1beta2
          - kind: Pod
            name: ''
            version: v1
        specDescriptors:
          - description: Desired number of Pods for the cluster
            displayName: Size
            path: replicas
            x-descriptors:
              - 'urn:alm:descriptor:com.tectonic.ui:podCount'
          - description: >-
              Limits describes the minimum/maximum amount of compute resources
              required/allowed
            displayName: Resource Requirements
            path: resources
            x-descriptors:
              - 'urn:alm:descriptor:com.tectonic.ui:resourceRequirements'
        version: v1
  description: >
    The Prometheus Operator for Kubernetes provides easy monitoring definitions
    for Kubernetes services and deployment and management of Prometheus
    instances.


    Once installed, the Prometheus Operator provides the following features:


    * **Create/Destroy**: Easily launch a Prometheus instance for your
    Kubernetes namespace, a specific application or team easily using the
    Operator.


    * **Simple Configuration**: Configure the fundamentals of Prometheus like
    versions, persistence, retention policies, and replicas from a native
    Kubernetes resource.


    * **Target Services via Labels**: Automatically generate monitoring target
    configurations based on familiar Kubernetes label queries; no need to learn
    a Prometheus specific configuration language.


    ### Other Supported Features


    **High availability**


    Multiple instances are run across failure zones and data is replicated. This
    keeps your monitoring available during an outage, when you need it most.


    **Updates via automated operations**


    New Prometheus versions are deployed using a rolling update with no
    downtime, making it easy to stay up to date.


    **Handles the dynamic nature of containers**


    Alerting rules are attached to groups of containers instead of individual
    instances, which is ideal for the highly dynamic nature of container
    deployment.
  displayName: Prometheus Operator
  icon:
    - base64data: >-
        PHN2ZyB3aWR0aD0iMjQ5MCIgaGVpZ2h0PSIyNTAwIiB2aWV3Qm94PSIwIDAgMjU2IDI1NyIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiBwcmVzZXJ2ZUFzcGVjdFJhdGlvPSJ4TWlkWU1pZCI+PHBhdGggZD0iTTEyOC4wMDEuNjY3QzU3LjMxMS42NjcgMCA1Ny45NzEgMCAxMjguNjY0YzAgNzAuNjkgNTcuMzExIDEyNy45OTggMTI4LjAwMSAxMjcuOTk4UzI1NiAxOTkuMzU0IDI1NiAxMjguNjY0QzI1NiA1Ny45NyAxOTguNjg5LjY2NyAxMjguMDAxLjY2N3ptMCAyMzkuNTZjLTIwLjExMiAwLTM2LjQxOS0xMy40MzUtMzYuNDE5LTMwLjAwNGg3Mi44MzhjMCAxNi41NjYtMTYuMzA2IDMwLjAwNC0zNi40MTkgMzAuMDA0em02MC4xNTMtMzkuOTRINjcuODQyVjE3OC40N2gxMjAuMzE0djIxLjgxNmgtLjAwMnptLS40MzItMzMuMDQ1SDY4LjE4NWMtLjM5OC0uNDU4LS44MDQtLjkxLTEuMTg4LTEuMzc1LTEyLjMxNS0xNC45NTQtMTUuMjE2LTIyLjc2LTE4LjAzMi0zMC43MTYtLjA0OC0uMjYyIDE0LjkzMyAzLjA2IDI1LjU1NiA1LjQ1IDAgMCA1LjQ2NiAxLjI2NSAxMy40NTggMi43MjItNy42NzMtOC45OTQtMTIuMjMtMjAuNDI4LTEyLjIzLTMyLjExNiAwLTI1LjY1OCAxOS42OC00OC4wNzkgMTIuNTgtNjYuMjAxIDYuOTEuNTYyIDE0LjMgMTQuNTgzIDE0LjggMzYuNTA1IDcuMzQ2LTEwLjE1MiAxMC40Mi0yOC42OSAxMC40Mi00MC4wNTYgMC0xMS43NjkgNy43NTUtMjUuNDQgMTUuNTEyLTI1LjkwNy02LjkxNSAxMS4zOTYgMS43OSAyMS4xNjUgOS41MyA0NS40IDIuOTAyIDkuMTAzIDIuNTMyIDI0LjQyMyA0Ljc3MiAzNC4xMzguNzQ0LTIwLjE3OCA0LjIxMy00OS42MiAxNy4wMTQtNTkuNzg0LTUuNjQ3IDEyLjguODM2IDI4LjgxOCA1LjI3IDM2LjUxOCA3LjE1NCAxMi40MjQgMTEuNDkgMjEuODM2IDExLjQ5IDM5LjYzOCAwIDExLjkzNi00LjQwNyAyMy4xNzMtMTEuODQgMzEuOTU4IDguNDUyLTEuNTg2IDE0LjI4OS0zLjAxNiAxNC4yODktMy4wMTZsMjcuNDUtNS4zNTVjLjAwMi0uMDAyLTMuOTg3IDE2LjQwMS0xOS4zMTQgMzIuMTk3eiIgZmlsbD0iI0RBNEUzMSIvPjwvc3ZnPg==
      mediatype: image/svg+xml
  install:
    spec:
      deployments:
        - name: prometheus-operator
          spec:
            replicas: 1
            selector:
              matchLabels:
                k8s-app: prometheus-operator
            template:
              metadata:
                labels:
                  k8s-app: prometheus-operator
              spec:
                containers:
                  - args:
                      - '-namespaces=$(K8S_NAMESPACE),{TARGET_NAMESPACE}'
                      - '-manage-crds=false'
                      - '-logtostderr=true'
                      - >-
                        --config-reloader-image=quay.io/coreos/configmap-reload:v0.0.1
                      - >-
                        --prometheus-config-reloader=quay.io/coreos/prometheus-config-reloader:v0.29.0
                    env:
                      - name: K8S_NAMESPACE
                        valueFrom:
                          fieldRef:
                            fieldPath: metadata.namespace
                    image: >-
                      quay.io/coreos/prometheus-operator@sha256:5abe9bdfd93ac22954e3281315637d9721d66539134e1c7ed4e97f13819e62f7
                    name: prometheus-operator
                    ports:
                      - containerPort: 8080
                        name: http
                    resources:
                      limits:
                        cpu: 200m
                        memory: 100Mi
                      requests:
                        cpu: 100m
                        memory: 50Mi
                    securityContext:
                      allowPrivilegeEscalation: false
                      readOnlyRootFilesystem: true
                nodeSelector:
                  beta.kubernetes.io/os: linux
                serviceAccount: prometheus-operator-0-29-0
      permissions:
        - rules:
            - apiGroups:
                - ''
              resources:
                - nodes
                - services
                - endpoints
                - pods
              verbs:
                - get
                - list
                - watch
            - apiGroups:
                - ''
              resources:
                - configmaps
              verbs:
                - get
          serviceAccountName: prometheus-k8s
        - rules:
            - apiGroups:
                - apiextensions.k8s.io
              resources:
                - customresourcedefinitions
              verbs:
                - '*'
            - apiGroups:
                - monitoring.coreos.com
              resources:
                - alertmanagers
                - prometheuses
                - prometheuses/finalizers
                - alertmanagers/finalizers
                - servicemonitors
                - prometheusrules
              verbs:
                - '*'
            - apiGroups:
                - apps
              resources:
                - statefulsets
              verbs:
                - '*'
            - apiGroups:
                - ''
              resources:
                - configmaps
                - secrets
              verbs:
                - '*'
            - apiGroups:
                - ''
              resources:
                - pods
              verbs:
                - list
                - delete
            - apiGroups:
                - ''
              resources:
                - services
                - endpoints
              verbs:
                - get
                - create
                - update
            - apiGroups:
                - ''
              resources:
                - nodes
              verbs:
                - list
                - watch
            - apiGroups:
                - ''
              resources:
                - namespaces
              verbs:
                - list
                - get
                - watch
          serviceAccountName: prometheus-operator-0-29-0
    strategy: deployment
  keywords:
    - prometheus
    - monitoring
    - tsdb
    - alerting
  labels:
    alm-owner-prometheus: prometheusoperator
    alm-status-descriptors: prometheusoperator.0.29.0
  links:
    - name: Prometheus
      url: 'https://www.prometheus.io/'
    - name: Documentation
      url: 'https://coreos.com/operators/prometheus/docs/latest/'
    - name: Prometheus Operator
      url: 'https://github.com/coreos/prometheus-operator'
  maintainers:
    - email: openshift-operators@redhat.com
      name: Red Hat
  maturity: beta
  provider:
    name: Red Hat
  replaces: prometheusoperator.0.22.2
  selector:
    matchLabels:
      alm-owner-prometheus: prometheusoperator
  version: 0.29.0
```

 Now you can simply create this new CSV by doing `oc apply` and use by referencing this new CSV in the subscription in the `startingCSV` field.

 Also for each extra namespace added, you must give access to prometheus operator's service account in the extra namespace. The permissions needed are below

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prometheus-{EXTRA_NAMESPACE}
  namespace: {EXTRA_NAMESPACE}
rules:
  - apiGroups:
      - ''
    resources:
      - nodes
      - services
      - endpoints
      - pods
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - configmaps
    verbs:
      - get
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prometheus-binding-{EXTRA_NAMESPACE}
  namespace: {EXTRA_NAMESPACE}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-{EXTRA_NAMESPACE}
subjects:
  - kind: ServiceAccount
    name: prometheus-k8s
    namespace: {NAMESPACE}
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prometheus-operator-{EXTRA_NAMESPACE}
  namespace: {EXTRA_NAMESPACE}
rules:
  - apiGroups:
      - apiextensions.k8s.io
    resources:
      - customresourcedefinitions
    verbs:
      - '*'
  - apiGroups:
      - monitoring.coreos.com
    resources:
      - alertmanagers
      - prometheuses
      - prometheuses/finalizers
      - alertmanagers/finalizers
      - servicemonitors
      - prometheusrules
    verbs:
      - '*'
  - apiGroups:
      - apps
    resources:
      - statefulsets
    verbs:
      - '*'
  - apiGroups:
      - ''
    resources:
      - configmaps
      - secrets
    verbs:
      - '*'
  - apiGroups:
      - ''
    resources:
      - pods
    verbs:
      - list
      - delete
  - apiGroups:
      - ''
    resources:
      - services
      - endpoints
    verbs:
      - get
      - create
      - update
  - apiGroups:
      - ''
    resources:
      - nodes
    verbs:
      - list
      - watch
  - apiGroups:
      - ''
    resources:
      - namespaces
    verbs:
      - list
      - get
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prometheus-operator-binding-{EXTRA_NAMESPACE}
  namespace: {EXTRA_NAMESPACE}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-operator-{EXTRA_NAMESPACE}
subjects:
  - kind: ServiceAccount
    name: prometheus-operator-0-29-0
    namespace: {NAMESPACE}
```

Replace `{NAMESPACE}` and `{EXTRA_NAMESPACE}` with appropriate values. In case of multiple namespaces, create one copy of above manifest for each extra namespace. The name of the service account should match the name provided in CSV above.