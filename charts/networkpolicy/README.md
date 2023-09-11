# Network Policy Chart
[![Artifact HUB](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/heywood8-helm-charts)](https://artifacthub.io/packages/search?repo=heywood8-helm-charts

## Description

The networkpolicy chart deploys a single Kubernetes Network Policy object.

## Installation

    helm repo add heywood8 https://heywood8.github.io/helm-charts/

    helm install mypolicy heywood8/networkpolicy

## Configuration

The following table lists the configurable parameters of the networkpolicy chart and their default values.

Parameter | Description | Default
--- | --- | ---
`nameOverride` | override name of the chart component | .Release.Name
`apiVersion` | api version of k8s object | `"v1"`
`annotations` | annotations in yaml map format to be added to the object | `null`
`labels` | labels to add to Network Policy object | `null`
`podSelector.matchLabels` | Map of key/value pairs to be used to select pods with matching labels to apply Network Policy to | `null`
`egressRules` | array of egress rules to apply to matching pods. | `[]`
`ingressRules` | array of ingress rules to apply to matching pods. | `[]`
`ingressRules[].selectors` | array of network policy ingress traffic selector statements | `[]`
`ingressRules[].ports` | array of port specifications to apply the ingress rule on | `[]`
`egressRules[].selectors` | array of network policy egress traffic selector statements | `[]`
`egressRules[].ports` | array of port specifications to apply the egress rule on | `[]`

## Example Configuration

For some examples of values used to configure this chart, see [the ci/example values for this chart](./ci/ci-values.yaml)
