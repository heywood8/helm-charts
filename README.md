# Heywood helm-charts

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/heywood8-helm-charts)](https://artifacthub.io/packages/search?repo=heywood8-helm-charts) [![License](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/Apache-2.0) ![Release Charts](https://github.com/heywood8/helm-charts/workflows/Release%20Charts/badge.svg?branch=main) [![Releases downloads](https://img.shields.io/github/downloads/heywood8/helm-charts/total.svg)](https://github.com/heywood8/helm-charts/releases)

This functionality is in beta and is subject to change. The code is provided as-is with no warranties. Beta features are not subject to the support SLA of official GA features.

## Usage

[Helm](https://helm.sh) must be installed to use the charts.
Please refer to Helm's [documentation](https://helm.sh/docs/) to get started.

Once Helm is set up properly, add the repository as follows:

```console
helm repo add heywood8 https://heywood8.github.io/helm-charts
```

You can then run `helm search repo heywood8` to see the charts.

## Helm Charts

* [Redisinsight](charts/redisinsight/)
* [Postgres Backup Local](charts/postgres-backup-local/)
* [Nominatim](charts/nominatim/)
* [NetworkPolicy](charts/networkpolicy)
* Coming soon...

## Contributing

The source code of all [Helm](https://helm.sh) charts can be found on Github: <https://github.com/heywood8/helm-charts/>

<!-- Keep full URL links to repo files because this README syncs from main to gh-pages.  -->
I'd love to have you contribute! For now I have no contribution guidelines

## License

<!-- Keep full URL links to repo files because this README syncs from main to gh-pages.  -->
[MIT License](https://github.com/prometheus-community/helm-charts/blob/main/LICENSE).

## Helm charts build status

![Release Charts](https://github.com/heywood8/helm-charts/workflows/Release%20Charts/badge.svg?branch=main)
