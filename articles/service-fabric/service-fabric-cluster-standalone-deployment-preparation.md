---
title: Préparation du déploiement d’un cluster autonome
description: Documentation relative à la préparation de l’environnement et la création de la configuration du cluster, à prendre en considération avant de déployer un cluster conçu pour gérer une charge de travail de production.
ms.topic: conceptual
ms.date: 9/11/2018
ms.openlocfilehash: 277c7e047815b3b4171f7cced203ecbe5b68b155
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509170"
---
# <a name="plan-and-prepare-your-service-fabric-standalone-cluster-deployment"></a>Planifier et préparer votre déploiement de cluster Service Fabric autonome

<a id="preparemachines"></a>Exécutez les étapes suivantes avant de créer votre cluster.

## <a name="plan-your-cluster-infrastructure"></a>Planifier votre infrastructure de cluster
Vous allez créer un cluster Service Fabric sur les ordinateurs que vous avez pour décider des types de défaillances auxquels le cluster doit survivre. Par exemple, devez-vous séparer les lignes d’alimentation des connexions internet reliées à ces ordinateurs ? Prenez en outre en compte la sécurité physique de ces ordinateurs. Où sont situés les ordinateurs et qui a besoin d’y accéder ? Une fois ces décisions prises, vous pouvez mapper logiquement les ordinateurs aux différents domaines d’erreur (voir l’étape suivante). La planification de l’infrastructure pour les clusters de production est plus complexe que pour les clusters de test.

## <a name="determine-the-number-of-fault-domains-and-upgrade-domains"></a>Déterminer le nombre de domaines d’erreur et de domaines de mise à niveau
Un [*domaine d’erreur* (FD)](service-fabric-cluster-resource-manager-cluster-description.md) est une unité physique de défaillance directement liée à l’infrastructure physique dans les centres de données. Un domaine d’erreur est constitué de composants matériels (ordinateurs, commutateurs, réseaux, etc.) qui partagent un point de défaillance unique. Bien qu’il n’existe aucun mappage 1:1 entre les domaines d’erreur et les racks, chaque rack peut être considéré au sens large comme un domaine d’erreur.

Quand vous spécifiez des domaines d’erreur dans ClusterConfig.json, vous pouvez choisir le nom de chaque domaine d’erreur. Service Fabric prend en charge les domaines d’erreur hiérarchiques pour vous permettre d’y refléter votre topologie d’infrastructure.  Par exemple, les domaines d’erreur suivants sont valides :

* "faultDomain": "fd:/Room1/Rack1/Machine1"
* "faultDomain": "fd:/FD1"
* "faultDomain": "fd:/Room1/Rack1/PDU1/M1"

Un *domaine de mise à niveau (UD)* est une unité logique de nœuds. Pendant les mises à niveau orchestrées par Service Fabric (mise à niveau de l’application ou du cluster), tous les nœuds dans un domaine de mise à niveau sont mis hors service pour effectuer la mise à niveau alors que les nœuds dans d’autres domaines de mise à niveau restent disponibles pour répondre aux requêtes. Les mises à niveau du microprogramme que vous effectuez sur vos ordinateurs ne respectent pas les domaines de mise à niveau. Vous devez donc les exécuter sur un ordinateur à la fois.

L’approche la plus simple de ces concepts consiste à considérer les domaines d’erreur comme unité de défaillance non planifiée et les domaines mise à niveau comme unité de maintenance planifiée.

Quand vous spécifiez des domaines de mise à niveau dans ClusterConfig.json, vous pouvez choisir le nom de chaque domaine de mise à niveau. Par exemple, les noms suivants sont valides :

* « upgradeDomain » : « UD0 »
* « upgradeDomain » : « UD1A »
* « upgradeDomain » : « DomainRed »
* « upgradeDomain » : « Blue »

Pour plus d’informations sur les domaines d’erreur et les domaines de mise à niveau, consultez [Description d’un cluster Service Fabric](service-fabric-cluster-resource-manager-cluster-description.md).

Un cluster en production doit s’étendre sur au moins trois domaines d’erreur afin d’être pris en charge dans un environnement de production si vous avez un contrôle total sur la maintenance et la gestion des nœuds, à savoir, si vous êtes responsable de la mise à jour et du remplacement des machines. Pour les clusters exécutés dans des environnements (à savoir, les instances de machine virtuelle Amazon Web Services) où vous n’avez pas un contrôle total sur les machines, vous devez avoir un minimum de cinq domaines d’erreur dans votre cluster. Chaque domaine d’erreur peut avoir un ou plusieurs nœuds. Cela permet d’éviter les problèmes causés par des mises à niveau et mises à jour de la machine qui, en fonction du calendrier, peuvent interférer avec l’exécution d’applications et de services dans les clusters.

## <a name="determine-the-initial-cluster-size"></a>Déterminer la taille initiale du cluster

En règle générale, le nombre de nœuds de votre cluster est déterminé par les besoins de votre entreprise, à savoir, le nombre de services et de conteneurs qui s’exécuteront sur le cluster et le nombre de ressources nécessaires pour gérer vos charges de travail. Pour les clusters de production, nous vous recommandons d’avoir au moins cinq nœuds dans votre cluster, qui s’étendent sur 5 domaines d’erreur. Toutefois, comme décrit ci-dessus, si vous avez un contrôle total sur les nœuds et que vous pouvez couvrir trois domaines d’erreur, trois nœuds doivent également s’avérer suffisants.

Les clusters de test exécutant des charges de travail avec état doivent avoir trois nœuds, tandis que les clusters de test exécutant uniquement des charges de travail sans état n’ont besoin que d’un nœud. Notez également qu’à des fins de développement, vous pouvez avoir plusieurs nœuds sur un ordinateur donné. Toutefois, dans un environnement de production, Service Fabric ne prend en charge qu’un seul nœud par ordinateur physique ou virtuel.

## <a name="prepare-the-machines-that-will-serve-as-nodes"></a>Préparer les ordinateurs qui serviront de nœuds

Voici les spécifications recommandées pour les ordinateurs d’un cluster Service Fabric :

* Un minimum de 16 Go de RAM
* Un minimum de 40 Go d’espace disque disponible
* Un processeur 4 cœurs ou plus
* Connectivité à un ou plusieurs réseaux sécurisés pour tous les ordinateurs
* Système d’exploitation Windows Server installé (versions valides 2012 R2, 2016, 1709 ou 1803). Les versions 6.4.654.9590 et ultérieures de Service Fabric prennent également en charge Windows Server 2019 et 1809.
* [.NET Framework 4.5.1 ou version ultérieure](https://www.microsoft.com/download/details.aspx?id=40773), installation complète
* [Windows PowerShell 3.0](/powershell/scripting/windows-powershell/install/installing-windows-powershell?view=powershell-7)
* Le [service RemoteRegistry](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754820(v=ws.11)) doit être exécuté sur tous les ordinateurs
* **Le lecteur d’installation de Service Fabric doit être un système de fichiers NTFS.**
* **Les services Windows *Journaux de performances et alertes* et *Journal des événements Windows* doivent [être activés](/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc755249(v=ws.11))** .

> [!IMPORTANT]
> L’administrateur de cluster déployant et configurant le cluster doit disposer de [privilèges d’administrateur](https://social.technet.microsoft.com/wiki/contents/articles/13436.windows-server-2012-how-to-add-an-account-to-a-local-administrator-group.aspx) sur chaque ordinateur. Vous ne pouvez pas installer Service Fabric sur un contrôleur de domaine.

## <a name="download-the-service-fabric-standalone-package-for-windows-server"></a>Télécharger le package autonome Service Fabric pour Windows Server
[Lien de téléchargement - Package autonome Service Fabric - Windows Server](https://go.microsoft.com/fwlink/?LinkId=730690) et décompressez le package sur un ordinateur de déploiement qui ne fait pas partie du cluster ou sur l’un des ordinateurs qui fera partie de votre cluster.

## <a name="modify-cluster-configuration"></a>Modifier la configuration du cluster
Pour créer un cluster autonome, vous devez créer un fichier ClusterConfig.json de configuration de cluster autonome qui décrit les spécifications du cluster. Vous pouvez baser le fichier de configuration sur les modèles accessibles avec le lien ci-dessous. <br>
[Configurations de cluster autonome](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)

Pour plus d’informations sur les sections de ce fichier, consultez [Paramètres de configuration pour un cluster Windows autonome](service-fabric-cluster-manifest.md).

Ouvrez l’un des fichiers ClusterConfig.json à partir du package que vous avez téléchargé et modifiez les paramètres suivants :

| **Paramètres de configuration** | **Description** |
| --- | --- |
| **NodeTypes** |Les types de nœuds permettent de séparer vos nœuds de cluster en différents groupes. Un cluster doit comprendre au moins un NodeType. Tous les nœuds d’un groupe ont les caractéristiques communes suivantes : <br> **Name** : il s’agit du nom de type de nœud. <br>**Endpoints Ports** : il s’agit de différents points de terminaison (ports) nommés qui sont associés à ce type de nœud. Vous pouvez utiliser n’importe quel numéro de port, tant qu’il n’entre pas en conflit avec un autre élément de ce manifeste et qu’il n’est pas déjà utilisé par une autre application en cours d’exécution sur l’ordinateur/la machine virtuelle. <br> **Placement Properties** : il s’agit des propriétés pour ce type de nœud que vous avez utilisées comme contraintes de positionnement pour les services système ou pour vos services. Ces propriétés sont des paires clé/valeur définies par l’utilisateur qui fournissent des métadonnées supplémentaires pour un nœud donné. La présence d’un disque dur ou d’une carte graphique sur le nœud, le nombre de rotations du disque dur, les noyaux et d’autres propriétés physiques sont des exemples de propriétés du nœud. <br> **Capacities** : les capacités du nœud définissent le nom et la quantité d’une ressource spécifique disponible sur un nœud particulier pour consommation. Par exemple, un nœud peut définir qu’il possède la capacité pour une mesure appelée « MemoryInMb » et qu’il dispose de 2 048 Mo de mémoire disponible par défaut. Ces capacités sont utilisées au moment de l’exécution pour garantir que les services qui nécessitent une quantité spécifique de ressources sont placés sur des nœuds où ces ressources sont disponibles dans la quantité requise.<br>**IsPrimary** : si plusieurs NodeType sont définis, vérifiez qu’un seul est défini comme primaire avec la valeur *true*; c’est là que les services système s’exécutent. Tous les autres types de nœuds doivent avoir la valeur *false* |
| **Nœuds** |Il s’agit des détails de chacun des nœuds qui font partie du cluster (type de nœud, nom de nœud, adresse IP, domaine d’erreur et domaine de mise à niveau du nœud). Les ordinateurs sur lesquels vous souhaitez créer le cluster doivent être répertoriés ici avec leur adresse IP. <br> Si vous utilisez la même adresse IP pour tous les nœuds, un cluster à boîtier unique est créé. Vous pouvez l’utiliser à des fins de test. N’utilisez pas les clusters à boîtier unique pour le déploiement de charges de travail de production. |

Une fois tous les paramètres de la configuration du cluster configurés dans l’environnement, vous pouvez tester la configuration dans l’environnement du cluster (étape 7).

<a id="environmentsetup"></a>

## <a name="environment-setup"></a>Configuration de l’environnement

Lorsqu’un administrateur de cluster configure un cluster autonome Service Fabric, l’environnement doit être configuré avec les critères suivants : <br>
1. L’utilisateur qui crée le cluster doit disposer de privilèges de sécurité de niveau administrateur sur tous les ordinateurs qui sont répertoriés en tant que nœuds dans le fichier de configuration du cluster.
2. L’ordinateur à partir duquel le cluster est créé, ainsi que chaque ordinateur nœud du cluster doit respecter les conditions suivantes :
   * le Kit de développement logiciel (SDK) Service Fabric doit être désinstallé ;
   * le runtime Service Fabric doit être désinstallé ;
   * le service pare-feu Windows (mpssvc) doit être activé ;
   * le service Registre distant (remoteregistry) doit être activé ;
   * le partage de fichiers (SMB) doit être activé ;
   * les ports requis doivent être ouverts, selon les ports de configuration du cluster ;
   * les ports requis doivent êtres ouverts pour le service Registre distant et SMB Windows : 135, 137, 138, 139 et 445 ;
   * la connectivité réseau entre les ordinateurs doit être activée.
3. Aucun des ordinateurs nœuds de cluster ne doit être un contrôleur de domaine.
4. Si le cluster à déployer est un cluster sécurisé, vérifiez que les conditions préalables de sécurité requises sont en place et correctement configurées en fonction de la configuration.
5. Si les ordinateurs du cluster ne sont pas accessibles via Internet, définissez les éléments suivants dans la configuration du cluster :
   * Désactiver la télémétrie : sous *Propriétés*, définissez *« enableTelemetry » : false*
   * Désactiver le téléchargement de version Fabric automatique et les notifications indiquant que la prise en charge de la version actuelle touche à sa fin : sous *Propriétés*, définissez *« fabricClusterAutoupgradeEnabled » : false*
   * De même, si l’accès réseau à Internet est limité à des domaines sur liste d’autorisation, les domaines ci-dessous sont nécessaires pour la mise à niveau automatique : go.microsoft.com download.microsoft.com

6. Définissez les exclusions antivirus Service Fabric appropriées :

| **Répertoires exclus de l’antivirus** |
| --- |
| Program Files\Microsoft Service Fabric |
| FabricDataRoot (dans la configuration du cluster) |
| FabricLogRoot (dans la configuration du cluster) |

| **Processus exclus de l’antivirus** |
| --- |
| Fabric.exe |
| FabricHost.exe |
| FabricInstallerService.exe |
| FabricSetup.exe |
| FabricDeployer.exe |
| ImageBuilder.exe |
| FabricGateway.exe |
| FabricDCA.exe |
| FabricFAS.exe |
| FabricUOS.exe |
| FabricRM.exe |
| FileStoreService.exe |

## <a name="validate-environment-using-testconfiguration-script"></a>Valider l’environnement à l’aide du script TestConfiguration
Vous trouverez le script TestConfiguration.ps1 dans le package autonome. Il est utilisé comme un outil d’analyse des meilleures pratiques pour valider certains des critères ci-dessus et doit être utilisé comme contrôle de la validité pour vérifier si un cluster peut être déployé dans un environnement donné. En cas de défaillance, consultez la liste sous [Configuration de l’environnement](service-fabric-cluster-standalone-deployment-preparation.md) pour la résolution des problèmes.

Ce script peut être exécuté sur n’importe quel ordinateur disposant d’un accès administrateur à tous les ordinateurs qui sont répertoriés en tant que nœuds dans le fichier de configuration du cluster. L’ordinateur sur lequel ce script est exécuté ne doit pas nécessairement faire partie du cluster.

```powershell
PS C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer> .\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json
Trace folder already exists. Traces will be written to existing trace folder: C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer\DeploymentTraces
Running Best Practices Analyzer...
Best Practices Analyzer completed successfully.


LocalAdminPrivilege        : True
IsJsonValid                : True
IsCabValid                 : True
RequiredPortsOpen          : True
RemoteRegistryAvailable    : True
FirewallAvailable          : True
RpcCheckPassed             : True
NoConflictingInstallations : True
FabricInstallable          : True
Passed                     : True
```

Actuellement, ce module de test de configuration ne valide pas la configuration de sécurité. Cette opération doit donc être effectuée de manière indépendante.

> [!NOTE]
> Nous améliorons en permanence ce module afin de le rendre plus fiable. Par conséquent, s’il existe un cas défectueux ou manquant qui, selon vous, n’est pas actuellement détecté par TestConfiguration, faites-le nous savoir via nos [canaux de prise en charge](./service-fabric-support.md).
>
>

## <a name="next-steps"></a>Étapes suivantes
* [Créer un cluster autonome s’exécutant sur Windows Server](service-fabric-cluster-creation-for-windows-server.md)
