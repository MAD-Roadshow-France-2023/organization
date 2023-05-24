# Base de données de production en VM avec cloud-init

## Développement

```sh
sudo ./install.sh
psql -U appli -h 192.168.122.160 appli
```

## Ressources KubeVirt

```sh
oc new-project prod
oc create -f vm-database.yaml -n prod
```
