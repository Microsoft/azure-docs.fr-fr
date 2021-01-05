---
title: Déployer une instance Prometheus dans un cluster Azure Red Hat OpenShift
description: Créez une instance Prometheus sur un cluster Azure Red Hat OpenShift pour superviser les métriques de votre application.
author: makdaam
ms.author: b-lejaku
ms.service: container-service
ms.topic: conceptual
ms.date: 06/17/2019
keywords: prometheus, aro, openshift, métriques, red hat
ms.openlocfilehash: 42ed8c90b35eba57fdc3db1f0ed93d44cf9a5e41
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996846"
---
# <a name="deploy-a-standalone-prometheus-instance-in-an-azure-red-hat-openshift-cluster"></a>Déployer une instance Prometheus autonome dans un cluster Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service le 30 juin 2022. La prise en charge de la création de nouveaux clusters Azure Red Hat OpenShift 3.11 se poursuit jusqu’au 30 novembre 2020. Après la mise hors service, les clusters Azure Red Hat OpenShift 3.11 restants seront arrêtés pour éviter des failles de sécurité.
> 
> Suivez ce guide pour [créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Si vous avez des questions spécifiques, n’hésitez pas à [nous contacter](mailto:arofeedback@microsoft.com).

Cet article décrit comment configurer une instance Prometheus autonome avec la découverte des services dans un cluster Azure Red Hat OpenShift.

> [!NOTE]
> L’accès administrateur client au cluster Azure Red Hat OpenShift n’est pas obligatoire.

Configuration cible :

- Un projet (prometheus-project), qui contient Prometheus et Alertmanager.
- Deux projets (app-project1 et app-project2), qui contiennent les applications à superviser.

Vous allez préparer certains fichiers de configuration Prometheus localement. Créez un dossier pour les stocker. Ces fichiers de configuration seront stockés dans le cluster en tant que secrets pour le cas où des jetons secrets leur seraient ajoutés plus tard.

## <a name="sign-in-to-the-cluster-by-using-the-oc-tool"></a>Connectez-vous au cluster à l’aide de l’outil OC

1. Ouvrez un navigateur web, accédez à la console web de votre cluster (https://openshift.*random-id*.*region*.azmosa.io).
2. Connectez-vous avec vos informations d’identification Azure.
3. Dans l’angle supérieur droit, sélectionnez votre nom d’utilisateur, puis **Copier la commande de connexion**.
4. Collez votre nom d’utilisateur dans le terminal que vous allez utiliser.

> [!NOTE]
> Pour vérifier si vous êtes connecté au bon cluster, exécutez la commande `oc whoami -c`.

## <a name="prepare-the-projects"></a>Préparer les projets

Pour créer les projets, exécutez les commandes suivantes :
```
oc new-project prometheus-project
oc new-project app-project1
oc new-project app-project2
```


> [!NOTE]
> Vous pouvez utiliser le paramètre `-n` ou `--namespace` sélectionner un projet actif en exécutant la commande `oc project`.

## <a name="prepare-the-prometheus-configuration-file"></a>Préparer le fichier de configuration Prometheus
Créez un fichier prometheus.yml en saisissant le contenu suivant :
```
global:
  scrape_interval: 30s
  evaluation_interval: 5s

scrape_configs:
    - job_name: prom-sd
      scrape_interval: 30s
      scrape_timeout: 10s
      metrics_path: /metrics
      scheme: http
      kubernetes_sd_configs:
      - api_server: null
        role: endpoints
        namespaces:
          names:
          - prometheus-project
          - app-project1
          - app-project2
```
Créez une clé secrète appelée Prom en saisissant la configuration suivante :
```
oc create secret generic prom --from-file=prometheus.yml -n prometheus-project
```

Le fichier Prometheus. yml est un fichier de configuration Prometheus de base. Il définit les intervalles et configure la découverte automatique dans trois projets (prometheus-project, app-project1, app-project2). Dans le fichier de configuration précédent, les points de terminaison détectés automatiquement sont rebutés via HTTP sans authentification.

Pour plus d’informations sur la mise en rebut des points de terminaison, consultez [Configuration de mise en rebut Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#scrape_config).


## <a name="prepare-the-alertmanager-config-file"></a>Préparez le fichier de configuration Alertmanager
Créez un fichier alertmanager.yml en saisissant le contenu suivant :
```
global:
  resolve_timeout: 5m
route:
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: default
  routes:
  - match:
      alertname: DeadMansSwitch
    repeat_interval: 5m
    receiver: deadmansswitch
receivers:
- name: default
- name: deadmansswitch
```
Créez une clé secrète appelée Prom-Alerts en saisissant la configuration suivante :
```
oc create secret generic prom-alerts --from-file=alertmanager.yml -n prometheus-project
```

Alertmanager .yml est le fichier de configuration Alertmanager.

> [!NOTE]
> Pour vérifier les deux étapes précédentes, exécutez la commande `oc get secret -n prometheus-project`.

## <a name="start-prometheus-and-alertmanager"></a>Démarrer Prometheus et Alertmanager
Accédez au [référentiel openshift/origin](https://github.com/openshift/origin/tree/release-3.11/examples/prometheus) et téléchargez le modèle [Prometheus-standalone.yaml](
https://raw.githubusercontent.com/openshift/origin/release-3.11/examples/prometheus/prometheus-standalone.yaml). Appliquez le modèle à prometheus-project en saisissant la configuration suivante :
```
oc process -f https://raw.githubusercontent.com/openshift/origin/release-3.11/examples/prometheus/prometheus-standalone.yaml | oc apply -f - -n prometheus-project
```
Le fichier prometheus-standalone.yaml est un modèle OpenShift. Il crée une instance Prometheus avec le proxy OAuth devant et une instance AlertManager, également sécurisée avec OAuth-proxy. Dans ce modèle, oauth-proxy est configuré pour autoriser tout utilisateur capable d’obtenir (« get ») l’espace de noms « prometheus-project » (voir l’indicateur `-openshift-sar`).

> [!NOTE]
> Pour vérifier si le StatefulSet « prom » a un nombre égal de réplicas DESIRED et CURRENT exécutez la commande `oc get statefulset -n prometheus-project`. Pour vérifier toutes les ressources dans le projet, exécutez la commande `oc get all -n prometheus-project`.

## <a name="add-permissions-to-allow-service-discovery"></a>Ajoutez des autorisations pour permettre la découverte des services

Créez un fichier prometheus-sdrole.yml en saisissant le contenu suivant :
```
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: prometheus-sdrole
  annotations:
    "openshift.io/display-name": Prometheus service discovery role
    description: |
      Role and rolebinding added permissions required for service discovery in a given project.
    iconClass: fa fa-cogs
    tags: "monitoring,prometheus,alertmanager,time-series"
parameters:
- description: The project name, where a standalone Prometheus is deployed
  name: PROMETHEUS_PROJECT
  value: prometheus-project
objects:
- apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: prometheus-sd
  rules:
  - apiGroups:
    - ""
    resources:
    - services
    - endpoints
    - pods
    verbs:
    - list
    - get
    - watch
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: prometheus-sd
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: prometheus-sd
  subjects:
  - kind: ServiceAccount
    name: prom
    namespace: ${PROMETHEUS_PROJECT}
```
Pour appliquer le modèle à tous les projets à partir desquels vous souhaitez autoriser la découverte de services, exécutez les commandes suivantes :
```
oc process -f prometheus-sdrole.yml | oc apply -f - -n app-project1
oc process -f prometheus-sdrole.yml | oc apply -f - -n app-project2
oc process -f prometheus-sdrole.yml | oc apply -f - -n prometheus-project
```

> [!NOTE]
> Pour vérifier que le rôle et les RoleBinding ont été créés correctement, exécutez les commandes `oc get role` et `oc get rolebinding`.

## <a name="optional-deploy-example-application"></a>Facultatif : Déployer l’exemple d’application

Tout fonctionne, mais il n’y a aucune source de métriques. Accédez à l’URL Prometheus (https://prom-prometheus-project.apps.*random-id*.*region*.azmosa.io/). Vous pouvez la trouver à l’aide de la commande suivante :

```
oc get route prom -n prometheus-project
```
> [!IMPORTANT]
> N’oubliez pas d’ajouter le préfixe https:// au début du nom d’hôte.

La page **Status > Service Discovery** (États > Découverte des services) affiche 0/0 cible active.

Pour déployer un exemple d’application qui expose des métriques de base Python sous le point de terminaison /metrics, exécutez les commandes suivantes :
```
oc new-app python:3.6~https://github.com/Makdaam/prometheus-example --name=example1 -n app-project1

oc new-app python:3.6~https://github.com/Makdaam/prometheus-example --name=example2 -n app-project2
```
Les nouvelles applications devraient apparaître en tant que cibles valides dans la page de découverte des services dans les 30 secondes après le déploiement.

Pour plus d’informations, sélectionnez **État** > **Cibles**.

> [!NOTE]
> Pour chaque cible correctement interrogée, Prometheus ajoute un point de données dans la métrique « up ». Cliquez sur **Prometheus** dans le coin supérieur gauche, puis entrez **up** comme expression et cliquez sur **Exécuter**.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez ajouter une instrumentation Prometheus personnalisée à vos applications. La bibliothèque de client Prometheus, qui simplifie la préparation des métriques Prometheus, est prête pour différents langages de programmation.

Pour plus d’informations, consultez les bibliothèques GitHub suivantes :

 - [Java](https://github.com/prometheus/client_java)
 - [Python](https://github.com/prometheus/client_python)
 - [Go](https://github.com/prometheus/client_golang)
 - [Ruby](https://github.com/prometheus/client_ruby)
