#  Project 02: Helm Chart - Package Management for Kubernetes

### Learning Objectives
- Understand the concept and purpose of Helm and Helm Charts in the Kubernetes ecosystem.
- Identify the structure and main components of a Helm Chart.
- Learn how to create, customize, and deploy applications on Kubernetes using Helm Charts.
- Gain hands-on experience with Helm commands for installation, upgrade, and removal of applications.
- Know how to modify deployment parameters using the `values.yaml` file.

---

### Prerequisites
- Completed the Kubernetes fundamentals and on-premises cluster deployment module.
- Familiarity with Kubernetes core concepts: Pods, Deployments, Services, Nodes.
- Basic experience with command-line tools (`kubectl`, terminal usage).
- A running Kubernetes cluster (on-premise or local, e.g., Minikube).
- Administrative access to the cluster for deploying applications.

---

## 1. Introduction To Helm 

### What is Helm ?
Helm is an open-source package manager for Kubernetes. It helps you define, install, and manage Kubernetes applications using packages called charts. Charts are collections of YAML files that describe a related set of Kubernetes resources.

### Why we need Helm ?
Helm simplifies the deployment and management of applications on Kubernetes. Without Helm, deploying applications often requires writing and maintaining multiple complex YAML files, making the process error-prone and difficult to manage as the application grows. Helm packages Kubernetes resources into reusable charts, allowing for easier installation, configuration, versioning, and sharing of applications. This significantly improves productivity, consistency, and maintainability for DevOps teams working with Kubernetes.

## 2. Helm Chart Structure And Components

### Helm Chart Structure 

A Helm Chart is organized in a specific directory structure that makes it easy to manage, reuse, and share Kubernetes application deployments. The main components of a Helm Chart include:

- **Chart.yaml**: This file contains metadata about the chart such as its name, version, description, and maintainers.

- **values.yaml**: The default configuration values for the chart. Users can override these values during installation or upgrade.

- **templates/**: A directory containing Kubernetes manifest templates (YAML files) that will be rendered using the values provided in `values.yaml`.

- **charts/**: (Optional) A directory for any chart dependencies.

- **README.md**: (Optional) Documentation about how to use the chart.

- **.helmignore**: (Optional) Specifies files and directories to ignore when packaging the chart.

**Typical Helm Chart directory structure:**
```
my-chart/
  Chart.yaml
  values.yaml
  charts/
  templates/
  README.md
  .helmignore
```
Each file and directory has a specific role in defining, configuring, and deploying an application using Helm. Understanding the structure helps you customize and extend charts efficiently.

### Helm Components (Chart - Repository - Release - Revision)

- **Helm Charts**: is a collection of files that defines the structure and the configuration of a set of Kubernetest objects that are part of the same application. 
- **Helm Repository**: is a remote location for storing and sharing helm charts. It is similar to Github for storing code and DockerHub for storing docker images.
- **Helm Release**: When installing a chart, a release is created. So, A release = a single installation of a Helm chart.
- **Helm Revision**: is a specific version or snapshot of a release in Helm. Every time you install, upgrade, or roll back a Helm release, Helm creates a new revision number for that release. This allows you to track the history of changes made to the release and easily roll back to a previous revision if necessary. 

## 3. Helm Cheat Sheet
```bash
# General Help
helm --help                                      # Display help and usage for Helm commands

## Search Charts
helm search repo <repo_name>                     # Search for charts in locally added repositories
helm search hub <keyword>                        # Search for charts on Artifact Hub (all public repositories)

## Repository Management
helm repo list                                   # List all locally added Helm repositories
helm repo add <repo_name> <url>                  # Add a new Helm repository
helm repo remove <repo_name>                     # Remove a repository from local configuration

## Release Management
helm list                                        # List releases in the current namespace
helm list --all-namespaces                       # List releases in all namespaces
helm status <release_name>                       # Show the status of a release
helm install <release_name> <chart> --namespace <ns>          # Install a chart into a specified namespace
helm install <release_name> <chart> --values <file/url>       # Install a chart with custom values
helm install <release_name> <chart> --dry-run --debug         # Simulate an install and show output (no resources created)
helm install <release_name> <chart> --dependency-update       # Install and update chart dependencies automatically
helm uninstall <release_name>                                 # Uninstall (delete) a release

helm get all <release_name>                      # Get all information about a release
helm get manifest <release_name>                 # Show the manifest (YAML) of a release
helm get notes <release_name>                    # Show the NOTES.txt for a release (usage instructions)
helm get values <release_name>                   # Show the values used to deploy a release

## Revision & Upgrade
helm history <release_name>                      # Show the revision history for a release
helm upgrade <release_name> <chart>              # Upgrade a release with a new chart or values
helm upgrade <release_name> <chart> --atomic     # Upgrade a release, rolling back on failure
helm upgrade <release_name> <chart> --install    # Upgrade if the release exists, or install if not
helm rollback <release_name> <revision_number>   # Rollback a release to a previous revision

## Chart Management
helm create <chart_name>                         # Create a new chart scaffold
helm package <chart-path>                        # Package a chart directory into a .tgz archive
helm lint <chart>                                # Lint (validate) a chart for errors
helm show all <chart>                            # Show all information about a chart
helm pull <chart>                                # Download a chart to local storage
helm dependency list <chart>                     # List chart dependencies
```

## 4. Hands-on: Creating, Customizing, and Deploying Applications with Helm Chart

