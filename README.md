<!--
    Copyright (c) 2021-2022 Contributors to the Eclipse Foundation

    See the NOTICE file(s) distributed with this work for additional
    information regarding copyright ownership.

    This program and the accompanying materials are made available under the
    terms of the Apache License, Version 2.0 which is available at
    https://www.apache.org/licenses/LICENSE-2.0.

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
    WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
    License for the specific language governing permissions and limitations
    under the License.

    SPDX-License-Identifier: Apache-2.0
-->

# Discovery Finder
The Discovery Finder is a logical and architectural component of Tractus-X. The source code under this folder contains reference implementations of the SLDT Discovery Finder.

## Build Packages
Run `mvn clean install` to run unit tests, build and install the package.

## Run Package Locally
To check whether the build was successful, you can start the resulting JAR file from the build process by running `java -jar backend/target/discovery-finder-backend-{current-version}.jar --spring.profiles.active=local`.

## Build Docker
Run `docker build -f backend/Dockerfile -t sldt-discovery-finder .`

In case you want to publish your image into a remote container registry, apply the tag accordingly and `docker push` the image.

## Deploy using Helm and K8s
If you have a running Kubernetes cluster available, you can deploy the Discovery Finder using our Helm Chart, which is located under `charts/discoveryfinder`.
In case you don't have a running cluster, you can set up one by yourself locally, using [minikube](https://minikube.sigs.k8s.io/docs/start/).
In the following, we will use a minikube cluster for reference.

Before deploying the Discovery Finder, enable a few add-ons in your minikube cluster by running the following commands:

`minikube addons enable storage-provisioner`

`minikube addons enable default-storageclass`

`minikube addons enable ingress`

Fetch all dependencies by running `helm dep up charts/discoveryfinder`.

In order to deploy the helm chart, first create a new namespace "discovery": `kubectl create namespace discovery`.

Then run `helm install discoveryfinder -n discovery charts/discoveryfinder`. This will set up a new helm deployment in the discovery namespace. By default, the deployment contains the Discovery Finder instance itself, and a Postgresql.

Check that the two containers are running by calling `kubectl get pod -n discovery`.

To access the Discovery Finder API from the host, you need to configure the `Ingress` resource.
By default, the `Ingress` is disabled.

If you enable the `Ingress`, the Discovery Finder exposes the API on https://minikube/discovery/discoveryfinder.
For that to work, you need to append `/etc/hosts` by running `echo "minikube $(minikube ip)" | sudo tee -a /etc/hosts`.

For automated certificate generation, use and configure [cert-manager](https://cert-manager.io/).
By default, authentication is deactivated, please adjust `discoveryfinder.authentication` if needed.

## Parameters
The Helm Chart can be configured using the following parameters. For a full overview, please see the [values.yaml](./charts/discoveryfinder/values.yaml).

### Discovery Finder parameters
| Key                                                                                      | Type                      | Default                           | Description                                                                                                                                                                                                                              |
|------------------------------------------------------------------------------------------|---------------------------|-----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| discoveryfinder.authentication                                                           | bool                      | `false`                           | Enables OAuth2 based authentication/authorization                                                                                                                                                                                        |
| discoveryfinder.containerPort                                                            | int                       | `4243`                            | Containerport                                                                                                                                                                                                                            |
| discoveryfinder.dataSource.driverClassName                                               | string                    | `"org.postgresql.Driver"`         | The driver class name for the database connection                                                                                                                                                                                        |
| discoveryfinder.dataSource.password                                                      | string                    | `"password"`                      | Datasource password                                                                                                                                                                                                                      |
| discoveryfinder.dataSource.sqlInitPlatform                                               | string                    | `"pg"`                            | Datasource InitPlatform                                                                                                                                                                                                                  |
| discoveryfinder.dataSource.url                                                           | string                    | `"jdbc:postgresql://database:5432"` | Datasource URL                                                                                                                                                                                                                           |
| discoveryfinder.dataSource.user                                                          | string                    | `"user"`                          | Datasource user                                                                                                                                                                                                                          |
| discoveryfinder.host                                                                     | string                    | `"localhost"`                     | This value is used by the Ingress object (if enabled) to route traffic                                                                                                                                                                   |
| discoveryfinder.idp.issuerUri                                                            | string                    | `""`               | The issuer URI of the OAuth2 identity provider                                                                                                                                                                                           |
| discoveryfinder.idp.publicClientId                                                       | string                    | `""`                   | ClientId                                                                                                                                                                                                                                 |
| discoveryfinder.image.imagePullPolicy                                                    | string                    | `"IfNotPresent"`                  | ImagepullPolicy                                                                                                                                                                                                                          |
| discoveryfinder.image.registry                                                           | string                    | `"ghcr.io/catenax-ng"`            | Image registry                                                                                                                                                                                                                           |
| discoveryfinder.image.repository                                                         | string                    | `"sldt-discovery-finder"`         | Image repository                                                                                                                                                                                                                         |
| discoveryfinder.image.version                                                            | string                    | `""`                              | Version of image. By default the app Version from Chart.yml is used. You can overwrite the version to use an  other version of sldt-discovery-finder                                                                                     |
| discoveryfinder.ingress.annotations."cert-manager.io/cluster-issuer"                     | string                    | `"selfsigned-cluster-issuer"`     |                                                                                                                                                                                                                                          |
| discoveryfinder.ingress.annotations."nginx.ingress.kubernetes.io/cors-allow-credentials" | string                    | `"true"`                          |                                                                                                                                                                                                                                          |
| discoveryfinder.ingress.annotations."nginx.ingress.kubernetes.io/enable-cors"            | string                    | `"true"`                          |                                                                                                                                                                                                                                          |
| discoveryfinder.ingress.annotations."nginx.ingress.kubernetes.io/rewrite-target"         | string                    | `"/$2"`                           |                                                                                                                                                                                                                                          |
| discoveryfinder.ingress.annotations."nginx.ingress.kubernetes.io/use-regex"              | string                    | `"true"`                          |                                                                                                                                                                                                                                          |
| discoveryfinder.ingress.annotations."nginx.ingress.kubernetes.io/x-forwarded-prefix"     | string                    | `"/discoveryfinder"`              |                                                                                                                                                                                                                                          |
| discoveryfinder.ingress.className                                                        | string                    | `"nginx"`                         | The Ingress class name                                                                                                                                                                                                                   |
| discoveryfinder.ingress.enabled                                                          | bool                      | `false`                           | Configures if an Ingress resource is created                                                                                                                                                                                             |
| discoveryfinder.ingress.tls                                                              | bool                      | `false`                           | Configures whether the `Ingress` should include TLS configuration. In that case, a separate `Secret` (as defined by `registry.ingress.tlsSecretName`) needs to be provided manually or by using [cert-manager](https://cert-manager.io/) |
| discoveryfinder.ingress.urlPrefix                                                        | string                    | `"/discoveryfinder"`              | The url prefix that is used by the Ingress resource to route traffic                                                                                                                                                                     |
| discoveryfinder.replicaCount                                                             | int                       | `1`                               | Replica count                                                                                                                                                                                                                            |
| discoveryfinder.resources.limits.memory                                                  | string                    | `"1024Mi"`                        | Resources limit memory                                                                                                                                                                                                                   |
| discoveryfinder.resources.requests.memory                                                | string                    | `"512Mi"`                         | Resources request memory                                                                                                                                                                                                                 |
| discoveryfinder.service.port                                                             | int                       | `8080`                            | Service port                                                                                                                                                                                                                             |
| discoveryfinder.service.type                                                             | string                    | `"ClusterIP"`                     | Service type                                                                                                                                                                                                                             |
| discoveryfinder.properties.discoveryfinder.initialEndpoints                              | list of initialEndpoints. |  | This attribute is optional. If it is given, the defined initial endpoints are saved in the database when the application is started.                                                                                                     |
| discoveryfinder.properties.discoveryfinder.initialEndpoints[*].description               | string                    | `` | Description. Value is optional                                                                                                                                                                                                           |
| discoveryfinder.properties.discoveryfinder.initialEndpoints[*].documentation             | string                    | `` | Documentation. Value is optional                                                                                                                                                                                                         |
| discoveryfinder.properties.discoveryfinder.initialEndpoints[*].endpointAddress           | string                    | `` | EndpointAddress. Value is required if initial endpoint is defined.                                                                                                                                                                       |
| discoveryfinder.properties.discoveryfinder.initialEndpoints[*].type                      | string                    | `` | Type. Value is required if initial endpoint is defined.                                                                                                                                                                                  |
### PostgreSQL parameters
| Key | Type | Default                             | Description                                                                                   |
|-----|------|-------------------------------------|-----------------------------------------------------------------------------------------------|
| enablePostgres | bool | `true`                              | If enabled, the postgreSQL instance will be run. Disable if you use your own hosted postgreSQL. |
| postgresql.auth.database | string | `"discoveryfinder"`                 | Database name                                                                                 |
| postgresql.auth.password | string | `"password"`                        | Password for authentication at the database                                                   |
| postgresql.auth.username | string | `"catenax"`                         | Username that is used to authenticate at the database                                         |
| postgresql.primary.persistence.enabled | bool | `true`                              | Persistence enabled                                                                           |
| postgresql.primary.persistence.size | string | `"50Gi"`                            | Size of persistence                                                                           |
| postgresql.service.ports.postgresql | int | `5432`                              | Size of the PersistentVolume that persists the data                                           |
