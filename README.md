# AI-Stacks
This is a set of jobs to be run on the AI-Stacks pipeline to product optimized AI Stacks

## Creating the pipeline

```bash
oc login ...
oc create -f ai-stacks.yaml
```

## Create the AI-Stacks BuildConfigs

```bash
cd stack-images/ ; for d in `ls -1` ; do oc process -f ${d}/buildConfigAndImageStream.yaml | oc create -f - ; done ; cd ..
```

FIXME radanalytics will fail with this for-loop!

# Operations

## Cleanup and Garbage Collection

`oc adm prune builds --confirm --namespace aicoe`

# Known Issues

* location of image in tensorflow test Dockerfile
* version used for tensorflow is in Jenkinsfile and BC template as a parameter