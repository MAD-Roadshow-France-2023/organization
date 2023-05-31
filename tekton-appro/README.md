# Tâche Tekton qui demande une approbation via Slack

Voir le projet [Tekton-Pipeline-Slack-Bot](https://github.com/nmasse-itix/Tekton-Pipeline-Slack-Bot).

```sh
oc new-project test-tekton
oc create secret generic tekton-tokens --from-literal=bot-token=xoxb-.... --from-literal=app-token=xapp-....
oc apply -f task-foo.yaml
oc apply -f task-bar.yaml
oc apply -f task-slack-approval.yaml
oc apply -f pipeline.yaml
oc create -f pipelinerun.yaml
```
