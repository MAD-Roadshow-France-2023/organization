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
