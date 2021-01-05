---
title: Configuration des Reliable Services dans Azure Service Fabric
description: Apprenez-en davantage sur la configuration des Reliable Services avec état dans une application Azure Service Fabric à l’échelle mondiale et pour un service unique.
author: sumukhs
ms.topic: conceptual
ms.date: 10/02/2017
ms.author: sumukhs
ms.custom: devx-track-csharp
ms.openlocfilehash: cda0a9f988afae58a60bff051885a5eec8afe434
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023582"
---
# <a name="configure-stateful-reliable-services"></a>Configuration des services fiables (Reliable Services) avec état
Il existe deux ensembles de paramètres de configuration pour les services fiables (Reliable Services). L’un des ensembles est global pour tous les services fiables dans le cluster, alors que l’autre est spécifique à un service fiable.

## <a name="global-configuration"></a>Configuration globale
La configuration de service fiable globale est spécifiée dans le manifeste de cluster sous la section KtlLogger. Elle permet la configuration de l’emplacement et de la taille du journal partagé, ainsi que des limites de mémoire globales utilisées par l’enregistreur d’événements. Le manifeste de cluster est un fichier XML simple qui contient les paramètres et les configurations qui s’appliquent à l’ensemble des nœuds et des services du cluster. Le fichier a généralement pour nom ClusterManifest.xml. Vous pouvez voir le manifeste de cluster de votre cluster à l’aide de la commande PowerShell Get-ServiceFabricClusterManifest.

### <a name="configuration-names"></a>Noms de configuration
| Name | Unité | Valeur par défaut | Notes |
| --- | --- | --- | --- |
| WriteBufferMemoryPoolMinimumInKB |Ko |8388608 |Nombre minimal de Ko à allouer en mode noyau pour le pool de mémoire tampon d’écriture de l’enregistreur d’événements. Ce pool de mémoire est utilisé pour la mise en cache des informations d’état avant l’écriture sur le disque. |
| WriteBufferMemoryPoolMaximumInKB |Ko |Aucune limite |Taille maximale que peut atteindre le pool de mémoire tampon d’écriture de l’enregistreur d’événements. |
| SharedLogId |GUID |"" |Spécifie un GUID unique à utiliser pour identifier le fichier journal partagé par défaut utilisé par tous les services fiables sur tous les nœuds du cluster qui ne spécifient pas l’élément SharedLogId dans leur configuration de service spécifique. Si SharedLogId est spécifié, SharedLogPath doit l’être aussi. |
| SharedLogPath |Nom de chemin complet |"" |Spécifie le chemin d’accès complet du fichier journal partagé utilisé par tous les services fiables sur tous les nœuds du cluster qui ne spécifient pas l’élément SharedLogPath dans leur configuration de service spécifique. Toutefois, si SharedLogPath est spécifié, SharedLogId doit l'être aussi. |
| SharedLogSizeInMB |Mo |8 192 |Spécifie le nombre de Mo d’espace disque à allouer de manière statique pour le journal partagé. La valeur doit être supérieure ou égale à 2 048. |

Dans Azure ARM ou un modèle JSON local, l’exemple ci-dessous montre comment modifier le journal des transactions partagé qui est créé pour sauvegarder toutes les collections fiables pour les services avec état.

```json
"fabricSettings": [{
    "name": "KtlLogger",
    "parameters": [{
        "name": "SharedLogSizeInMB",
        "value": "4096"
    }]
}]
```

### <a name="sample-local-developer-cluster-manifest-section"></a>Exemple de section du manifeste de cluster pour développeur local
Si vous souhaitez modifier cette valeur dans votre environnement de développement local, vous devez modifier le fichier local clustermanifest.xml.

```xml
   <Section Name="KtlLogger">
     <Parameter Name="SharedLogSizeInMB" Value="4096"/>
     <Parameter Name="WriteBufferMemoryPoolMinimumInKB" Value="8192" />
     <Parameter Name="WriteBufferMemoryPoolMaximumInKB" Value="8192" />
     <Parameter Name="SharedLogId" Value="{7668BB54-FE9C-48ed-81AC-FF89E60ED2EF}"/>
     <Parameter Name="SharedLogPath" Value="f:\SharedLog.Log"/>
   </Section>
```

### <a name="remarks"></a>Notes
L’enregistreur dispose d’un pool global de mémoire allouée provenant de la mémoire du noyau non paginée disponible pour tous les services fiables sur un nœud pour mettre en cache les données d’état avant l’écriture dans le journal dédié associé du réplica de service fiable. La taille du pool est contrôlée par les paramètres WriteBufferMemoryPoolMinimumInKB et WriteBufferMemoryPoolMaximumInKB. WriteBufferMemoryPoolMinimumInKB spécifie la taille initiale de ce pool de mémoire et la taille minimale à laquelle peut se réduire le pool de mémoire. WriteBufferMemoryPoolMaximumInKB est la taille maximale que peut atteindre le pool de mémoire. Chaque réplica de service fiable qui est ouvert peut augmenter la taille du pool de mémoire d’une quantité déterminée par le système jusqu’à WriteBufferMemoryPoolMaximumInKB. Si la demande de mémoire au pool de mémoire dépasse la quantité disponible, les requêtes de mémoire sont retardées jusqu’à ce que la mémoire soit disponible. Par conséquent, si le pool de mémoire tampon d’écriture est trop petit pour une configuration particulière, les performances peuvent être affectées.

Les paramètres SharedLogId et SharedLogPath sont toujours utilisés ensemble pour définir le GUID et l’emplacement du journal partagé par défaut pour tous les nœuds du cluster. Le journal partagé par défaut est utilisé pour tous les services fiables qui ne spécifient pas les paramètres dans le fichier settings.xml pour le service spécifique. Pour des performances optimales, les fichiers journaux partagés doivent être placés sur des disques uniquement utilisés pour le fichier journal partagé afin de réduire la contention.

SharedLogSizeInMB spécifie la quantité d’espace disque à préallouer pour le journal partagé par défaut sur tous les nœuds.  Il n’est pas nécessaire de spécifier SharedLogId et SharedLogPath pour spécifier SharedLogSizeInMB.

## <a name="service-specific-configuration"></a>Configuration spécifiques à un service
Vous pouvez modifier la configuration par défaut des Reliable Services avec état par le biais du package de configuration (Config) ou l'implémentation du service (Code).

* **Config** : vous pouvez réaliser la configuration par le biais du package de configuration en modifiant le fichier Settings.xml généré dans la racine du package Microsoft Visual Studio, dans le dossier Config de chaque service dans l'application.
* **Code** - La configuration par l’intermédiaire du code est obtenue en créant un ReliableStateManager à l’aide d’un objet ReliableStateManagerConfiguration avec les options appropriées définies.

Par défaut, le runtime Azure Service Fabric recherche des noms de sections prédéfinis dans le fichier Settings.xml et utilise les valeurs de configuration pendant la création des composants runtime sous-jacents.

> [!NOTE]
> Ne supprimez **pas** les noms de sections des configurations suivantes dans le fichier Settings.xml généré dans la solution Visual Studio, sauf si vous envisagez de configurer votre service au moyen du code.
> Renommer les noms de package ou de la section de configuration nécessite une modification du code lors de la configuration de ReliableStateManager.
> 
> 

### <a name="replicator-security-configuration"></a>Configuration de la sécurité du réplicateur
Les configurations de sécurité du réplicateur sont utilisées pour sécuriser le canal de communication utilisé lors de la réplication. Cela signifie que les services ne sont pas en mesure d'afficher leur trafic de réplication mutuel, ce qui garantit que les données hautement disponibles soient également sécurisées. Par défaut, une section de configuration de sécurité vide empêche de sécuriser la réplication.

> [!IMPORTANT]
> Sur les nœuds Linux, les certificats doivent être au format PEM. Pour en savoir plus sur la localisation et la configuration des certificats pour Linux, consultez [Configurer des certificats sur Linux](./service-fabric-configure-certificates-linux.md). 
> 
> 

### <a name="default-section-name"></a>Nom de la section par défaut
ReplicatorSecurityConfig

> [!NOTE]
> Pour renommer ce nom de section, substituez le paramètre replicatorSecuritySectionName avec le constructeur ReliableStateManagerConfiguration lors de la création de la classe ReliableStateManager pour ce service.
> 
> 

### <a name="replicator-configuration"></a>Configuration du réplicateur
Les configurations du réplicateur configurent le réplicateur garantissant la haute fiabilité de l'état du service fiable avec état par la réplication et la conservation de l'état localement.
La configuration par défaut est générée par le modèle Visual Studio et devrait suffire. Cette section décrit les configurations supplémentaires disponibles pour paramétrer le réplicateur.

### <a name="default-section-name"></a>Nom de la section par défaut
ReplicatorConfig

> [!NOTE]
> Pour renommer ce nom de section, substituez le paramètre replicatorSettingsSectionName avec le constructeur ReliableStateManagerConfiguration lors de la création de la classe ReliableStateManager pour ce service.
> 
> 

### <a name="configuration-names"></a>Noms de configuration
| Name | Unité | Valeur par défaut | Notes |
| --- | --- | --- | --- |
| BatchAcknowledgementInterval |Secondes |0.015 |Durée d'attente du réplicateur secondaire après la réception d'une opération et avant de renvoyer un accusé de réception au réplicateur principal. Tous les autres accusés de réception à envoyer pour les opérations traitées durant cet intervalle sont envoyés sous la forme d'une réponse. |
| ReplicatorEndpoint |N/A |Aucune valeur par défaut (paramètre obligatoire) |Adresse IP et port que le réplicateur principal/secondaire utilise pour communiquer avec d'autres réplicateurs dans le jeu de réplicas. Doit faire référence à un point de terminaison de ressource TCP dans le manifeste de service. Reportez-vous aux [ressources du manifeste de service](service-fabric-service-manifest-resources.md) pour en savoir plus sur la définition des ressources de point de terminaison dans le manifeste de service. |
| MaxPrimaryReplicationQueueSize |Nombre d'opérations |8 192 |Nombre maximal d'opérations dans la file d'attente principale. Une opération est libérée quand le réplicateur principal reçoit un accusé de réception de tous les réplicateurs secondaires. Cette valeur doit être supérieure à 64 et être une puissance de 2. |
| MaxSecondaryReplicationQueueSize |Nombre d'opérations |16384 |Nombre maximal d'opérations dans la file d'attente secondaire. Une opération est libérée une fois son état devenu hautement disponible grâce à la persistance. Cette valeur doit être supérieure à 64 et être une puissance de 2. |
| CheckpointThresholdInMB |Mo |50 |Quantité d'espace du fichier journal après lequel l'état est vérifié. |
| MaxRecordSizeInKB |Ko |1 024 |Taille maximale de l'enregistrement que le réplicateur peut écrire dans le journal. Cette valeur doit être un multiple de 4 et supérieure à 16. |
| MinLogSizeInMB |Mo |0 (système déterminé) |Taille minimale du journal des transactions. Le journal ne peut pas être tronqué à une taille inférieure à ce paramètre. 0 indique que le réplicateur détermine la taille minimale du journal. Si vous augmentez cette valeur, vous augmentez la possibilité de faire des copies partielles et des sauvegardes incrémentielles, car la probabilité de la troncation des enregistrements de journaux pertinents est réduite. |
| TruncationThresholdFactor |Factor |2 |Détermine à partir de quelle taille du journal la troncation est déclenchée. Le seuil de troncation est déterminé par MinLogSizeInMB multiplié par TruncationThresholdFactor. TruncationThresholdFactor doit être supérieur à 1. MinLogSizeInMB * TruncationThresholdFactor doit être inférieur à MaxStreamSizeInMB. |
| ThrottlingThresholdFactor |Factor |4 |Détermine à partir de quelle taille du journal la troncation du réplication commence. Le seuil de limitation (en Mo) est déterminé par Max ((MinLogSizeInMB * ThrottlingThresholdFactor),(CheckpointThresholdInMB * ThrottlingThresholdFactor)). Le seuil de limitation (en Mo) doit être supérieur au seuil de troncation (en Mo). Le seuil de troncation (en Mo) doit être inférieur à MaxStreamSizeInMB. |
| MaxAccumulatedBackupLogSizeInMB |Mo |800 |Taille cumulée maximale (en Mo) des journaux d’activité de sauvegarde dans une séquence de journaux d’activité de sauvegarde donnée. Des demandes de sauvegarde incrémentielle échouent si la sauvegarde incrémentielle génère un journal de sauvegarde qui provoque la cumulation des journaux d’activité de sauvegarde, étant donné que la sauvegarde complète pertinente est supérieure à cette taille. Dans ce cas, l’utilisateur doit effectuer une sauvegarde complète. |
| SharedLogId |GUID |"" |Spécifie un GUID unique à utiliser pour identifier le fichier journal partagé utilisé avec ce réplica. En règle générale, les services ne doivent pas utiliser ce paramètre. Toutefois, si SharedLogId est spécifié, SharedLogPath doit l'être aussi. |
| SharedLogPath |Nom de chemin complet |"" |Spécifie le chemin d'accès complet où sera créé le fichier journal partagé pour ce réplica. En règle générale, les services ne doivent pas utiliser ce paramètre. Toutefois, si SharedLogPath est spécifié, SharedLogId doit l'être aussi. |
| SlowApiMonitoringDuration |Secondes |300 |Définit l’intervalle de surveillance pour les appels d’API gérées. Exemple : fonction de rappel de sauvegarde fournie par l’utilisateur. Une fois l’intervalle écoulé, un rapport d’intégrité Warning sera adressé au Gestionnaire d’intégrité. |
| LogTruncationIntervalSeconds |Secondes |0 |Intervalle configurable à partir duquel la troncature des journaux se lance sur chaque réplica. Il permet de s’assurer que le journal est également tronqué en fonction du temps plutôt que de sa taille uniquement. Ce paramètre permet également de forcer le vidage des entrées supprimées dans un dictionnaire fiable. Il peut donc servir à s’assurer que les éléments supprimés sont nettoyés en temps opportun. |
| EnableStableReads |Boolean |False |Le fait d’activer les lectures stables a pour effet de restreindre les réplicas secondaires aux valeurs retournées demandées par le quorum. |

### <a name="sample-configuration-via-code"></a>Exemple de configuration au moyen du code
```csharp
class Program
{
    /// <summary>
    /// This is the entry point of the service host process.
    /// </summary>
    static void Main()
    {
        ServiceRuntime.RegisterServiceAsync("HelloWorldStatefulType",
            context => new HelloWorldStateful(context, 
                new ReliableStateManager(context, 
        new ReliableStateManagerConfiguration(
                        new ReliableStateManagerReplicatorSettings()
            {
                RetryInterval = TimeSpan.FromSeconds(3)
                        }
            )))).GetAwaiter().GetResult();
    }
}    
```
```csharp
class MyStatefulService : StatefulService
{
    public MyStatefulService(StatefulServiceContext context, IReliableStateManagerReplica stateManager)
        : base(context, stateManager)
    { }
    ...
}
```


### <a name="sample-configuration-file"></a>Exemple de fichier de configuration
```xml
<?xml version="1.0" encoding="utf-8"?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Section Name="ReplicatorConfig">
      <Parameter Name="ReplicatorEndpoint" Value="ReplicatorEndpoint" />
      <Parameter Name="BatchAcknowledgementInterval" Value="0.05"/>
      <Parameter Name="CheckpointThresholdInMB" Value="512" />
   </Section>
   <Section Name="ReplicatorSecurityConfig">
      <Parameter Name="CredentialType" Value="X509" />
      <Parameter Name="FindType" Value="FindByThumbprint" />
      <Parameter Name="FindValue" Value="9d c9 06 b1 69 dc 4f af fd 16 97 ac 78 1e 80 67 90 74 9d 2f" />
      <Parameter Name="StoreLocation" Value="LocalMachine" />
      <Parameter Name="StoreName" Value="My" />
      <Parameter Name="ProtectionLevel" Value="EncryptAndSign" />
      <Parameter Name="AllowedCommonNames" Value="My-Test-SAN1-Alice,My-Test-SAN1-Bob" />
   </Section>
</Settings>
```


### <a name="remarks"></a>Notes
BatchAcknowledgementInterval contrôle la latence de réplication. La valeur « 0 » entraîne la latence la plus faible possible, au détriment du débit (car davantage de messages d'accusé de réception doivent être envoyés et traités, chacun contenant moins d'accusés de réception).
Plus la valeur de BatchAcknowledgementInterval est élevée, plus le débit de réplication général est élevé, au détriment d'une plus grande latence de l'opération. Cela se traduit directement par une latence dans la validation des transactions.

La valeur CheckpointThresholdInMB contrôle la quantité d'espace disque que le réplicateur peut utiliser pour stocker des informations d'état dans le fichier journal dédié au réplica. Son augmentation pour une valeur supérieure à celle par défaut peut entraîner des temps de reconfiguration plus rapides quand un nouveau réplica est ajouté à l'ensemble. Cela est dû au transfert d'état partiel qui intervient suite à la disponibilité d'un historique des opérations plus important dans le journal. Cela peut potentiellement accroître le temps de récupération d’un réplica après un blocage.

Le paramètre MaxRecordSizeInKB définit la taille maximale d’un enregistrement que le réplicateur peut écrire dans le fichier journal. Dans la plupart des cas, la taille d’enregistrement par défaut de 1 024 Ko est optimale. Toutefois, si le service ajoute des données plus volumineuses aux informations d’état, cette valeur devra éventuellement être augmentée. Il n'est pas recommandé de choisir une valeur MaxRecordSizeInKB inférieure à 1 024 Ko, car des enregistrements plus petits utilisent uniquement l'espace nécessaire à l'enregistrement le plus petit. Cette valeur ne doit être modifiée qu'en de rares occasions.

Les paramètres SharedLogId et SharedLogPath sont toujours utilisés ensemble pour permettre à un service d'utiliser un journal partagé distinct du journal partagé par défaut pour le nœud. Pour plus d'efficacité, vous devriez spécifier autant de services que possible dans le même journal partagé. Les fichiers journaux partagés doivent être placés sur des disques uniquement utilisés pour le fichier journal partagé afin de réduire la contention des mouvements de la tête. Cette valeur ne doit être modifiée qu'en de rares occasions.

## <a name="next-steps"></a>Étapes suivantes
* [Déboguer votre application Service Fabric dans Visual Studio](service-fabric-debugging-your-application.md)
* [Référence du développeur pour les services fiables](/previous-versions/azure/dn706529(v=azure.100))
