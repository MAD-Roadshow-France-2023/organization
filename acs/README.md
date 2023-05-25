# Déploiement et configuration d'ACS

```sh
oc apply -f acs.yaml
```

```sh
ROX_CENTRAL_ADMIN_PASSWORD=secret
export ROX_CENTRAL_ADDRESS="$(oc get route central -n stackrox -o go-template='{{.spec.host}}'):443"
curl -sfLo /usr/local/bin/roxctl  https://mirror.openshift.com/pub/rhacs/assets/4.0.0/bin/Linux/roxctl
chmod 755 /usr/local/bin/roxctl
POLICY_JSON='{ "name": "init-token", "role":"Admin"}'
APIURL="https://$ROX_CENTRAL_ADDRESS/v1/apitokens/generate"
export ROX_API_TOKEN=$(curl -s -k -u admin:$ROX_CENTRAL_ADMIN_PASSWORD -H 'Content-Type: application/json' -X POST -d "$POLICY_JSON" "$APIURL" | jq -r '.token')
roxctl -e "$ROX_CENTRAL_ADDRESS" central init-bundles generate local-cluster --output-secrets cluster_init_bundle.yaml
oc apply -f cluster_init_bundle.yaml -n stackrox
```

```sh
oc apply -f scs.yaml
```

```sh
oc new-project dev
curl -sSfL https://github.com/sigstore/cosign/releases/download/v2.0.2/cosign-linux-amd64 -o ~/local/bin/cosign
chmod 755 ~/local/bin/cosign
COSIGN_PASSWORD=hackthis cosign generate-key-pair k8s://dev/code-signature-keypair
ls -l cosign.pub
```

```sh
oc create route reencrypt image-registry --service=image-registry -n openshift-image-registry
REGISTRY=$(oc get route -n openshift-image-registry image-registry -o jsonpath={.spec.host})
oc adm policy add-role-to-user admin -z default -n dev
REGISTRY_TOKEN="$(oc get secrets -n dev -o json | jq -r '.items[] | select(.metadata.annotations["kubernetes.io/service-account.name"]=="default" and .type=="kubernetes.io/dockercfg") | .data[".dockercfg"]' | base64 -d | jq -r '.["image-registry.openshift-image-registry.svc:5000"].password')"
podman login "$REGISTRY" --username sa --password "$REGISTRY_TOKEN"
oc get secrets -n dev -o json | jq -r '.items[] | select(.metadata.annotations["kubernetes.io/service-account.name"]=="default" and .type=="kubernetes.io/dockercfg") | .data[".dockercfg"]' | base64 -d | jq --arg registry "$REGISTRY" '.["image-registry.openshift-image-registry.svc:5000"] as $conf | { ($registry) : $conf}' > dockercfg
oc apply -n dev -f - <<EOF
kind: Secret
apiVersion: v1
metadata:
  name: external-registry
data:
  .dockercfg: $(base64 -w0 dockercfg)
type: kubernetes.io/dockercfg
EOF
oc secrets link default external-registry --for=pull -n dev

```

```sh
podman pull quay.io/podman/hello:latest
podman push quay.io/podman/hello:latest "$REGISTRY/dev/podman-hello:latest"
COSIGN_PASSWORD=hackthis cosign sign -y --upload=false --key k8s://dev/code-signature-keypair "$REGISTRY/dev/podman-hello:latest"
cosign verify --key k8s://dev/code-signature-keypair "$REGISTRY/dev/podman-hello:latest"
```

```sh
oc apply -f - <<EOF
kind: Deployment
apiVersion: apps/v1
metadata:
  name: signed
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: signed
  template:
    metadata:
      creationTimestamp: null
      labels:
        deployment: signed
    spec:
      containers:
        - name: signed
          image: >-
            $REGISTRY/dev/podman-hello:latest
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
EOF
```

```sh
podman push quay.io/podman/hello:latest "$REGISTRY/dev/podman-hello-unsigned:latest"
oc apply -f - <<EOF
kind: Deployment
apiVersion: apps/v1
metadata:
  name: unsigned
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: unsigned
  template:
    metadata:
      creationTimestamp: null
      labels:
        deployment: unsigned
    spec:
      containers:
        - name: unsigned
          image: >-
            $REGISTRY/dev/podman-hello-unsigned:latest
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
EOF
```
