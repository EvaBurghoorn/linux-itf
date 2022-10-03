> Writing all that YAML for everything? No you silly!

# Helm - Your package manager for Kubernetes

![Helm](./helm.svg)

Helm is a package manager for Kubernetes. It allows you to fetch and install applications or to write some yourself.
Helm will be able to manage the lifecycle of your application, from installation to upgrades and even rollbacks.

Helm (today, v2 was not) is a clientside command line tool that does this all for you. Underneath is uses Kubernetes ConfigMaps to store all data it needs to function. The power of Helm is that it uses a templating engine called [Go Templates](https://golang.org/pkg/text/template/) to generate the YAML files that are needed to deploy your application. This means that you can use scripting logic inside it and use includes for common replicated parts like the labels, and have a configuration file for all options in one place! This is a huge advantage over writing all that YAML by hand.

:::note
Helm is not the only option to do this. It often is critisised for using a template engine which is sometimes hard to debug. There are other options like [Kustomize](https://kustomize.io/). But Helm is the most popular one and the one we will use in this course.
:::

## Installing Helm

Getting helm is easy. Just follow the instructions on the [Helm website](https://helm.sh/docs/intro/quickstart/).
Or just run the following command which we very conveniently copied for you (just say thank you after the class):

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

This adds a new `helm` command to your system. You can check if it is installed by running `helm version`.

## Using Helm

Helm will help us installing apps in Kubernetes, it does this by using a "Chart" (literally: an outline map exhibiting something). This is a collection of YAML templates it will deploy. On top of that Helm keeps track of the state of the deployment and can upgrade it for you or rollback if needed. It also supports running hooks on install/upgrade/rollback to facilitate application needs like a database migration.

### Chart Repositories

A chart can be found in the filesystem (local charts) or in a repository. A repository is a collection of charts. You can add repositories to your local helm installation by running `helm repo add <name> <url>`. This will add the repository to your local helm installation. Helm no longer has an official repository, but there are many others.

The [Artifact Hub](https://artifacthub.io/) is a good place to find charts and repositories (10774 apps and counting!). It keeps a large collection, we reccomend to sort by the number of stars to find the most popular ones. Let's take this [MariaDB chart by Bitnami](https://artifacthub.io/packages/helm/bitnami/mariadb) it offers a wide range of options to set! We'll explore this later.

For many applications there are existing charts to use! But we'll also be using this power to build our very own!

### Charts

What does a Chart look like? First of all it is a directory with files... Let's take a look at the [cert-manager chart](https://github.com/cert-manager/cert-manager/tree/master/deploy/charts/cert-manager) on GitHub for example:

- `Chart.yaml` - This is the metadata of the chart. It contains the name, version, description, logo, etc.
- `values.yaml` - This is the **default** configuration file for the chart. It contains all the options you can set for the chart.
- `templates/` - This is the directory containing all the templates. These are the files that will be rendered by Helm and deployed to Kubernetes.
  - `NOTES.txt` - This is a file that will be printed after the chart is deployed. It can contain useful instructions for the user.
  - `_helpers.tpl` - This is a file that contains helper functionsfor the templates.
- `charts/` - This is the directory containing all sub-charts, it is a way to import other charts into your chart. It is not always used.
- `README.md` - This is the README file for the chart.

Inside `templates/` we find all templates to be rendered and deployed. They often carry the name of the resource type like `deployment.yaml` or `service.yaml`. If you need more than one we often prefix them with what service they are used for, in case of cert-manager we see `webhook-deployment.yaml` for example.

When we do a `helm install` or `helm upgrade` we pass a configuration file. This file will be merged with the default values from `values.yaml` and then passed to the templates. This means that we can override any value we want. An advice is to take the original `values.yaml` and copy it to a new file, then edit that file and use it as your personal values. That way you have a good overview of all options and documentation in the comments.

### Using Charts

Using charts is as easy as `helm install`. That's all!

Let's try and install something... We'll look into [Nextcloud](https://artifacthub.io/packages/helm/nextcloud/nextcloud) a PHP based DropBox alternative.

We will have to add the repository first:

```bash
helm repo add nextcloud https://nextcloud.github.io/helm/
helm repo update
```

Now we can install it:

```bash
helm install nextcloud nextcloud/nextcloud
```

In the install we first give a name to our deployment, then we give the name of the chart and the repository it is in. This will install the latest version of the chart.
When done we get a ton of info, feel free to read it.

We can get a list now of all deployed apps in our current namespace:

```bash
$ helm list
NAME            NAMESPACE       REVISION        UPDATED                                         STATUS          CHART           APP VERSION
nextcloud       default         1               2022-10-03 20:39:13.014255745 +0200 CEST        deployed        nextcloud-3.1.2 24.0.5
```

You can also see the pods running!

So this is a little bit useless... We just pushed the default configuration...

We now have two methods (they both work for `upgrade` and `install`):

- We can make a `values.yaml` file
- We can use the `--set` flag

Let's try the `--set` flag first:

```bash
helm upgrade nextcloud nextcloud/nextcloud --set nextcloudUsername=admin,nextcloudPassword=supersecret
```

When we now look in `helm list` we see it is now at revision 2. But `supersecret` is not a very secure password... Let's undo that:

```bash
helm rollback nextcloud 1
```

Now we are back at revision 1. We can also see the history of our deployment:

```bash
$ helm history nextcloud
REVISION        UPDATED                                         STATUS          CHART           APP VERSION
1               Mon Oct  3 20:39:13 2022        superseded      nextcloud-3.1.2 24.0.5          Install complete
2               Mon Oct  3 20:42:29 2022        superseded      nextcloud-3.1.2 24.0.5          Upgrade complete
3               Mon Oct  3 20:43:14 2022        deployed        nextcloud-3.1.2 24.0.5          Rollback to 1
```

Let's try the `values.yaml` file now. We'll use the one from the chart as a base:

```bash
helm show values nextcloud/nextcloud > values.yaml
```

Now change the password in the file and run:

```bash
helm upgrade nextcloud nextcloud/nextcloud -f values.yaml
```

Cool! Now we have a Nextcloud instance running in our cluster! But we don't need it so let's delete it:

```bash
helm uninstall nextcloud
```

When we now look in our running pods we see that they are all gone.

## Writing Charts

We found out what Charts can do for us. So time to build our own!

To create a Chart just ask Helm to place some files on your disk: (tip: change the name)

```bash
helm create name-of-chart
```

This will create a directory with the name of your chart. Inside you will find the files we saw earlier. Let's take a look at the `Chart.yaml`:

```yaml
apiVersion: v2
name: test
description: A Helm chart for Kubernetes

type: application

version: 0.1.0
appVersion: 1.16.0
```

We see it has a name and a description, you might want to change the last one. We see `version` and `appVersion`. The first is the version of the chart, you should increase them following the [semver](https://semver.org/) rules on each change. The second is the version of the application you are deploying.

In `templates/` you will find the templates. Helm will provide you with:

- `deployment.yaml` - A deployment with a single container by default
- `ingress.yaml` - An ingress service which can be enabled in the `values.yaml`
- `service.yaml` - A service file that points to the deployment
- `serviceaccount.yaml` - A service account, this should only be used when your application needs to talk to the Kubernetes API
- `tests/test-connection.yaml` - A test to check if the application is running, this is a unit test to see if the chart works

Feel free to explore those files!

We take a look at `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: { { include "test.fullname" . } }
  labels: { { - include "test.labels" . | nindent 4 } }
spec:
  type: { { .Values.service.type } }
  ports:
    - port: { { .Values.service.port } }
      targetPort: http
      protocol: TCP
      name: http
  selector: { { - include "test.selectorLabels" . | nindent 4 } }
```

You notice this is a bit different than what we are used to! They contain Go Template syntax (quite diffrent than Go itself...).

We can use the `{{ }}` syntax to insert values. We can also use `{{-` and `-}}` to remove whitespace. This is useful when you want to have a newline in your template but don't want it to be rendered.

We can also use `{{- if }}` and `{{- end }}` to create if statements. We can also use `{{- range }}` and `{{- end }}` to create loops.

The `{{- include` comes from the helpers written in `_helpers.tpl`. The common ones are `<name>.fullname` and `<name>.labels`. You can also use `<name>.selectorLabels` when needed

TODO WRITE MORE
