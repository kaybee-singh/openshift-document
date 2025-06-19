# Openshift Pipelines

Openshift pipeline is a CICD solution based on Tekton pipelines.

Components of pipeline. 

Task - A collection of steps which we need to run. A steps can be a set of commands or a single command.
Pipeline - Pipeline will have a list of different tasks which we need to execute.
PipelineRun - PipelineRun will actually execute the pipeline.

So we need 3 yamls.

## Lets create a simple 3 step pipeline and then run it with pipelinerun.
Create a task file. Which expects a task input as well in form of parameter.
```yaml
cat task.yaml 
apiVersion: tekton.dev/v1
kind: task
metadata:
  name: task1
  namespace: pipe
spec:
  params:
    - name: MESSAGE
      type: string
  steps:
    - name: echo
      image: registry.access.redhat.com/ubi8/ubi
      script: |
        echo "$(params.MESSAGE)"
```
- Create a pipeline to run above task.
- In pipeline we have specified the params twice. One is under spec.params and other is spec.tasks.params. spec.params tell that pipeline expects an input as MESSAGE params.
- As task also expects an input so we are passing the input to task by spec.tasks.params
```yaml
cat pipeline1.yaml 
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: pipeline1
  namespace: pipe
spec:
  params: 
    - name: MESSAGE
      type: string
  tasks:
    - name: run-echo-task
      taskRef:
        name: task1
      params: 
       - name: MESSAGE
         value: $(params.MESSAGE)
```
- Now create a pipelinerun to execute the pipeline.
```yaml
cat run1.yaml 
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  name: run1
  namespace: pipe
spec:
  pipelineRef:
    name: pipeline1
  params:
    - name: MESSAGE
      value: "Hello It is wokring :)"
```
- Execute above
```bash
oc apply -f run1.yaml
```
- Now check logs
```bash
tkn pipelinerun list
```
```bash
tkn pipelinerun logs run1
```

# Task2 - Using workspaces in pipeline

## Workspace is a shared storage volume used to tasks so that they can shared pass data between each other. It is like a shared folder.

We need to define workspace at below places:-

1. task - It declares that task expects a workspace.

```yaml
spec:
  workspaces:
    - name: source
  steps:
    - name: list
      image: busybox
      script: |
        ls -l $(workspaces.source.path)
```
3. pipeline - Declares a named workspace and connects it to the task.
```yaml
spec:
  workspaces:
    - name: shared
  tasks:
    - name: show-files
      taskRef:
        name: show-task
      workspaces:
        - name: source         # 👈 task's workspace
          workspace: shared    # 👈 pipeline's workspace
```

5. pipelinerun - Provides the actual volume(PVC, dir) which will be then attached to the task
```yaml
spec:
  workspaces:
    - name: shared
      volumeClaimTemplate:
        metadata:
          name: shared-pvc
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 500Mi
```
