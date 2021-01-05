---
title: Créer une instance gérée Azure SQL sur Azure Arc
description: Créer une instance gérée Azure SQL sur Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 0d1d6c7349adc65270e13d09d771963cb9cacea3
ms.sourcegitcommit: 58f12c358a1358aa363ec1792f97dae4ac96cc4b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93280406"
---
# <a name="create-an-azure-sql-managed-instance-on-azure-arc"></a>Créer une instance gérée Azure SQL sur Azure Arc

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="login-to-the-azure-arc-data-controller"></a>Se connecter au contrôleur de données Azure Arc

Avant de pouvoir créer une instance, connectez-vous au contrôleur de données Azure Arc si vous ne l’avez pas encore fait.

```console
azdata login
```

Vous serez ensuite invité à entrer le nom d’utilisateur, le mot de passe et l’espace de noms système.  

```console
Username: arcadmin
Password:
Namespace: arc
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="create-an-azure-sql-managed-instance"></a>Créer une instance gérée SQL Azure

Pour afficher les options de création disponibles pour SQL Managed Instance, utilisez la commande suivante :
```console
azdata arc sql mi create --help
```

Pour créer une instance SQL Managed Instance, utilisez la commande suivante :

```console
azdata arc sql mi create -n <instanceName> --storage-class-data <storage class> --storage-class-logs <storage class>
```

Exemple :

```console
azdata arc sql mi create -n sqldemo --storage-class-data managed-premium --storage-class-logs managed-premium
```
> [!NOTE]
>  La longueur des noms doit être inférieure à 13 caractères et conforme aux [conventions d’affectation des noms DNS](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names)
>
>  Lorsque vous spécifiez l’allocation de mémoire et l’allocation vCore, utilisez cette formule pour vous assurer de la réussite de votre création. Pour chaque vCore, vous devez disposer d’au moins 4 Go de RAM de la capacité disponible sur le nœud Kubernetes où le pod d’instance gérée SQL s’exécutera.
>
>  Lors de la création d’une instance SQL, le nom ne doit pas comporter de majuscules si vous procédez à l’approvisionnement dans Azure
>
>  Pour répertorier les classes de stockage disponibles dans votre cluster Kubernetes, exécutez `kubectl get storageclass` 


> [!NOTE]
> Si vous souhaitez automatiser la création d’instances SQL et éviter l’invite interactive pour le mot de passe d’administrateur, vous pouvez définir les variables d’environnement `AZDATA_USERNAME` et `AZDATA_PASSWORD` sur le nom d’utilisateur et le mot de passe souhaités avant d’exécuter la commande `azdata arc sql mi create`.
> 
>  Si vous avez créé le contrôleur de données à l’aide d’AZDATA_USERNAME et d’AZDATA_PASSWORD dans la même session de terminal, les valeurs d’AZDATA_USERNAME et d’AZDATA_PASSWORD seront également utilisées pour créer l’instance gérée SQL.

> [!NOTE]
> La création de Azure SQL Managed Instance n’inscrira pas les ressources dans Azure. Les étapes d’inscription de la ressource sont décrites dans les articles suivants : 
> - [Afficher les journaux et les métriques à l’aide de Kibana et Grafana](monitor-grafana-kibana.md)
> - [Charger des données de facturation dans Azure et les afficher dans le portail Azure](view-billing-data-in-azure.md) 


## <a name="view-instance-on-azure-arc"></a>Afficher l’instance sur Azure Arc

Pour afficher l’instance, utilisez la commande suivante :

```console
azdata arc sql mi list
```

La sortie doit se présenter comme suit :

```console
Name    Replicas    ServerEndpoint    State
------  ----------  ----------------  -------
sqldemo 1/1         10.240.0.4:32023  Ready
```

Si vous utilisez AKS ou `kubeadm` ou OpenShift, etc., vous pouvez copier l’adresse IP externe et le numéro de port à partir d’ici et vous y connecter à l’aide de votre outil favori pour vous connecter à une instance SQL Server/SQL Azure, par exemple Azure Data Studio ou SQL Server Management Studio. Toutefois, si vous utilisez la machine virtuelle de démarrage rapide, consultez l’article [Se connecter à SQL Managed Instance avec Azure Arc](connect-managed-instance.md) pour obtenir des instructions spéciales.


## <a name="next-steps"></a>Étapes suivantes
- [Se connecter à SQL Managed Instance avec Azure Arc](connect-managed-instance.md)
- [Inscrire votre instance auprès d’Azure et charger les métriques et les journaux relatifs à votre instance](upload-metrics-and-logs-to-azure-monitor.md)
- [Déployer Azure SQL Managed Instance en utilisant Azure Data Studio](create-sql-managed-instance-azure-data-studio.md)
