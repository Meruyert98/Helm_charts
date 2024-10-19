# Single Helm Chart, Multiple environments

[Youtube](https://www.youtube.com/watch?v=ZvG8rfJ4XxM&ab_channel=DeekshithSN)

Create helm chart, it will create standard helm directory

```
helm create ui
```

Clean up, what is not needed.
Delete everything inside files and replace with your manifests.

Write values.yaml

- define variable to values
- replicas, labels, image can be different

Render your helm charts

```
helm template dev-dep -f .\ui\values-dev.yaml .\ui\ > dev.yaml
```

Show debug errors of dev

```
helm template dev-dep -f .\ui\values-dev.yaml .\ui\  --debug > dev.yaml
```

Show helm template for qa

```
helm template dev-dep -f .\ui\values-qa.yaml .\ui\   > qa.yaml
```

Show helm template for prod

```
helm template dev-dep -f .\ui\values-prod.yaml .\ui\   > prod.yaml
```

Configure configmap.yaml for different environments

Configure config.xml for different environments
