---
title: Configuration du stockage
description: Explique les options de configuration du stockage des services de données activés pour Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: uc-msft
ms.author: umajay
ms.reviewer: mikeray
ms.date: 10/12/2020
ms.topic: conceptual
ms.openlocfilehash: 7b683029b7fd05078755d4e8cd027f55c805f991
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107258"
---
# <a name="storage-configuration"></a>Configuration du stockage

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="kubernetes-storage-concepts"></a>Concepts du stockage Kubernetes

Kubernetes fournit une couche d’abstraction de l’infrastructure sur la pile technique de virtualisation sous-jacente (facultatif) et le matériel. Kubernetes rend abstrait le stockage via des **[classes de stockage](https://kubernetes.io/docs/concepts/storage/storage-classes/)** . Au moment du provisionnement d’un pod, vous pouvez spécifier qu’une classe de stockage doit être utilisée pour chaque volume. Au moment du provisionnement du pod, le **[provisionneur](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)** de la classe de stockage est appelé pour provisionner le stockage, puis un **[volume persistant](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)** est créé sur ce stockage provisionné et enfin, le pod est monté sur le volume persistant par une **[revendication de volume persistant](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)** .

Kubernetes offre aux fournisseurs d’infrastructure de stockage un moyen d’introduire des pilotes (également appelés « modules complémentaires » ou « add-ons ») qui étendent Kubernetes. Les modules complémentaires de stockage sont conformes au **[standard de Container Storage Interface](https://kubernetes.io/blog/2019/01/15/container-storage-interface-ga/)** . Vous trouverez des dizaines de modules complémentaires dans cette **[liste des pilotes CSI](https://kubernetes-csi.github.io/docs/drivers.html)** non définitive. Le pilote CSI que vous utilisez dépend de facteurs comme le fait que vous exécutez un service Kubernetes managé et hébergé dans le cloud, ou le fournisseur OEM que vous utilisez pour votre matériel.

Vous pouvez voir les classes de stockage qui sont configurées dans votre cluster Kubernetes en exécutant la commande suivante :

```console
kubectl get storageclass
```

Exemple de sortie d’un cluster Azure Kubernetes Service (AKS) :

```console
NAME                PROVISIONER                AGE
azurefile           kubernetes.io/azure-file   15d
azurefile-premium   kubernetes.io/azure-file   15d
default (default)   kubernetes.io/azure-disk   4d3h
managed-premium     kubernetes.io/azure-disk   4d3h
```

Vous pouvez obtenir des détails sur une classe de stockage en exécutant la commande suivante :

```console
kubectl describe storageclass/<storage class name>
```

Exemple :

```console
kubectl describe storageclass/azurefile

Name:            azurefile
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"allowVolumeExpansion":true,"apiVersion":"storage.k8s.io/v1beta1","kind":"StorageClass","metadata":{"annotations":{},"labels":{"kubernetes.io/cluster-service":"true"},"name":"azurefile"},"parameters":{"sku
Name":"Standard_LRS"},"provisioner":"kubernetes.io/azure-file"}

Provisioner:           kubernetes.io/azure-file
Parameters:            skuName=Standard_LRS
AllowVolumeExpansion:  True
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

Vous pouvez voir les volumes persistants actuellement provisionnés et les revendications de volume persistant en exécutant les commandes suivantes :

```console
kubectl get persistentvolumes -n <namespace>

kubectl get persistentvolumeclaims -n <namespace>
```

Exemple d’affichage de volumes persistants :

```console

kubectl get persistentvolumes -n arc

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS   REASON   AGE
pvc-07fc7b9f-9a37-4796-9442-4405147120da   15Gi       RWO            Delete           Bound    arc/sqldemo11-data-claim   default                 7d3h
pvc-3e772f20-ed89-4642-b34d-8bb11b088afa   15Gi       RWO            Delete           Bound    arc/data-metricsdb-0       default                 7d14h
pvc-41b33bbd-debb-4153-9a41-02ce2bf9c665   10Gi       RWO            Delete           Bound    arc/sqldemo11-logs-claim   default                 7d3h
pvc-4ccda3e4-fee3-4a89-b92d-655c04fa62ad   15Gi       RWO            Delete           Bound    arc/data-controller        default                 7d14h
pvc-63e6bb4c-7240-4de5-877e-7e9ea4e49c91   10Gi       RWO            Delete           Bound    arc/logs-controller        default                 7d14h
pvc-8a1467fe-5eeb-4d73-b99a-f5baf41eb493   10Gi       RWO            Delete           Bound    arc/logs-metricsdb-0       default                 7d14h
pvc-8e2cacbc-e953-4901-8591-e77df9af309c   10Gi       RWO            Delete           Bound    arc/sqldemo10-logs-claim   default                 7d14h
pvc-9fb79ba3-bd3e-42aa-aa09-3090135d4513   15Gi       RWO            Delete           Bound    arc/sqldemo10-data-claim   default                 7d14h
pvc-a39c85d4-5cd9-4249-9915-68a70a9bb5e5   15Gi       RWO            Delete           Bound    arc/data-controldb         default                 7d14h
pvc-c9cbd74a-76ca-4be5-b598-0c7a45749bfb   10Gi       RWO            Delete           Bound    arc/logs-controldb         default                 7d14h
pvc-d576e9d4-0a09-4dd7-b806-be8ed461f8a4   10Gi       RWO            Delete           Bound    arc/logs-logsdb-0          default                 7d14h
pvc-ecd7d07f-2c2c-421d-98d7-711ec5d4a0cd   15Gi       RWO            Delete           Bound    arc/data-logsdb-0          default                 7d14h
```

Exemple d’affichage de revendications de volume persistant :

```console

kubectl get persistentvolumeclaims -n arc

NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-controldb         Bound    pvc-a39c85d4-5cd9-4249-9915-68a70a9bb5e5   15Gi       RWO            default        7d14h
data-controller        Bound    pvc-4ccda3e4-fee3-4a89-b92d-655c04fa62ad   15Gi       RWO            default        7d14h
data-logsdb-0          Bound    pvc-ecd7d07f-2c2c-421d-98d7-711ec5d4a0cd   15Gi       RWO            default        7d14h
data-metricsdb-0       Bound    pvc-3e772f20-ed89-4642-b34d-8bb11b088afa   15Gi       RWO            default        7d14h
logs-controldb         Bound    pvc-c9cbd74a-76ca-4be5-b598-0c7a45749bfb   10Gi       RWO            default        7d14h
logs-controller        Bound    pvc-63e6bb4c-7240-4de5-877e-7e9ea4e49c91   10Gi       RWO            default        7d14h
logs-logsdb-0          Bound    pvc-d576e9d4-0a09-4dd7-b806-be8ed461f8a4   10Gi       RWO            default        7d14h
logs-metricsdb-0       Bound    pvc-8a1467fe-5eeb-4d73-b99a-f5baf41eb493   10Gi       RWO            default        7d14h
sqldemo10-data-claim   Bound    pvc-9fb79ba3-bd3e-42aa-aa09-3090135d4513   15Gi       RWO            default        7d14h
sqldemo10-logs-claim   Bound    pvc-8e2cacbc-e953-4901-8591-e77df9af309c   10Gi       RWO            default        7d14h
sqldemo11-data-claim   Bound    pvc-07fc7b9f-9a37-4796-9442-4405147120da   15Gi       RWO            default        7d4h
sqldemo11-logs-claim   Bound    pvc-41b33bbd-debb-4153-9a41-02ce2bf9c665   10Gi       RWO            default        7d4h

```

## <a name="factors-to-consider-when-choosing-your-storage-configuration"></a>Facteurs à prendre en compte lors du choix de la configuration de votre stockage

La sélection de la classe de stockage appropriée est importante pour la résilience et les performances des données. Si vous choisissez une classe de stockage incorrecte, vous risquez de perdre des données en cas de panne matérielle ou d’obtenir de moins bonnes performances.

Il existe en général deux types de stockage :

- **Stockage local** - stockage provisionné sur des disques durs locaux sur un nœud donné. Ce type de stockage peut être idéal en termes de performances, mais il demande une conception spécifique de la redondance des données via la réplication des données sur plusieurs nœuds.
- **Stockage partagé distant** - stockage provisionné sur un dispositif de stockage distant, par exemple un SAN, un NAS ou un service de stockage cloud, comme EBS ou Azure Files. Ce type de stockage fournit généralement une redondance des données automatique, mais il n’est généralement pas aussi rapide que le stockage local.

> [!NOTE]
> Pour le moment, si vous utilisez NFS, vous devez définir allowRunAsRoot sur true dans votre fichier de profil de déploiement avant de déployer le contrôleur de données Azure Arc.

### <a name="data-controller-storage-configuration"></a>Configuration du stockage du contrôleur de données

Certains services dans Azure Arc pour les services de données dépendent de la configuration pour utiliser un stockage partagé distant, car les services n’ont pas la capacité de répliquer les données. Ces services se trouvent dans la collection de pods du contrôleur de données :

|**Service**|**Revendications de volume persistant**|
|---|---|
|**ElasticSearch**|`<namespace>/logs-logsdb-0`, `<namespace>/data-logsdb-0`|
|**InfluxDB**|`<namespace>/logs-metricsdb-0`, `<namespace>/data-metricsdb-0`|
|**Instance SQL du contrôleur**|`<namespace>/logs-controldb`, `<namespace>/data-controldb`|
|**Service d’API du contrôleur**|`<namespace>/data-controller`|

Au moment où le contrôleur de données est provisionné, la classe de stockage à utiliser pour chacun de ces volumes persistants est spécifiée en passant le paramètre --storage-class | -sc à la commande `azdata arc dc create` ou en définissant les classes de stockage dans le fichier de modèle de déploiement control.json utilisé.

Les modèles de déploiement fournis d’origine spécifient une classe de stockage par défaut appropriée pour l’environnement cible, mais elle peut être remplacée au moment du déploiement. Pour modifier la configuration de la classe de stockage pour les pods de contrôleur de données au moment du déploiement, consultez les étapes détaillées pour [modifier le profil de déploiement](create-data-controller.md).

Si vous définissez la classe de stockage en utilisant le paramètre --storage-class | -sc, la classe de stockage sera utilisée pour les classes de stockage des journaux et des données. Si vous définissez les classes de stockage dans le fichier de modèle de déploiement, vous pouvez spécifier des classes de stockage différentes pour les journaux et pour les données.

Voici les facteurs importants à prendre en compte lors du choix d’une classe de stockage pour les pods de contrôleur de données :

- Vous **devez** utiliser une classe de stockage partagée distante afin de garantir la durabilité des données et que, si un pod ou un nœud est défaillant, quand le pod est rétabli, il peut se reconnecter au volume persistant.
- Les données écrites dans l’instance SQL du contrôleur, dans la base de données des métriques et dans la base de données des journaux sont généralement relativement peu volumineuses et ne sont pas sensibles à la latence : un stockage avec des performances ultrarapides n’est donc pas critique. Si vous avez des utilisateurs qui utilisent fréquemment les interfaces Grafana et Kibana, et que vous avez un grand nombre d’instances de base de données, vos utilisateurs pourraient tirer un bénéfice d’un stockage plus rapide.
- La capacité de stockage nécessaire varie en fonction du nombre d’instances de base de données que vous avez déployées, car les journaux et les métriques sont collectés pour chaque instance de base de données. Les données sont conservées dans les bases de données des journaux et des métriques pendant deux (2) semaines avant d’être supprimées. 
- Changer la classe de stockage après le déploiement est difficile, non documenté et non pris en charge. Veillez à choisir la classe de stockage correctement au moment du déploiement.

> [!NOTE]
> Si aucune classe de stockage n’est spécifiée, la classe de stockage par défaut sera utilisée. Il ne peut y avoir qu’une seule classe de stockage par défaut par cluster Kubernetes. Vous pouvez [changer la classe de stockage par défaut](https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/).

### <a name="database-instance-storage-configuration"></a>Configuration du stockage de l’instance de base de données

Chaque instance de base de données a des volumes persistants pour les données, pour les journaux et pour les sauvegardes. Les classes de stockage de ces volumes persistants peuvent être spécifiées au moment du déploiement. Si aucune classe de stockage n’est spécifiée, la classe de stockage par défaut sera utilisée.

Quand vous créez une instance en utilisant les commandes `azdata arc sql mi create` ou `azdata arc postgres server create`, vous pouvez utiliser deux paramètres pour définir les classes de stockage :

> [!NOTE]
> Certains de ces paramètres sont en cours de développement et seront disponibles sur `azdata arc sql mi create` et sur `azdata arc postgres server create` dans les versions à venir.

|Nom du paramètre, nom abrégé|Utilisé pour|
|---|---|
|`--storage-class-data`, `-scd`|Utilisé pour spécifier la classe de stockage pour tous les fichiers de données, y compris les fichiers journaux de transactions|
|`--storage-class-logs`, `-scl`|Utilisé pour spécifier la classe de stockage pour tous les fichiers journaux|
|`--storage-class-data-logs`, `-scdl`|Utilisé pour spécifier la classe de stockage pour les fichiers journaux de transactions de la base de données. **Remarque : Pas encore disponible.**|
|`--storage-class-backups`, `-scb`|Utilisé pour spécifier la classe de stockage pour tous les fichiers de sauvegarde. **Remarque : Pas encore disponible.**|

Le tableau ci-dessous liste les chemins dans le conteneur Azure SQL Managed Instance qui est mappé au volume persistant pour les données et les journaux :

|Nom du paramètre, nom abrégé|Chemin dans le conteneur mssql-miaa|Description|
|---|---|---|
|`--storage-class-data`, `-scd`|/var/opt|Contient des répertoires pour l’installation de mssql et d’autres processus système. Le répertoire mssql contient les données par défaut (y compris les journaux des transactions), le journal des erreurs et les répertoires de sauvegarde|
|`--storage-class-logs`, `-scl`|/var/log|Contient des répertoires qui stockent la sortie de la console (stderr, stdout) ainsi que d’autres informations de journalisation des processus au sein du conteneur|

Le tableau ci-dessous liste les chemins dans le conteneur de l’instance PostgreSQL qui est mappé au volume persistant pour les données et les journaux :

|Nom du paramètre, nom abrégé|Chemin au sein du conteneur postgres|Description|
|---|---|---|
|`--storage-class-data`, `-scd`|/var/opt/postgresql|Contient des répertoires de données et de journaux pour l’installation de postgres|
|`--storage-class-logs`, `-scl`|/var/log|Contient des répertoires qui stockent la sortie de la console (stderr, stdout) ainsi que d’autres informations de journalisation des processus au sein du conteneur|

Chaque instance de base de données aura un volume persistant distinct pour les fichiers de données, les journaux et les sauvegardes. Cela signifie qu’il y aura une séparation des E/S pour chacun de ces types de fichiers, en fonction de la façon dont le provisionneur de volume va provisionner le stockage. Chaque instance de base de données a ses propres revendications de volume persistant et ses propres volumes persistants.

S’il existe plusieurs bases de données sur une instance de base de données spécifique, toutes les bases de données utilisent la même revendication de volume persistant, le même volume persistant et la même classe de stockage. Toutes les sauvegardes, à la fois les sauvegardes des journaux différentiels et les sauvegardes complètes, utilisent la même revendication de volume persistant et le même volume persistant. Les revendications de volume persistant pour les pods d’instances de base de données sont présentées ci-dessous :

|**Instance**|**Revendications de volume persistant**|
|---|---|
|**Azure SQL Managed Instance**|`<namespace>/logs-<instance name>-0`, `<namespace>/data-<instance name>-0`|
|**Instance Azure Database pour PostgreSQL**|`<namespace>/logs--<instance name>-0`, `<namespace>/data--<instance name>-0`|
|**Azure PostgreSQL HyperScale**|`<namespace>/logs-<instance namme>-<ordinal>`, `<namespace>/data-<instance namme>-<ordinal>` *(Plages ordinales comprises entre 0 et W, où W est le nombre de workers)*|

Voici les facteurs importants à prendre en compte lors du choix d’une classe de stockage pour les pods d’instances de base de données :

- Les instances de base de données peuvent être déployées dans un modèle de pod unique ou dans un modèle à plusieurs pods. Une instance développeur d’Azure SQL Managed Instance ou une instance d’Azure SQL Managed Instance de niveau tarifaire Usage général est un exemple de modèle de pod unique. Une instance d’Azure SQL Managed Instance de niveau tarifaire Critique pour l’entreprise est un exemple de modèle à plusieurs pods (Remarque : les niveaux tarifaires sont en cours de développement et ne sont pas encore disponibles pour les clients.)  Les instances de base de données déployées avec le modèle de pod unique **doivent** utiliser une classe de stockage partagée distante afin de garantir la durabilité des données et que, si un pod ou un nœud est défaillant, quand le pod est rétabli, il peut se reconnecter au volume persistant. En revanche, une instance managée Azure SQL à haute disponibilité utilise des groupes de disponibilité Always On pour répliquer les données d’une instance vers une autre de façon synchrone ou asynchrone. En particulier dans le cas où les données sont répliquées de façon synchrone, il y a toujours plusieurs copies des données, en général trois (3) copies. Pour cette raison, il est possible d’utiliser un stockage local ou des classes de stockage partagé distant pour les fichiers de données et les fichiers journaux. Si vous utilisez un stockage local, les données sont toujours préservées, même dans le cas de la défaillance d’un pod, d’un nœud ou d’un matériel de stockage. Étant donné cette flexibilité, vous pouvez choisir d’utiliser un stockage local pour de meilleures performances.
- Les performances de la base de données sont en grande partie fonction du débit d’E/S d’un dispositif de stockage donné. Si votre base de données fait l’objet de lectures ou d’écritures intensives, vous devez choisir une classe de stockage qui a un matériel conçu pour ce type de charge de travail. Par exemple, si votre base de données est principalement utilisée pour des écritures, vous pouvez choisir un stockage local avec un niveau RAID 0. Si votre base de données est principalement utilisée pour des lectures d’une petite quantité de « données chaudes », mais qu’il existe un grand volume de stockage global des données froides, vous pouvez choisir un périphérique SAN avec des capacités de stockage hiérarchisé. Le choix de la classe de stockage appropriée n’est pas très différent du choix du type de stockage que vous utiliseriez pour une base de données.
- Si vous utilisez un provisionneur de volume de stockage local, vérifiez que les volumes locaux provisionnés pour les données, les journaux et les sauvegardes sont placés sur des dispositifs de stockage sous-jacents différents afin d’éviter la contention sur les E/S disque. Le système d’exploitation doit également se trouver sur un volume monté sur un ou plusieurs disques distincts. Il s’agit essentiellement des mêmes conseils que ceux qui seraient à suivre pour une instance de base de données sur un matériel physique.
- Étant donné que toutes les bases de données sur une instance spécifique partagent une revendication de volume persistant et un volume persistant, veillez à ne pas colocaliser les instances de base de données occupées sur la même instance de base de données. Si possible, placez séparément les bases de données occupées dans leurs propres instances de base de données pour éviter la contention des E/S. En outre, utilisez le ciblage des étiquettes de nœud pour placer les instances de base de données sur des nœuds distincts afin de distribuer le trafic d’E/S global entre plusieurs nœuds. Si vous utilisez la virtualisation, veillez à distribuer le trafic d’E/S non seulement au niveau du nœud, mais aussi l’activité d’E/S combinée qui se produit sur toutes les machines virtuelles du nœud sur un hôte physique donné.

## <a name="estimating-storage-requirements"></a>Estimation des besoins en stockage
Chaque pod qui contient des données avec état utilise deux volumes persistants dans cette version : un volume persistant pour les données et un autre pour les journaux. Le tableau ci-dessous indique le nombre de volumes persistants nécessaires pour un seul contrôleur de données, une instance Azure SQL Managed Instance, une instance Azure Database pour PostgreSQL et une instance Azure PostgreSQL Hyperscale :

|Type de ressource|Nombre de pods avec état|Nombre de volumes persistants nécessaires|
|---|---|---|
|Contrôleur de données|4 (`control`, `controldb`, `logsdb`, `metricsdb`)|4 * 2 = 8|
|Azure SQL Managed Instance|1|2|
|Instance Azure Database pour PostgreSQL|1| 2|
|Azure PostgreSQL HyperScale|1 + W (W = nombre de workers)|2 * (1 + W)|

Le tableau ci-dessous indique le nombre total de volumes persistants nécessaires pour un exemple de déploiement :

|Type de ressource|Nombre d’instances|Nombre de volumes persistants nécessaires|
|---|---|---|
|Contrôleur de données|1|4 * 2 = 8|
|Azure SQL Managed Instance|5|5 * 2 = 10|
|Instance Azure Database pour PostgreSQL|5| 5 * 2 = 10|
|Azure PostgreSQL HyperScale|2 (nombre de workers = 4 par instance)|2 * 2 * (1 + 4) = 20|
|***Nombre total de volumes persistants** _||8 + 10 + 10 + 20 = 48|

Ce calcul peut être utilisé pour planifier le stockage de votre cluster Kubernetes en fonction de l’environnement ou du provisionneur de stockage. Par exemple, si le provisionneur de stockage local est utilisé pour un cluster Kubernetes avec cinq (5) nœuds, pour l’exemple de déploiement ci-dessus, chaque nœud nécessite du stockage pour au moins 10 volumes persistants. De même, lors du provisionnement d’un cluster Azure Kubernetes Service (AKS) avec cinq (5) nœuds, il est important de choisir une taille de machine virtuelle appropriée pour le pool de nœuds pour que 10 disques de données puissent y être attachés. Vous trouverez plus d’informations sur la taille des nœuds pour les besoins en stockage des nœuds AKS [ici](../../aks/operator-best-practices-storage.md#size-the-nodes-for-storage-needs).

## <a name="choosing-the-right-storage-class"></a>Choix de la classe de stockage appropriée

### <a name="on-premises-and-edge-sites"></a>Sites locaux et périphériques

Microsoft et ses partenaires OEM, OS et Kubernetes travaillent sur un programme de certification pour les services de données Azure Arc. Ce programme fournira aux clients des résultats de tests comparables provenant d’une boîte à outils de tests de certification. Les tests évalueront la compatibilité des fonctionnalités, les résultats des tests de stress, et les performances et la scalabilité. Chacun de ces résultats de test indiquera les composants utilisés : le système d’exploitation, la distribution Kubernetes, le matériel, le module complémentaire CSI et les classes de stockage. Ceci permettra aux clients de choisir les meilleurs composants en fonction de leurs besoins : la classe de stockage, le système d’exploitation, la distribution Kubernetes et le matériel. Des informations plus complètes sur ce programme et les résultats des tests initiaux seront fournies à un moment proche de la disponibilité générale des services de données Azure Arc.

#### <a name="public-cloud-managed-kubernetes-services"></a>Cloud public, services Kubernetes managés

Pour les services Kubernetes managés et basés sur le cloud public, nous pouvons formuler les recommandations suivantes :

|Service cloud public|Recommandation|
|---|---|
|_ *Azure Kubernetes Service (AKS)* *|Azure Kubernetes Service (AKS) a deux types de stockage : Azure Files et Disques managés Azure. Chaque type de stockage a deux niveaux tarifaires/de performance : Standard (HDD) et Premium (SSD). Ainsi, les quatre classes de stockage fournies dans AKS sont `azurefile` (niveau Standard d’Azure Files), `azurefile-premium` (niveau Premium d’Azure Files), `default` (niveau Standard de Disques Azure) et `managed-premium` (niveau Premium de Disques Azure). La classe de stockage par défaut est `default` (niveau Standard de Disques Azure). Il existe des **[différences tarifaires](https://azure.microsoft.com/en-us/pricing/details/storage/)** substantielles entre les types et les niveaux, et elles doivent être prises en compte dans votre décision. Pour les charges de travail de production avec des exigences de performances élevées, nous vous recommandons d’utiliser `managed-premium` pour toutes les classes de stockage. Pour les charges de travail de développement/test, les preuves de concept, etc., où le coût est un critère important, `azurefile` est l’option la moins coûteuse. Ces quatre options peuvent être utilisées dans les situations nécessitant un stockage partagé distant, car toutes les quatre correspondent à des dispositifs de stockage attachés à un réseau dans Azure. Pour plus d’informations, consultez [Stockage AKS](../../aks/concepts-storage.md).|
|**AWS Elastic Kubernetes Service (EKS)**| Le service Elastic Kubernetes Service d’Amazon a une classe de stockage principale basée sur le [pilote de stockage EBS CSI](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html). Ceci est recommandé pour les charges de travail de production. Il existe un nouveau pilote de stockage, [Pilote de stockage EFS CSI](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html), qui peut être ajouté à un cluster EKS, mais il est actuellement en phase bêta et susceptible d’être modifié. Bien qu’AWS indique que ce pilote de stockage est pris en charge pour la production, nous ne recommandons pas de l’utiliser, car il est toujours en version bêta et susceptible d’être modifié. La classe de stockage EBS est la classe par défaut et elle est appelée `gp2`. Pour plus d’informations, consultez [Stockage EKS](https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html).|
|**Google Kubernetes Engine (GKE)**|Google Kubernetes Engine (GKE) a une seule classe de stockage appelée `standard`, qui est utilisée pour les [disques persistants GCE](https://kubernetes.io/docs/concepts/storage/volumes/#gcepersistentdisk). Comme c’est la seule, elle est également la classe de stockage par défaut. Bien qu’il existe un [provisionneur de volume statique local](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/local-ssd#run-local-volume-static-provisioner) pour GKE que vous pouvez utiliser avec des disques SSD en attachement direct, nous vous déconseillons de l’utiliser, car il n’est pas maintenu ni pris en charge par Google. Pour plus d’informations, consultez [Stockage GKE](https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes).
