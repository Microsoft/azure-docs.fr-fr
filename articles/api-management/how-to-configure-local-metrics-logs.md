---
title: Configurer des métriques et journaux locaux pour la passerelle auto-hébergée de Gestion des API Azure | Microsoft Docs
description: Découvrir comment configurer des métriques et journaux locaux pour la passerelle auto-hébergée de Gestion des API Azure
services: api-management
documentationcenter: ''
author: miaojiang
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 04/30/2020
ms.author: apimpm
ms.openlocfilehash: ac147863fe54be3343eda653fc863ebd08dac54d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86254501"
---
# <a name="configure-local-metrics-and-logs-for-azure-api-management-self-hosted-gateway"></a>Configurer des métriques et journaux locaux pour la passerelle auto-hébergée de Gestion des API Azure

Cet article fournit des informations détaillées sur la configuration des métriques et journaux locaux pour la [passerelle auto-hébergée](./self-hosted-gateway-overview.md). Pour configurer les journaux et métriques cloud, consultez [cet article](how-to-configure-cloud-metrics-logs.md). 

## <a name="metrics"></a>Mesures
La passerelle auto-hébergée prend en charge [StatsD](https://github.com/statsd/statsd), qui est devenu un protocole unifiant pour la collecte et l’agrégation des métriques. Cette section décrit les étapes de déploiement de StatsD sur Kubernetes, de configuration de la passerelle pour émettre des métriques via StatsD, et d’utilisation de [Prometheus](https://prometheus.io/) pour surveiller les métriques. 

### <a name="deploy-statsd-and-prometheus-to-the-cluster"></a>Déployer StatsD et Prometheus sur le cluster

Voici un exemple de configuration YAML pour le déploiement des services StatsD et Prometheus sur le cluster Kubernetes dans lequel une passerelle auto-hébergée est déployée. Il crée également un [Service](https://kubernetes.io/docs/concepts/services-networking/service/) pour chaque service. La passerelle auto-hébergée publie des métriques sur le service StatsD. Nous allons accéder au tableau de bord de Prometheus via son service.   

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sputnik-metrics-config
data:
  statsd.yaml: ""
  prometheus.yaml: |
    global:
      scrape_interval:     3s
      evaluation_interval: 3s
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      - job_name: 'test_metrics'
        static_configs:
          - targets: ['localhost:9102']
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sputnik-metrics
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sputnik-metrics
  template:
    metadata:
      labels:
        app: sputnik-metrics
    spec:
      containers:
      - name: sputnik-metrics-statsd
        image: prom/statsd-exporter
        ports:
        - name: tcp
          containerPort: 9102
        - name: udp
          containerPort: 8125
          protocol: UDP
        args:
          - --statsd.mapping-config=/tmp/statsd.yaml
          - --statsd.listen-udp=:8125
          - --web.listen-address=:9102
        volumeMounts:
          - mountPath: /tmp
            name: sputnik-metrics-config-files
      - name: sputnik-metrics-prometheus
        image: prom/prometheus
        ports:
        - name: tcp
          containerPort: 9090
        args:
          - --config.file=/tmp/prometheus.yaml
        volumeMounts:
          - mountPath: /tmp
            name: sputnik-metrics-config-files
      volumes:
        - name: sputnik-metrics-config-files
          configMap:
            name: sputnik-metrics-config
---
apiVersion: v1
kind: Service
metadata:
  name: sputnik-metrics-statsd
spec:
  type: NodePort
  ports:
  - name: udp
    port: 8125
    targetPort: 8125
    protocol: UDP
  selector:
    app: sputnik-metrics
---
apiVersion: v1
kind: Service
metadata:
  name: sputnik-metrics-prometheus
spec:
  type: LoadBalancer
  ports:
  - name: http
    port: 9090
    targetPort: 9090
  selector:
    app: sputnik-metrics
```

Enregistrez les configurations dans un fichier nommé `metrics.yaml` et utilisez la commande ci-dessous pour tout déployer sur le cluster :

```console
kubectl apply -f metrics.yaml
```

Une fois le déploiement terminé, exécutez la commande ci-dessous pour vérifier que les pods sont en cours d’exécution. Notez que le nom de votre pod va être différent. 

```console
kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
sputnik-metrics-f6d97548f-4xnb7        2/2     Running   0          1m
```

Exécutez la commande ci-dessous pour vérifier que les services sont en cours d’exécution. Prenez note du `CLUSTER-IP` et du `PORT` du service StatsD, car vous en aurez besoin plus tard. Vous pouvez visiter le tableau de bord Prometheus en utilisant son `EXTERNAL-IP` et son `PORT`.

```console
kubectl get services
NAME                         TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE
sputnik-metrics-prometheus   LoadBalancer   10.0.252.72   13.89.141.90    9090:32663/TCP               18h
sputnik-metrics-statsd       NodePort       10.0.41.179   <none>          8125:32733/UDP               18h
```

### <a name="configure-the-self-hosted-gateway-to-emit-metrics"></a>Configurer la passerelle auto-hébergée pour émettre des métriques

Maintenant que les services StatsD et Prometheus ont été déployés, nous pouvons mettre à jour les configurations de la passerelle auto-hébergée pour commencer à émettre des métriques via le service StatsD. Vous pouvez activer ou désactiver la fonctionnalité en utilisant la clé `telemetry.metrics.local` dans le fichier Configmap du déploiement de la passerelle auto-hébergée avec des options supplémentaires. Voici la liste des options disponibles :

| Champ  | Default | Description |
| ------------- | ------------- | ------------- |
| telemetry.metrics.local  | `none` | Active la journalisation via StatsD. La valeur peut être `none` ou `statsd`. |
| telemetry.metrics.local.statsd.endpoint  | n/a | Spécifie le point de terminaison de StatsD. |
| telemetry.metrics.local.statsd.sampling  | n/a | Spécifie le taux d’échantillonnage des métriques. La valeur peut être comprise entre 0 et 1. Par exemple, `0.5`|
| telemetry.metrics.local.statsd.tag-format  | n/a | [Format d’étiquetage](https://github.com/prometheus/statsd_exporter#tagging-extensions) de l’exportateur StatsD. La valeur peut être `none`, `librato`, `dogStatsD` ou `influxDB`. |

Voici un exemple de configuration :

```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: contoso-gateway-environment
    data:
        config.service.endpoint: "<self-hosted-gateway-management-endpoint>"
        telemetry.metrics.local: "statsd"
        telemetry.metrics.local.statsd.endpoint: "10.0.41.179:8125"
        telemetry.metrics.local.statsd.sampling: "1"
        telemetry.metrics.local.statsd.tag-format: "dogStatsD"
```

Mettez à jour le fichier YAML du déploiement de la passerelle auto-hébergée avec les configurations ci-dessus, et appliquez les modifications à l’aide de la commande ci-dessous : 

```console
kubectl apply -f <file-name>.yaml
 ```

Pour prélever les dernières modifications de configuration, redémarrez le déploiement de la passerelle à l’aide de la commande ci-dessous :

```console
kubectl rollout restart deployment/<deployment-name>
```

### <a name="view-the-metrics"></a>Afficher les métriques

À présent que tout est déployé et configuré, la passerelle auto-hébergée doit signaler les métriques via StatsD. Prometheus prélève les métriques dans StatsD. Accédez au tableau de bord Prometheus en utilisant les valeurs `EXTERNAL-IP` et `PORT` du service Prometheus. 

Effectuez quelques appels d’API via la passerelle auto-hébergée. Si tout est bien configuré, vous devez être en mesure d’afficher les métriques ci-dessous :

| Métrique  | Description |
| ------------- | ------------- |
| Demandes  | Nombre de demandes API au cours de la période |
| DurationInMS | Nombre de millisecondes entre le moment où la passerelle a reçu la requête et celui où la réponse complète a été envoyée |
| BackendDurationInMS | Nombre de millisecondes consacrées à l’ensemble des E/S du serveur principal (connexion, envoi et réception d’octets)  |
| ClientDurationInMS | Nombre de millisecondes consacrées à l’ensemble des E/S du client (connexion, envoi et réception d’octets)  |

## <a name="logs"></a>Journaux d’activité

La passerelle auto-hébergée génère des journaux dans `stdout` et `stderr` par défaut. Vous pouvez facilement afficher les journaux à l’aide de la commande suivante :

```console
kubectl logs <pod-name>
```

Si votre passerelle auto-hébergée est déployée dans Azure Kubernetes Service, vous pouvez activer [Azure Monitor pour conteneurs](../azure-monitor/insights/container-insights-overview.md) afin de collecter `stdout` et `stderr` à partir de vos charges de travail et d’afficher les journaux dans Log Analytics. 

La passerelle auto-hébergée prend également en charge un certain nombre de protocoles, notamment `localsyslog`, `rfc5424` et `journal`. Le tableau ci-dessous récapitule toutes les options prises en charge. 

| Champ  | Default | Description |
| ------------- | ------------- | ------------- |
| telemetry.logs.std  | `text` | Active la journalisation dans des flux standard. La valeur peut être `none`, `text` ou `json`. |
| telemetry.logs.local  | `none` | Active la journalisation locale. La valeur peut être `none`, `auto`, `localsyslog`, `rfc5424` ou `journal`.  |
| telemetry.logs.local.localsyslog.endpoint  | n/a | Spécifie le point de terminaison de localsyslog.  |
| telemetry.logs.local.localsyslog.facility  | n/a | Specifies le [code de facilité](https://en.wikipedia.org/wiki/Syslog#Facility) de localsyslog. Par exemple, `7` 
| telemetry.logs.local.rfc5424.endpoint  | n/a | Spécifie le point de terminaison de rfc5424.  |
| telemetry.logs.local.rfc5424.facility  | n/a | Spécifie le code de facilité par [rfc5424](https://tools.ietf.org/html/rfc5424). Par exemple, `7`  |
| telemetry.logs.local.journal.endpoint  | n/a | Spécifie le point de terminaison du journal.  |

Voici un exemple de configuration de journalisation locale :

```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: contoso-gateway-environment
    data:
        config.service.endpoint: "<self-hosted-gateway-management-endpoint>"
        telemetry.logs.std: "text"
        telemetry.logs.local.localsyslog.endpoint: "/dev/log"
        telemetry.logs.local.localsyslog.facility: "7"
```
 
## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la passerelle auto-hébergée, consultez [Vue d’ensemble de la passerelle auto-hébergée Gestion des API](self-hosted-gateway-overview.md)
* En savoir plus sur la [configuration et la persistance des journaux dans le cloud](how-to-configure-local-metrics-logs.md)
