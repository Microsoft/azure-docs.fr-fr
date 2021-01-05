---
title: Corriger le système d’exploitation Windows dans votre cluster Service Fabric
description: Cet article explique comment automatiser les mises à jour correctives du système d’exploitation sur un cluster Service Fabric à l’aide de Patch Orchestration Application (application d’orchestration des correctifs).
services: service-fabric
documentationcenter: .net
author: athinanthny
manager: chackdan
editor: ''
ms.assetid: de7dacf5-4038-434a-a265-5d0de80a9b1d
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/01/2019
ms.author: atsenthi
ms.openlocfilehash: 8f92501bdb8261a67d3dc2b8aefbe1fb1498ef1e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91445901"
---
# <a name="patch-the-windows-operating-system-in-your-service-fabric-cluster"></a>Corriger le système d’exploitation Windows dans votre cluster Service Fabric

> 
> [!IMPORTANT]
> Depuis le 30 avril 2019, la version de Patch Orchestration Application (application d’orchestration des correctifs) 1.2.* n’est plus prise en charge. Veillez à mettre à niveau vers la dernière version.

> [!NOTE]
> L’obtention de [mises à niveau d’images de système d’exploitation automatiques sur votre groupe de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) est la meilleure façon d’assurer la mise à jour corrective de votre système d’exploitation dans Azure. Le groupe de machines virtuelles identiques basé sur des mises à niveau d’images de système d’exploitation automatiques nécessitent une durabilité Silver ou supérieure sur un groupe identique.
>

 Patch Orchestration Application (POA) est un wrapper autour du service Gestionnaire des réparations Azure Service Fabric, qui permet la planification des correctifs de système d’exploitation basés sur la configuration pour les clusters non hébergés dans Azure. POA n’est pas obligatoire pour les clusters hébergés ailleurs que sur Azure, mais la planification de l’installation de correctifs par le domaine de mise à jour est nécessaire pour corriger les hôtes du cluster Service Fabric sans temps d’arrêt.

POA est une application Service Fabric qui automatise les mises à jour correctives du système d’exploitation sur un cluster Service Fabric sans temps d’arrêt.

POA fournit les fonctionnalités suivantes :

- **Installation automatique de mise à jour du système d’exploitation**. Des mises à jour du système d’exploitation sont téléchargées et installées automatiquement. Les nœuds de cluster sont redémarrés en fonction des besoins sans temps d’arrêt du cluster.

- **Application de correctifs et intégration de l’intégrité adaptées au cluster**. Alors que POA applique des mises à jour, il surveille l’intégrité des nœuds du cluster. Les nœuds du cluster sont mis à jour l’un après l’autre, ou domaine de mise à jour après domaine de mise à jour. Si l’intégrité du cluster se dégrade en raison du processus de mise à jour corrective, la mise à jour est arrêtée pour éviter d’aggraver le problème.

## <a name="internal-details-of-poa"></a>Détails internes de POA

POA est composé des sous-éléments suivants :

- **Service coordinateur** : Ce service avec état est responsable des aspects suivants :
    - coordination de la tâche Windows Update sur le cluster tout entier ;
    - stockage du résultat des opérations Windows Update accomplies.

- **Service Agent du nœud** : Ce service sans état s’exécute sur tous les nœuds de cluster Service Fabric. Les responsabilités du service sont les suivantes :
    - amorçage du service NT Agent du nœud ;
    - surveillance du service NT Agent du nœud.

- **Service NT Agent du nœud** : Ce service Windows NT s’exécute avec un privilège de niveau supérieur (SYSTEM). À l’opposé, les services Agent du nœud et Coordinateur s’exécutent avec des privilèges inférieurs (service réseau). Le service est chargé de l’exécution sur tous les nœuds de cluster des tâches Windows Update suivantes :
    - désactivation des mises à jour automatiques Windows Update sur le nœud ;
    - téléchargement et installation des mises à jour Windows Update en fonction de la stratégie définie par l’utilisateur ;
    - redémarrage de l’ordinateur après installation de Windows Update ;
    - chargement des résultats des mises à jour Windows apportées au service Coordinateur ;
    - génération de rapport d’intégrité en cas d’échec de l’opération une fois toutes les nouvelles tentatives effectuées.

> [!NOTE]
> POA utilise le service Gestionnaire des réparations Service Fabric pour désactiver ou activer le nœud et effectuer des vérifications d’intégrité. La tâche de réparation créée par POA suit la progression de l’exécution de Windows Update pour chaque nœud.

## <a name="prerequisites"></a>Prérequis

> [!NOTE]
> La version minimale de .NET Framework requise est la version 4.6.

### <a name="enable-the-repair-manager-service-if-its-not-running-already"></a>Activer le service Gestionnaire des réparations (s’il est inactif)

Pour que POA puisse fonctionner, le service Gestionnaire des réparations doit être activé sur le cluster.

#### <a name="azure-clusters"></a>Clusters Azure

Le service Gestionnaire des réparations est activé par défaut pour les clusters Azure au niveau de durabilité Silver. Le service Gestionnaire des réparations peut être activé ou non pour les clusters Azure au niveau de durabilité Gold, en fonction du moment de leur création. Le service Gestionnaire des réparations est déactivé par défaut pour les clusters Azure au niveau de durabilité Bronze. Si le service est déjà activé, vous pouvez le voir s’exécuter dans la section des services système de Service Fabric Explorer.

##### <a name="the-azure-portal"></a>Le portail Azure
Vous pouvez activer le service Gestionnaire des réparations à partir du portail Azure lorsque vous configurez le cluster. Lorsque vous configurez le cluster, sélectionnez l’option **Inclure le Gestionnaire des réparations** sous **Fonctionnalités complémentaires**.

![Image représentant l’activation du service Gestionnaire des réparations à partir du portail Azure](media/service-fabric-patch-orchestration-application/EnableRepairManager.png)

##### <a name="the-azure-resource-manager-deployment-model"></a>Le modèle de déploiement Azure Resource Manager
Vous pouvez également utiliser le [modèle de déploiement Azure Resource Manager](./service-fabric-cluster-creation-via-arm.md) pour activer le service Gestionnaire des réparations sur les clusters Service Fabric nouveaux et existants. Récupérez le modèle approprié pour le cluster que vous souhaitez déployer. Vous pouvez soit utiliser les exemples de modèles, soit créer un modèle de déploiement Azure Resource Manager personnalisé. 

Pour activer le service Gestionnaire des réparations à l’aide du [modèle de déploiement Azure Resource Manager](./service-fabric-cluster-creation-via-arm.md), procédez comme suit :

1. Vérifiez que `apiVersion` est défini sur *2017-07-01-preview* pour la ressource *Microsoft.ServiceFabric/clusters*. Si cela ne correspond pas, vous devez mettre à jour `apiVersion` vers *2017-07-01-preview* ou version ultérieure :

    ```json
    {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

1. Activez le service Gestionnaire des réparations en ajoutant la section `addonFeatures` suivante après la section `fabricSettings` :

    ```json
    "fabricSettings": [
        ...      
    ],
    "addonFeatures": [
        "RepairManager"
    ],
    ```

3. Après avoir mis à jour votre modèle de cluster avec ces modifications, appliquez-les et laissez la mise à jour s’accomplir. Vous pouvez maintenant voir le service Gestionnaire des réparations s’exécuter dans votre cluster. Il est nommé *fabric:/System/RepairManagerService* dans la section des services système de Service Fabric Explorer. 

### <a name="standalone-on-premises-clusters"></a>Clusters locaux autonomes

Vous pouvez utiliser les [paramètres de configuration de cluster Windows autonome](./service-fabric-cluster-manifest.md) pour activer le service Gestionnaire des réparations sur un cluster Service Fabric nouveau ou existant.

Pour activer le service Gestionnaire des réparations :

1. Vérifiez que `apiVersion` dans [Configurations de cluster générales](./service-fabric-cluster-manifest.md#general-cluster-configurations) est défini sur *04-2017* ou version ultérieure, comme illustré ici :

    ```json
    {
        "name": "SampleCluster",
        "clusterConfigurationVersion": "1.0.0",
        "apiVersion": "04-2017",
        ...
    }
    ```

1. Activez le service Gestionnaire des réparations en ajoutant la section `addonFeatures` suivante après la section `fabricSettings`, comme illustré ici :

    ```json
    "fabricSettings": [
        ...      
    ],
    "addonFeatures": [
        "RepairManager"
    ],
    ```

1. Mettez à jour le manifeste de cluster avec ces modifications, en utilisant le manifeste de cluster mis à jour, [créez un cluster](./service-fabric-cluster-creation-for-windows-server.md) ou [mettez à niveau la configuration du cluster](./service-fabric-cluster-upgrade-windows-server.md). 

   Une fois que le cluster est en cours d’exécution avec un manifeste de cluster mis à jour, vous pouvez voir le service Gestionnaire des réparations s’exécuter dans votre cluster. Il est nommé *fabric:/System/RepairManagerService* et se situe dans la section des services système de Service Fabric Explorer.

### <a name="configure-windows-updates-for-all-nodes"></a>Configurer les mises à jour Windows pour tous les nœuds

Les mises à jour automatiques Windows peuvent entraîner une perte de disponibilité, car plusieurs nœuds de cluster peuvent redémarrer en même temps. Par défaut, OPA tente de désactiver les mises à jour automatiques Windows sur chaque nœud du cluster. Toutefois, si les paramètres sont gérés par une stratégie d’administrateur ou de groupe, nous vous recommandons de définir la stratégie Windows Update en optant explicitement pour l’envoi d’une notification avant le téléchargement.

## <a name="download-the-application-package"></a>Télécharger le package d’application

Pour télécharger le package d’application, rendez-vous sur la [page de publication Patch Orchestration Application](https://github.com/microsoft/Service-Fabric-POA/releases/latest/) de GitHub.

## <a name="configure-poa-behavior"></a>Configurer le comportement de POA

Vous pouvez configurer le comportement de POA pour répondre à vos besoins. Remplacez les valeurs par défaut en entrant le paramètre d’application durant la création ou la mise à jour de l’application. Il est possible de fournir des paramètres d’application en spécifiant `ApplicationParameter` pour les cmdlets `Start-ServiceFabricApplicationUpgrade` ou `New-ServiceFabricApplication`.

| Paramètre        | Type                          | Détails |
|:-|-|-|
|MaxResultsToCache    |Long                              | Nombre maximal de résultats d’exécution de Windows Update à mettre en cache. <br><br>La valeur par défaut est 3 000, en supposant ce qui suit : <br> &nbsp;&nbsp;- Le nombre de nœuds est 20. <br> &nbsp;&nbsp;- Les mises à jour effectuées sur un nœud sont au nombre de 5 par mois. <br> &nbsp;&nbsp;- Le nombre maximal de résultats par opération est 10. <br> &nbsp;&nbsp;- Les résultats des trois derniers mois doivent être stockés. |
|TaskApprovalPolicy   |Enum <br> { NodeWise, UpgradeDomainWise }                          |TaskApprovalPolicy indique la stratégie que le service Coordinateur doit utiliser pour installer les mises à jour Windows Update sur les nœuds du cluster Service Fabric.<br><br>Les valeurs autorisées sont les suivantes : <br>*NodeWise* : Les mises à jour Windows s’installent de façon séquentielle, nœud après nœud. <br> *UpgradeDomainWise* : Les mises à jour Windows s’installent sur un domaine de mise à jour à la fois. (Au maximum, tous les nœuds appartenant à un domaine de mise à jour peuvent bénéficier d’une mise à jour Windows.)<br><br> Pour déterminer la stratégie la mieux adaptée à votre cluster, consultez la section [FAQ](#frequently-asked-questions).
|LogsDiskQuotaInMB   |Long  <br> (Par défaut : *1024*)               | Taille maximale en Mo des journaux d’activité de l’application d’orchestration des correctifs qui peuvent être conservés localement sur des nœuds.
| WUQuery               | string<br>(Par défaut : *IsInstalled=0*)                | Requête pour obtenir les mises à jour Windows Update. Pour plus d’informations, voir [WuQuery](/windows/win32/api/wuapi/nf-wuapi-iupdatesearcher-search).
| InstallWindowsOSOnlyUpdates | *Booléen* <br> (par défaut : false)                 | Utilisez cet indicateur pour contrôler les mises à jour qui doivent être téléchargées et installées. Les valeurs suivantes sont autorisées <br>True : installe uniquement les mises à jour du système d’exploitation Windows.<br>False : installe toutes les mises à jour disponibles sur l’ordinateur.          |
| WUOperationTimeOutInMinutes | Int <br>(Par défaut : *90*)                   | Spécifie le délai d’expiration de toute opération de Windows Update (rechercher, télécharger ou installer). Si l’opération n’est pas terminée dans le délai imparti, elle est abandonnée.       |
| WURescheduleCount     | Int <br> (Par défaut : *5*)                  | Nombre maximal de fois que le service replanifie une mise à jour Windows en cas d’échec persistant d’une opération.          |
| WURescheduleTimeInMinutes | Int <br>(Par défaut : *30*) | Intervalle auquel le service replanifie les mises à jour de Windows en cas d’échec persistant. |
| WUFrequency           | Chaîne de valeurs séparées par une virgule (par défaut : *Toutes les semaines, Mercredi, 7:00:00*)     | Fréquence d’installation des mises à jour Windows. Le format et les valeurs possibles sont les suivants : <br>-Mensuel, JJ, HH:MM:SS (exemple : *Mensuel, 5, 12:22:32*). Les valeurs autorisées pour le champ _JJ_ (jour) sont des nombres compris entre 1 et 28, et _last_ (dernier). <br>- Hebdomadaire, Jour, HH:MM:SS (exemple : *Hebdomadaire, mardi, 12:22:32*)  <br>- Quotidien, HH:MM:SS (exemple : *Quotidien, 12:22:32*)  <br>- Semaine, Jour, HH:MM:SS (exemple : *2, vendredi, 21:00:00* indique 21h00 UTC le vendredi de la 2e semaine de chaque mois) <br>- *Aucun* indique que les mises à jour Windows ne doivent pas être effectuées.  <br><br> Les heures sont exprimées en UTC.|
| AcceptWindowsUpdateEula | Boolean <br>(Par défaut : *true*) | Lorsque cet indicateur est défini, l’application accepte le contrat de licence utilisateur final de Windows Update pour le compte du propriétaire de l’ordinateur.              |

> [!TIP]
> Si vous souhaitez que les mises à jour Windows aient lieu immédiatement, définissez `WUFrequency` par rapport à l’heure de déploiement de l’application. Par exemple, supposons que vous disposez d’un cluster de test de cinq nœuds et que vous prévoyez de déployer l’application vers 17 heures UTC. Si vous estimez que la mise à niveau ou le déploiement de l’application nécessite au maximum 30 minutes, définissez WUFrequency sur *Quotidienne, 17:30:00*.

## <a name="deploy-poa"></a>Déployer POA

1. Accomplissez toutes les étapes requises pour préparer le cluster.
1. Déployez POA comme n’importe quelle autre application Service Fabric. Pour la déployer avec PowerShell, consultez [Déployer et supprimer des applications avec PowerShell](./service-fabric-deploy-remove-applications.md).
1. Pour configurer l’application au moment du déploiement, passez `ApplicationParameter` à l’applet de commande `New-ServiceFabricApplication`. Pour votre commodité, nous vous avons fourni le script Deploy.ps1 avec l’application. Pour utiliser le script :

    - Connectez-vous à un cluster Service Fabric en utilisant `Connect-ServiceFabricCluster`.
    - Exécutez le script PowerShell Deploy.ps1 avec la valeur `ApplicationParameter` appropriée.

> [!NOTE]
> Conservez le script et le dossier d’application *PatchOrchestrationApplication* dans le même répertoire.

## <a name="upgrade-poa"></a>Mettre POA à niveau

Pour mettre à niveau votre version de POA à l’aide de PowerShell, procédez de la manière décrite dans [Mise à niveau d’applications Service Fabric à l’aide de PowerShell](./service-fabric-application-upgrade-tutorial-powershell.md).

## <a name="remove-poa"></a>Supprimer POA

Pour supprimer l’application, procédez de la manière décrite dans [Déployer et supprimer des applications avec PowerShell](./service-fabric-deploy-remove-applications.md).

Pour votre commodité, nous avons fourni le script Undeploy.ps1 avec l’application. Pour utiliser le script :

  - Connectez-vous à un cluster Service Fabric en utilisant ```Connect-ServiceFabricCluster```.
  - Exécutez le script PowerShell Undeploy.ps1.

> [!NOTE]
> Conservez le script et le dossier d’application *PatchOrchestrationApplication* dans le même répertoire.

## <a name="view-the-windows-update-results"></a>Afficher les résultats des mises à jour Windows Update

POA expose les API REST pour afficher l’historique des résultats aux utilisateurs. Voici un exemple de JSON de résultat :

```json
[
  {
    "NodeName": "_stg1vm_1",
    "WindowsUpdateOperationResults": [
      {
        "OperationResult": 0,
        "NodeName": "_stg1vm_1",
        "OperationTime": "2019-05-13T08:44:56.4836889Z",
        "OperationStartTime": "2019-05-13T08:44:33.5285601Z",
        "UpdateDetails": [
          {
            "UpdateId": "7392acaf-6a85-427c-8a8d-058c25beb0d6",
            "Title": "Cumulative Security Update for Internet Explorer 11 for Windows Server 2012 R2 (KB3185319)",
            "Description": "A security issue has been identified in a Microsoft software product that could affect your system. You can help protect your system by installing this update from Microsoft. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article. After you install this update, you may have to restart your system.",
            "ResultCode": 0,
            "HResult": 0
          }
        ],
        "OperationType": 1,
        "WindowsUpdateQuery": "IsInstalled=0",
        "WindowsUpdateFrequency": "Daily,10:00:00",
        "RebootRequired": false
      }
    ]
  },
  ...
]
```

Les champs JSON sont décrits dans le tableau suivant :

Champ | Valeurs | Détails
-- | -- | --
OperationResult | 0 - Réussi<br> 1 - Réussi avec erreurs<br> 2 - Échec<br> 3 - Abandonné<br> 4 - Abandonné avec délai d’expiration | Indique le résultat de l’opération globale, qui implique généralement l’installation d’une ou de plusieurs mises à jour.
ResultCode | Identique à OperationResult | Ce champ indique le résultat de l’opération d’installation pour une mise à jour individuelle.
OperationType | 1 - Installation<br> 0 - Rechercher et télécharger| Par défaut, l’installation est le seul OperationType qui s’affiche dans les résultats.
WindowsUpdateQuery | La valeur par défaut est « IsInstalled=0 » | Requête Windows Update utilisée pour rechercher les mises à jour. Pour plus d’informations, voir [WuQuery](/windows/win32/api/wuapi/nf-wuapi-iupdatesearcher-search).
RebootRequired | true : le redémarrage était requis<br> false : le redémarrage n’était pas requis | Indique si un redémarrage était requis pour terminer l’installation des mises à jour.
OperationStartTime | DateTime | Indique l’heure de démarrage de l’opération (Téléchargement/Installation).
OperationTime | DateTime | Indique l’heure de fin de l’opération (Téléchargement/Installation).
HResult | 0 - Réussite<br> Autre valeur - Échec| Indique la raison de l’échec de la mise à jour Windows avec updateID « 7392acaf-6a85-427c-8a8d-058c25beb0d6 ».

Si aucune mise à jour n’est planifiée, le JSON de résultat est vide.

Connectez-vous au cluster pour interroger les résultats des mises à jour Windows Update. Déterminez l’adresse IP de réplica pour l’adresse principale du Service Coordinateur, puis ouvrez l’URL suivante à partir du navigateur : http://&lt;REPLICA-IP&gt;:&lt;ApplicationPort&gt;/PatchOrchestrationApplication/v1/GetWindowsUpdateResults.

Le point de terminaison REST pour le service Coordinateur a un port dynamique. Pour vérifier l’URL exacte, reportez-vous à Service Fabric Explorer. Par exemple, les résultats sont disponibles à l’adresse *http://10.0.0.7:20000/PatchOrchestrationApplication/v1/GetWindowsUpdateResults* .

![Image de point de terminaison REST](media/service-fabric-patch-orchestration-application/Rest_Endpoint.png)

Si le proxy inverse est activé sur le cluster, vous pouvez accéder à l’URL également à partir de l’extérieur du cluster.

Le point de terminaison à atteindre est *http://&lt;SERVERURL&gt;:&lt;REVERSEPROXYPORT&gt;/PatchOrchestrationApplication/CoordinatorService/v1/GetWindowsUpdateResults*.

Pour activer le proxy inverse sur le cluster, procédez de la manière décrite dans [Proxy inverse dans Azure Service Fabric](./service-fabric-reverseproxy.md). 

> 
> [!WARNING]
> Une fois le proxy inverse configuré, tous les microservices au sein du cluster, qui exposent un point de terminaison HTTP, sont adressables à partir de l’extérieur du cluster.

## <a name="diagnostics-and-health-events"></a>Diagnostics et événements d’intégrité

Cette section explique comment déboguer ou diagnostiquer les problèmes liés aux mises à jour correctives par le biais de POA sur des clusters Service Fabric.

> [!NOTE]
> La version 1.4.0 de POA ou une version ultérieure doit être installée pour bénéficier de plusieurs des améliorations d’auto-diagnostic ci-dessous.

Le service Node Agent NTService crée des [tâches de réparation](/dotnet/api/system.fabric.repair.repairtask?view=azure-dotnet) pour installer les mises à jour sur les nœuds. Chaque tâche est ensuite préparée par le Service Coordinateur conformément à la stratégie d’approbation des tâches. Enfin, les tâches préparées sont approuvées par le service Gestionnaire des réparations qui n’approuve aucune tâche si le cluster est dans un état non sain. 

Découvrons étape par étape comment se déroulent les mises à jour sur un nœud :

1. S’exécutant sur chaque nœud, NodeAgentNTService recherche les mises à jour Windows disponibles à l’heure prévue. Si des mises à jour sont disponibles, il les télécharge sur le nœud.

1. Une fois les mises à jour téléchargées, le service NT Agent du nœud crée une tâche de réparation correspondante pour le nœud sous le nom *POS___\<unique_id>* . Ces tâches de réparation sont visibles à l’aide de la cmdlet [Get-ServiceFabricRepairTask](/powershell/module/servicefabric/get-servicefabricrepairtask?view=azureservicefabricps) ou dans la section des détails du nœud dans SFX. Une fois créée, la tâche de réparation passe rapidement à [l’état *Revendiqué*](/dotnet/api/system.fabric.repair.repairtaskstate?view=azure-dotnet).

1. Le Service Coordinateur recherche périodiquement les tâches de réparation à l’état *Revendiqué*, puis les met à jour avec l’état *Préparation* conformément à la TaskApprovalPolicy. Si la TaskApprovalPolicy est configurée pour être NodeWise, une tâche de réparation associée à un nœud n’est préparée que si aucune autre tâche de réparation n’est actuellement à l’état *Préparation*, *Approuvé*, *Exécution* ou *Restauration*. 

   De même, une TaskApprovalPolicy UpgradeWise garantit qu’à tout moment seuls les nœuds appartenant au même domaine de mise à jour exécutent des tâches dans les états susmentionnés. Dès qu’une tâche de réparation passe à l’état *Préparation*, le nœud Service Fabric correspondant est [désactivé](/powershell/module/servicefabric/disable-servicefabricnode?view=azureservicefabricps) avec l’intention de *Redémarrer*.

   Les versions 1.4.0 et ultérieures de POA publient des événements avec la propriété ClusterPatchingStatus sur le Service Coordinateur pour afficher les nœuds qui sont corrigés. Les mises à jour sont installées sur _poanode_0, comme illustré dans l’image suivante :

    [![Image de l’état de mise à jour corrective de cluster](media/service-fabric-patch-orchestration-application/clusterpatchingstatus.png)](media/service-fabric-patch-orchestration-application/clusterpatchingstatus.png#lightbox)

1. Une fois le nœud désactivé, la tâche de réparation passe à l’état *Exécution*. 
   
   > [!NOTE]
   > Un nœud bloqué dans un état *Désactivé* peut bloquer une nouvelle tâche de réparation, ce qui entraîne l’arrêt de l’opération de mise à jour corrective sur le cluster.

1. Une fois que la tâche de réparation passe à l’état *Exécution*, l’installation du correctif commence sur ce nœud. Une fois le correctif installé, le nœud peut ou non redémarrer en fonction de celui-ci. Ensuite, la tâche de réparation est déplacée vers l’état *Restauration*, qui réactive le nœud. La tâche de réparation est alors marquée comme terminée.

   Dans les versions 1.4.0 et ultérieures de POA, vous pouvez rechercher l’état de la mise à jour en consultant les événements d’intégrité sur NodeAgentService avec la propriété WUOperationStatus-\<NodeName>. Les sections en surbrillance dans les images ci-dessous montrent l’état des mises à jour Windows sur les nœuds *poanode_0* et *poanode_2* :

   [![Capture d’écran montrant la fenêtre de console de l’état d’opération Windows Update avec poanode_0 en surbrillance.](media/service-fabric-patch-orchestration-application/wuoperationstatusa.png)](media/service-fabric-patch-orchestration-application/wuoperationstatusa.png#lightbox)

   [![Capture d’écran montrant la fenêtre de console de l’état d’opération Windows Update avec poanode_1 en surbrillance.](media/service-fabric-patch-orchestration-application/wuoperationstatusb.png)](media/service-fabric-patch-orchestration-application/wuoperationstatusb.png#lightbox)

   Vous pouvez également vous procurer les détails à l’aide de PowerShell. Pour ce faire, vous vous connectez au cluster et récupérez l’état de la tâche de réparation à l’aide de [Get-ServiceFabricRepairTask](/powershell/module/servicefabric/get-servicefabricrepairtask?view=azureservicefabricps). 
   
   L’exemple ci-dessous montre que la tâche « POS__poanode_2_125f2969-933c-4774-85d1-ebdf85e79f15 » se trouve dans l’état *DownloadComplete*. Cela signifie que les mises à jour ont été téléchargées sur le nœud *poanode_2* et que l’installation sera tentée lorsque la tâche passera à l’état *Exécution*.

   ``` powershell
    D:\service-fabric-poa-bin\service-fabric-poa-bin\Release> $k = Get-ServiceFabricRepairTask -TaskId "POS__poanode_2_125f2969-933c-4774-85d1-ebdf85e79f15"

    D:\service-fabric-poa-bin\service-fabric-poa-bin\Release> $k.ExecutorData
    {"ExecutorSubState":2,"ExecutorTimeoutInMinutes":90,"RestartRequestedTime":"0001-01-01T00:00:00"}
    ```

   S’il y a encore d’autres problèmes, connectez-vous à votre ou vos machines virtuelles et apprenez-en plus sur les problèmes à l’aide des journaux des événements Windows. La tâche de réparation mentionnée précédemment peut exister uniquement dans les sous-états d’exécuteurs suivants :

      ExecutorSubState | Description
    -- | -- 
      None=1 |  Implique qu’il n’y avait pas d’opération en cours sur le nœud. L’état est peut-être en transition.
      DownloadCompleted=2 | Signifie que l’opération de téléchargement s’est achevée avec succès, en échec partiel ou en échec total.
      InstallationApproved=3 | Signifie que l’opération de téléchargement s’est achevée plus tôt et que le Gestionnaire des réparations a approuvé l’installation.
      InstallationInProgress=4 | Correspond à l’état d’exécution de la tâche de réparation.
      InstallationCompleted=5 | Signifie que l’installation s’est achevée avec succès, en réussite partielle ou en échec.
      RestartRequested=6 | Signifie que l’installation du correctif s’est achevée et qu’une action de redémarrage est en attente sur le nœud.
      RestartNotNeeded=7 |  Signifie que le redémarrage n’était pas nécessaire une fois l’installation du correctif terminée.
      RestartCompleted=8 | Signifie que le redémarrage s’est achevé avec succès.
      OperationCompleted=9 | L’opération Windows Update s’est achevée avec succès.
      OperationAborted=10 | Signifie que l’opération Windows Update a été interrompue.

1. Dans les versions 1.4.0 et ultérieures de POA, lorsqu’une tentative de mise à jour de nœud se termine, un événement avec la propriété « WUOperationStatus-[NodeName] » est publié sur NodeAgentService pour vous avertir du démarrage de la prochaine tentative de téléchargement et d’installation des mises à jour Windows. Ceci s’affiche dans l’image suivante :

     [![Capture d’écran montrant état d’opération la fenêtre de console de Windows Update avec NodeAgentService.](media/service-fabric-patch-orchestration-application/wuoperationstatusc.png)](media/service-fabric-patch-orchestration-application/wuoperationstatusc.png#lightbox)

### <a name="diagnostics-logs"></a>Journaux de diagnostics

Les journaux d’activité de l’application d’orchestration des correctifs sont collectés en même temps que les journaux d’activité du runtime Service Fabric.

Vous pouvez capturer des journaux à l’aide de l’outil de diagnostic ou du pipeline de votre choix. POA utilise les ID de fournisseurs fixes suivants pour consigner les événements via la [source de l’événement](/dotnet/api/system.diagnostics.tracing.eventsource?view=netframework-4.5.1) :

- e39b723c-590c-4090-abb0-11e3e6616346
- fc0028ff-bfdc-499f-80dc-ed922c52c5e9
- 24afa313-0d3b-4c7c-b485-1047fd964b60
- 05dc046c-60e9-4ef7-965e-91660adffa68

### <a name="health-reports"></a>Rapports d'intégrité

POA publie également des rapports d’intégrité concernant Node Agent Service ou le Service Coordinateur dans les cas suivants :

* Le service NT Agent du nœud est inactif

   Si le service NT Agent du nœud est inactif sur un nœud, un rapport d’intégrité de niveau avertissement est généré concernant le service Agent du nœud.

* Le service Gestionnaire des réparations n’est pas activé

   Si le service Gestionnaire des réparations est introuvable sur le cluster, un rapport d’intégrité de niveau avertissement est généré concernant le Service Coordinateur.

## <a name="frequently-asked-questions"></a>Forum aux questions

**Q : Pourquoi mon cluster présente-t-il un état d’erreur lorsque POA est en cours d’exécution ?**

A : Durant le processus d’installation, POA désactive ou redémarre des nœuds, ce qui peut entraîner temporairement un cluster défectueux.

Selon la stratégie de l’application, une opération de mise à jour corrective peut entraîner la dégradation d’un nœud isolé *ou* d’un domaine de mise à jour entier.

À la fin de l’installation des mises à jour Windows, les nœuds sont réactivés après redémarrage.

Dans l’exemple suivant, le cluster a présenté temporairement un état d’erreur, car deux nœuds ont été désactivés et la stratégie MaxPercentageUnhealthyNodes a été enfreinte. L’erreur persiste temporairement jusqu’à ce que l’opération de mise à jour corrective puisse commencer.

![Image de cluster défectueux](media/service-fabric-patch-orchestration-application/MaxPercentage_causing_unhealthy_cluster.png)

Si le problème persiste, voir la section Résolution des problèmes.

**Q : Que puis-je faire si POA est dans un état d’avertissement ?**

A : Vérifiez si un rapport d’intégrité publié concernant l’application indique la cause première. En règle générale, l’avertissement contient des détails concernant le problème. Si le problème est temporaire, l’application est supposée récupérer automatiquement.

**Q : Que faire si mon cluster est défectueux et que je dois effectuer une mise à jour urgente du système d’exploitation ?**

A : POA n’installe pas de mises à jour lorsque le cluster est défectueux. Essayez de ramener votre cluster à un état sain pour débloquer le flux de travail de POA.

**Q : Dois-je définir TaskApprovalPolicy en tant que « NodeWise » ou « UpgradeDomainWise » pour mon cluster ?**

A : Le paramètre « UpgradeDomainWise » accélère la réparation globale du cluster en corrigeant en parallèle tous les nœuds qui appartiennent à un domaine de mise à jour. Pendant le processus, les nœuds qui appartiennent à un domaine de mise à jour entier ne sont pas disponibles (état [*Désactivé*](/dotnet/api/system.fabric.query.nodestatus?view=azure-dotnet#System_Fabric_Query_NodeStatus_Disabled)).

En revanche, le paramètre « NodeWise » ne corrige qu’un seul nœud à la fois, ce qui implique que la mise à jour corrective globale du cluster peut prendre plus de temps. Toutefois, au maximum un nœud est indisponible (en état *Désactivé*) pendant le processus de mise à jour corrective.

Si votre cluster peut tolérer l’exécution sur le nombre N-1 de domaines de mise à jour durant le cycle de mise à jour corrective (où N est le nombre total de domaines de mise à jour sur votre cluster), vous pouvez définir la stratégie en tant que « UpgradeDomainWise ». Autrement, affectez-lui la valeur « NodeWise ».

**Q : Combien de temps prend la correction d’un nœud ?**

A : La mise à jour corrective d’un nœud peut prendre de quelques minutes (par exemple, pour les [mises à jour de définitions de Windows Defender](https://www.microsoft.com/en-us/wdsi/definitions)) à plusieurs heures (par exemple, pour les [mises à jour cumulatives de Windows](https://www.catalog.update.microsoft.com/Search.aspx?q=windows%20server%20cumulative%20update)). Le temps nécessaire pour appliquer un correctif à un nœud dépend principalement des éléments suivants : 
 - Taille des mises à jour.
 - Nombre de mises à jour qui doivent être appliquées dans une fenêtre de mise à jour corrective.
 - Temps nécessaire pour installer les mises à jour, redémarrer le nœud (si nécessaire), puis achever les étapes d’installation après redémarrage.
 - Performances de machine virtuelle ou de l’ordinateur et conditions du réseau.

**Q : Combien de temps faut-il pour appliquer un correctif à un cluster entier ?**

A : Le temps nécessaire pour appliquer un correctif à un cluster entier dépend des éléments suivants :

- Temps nécessaire pour appliquer un correctif à un nœud.

- La stratégie du service Coordinateur. La stratégie par défaut, « NodeWise », entraîne l’application d’un correctif à un seul nœud à la fois, une approche plus lente que l’utilisation de « UpgradeDomainWise ». 

   Par exemple : Si l’application d’un correctif à un nœud prend environ 1 heure, pour appliquer un correctif à un cluster de 20 nœuds (du même type) avec 5 domaines de mise à jours, contenant 4 nœuds chacun :
    - Pour « NodeWise » : environ 20 heures.
    - Pour « UpgradeDomainWise » : environ 5 heures.

- Charge du cluster. Chaque opération de mise à jour corrective requiert un déplacement de la charge de travail client vers d’autres nœuds disponibles dans le cluster. Pendant ce temps, le nœud auquel est appliqué le correctif est en état de [*désactivation*](/dotnet/api/system.fabric.query.nodestatus?view=azure-dotnet#System_Fabric_Query_NodeStatus_Disabling). Si le cluster exécute une charge proche du pic, le processus de désactivation prend plus de temps. Par conséquent, le processus de mise à jour corrective global peut sembler lent dans de telles conditions de sollicitation.

- Échecs d’intégrité du cluster pendant la mise à jour corrective. Toute [dégradation](/dotnet/api/system.fabric.health.healthstate?view=azure-dotnet#System_Fabric_Health_HealthState_Error) de [l’intégrité du cluster](./service-fabric-health-introduction.md) interrompt le processus de mise à jour corrective. Ce problème s’ajoute au temps global nécessaire pour appliquer le correctif au cluster entier.

**Q : Pourquoi certaines mises à jour sont-elles affichées dans les résultats Windows Update obtenus via les API REST, et non dans l’historique Windows Update de l’ordinateur ?**

A : Certaines mises à jour de produits s’affichent uniquement dans leur propre historique des mises à jour ou des correctifs. Par exemple, les mises à jour de Windows Defender peuvent apparaître ou non dans l’historique de Windows Update sur Windows Server 2016.

**Q : Puis-je utiliser POA appliquer un correctif à mon cluster de développement (cluster à un nœud) ?**

A : Non, POA ne peut pas être utilisé pour appliquer un correctif à un cluster à un nœud. Cette limitation est due à la conception, car les [services système Service Fabric](./service-fabric-technical-overview.md#system-services) ou d’autres applications clientes occasionnent des temps d’arrêt. Par conséquent, la mise à jour corrective des travaux de réparation n’est jamais approuvée par le service Gestionnaire des réparations.

**Q : Comment corriger des nœuds de cluster sur Linux ?**

A : Consultez [Mises à niveau automatiques d’images de système d’exploitation de groupes de machines virtuelles identiques Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) pour orchestrer les mises à jour sur Linux.

**Q : Pourquoi le cycle de mise à jour prend-il autant de temps ?**

A : Interrogez le JSON de résultat, entrez dans le cycle de mise à jour pour tous les nœuds et vous pouvez essayer de connaître le temps mis par l’installation de la mise à jour sur chaque nœud en utilisant OperationStartTime et OperationTime (OperationCompletionTime). 

S’il existe une fenêtre de temps importante dans laquelle aucune mise à jour n’a lieu, le cluster peut être dans un état d’erreur et, par conséquent, le service Gestionnaire des réparations ne peut pas approuver les tâches de réparation POA. Si l’installation de la mise à jour prend beaucoup de temps sur un nœud quelconque, ce nœud n’a peut-être pas été mis à jour depuis longtemps. Plusieurs mises à jour peuvent être en attente d’installation, ce qui peut entraîner des retards. 

Il est également possible que la mise à jour corrective des nœuds soit bloquée dans l’état *Désactivation*. Cela se produit généralement lorsque la désactivation du nœud peut entraîner des situations de quorum ou de perte de données.

**Q : Pourquoi le nœud doit-il être désactivé lorsque POA lui applique une mise à jour corrective ?**

A : POA désactive le nœud avec l’intention *Redémarrer*, ce qui arrête ou réalloue tous les services Azure Service Fabric qui s’exécutent sur le nœud. POA fait cela pour éviter que les applications utilisent un mélange de nouvelles DLL et d’anciennes DLL. Il est donc recommandé de désactiver un nœud avant d’y installer un correctif.

**Q : Quel est le nombre maximal de nœuds pouvant être mis à jour à l’aide de POA ?**

A : POA utilise le Gestionnaire des réparations Service Fabric pour créer des tâches de réparation pour les nœuds à des fins de mise à jour. Cela étant, un maximum de 250 tâches de réparation peuvent être présentes en même temps. Actuellement, POA crée des tâches de réparation pour chaque nœud simultanément et dès lors, POA ne peut mettre à jour plus de 250 nœuds dans un cluster. 

## <a name="disclaimers"></a>Clauses d’exclusion de responsabilité

- POA accepte le contrat de licence utilisateur final de Windows Update à la place de l’utilisateur. Il est possible de désactiver ce paramètre dans la configuration de l’application.

- POA collecte des données de télémétrie pour effectuer le suivi de l’utilisation et des performances. La télémétrie de l’application suit la valeur du paramètre de télémétrie du runtime Service Fabric (activé par défaut).

## <a name="troubleshooting"></a>Dépannage

Cette section fournit des solutions possibles pour résoudre les problèmes liés à la mise à jour corrective des nœuds.

### <a name="a-node-is-not-coming-back-to-up-state"></a>Un nœud ne se réactive pas

* Le nœud peut être bloqué en état *Désactivation* pour les raisons suivantes :

  - Une vérification de sécurité est en attente. Pour résoudre ce problème, assurez-vous que les nœuds en état d’intégrité normal disponibles sont en nombre suffisant.

* Le nœud peut être bloqué en état *Désactivé* pour les raisons suivantes :

  - Il a été désactivé manuellement.
  - Il a été désactivé en raison d’un travail en cours sur l’infrastructure Azure.
  - Il a été temporairement désactivé par POA pour corriger le nœud.

* Le nœud peut être bloqué en état inactif pour les raisons suivantes :

  - Il a été mis en état inactif manuellement.
  - Il est en cours de redémarrage (qui peut être déclenché par POA).
  - Il a une défaillance de machine virtuelle ou d’ordinateur, ou des problèmes de connectivité réseau.

### <a name="updates-were-skipped-on-some-nodes"></a>Des mises à jour ont été ignorées sur certains nœuds

POA tente d’installer une mise à jour Windows Update conformément à la stratégie de replanification. Le service tente de récupérer le nœud et ignore la mise à jour en vertu de la stratégie d’application.

Dans ce cas, un rapport d’intégrité de niveau avertissement est généré concernant le service Agent du nœud. Le résultat de la mise à jour Windows Update contient également la cause possible de l’échec.

### <a name="the-health-of-the-cluster-goes-to-error-while-the-update-is-being-installed"></a>L’intégrité du cluster présente un état d’erreur pendant l’installation de la mise à jour

Une mise à jour Windows Update défectueuse peut dégrader l’intégrité d’une application ou d’un cluster sur un nœud ou un domaine de mise à jour particuliers. POA interrompt toute autre opération de Windows Update jusqu’à ce que le cluster retrouve son intégrité.

Un administrateur doit intervenir et déterminer la raison pour laquelle l’application ou le cluster sont devenus défectueux en raison d’une opération de Windows Update.

## <a name="poa-release-notes"></a>Notes de publication de POA

>[!NOTE]
> Pour les versions 1.4.0 et versions ultérieures de POA, vous trouverez des notes de publication et des mises en production dans la [page de publication Patch Orchestration Application](https://github.com/microsoft/Service-Fabric-POA/releases/) sur GitHub.

### <a name="version-110"></a>Version 1.1.0
- Version publique

### <a name="version-111"></a>Version 1.1.1
- Correction d’un bogue dans le paramètre SetupEntryPoint de NodeAgentService, qui empêchait l’installation de NodeAgentNTService.

### <a name="version-120"></a>Version 1.2.0

- Résolution de bogues liés au flux de travail de redémarrage du système.
- Résolution d’un bogue lié à la création de tâches RM, suite auquel le contrôle d’intégrité survenant lors de la préparation des tâches de réparation ne s’exécutait pas comme convenu.
- Transition du mode de démarrage du service Windows POANodeSvc, depuis le démarrage automatique vers le démarrage automatique différé.

### <a name="version-121"></a>Version 1.2.1

- Correctif de bogue dans le flux de travail de réduction du cluster. Introduction d’une logique garbage collection pour les tâches de réparation POA appartenant à des nœuds non existants.

### <a name="version-122"></a>Version 1.2.2

- Divers correctifs de bogues.
- Les fichiers binaires sont maintenant signés.
- Ajout du lien sfpkg pour l’application.

### <a name="version-130"></a>Version 1.3.0

- L’affectation de la valeur False à InstallWindowsOSOnlyUpdates installe maintenant toutes les mises à jour disponibles.
- Changement de la logique de désactivation des mises à jour automatiques. Cela résout un bogue à cause duquel les mises à jour automatiques n’étaient pas désactivées sur Server 2016 et ultérieur.
- Contrainte de placement paramétrable pour les deux microservices de l’application d’orchestration des correctifs pour les cas d’utilisation avancés.

### <a name="version-131"></a>Version 1.3.1
- Correction d’une régression où POA 1.3.0 ne fonctionnait pas sur Windows Server 2012 R2 ou antérieur en raison d’une impossibilité de désactiver les mises à jour automatiques. 
- Résolution du bogue avec lequel la configuration InstallWindowsOSOnlyUpdates avait toujours la valeur True.
- Remplacement de la valeur par défaut de InstallWindowsOSOnlyUpdates par la valeur False.

### <a name="version-132"></a>Version 1.3.2
- Correction d’un problème qui affecte le cycle de vie de la correction sur un nœud dans le cas où il existe des nœuds dont le nom est un sous-ensemble du nom du nœud courant. Pour de tels nœuds, il est possible qu’un correctif ait été omis ou qu’un redémarrage soit en cours.
