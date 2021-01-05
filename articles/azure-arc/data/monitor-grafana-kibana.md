---
title: Afficher les journaux et les métriques à l’aide de Kibana et Grafana
description: Afficher les journaux et les métriques à l’aide de Kibana et Grafana
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 3693c30a34601512770f5d9071f5d786410fb00e
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92360375"
---
# <a name="view-logs-and-metrics-using-kibana-and-grafana"></a>Afficher les journaux et les métriques à l’aide de Kibana et Grafana

Les tableaux de bord web Kibana et Grafana sont fournis pour apporter des informations et plus clarté aux espaces de noms Kubernetes utilisés par les services de données Azure Arc activés.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="retrieve-the-ip-address-of-your-cluster"></a>Récupérer l’adresse IP de votre cluster

Pour accéder aux tableaux de bord, vous devez récupérer l’adresse IP de votre cluster. La méthode de récupération de la bonne adresse varie selon la façon que vous avez choisi de déployer Kubernetes. Parcourez les options ci-dessous pour trouver celle qui vous convient le mieux.

### <a name="azure-virtual-machine"></a>Machine virtuelle Azure

Utilisez la commande suivante pour récupérer l’adresse IP publique :

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

### <a name="kubeadm-cluster"></a>Cluster Kubeadm

Utilisez la commande suivante pour récupérer l’adresse IP du cluster :

```console
kubectl cluster-info
```


### <a name="aks-or-other-load-balanced-cluster"></a>AKS ou un autre cluster avec équilibrage de charge

Pour surveiller votre environnement dans AKS ou un autre cluster avec équilibrage de charge, vous devez récupérer l’adresse IP du service de proxy de gestion. Utilisez cette commande pour récupérer l’ **adresse IP externe**  :

```console
kubectl get svc mgmtproxy-svc-external -n <namespace>

#Example:
#kubectl get svc mgmtproxy-svc-external -n arc
NAME                     TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)           AGE
mgmtproxy-svc-external   LoadBalancer   10.0.186.28   52.152.148.25   30777:30849/TCP   19h
```

## <a name="additional-firewall-configuration"></a>Configuration complémentaire du pare-feu

Il se peut que vous deviez ouvrir des ports sur votre pare-feu pour accéder aux points de terminaison Kibana et Grafana.

Voici un exemple montrant comment faire cela pour une machine virtuelle Azure. Vous devrez effectuer cette opération si vous avez déployé Kubernetes à l’aide du script.

Les étapes ci-dessous expliquent comment créer une règle de groupe de sécurité réseau pour les points de terminaison Kibana et Grafana :

### <a name="find-the-name-of-the-nsg"></a>Recherchez le nom du groupe de sécurité réseau

```azurecli
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

### <a name="add-the-nsg-rule"></a>Ajoutez la règle du groupe de sécurité réseau

Une fois que vous avez le nom du groupe de sécurité réseau, vous pouvez ajouter une règle à l’aide de la commande suivante :

```azurecli
az network nsg rule create -n ports_30777 --nsg-name azurearcvmNSG --priority 600 -g azurearcvm-rg --access Allow --description 'Allow Kibana and Grafana ports' --destination-address-prefixes '*' --destination-port-ranges 30777 --direction Inbound --protocol Tcp --source-address-prefixes '*' --source-port-ranges '*'
```

## <a name="monitor-azure-sql-managed-instances-on-azure-arc"></a>Suivre les instances managées Azure SQL sur Azure Arc

Maintenant que vous disposez de l’adresse IP et que vous avez exposé les ports, vous devez être en mesure d’accéder à Grafana et Kibana.

> [!NOTE]
>  Lorsque vous êtes invité à entrer un nom d’utilisateur et un mot de passe, entrez le nom d’utilisateur et le mot de passe que vous avez fournis au moment où vous avez créé le contrôleur de données Azure Arc.

> [!NOTE]
>  Un avertissement de certificat s’affiche, car les certificats utilisés dans la préversion sont des certificats auto-signés.

Utilisez le modèle d’URL suivant pour accéder aux tableaux de bord de journalisation et de surveillance pour l’instance managée Azure SQL :

```html
https://<external-ip-from-above>:30777/grafana
https://<external-ip-from-above>:30777/kibana
```

Les tableaux de bord pertinents sont les suivants :

* « Métriques Azure SQL Managed Instance »
* « Métriques de nœud hôte »
* « Métriques des pods hôtes »

## <a name="monitor-azure-database-for-postgresql-hyperscale---azure-arc"></a>Suivre Azure Database pour PostgreSQL Hyperscale - Azure Arc

Utilisez le modèle d’URL suivant pour accéder aux tableaux de bord de journalisation et de surveillance pour PostgreSQL Hyperscale :

```html
https://<external-ip-from-above>:30777/grafana
https://<external-ip-from-above>:30777/kibana
```

Les tableaux de bord pertinents sont les suivants :

* « Métriques Postgres »
* « Métriques de table Postgres »
* « Métriques de nœud hôte »
* « Métriques des pods hôtes »

## <a name="next-steps"></a>Étapes suivantes
- Essayez [Charger les métriques sur Azure Monitor](upload-metrics-and-logs-to-azure-monitor.md)
- En savoir plus sur Grafana :
   - [Bien démarrer](https://grafana.com/docs/grafana/latest/getting-started/getting-started)
   - [Fondamentaux de Grafana](https://grafana.com/tutorials/grafana-fundamentals/#1)
   - [Tutoriels Grafana](https://grafana.com/tutorials/grafana-fundamentals/#1)
- En savoir plus sur Kibana
   - [Introduction](https://www.elastic.co/webinars/getting-started-kibana?baymax=default&elektra=docs&storm=top-video)
   - [Guide Kibana](https://www.elastic.co/guide/en/kibana/current/index.html)
   - [Présentation des explorations du tableau de bord avec des visualisations de données dans Kibana](https://www.elastic.co/webinars/dashboard-drilldowns-with-data-visualizations-in-kibana/)
   - [Comment créer des tableaux de bord Kibana](https://www.elastic.co/webinars/how-to-build-kibana-dashboards/)

