# Organization

## Durée de la session

30 minutes

## Scénario

Etat initial:

* L'application Quarkus est dans le repository Git
* Les environnement de test et prod sont vierges
* La plateforme (DevSpaces, ACS, Tekton) est déjà installée

Etape 1:

* Nicolas déploie l'environnement de test via Helm + Opérateur
* Nicolas déploie l'environnement de prod via Helm + VM

Etape 2:

* Guillaume ouvre DevSpaces pour éditer le projet
* Montrer que la base de données est provisionnée avec le workspace (via le plugin openshift)
* Lancement de l'appli
* Modifications
* Live reload Quarkus
* Commit

Etape 3:

* Guillaume lance le pipeline Tekton
* Le pipeline construit l'application et la déploie en test
* Le pipeline attend l'approbation via slack pour continuer
* Nicolas donne l'appro
* Le pipeline déploie en prod
* Nicolas ou Guillaume montre que l'application fonctionne en test et en prod

Etape 4:

* Nicolas est le pirate, il veut changer l'image utilisée en prod
* ACS bloque le déploiement
* Nicolas veut pirater la supply chain
* Guillaume lui montre Tekton Chains, il renonce.

## TODO

* 🟢 **Guillaume**: Créer ou trouver une application Quarkus stateful (si possible avec support d'une base de données simple à déployer comme postgresql. [Exemple 1](https://github.com/nmasse-itix/demo-appdev))
* 🟢 **Guillaume**: Base de données de test: [Opérateur](https://github.com/MAD-Roadshow-France-2023/gitops/tree/main/kustomize/postgres)
* 🟢 **Nicolas**: Base de données de prod: [préparer le cloud-init](cloud-init/README.md)
* 🟢 **Guillaume**: [Devfile](https://github.com/MAD-Roadshow-France-2023/devspaces/blob/main/devfile.yaml) (composant DB + plugin OCP)
* 🟢 **Nicolas**: GitOps (Setup [Tekton](https://github.com/nmasse-itix/demo-apimgmt/tree/gitops/infrastructure/templates), Tekton Chains, ACS, )
* 🟢 **Nicolas**: [Configurer ACS pour Sigstore](acs/README.md)
* 🟢 **Nicolas**: Faire le chart Helm de l'environnement de test & prod
* 🟢 **Guillaume**: [Créer le pipeline Tekton](https://github.com/MAD-Roadshow-France-2023/devspaces/tree/main/tekton)
* 🟢 **Nicolas**: [Créer la tache Tekton qui demande l'appro via Slack](tekton-appro/README.md)
