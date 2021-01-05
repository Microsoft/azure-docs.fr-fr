---
title: Sauvegarder et restaurer Service Fabric
description: Documentation conceptuelle relative à la sauvegarde et à la restauration de Service Fabric, un service permettant de configurer la sauvegarde des services Reliable Stateful et Reliable Actors.
author: mcoskun
ms.topic: conceptual
ms.date: 10/29/2018
ms.author: mcoskun
ms.custom: devx-track-csharp
ms.openlocfilehash: a60ebff06562c12415b2a106a9a11127feb94dab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89021984"
---
# <a name="backup-and-restore-reliable-services-and-reliable-actors"></a>Sauvegarder et restaurer Reliable Services et Reliable Actors
Azure Service Fabric est une plateforme haute disponibilité qui réplique l’état sur plusieurs nœuds afin de conserver cette haute disponibilité.  Ainsi, même si un nœud du cluster échoue, les services continuent à être disponibles. Bien que cette redondance intégrée fournie par la plateforme suffise pour certains, dans d’autres cas, il est souhaitable que le service sauvegarde les données (dans un magasin externe).

> [!NOTE]
> Il est essentiel de sauvegarder et restaurer vos données (et de tester qu’elles fonctionnent comme prévu) afin de pouvoir les récupérer en cas de perte.
> 

> [!NOTE]
> Microsoft vous recommande d’utiliser [Sauvegarde et restauration périodiques](service-fabric-backuprestoreservice-quickstart-azurecluster.md) pour la configuration de la sauvegarde des données des services fiables avec état et de Reliable Actors. 
> 


Par exemple, un service peut souhaiter sauvegarder des données afin de les protéger contre les scénarios suivants :

- Perte définitive de la totalité d’un cluster Service Fabric.
- Perte définitive de la majorité des réplicas d’une partition de service.
- Erreurs d’administration dans le cadre desquelles l’état est accidentellement supprimé ou endommagé. Par exemple, cela peut se produire si un administrateur disposant de privilèges suffisants supprime le service par erreur.
- Bogues dans le service qui provoquent l’altération des données. Par exemple, cela peut se produire lorsqu’une mise à niveau de code de service écrit des données erronées dans une collection fiable. Dans ce cas, le code et les données devront peut-être être restaurés à un état antérieur.
- Traitement des données hors connexion. Il peut être utile de disposer du traitement des données hors connexion pour le décisionnel qui a lieu séparément du service qui génère les données.

La fonctionnalité Sauvegarder/Restaurer permet aux services reposant sur l’API Reliable Services de créer et de restaurer des sauvegardes. Les API de sauvegarde fournies par la plateforme permettent d’effectuer une ou plusieurs sauvegardes de l’état d’une partition de service, sans bloquer les opérations de lecture et d’écriture. Les API de restauration permettent la restauration de l’état d’une partition d’être à partir d’une sauvegarde choisie.

## <a name="types-of-backup"></a>Types de sauvegarde
Il existe deux options de sauvegarde : complète et incrémentielle.
Une sauvegarde complète est une sauvegarde qui contient toutes les données nécessaires pour recréer l’état du réplica, à savoir des points de contrôle et l’ensemble des enregistrements de journal.
Dans la mesure où une sauvegarde complète regroupe les points de contrôle et les journaux, elle ne peut être restaurée d’elle-même.

Les sauvegardes complètes posent problème dès lors qu’elles impliquent des points de contrôle volumineux.
Par exemple, un réplica associé à un état de 16 Go verra ses points de contrôle s’accumuler jusqu’à environ 16 Go.
Si l’objectif de point de récupération est fixé à cinq minutes, le réplica devra être sauvegardé toutes les cinq minutes.
Chaque sauvegarde suppose de copier 16 Go de points de contrôle en plus des 50 Mo (configurables avec `CheckpointThresholdInMB`) de journaux d’activité.

![Exemple de sauvegarde complète.](media/service-fabric-reliable-services-backup-restore/FullBackupExample.PNG)

La solution à ce problème consiste à recourir à des sauvegardes incrémentielles, qui ne contiennent que les enregistrements modifiés depuis la dernière sauvegarde.

![Exemple de sauvegarde incrémentielle.](media/service-fabric-reliable-services-backup-restore/IncrementalBackupExample.PNG)

Puisque les sauvegardes incrémentielles ne portent que sur les modifications apportées depuis la dernière sauvegarde (sans les points de contrôle), elles sont généralement plus rapides, mais ne peuvent pas être restaurées d’elles-mêmes.
La restauration d’une sauvegarde incrémentielle implique l’ensemble de la chaîne de sauvegarde.
Une chaîne de sauvegarde est une chaîne de sauvegardes commençant par une sauvegarde complète suivie d’un certain nombre de sauvegardes incrémentielles contiguës.

## <a name="backup-reliable-services"></a>Sauvegarder Reliable Services
Le créateur du service contrôle intégralement le moment auquel les sauvegardes sont effectuées et leur emplacement de stockage.

Pour démarrer une sauvegarde, le service doit appeler la fonction de membre hérité `BackupAsync`.  
Les sauvegardes peuvent uniquement être effectuées à partir de réplicas principaux et ont besoin de bénéficier du statut d’écriture.

Comme indiqué ci-dessous, `BackupAsync` prend un objet `BackupDescription`, dans lequel il est possible de spécifier une sauvegarde complète ou incrémentielle, ainsi qu’une fonction de rappel, `Func<< BackupInfo, CancellationToken, Task<bool>>>`, qui est appelée quand le dossier de sauvegarde a été créé en local et est prêt à être déplacé dans un stockage externe.

```csharp

BackupDescription myBackupDescription = new BackupDescription(BackupOption.Incremental,this.BackupCallbackAsync);

await this.BackupAsync(myBackupDescription);

```

Une demande de sauvegarde incrémentielle peut échouer en déclenchant `FabricMissingFullBackupException`. Cette exception indique que l’on se trouve dans une des situations suivantes :

- le réplica n’a jamais fait l’objet d’une sauvegarde complète depuis qu’il est devenu principal,
- qu’une partie des enregistrements du journal écrits depuis la dernière sauvegarde a été tronquée ou
- le réplica a dépassé la limite `MaxAccumulatedBackupLogSizeInMB`.

Configurer `MinLogSizeInMB` ou `TruncationThresholdFactor` augmente les chances de parvenir à effectuer des sauvegardes incrémentielles.
L’augmentation de ces valeurs augmente l’utilisation du disque par réplica.
Pour plus d’informations, consultez [Configuration de Reliable Services](service-fabric-reliable-services-configuration.md).

`BackupInfo` fournit des informations sur la sauvegarde, notamment l’emplacement du dossier où le runtime l’a enregistrée (`BackupInfo.Directory`). La fonction de rappel peut déplacer `BackupInfo.Directory` vers un magasin externe ou un autre emplacement.  Cette fonction renvoie également une valeur booléenne qui indique si elle a été en mesure de déplacer correctement le dossier de sauvegarde vers son emplacement cible.

Le code suivant montre comment la méthode `BackupCallbackAsync` peut permettre de charger la sauvegarde vers le Stockage Azure :

```csharp
private async Task<bool> BackupCallbackAsync(BackupInfo backupInfo, CancellationToken cancellationToken)
{
    var backupId = Guid.NewGuid();

    await externalBackupStore.UploadBackupFolderAsync(backupInfo.Directory, backupId, cancellationToken);

    return true;
}
```

Dans l’exemple précédent, `ExternalBackupStore` est l’exemple de classe qui sert d’interface avec le stockage Blob Azure, et `UploadBackupFolderAsync` est la méthode qui compresse le dossier et le place dans le magasin d’objets blob Azure.

Notez les points suivants :

  - Il ne peut y avoir qu’une seule opération de sauvegarde en cours par réplica à un moment donné. Si plusieurs appels `BackupAsync` sont effectués à la fois, l’exception `FabricBackupInProgressException` est levée pour limiter les sauvegardes en cours à une seule.
  - Si un réplica bascule pendant une sauvegarde, celle-ci n’a peut-être pas été effectuée. Ainsi, une fois le basculement terminé, il incombe au service de redémarrer la sauvegarde en appelant `BackupAsync` si nécessaire.

## <a name="restore-reliable-services"></a>Restaurer Reliable Services
En règle générale, vous devrez peut-être effectuer une restauration dans les cas suivants :

  - La partition du service a perdu des données. Par exemple, le disque de deux réplicas sur trois pour une partition (y compris le réplica principal) est endommagé ou effacé. Le nouveau réplica principal peut être amené à restaurer des données à partir d’une sauvegarde.
  - L’ensemble du service est perdu. Par exemple, un administrateur supprime l’ensemble du service, et le service et les données doivent donc être restaurés.
  - Le service a répliqué des données d’application endommagées (par exemple, en raison d’un bogue dans l’application). Dans ce cas, le service doit être mis à niveau ou rétabli pour supprimer la cause de l’endommagement, et les données non endommagées doivent être restaurées.

Bien que de nombreuses approches soient possibles, nous proposons quelques exemples de récupération à l’aide de `RestoreAsync`dans les scénarios ci-dessus.

## <a name="partition-data-loss-in-reliable-services"></a>Perte de données de partition dans Reliable Services
Dans ce cas, le runtime détecte automatiquement la perte de données et appelle l’API `OnDataLossAsync`.

Le créateur du service doit effectuer les opérations suivantes pour assurer la récupération :

  - Remplacer la méthode de classe de base virtuelle `OnDataLossAsync`.
  - Rechercher la dernière sauvegarde dans l’emplacement externe qui contient les sauvegardes du service.
  - Télécharger la dernière sauvegarde (et la décompresser dans le dossier de sauvegarde le cas échéant).
  - La méthode `OnDataLossAsync` fournit un `RestoreContext`. Appeler l’API `RestoreAsync` sur le `RestoreContext` fourni.
  - Renvoyer l’état True en cas de succès de la restauration.

Voici un exemple d’implémentation de la méthode `OnDataLossAsync` :

```csharp
protected override async Task<bool> OnDataLossAsync(RestoreContext restoreCtx, CancellationToken cancellationToken)
{
    var backupFolder = await this.externalBackupStore.DownloadLastBackupAsync(cancellationToken);

    var restoreDescription = new RestoreDescription(backupFolder);

    await restoreCtx.RestoreAsync(restoreDescription);

    return true;
}
```

La variable `RestoreDescription`, transmise à l’appel `RestoreContext.RestoreAsync`, contient un membre nommé `BackupFolderPath`.
Lorsque vous restaurez une seule sauvegarde complète, ce `BackupFolderPath` doit être défini sur le chemin d’accès local du dossier qui contient votre sauvegarde complète.
Lorsque vous restaurez une sauvegarde complète ainsi qu’un certain nombre de sauvegardes incrémentielles, `BackupFolderPath` doit être défini sur le chemin d’accès local du dossier qui contient à la fois la sauvegarde complète et l’ensemble des sauvegardes incrémentielles.
Un appel `RestoreAsync` peut lever `FabricMissingFullBackupException` si le `BackupFolderPath` fourni ne contient pas une sauvegarde complète.
Il peut également lever `ArgumentException` si `BackupFolderPath` a une chaîne interrompue de sauvegardes incrémentielles.
Par exemple, s’il contient la sauvegarde complète, les première et troisième sauvegardes incrémentielles, mais pas la deuxième sauvegarde incrémentielle.

> [!NOTE]
> Par défaut, RestorePolicy est défini sur Sécurisé.  Cela signifie que l’API `RestoreAsync` va échouer avec ArgumentException si elle détecte que le dossier de sauvegarde contient un état antérieur ou égal à l’état contenu dans ce réplica.  `RestorePolicy.Force` permet d’ignorer cette vérification de sécurité, comme le spécifie `RestoreDescription`.
> 

## <a name="deleted-or-lost-service"></a>Service supprimé ou perdu
Si un service est supprimé, vous devez d’abord le recréer pour que les données puissent être restaurées.  Il est important de créer le service avec la même configuration, par exemple le schéma de partitionnement, afin que les données puissent être restaurées de manière fluide.  Une fois le service opérationnel, l’API permettant de restaurer les données (`OnDataLossAsync` ci-dessus) doit être appelée sur chacune de ses partitions. Un moyen d’y parvenir consiste à utiliser [FabricClient.TestManagementClient.StartPartitionDataLossAsync](/dotnet/api/system.fabric.fabricclient.testmanagementclient?view=azure-dotnet#System_Fabric_FabricClient_TestManagementClient_StartPartitionDataLossAsync_System_Guid_System_Fabric_PartitionSelector_System_Fabric_DataLossMode_) sur chaque partition.  

À ce stade, l’implémentation est identique au scénario ci-dessus. Chaque partition doit restaurer la dernière sauvegarde pertinente à partir du magasin externe. L’inconvénient est que l’ID de partition peut avoir changé, étant donné que le runtime crée des ID de partition de manière dynamique. Par conséquent, le service doit stocker les informations de partition appropriées et le nom du service pour identifier la dernière sauvegarde correcte à restaurer pour chaque partition.

> [!NOTE]
> Il n’est pas recommandé d’utiliser `FabricClient.ServiceManager.InvokeDataLossAsync` sur chaque partition pour restaurer l’ensemble du service, car cela pourrait endommager l’état de votre cluster.
> 

## <a name="replication-of-corrupt-application-data"></a>Réplication des données d’application endommagées
Si la mise à niveau de l’application récemment déployée comporte un bogue, des données risquent d’être endommagées. Par exemple, une mise à niveau de l’application peut commencer à mettre à jour les enregistrements de numéro de téléphone d’un Dictionnaire fiable en utilisant un indicatif non valide.  Dans ce cas, les numéros de téléphone non valides seront répliqués, car Service Fabric ne connaît pas la nature des données stockées.

La première chose que vous devez faire après avoir détecté un bogue si flagrant qui endommage les données est de figer le service au niveau de l’application et, si possible, de procéder à une mise à niveau vers une version du code d’application qui exclut le bogue.  Cependant, même une fois que le code du service est résolu, les données peuvent toujours être endommagées et par conséquent avoir besoin d’être restaurées.  Dans ce cas, restaurer la dernière sauvegarde peut ne pas suffire, étant donné qu’elle peut également être endommagée.  Il convient donc de rechercher la dernière sauvegarde effectuée avant que les données ne soient endommagées.

Si vous ne savez pas exactement quelles mises à jour sont endommagées, vous pouvez déployer un nouveau cluster Service Fabric et restaurer les sauvegardes des partitions affectées comme dans le scénario de « service supprimé ou perdu » ci-dessus.  Pour chaque partition, lancer la restauration de sauvegardes de la plus récente à la plus ancienne. Une fois que vous avez trouvé une sauvegarde ne présentant pas de corruption, déplacez/supprimez toutes les sauvegardes de partition plus récentes (que cette sauvegarde). Répétez ce processus pour chaque partition. À présent, lorsque `OnDataLossAsync` est appelé sur la partition dans le cluster de production, la dernière sauvegarde trouvée dans le magasin externe est celle choisie par le processus ci-dessus.

Vous pouvez maintenant utiliser les étapes de la section « Service supprimé ou perdu » pour restaurer l’état du service tel qu’il était avant endommagement par le code bogué.

Notez les points suivants :

  - Lorsque vous effectuez une restauration, il existe un risque que la sauvegarde restaurée soit antérieure à l’état de la partition avant la perte des données. Pour cette raison, vous ne devez effectuer une restauration qu’en dernier recours pour récupérer autant de données que possible.
  - La chaîne qui représente le chemin d’accès du dossier de sauvegarde et les chemins d’accès des fichiers dans le dossier de sauvegarde peut être supérieure à 255 caractères, selon le chemin d’accès FabricDataRoot et la longueur du nom du type d’application. Certaines méthodes .NET, par exemple `Directory.Move`, risquent de lever l’exception `PathTooLongException`. Il existe une solution de contournement, qui consiste à appeler directement les API kernel32, notamment `CopyFile`.

## <a name="back-up-and-restore-reliable-actors"></a>Sauvegarder et restaurer Reliable Actors


L’infrastructure Reliable Actors est basée sur Reliable Services. Le service ActorService, qui héberge le ou les acteur(s), est un service fiable avec état. Par conséquent, toutes les fonctionnalités de sauvegarde et de restauration disponibles dans Reliable Services sont également disponibles pour Reliable Actors (à l’exception des comportements spécifiques au fournisseur d’état). Étant donné que les sauvegardes sont effectuées par partition, les états de tous les acteurs de cette partition sont sauvegardés (et la restauration se fait de façon similaire, par partition). Afin d’effectuer la sauvegarde ou la restauration, le propriétaire du service doit créer une classe de service d’acteur personnalisée, qui dérive de la classe ActorService, puis effectuer une sauvegarde/restauration similaire à celle de Reliable Services, comme décrit dans les sections précédentes.

```csharp
class MyCustomActorService : ActorService
{
    public MyCustomActorService(StatefulServiceContext context, ActorTypeInformation actorTypeInfo)
          : base(context, actorTypeInfo)
    {
    }
    
    //
    // Method overrides and other code.
    //
}
```

Lorsque vous créez une classe de service d’acteur personnalisée, vous devez l’inscrire également lors de l’inscription de l’acteur.

```csharp
ActorRuntime.RegisterActorAsync<MyActor>(
    (context, typeInfo) => new MyCustomActorService(context, typeInfo)).GetAwaiter().GetResult();
```

Le fournisseur d’état par défaut de Reliable Actors est `KvsActorStateProvider`. La sauvegarde incrémentielle n’est pas activée par défaut pour `KvsActorStateProvider`. Vous pouvez activer la sauvegarde incrémentielle en créant `KvsActorStateProvider` avec le paramètre approprié dans son constructeur et en le transmettant au constructeur ActorService, comme l’indique l’extrait de code suivant :

```csharp
class MyCustomActorService : ActorService
{
    public MyCustomActorService(StatefulServiceContext context, ActorTypeInformation actorTypeInfo)
          : base(context, actorTypeInfo, null, null, new KvsActorStateProvider(true)) // Enable incremental backup
    {
    }
    
    //
    // Method overrides and other code.
    //
}
```

Une fois la sauvegarde incrémentielle activée, elle peut échouer avec l’erreur FabricMissingFullBackupException pour une des raisons suivantes et vous devrez effectuer une sauvegarde complète avant d’effectuer d’autres sauvegardes incrémentielles :

  - Le réplica n’a jamais fait l’objet d’une sauvegarde complète depuis qu’il est devenu le principal.
  - Certains enregistrements de journal ont été tronqués depuis la dernière sauvegarde.

Lorsque la sauvegarde incrémentielle est activée, `KvsActorStateProvider` n’utilise pas de mémoire tampon circulaire pour gérer ses enregistrements et les tronque de temps en temps. Si aucune sauvegarde n’est effectuée par l’utilisateur pendant une période de 45 minutes, le système tronque automatiquement les enregistrements du journal. Vous pouvez configurer cet intervalle en spécifiant `logTruncationIntervalInMinutes` dans le constructeur `KvsActorStateProvider` (comme pour activer la sauvegarde incrémentielle). Les enregistrements de journal peuvent également être tronqués si le réplica principal doit créer un autre réplica en envoyant toutes ses données.

Lors de la restauration à partir d’une chaîne de sauvegarde, comme pour Reliable Services, BackupFolderPath doit contenir des sous-répertoires avec un seul sous-répertoire contenant la sauvegarde complète et les autres sous-répertoires contenant les sauvegardes incrémentielles. Si la validation de la chaîne de sauvegarde échoue, l’API de restauration lance une exception FabricException avec le message d’erreur correspondant. 

> [!NOTE]
> `KvsActorStateProvider` ignore actuellement l’option RestorePolicy.Safe. La prise en charge de cette fonctionnalité est prévue dans une prochaine version.
> 

## <a name="testing-back-up-and-restore"></a>Test de la sauvegarde et de la restauration
Il est important de s’assurer que les données critiques soient sauvegardées et puissent être restaurées. Vous pouvez pour cela appeler la cmdlet `Start-ServiceFabricPartitionDataLoss` dans PowerShell, qui peut provoquer une perte de données dans une partition en particulier pour déterminer si les fonctionnalités de sauvegarde et de restauration de données de votre service fonctionnent comme prévu.  Il est également possible d’appeler par programme une perte de données et de la restaurer à partir de cet événement.

> [!NOTE]
> Vous pouvez trouver un exemple d’implémentation des fonctionnalités de sauvegarde et de restauration dans l’application de référence web sur GitHub. Pour plus de détails, voir le service `Inventory.Service`.
> 
> 

## <a name="under-the-hood-more-details-on-backup-and-restore"></a>Sous le capot : détails supplémentaires sur la sauvegarde et la restauration
Voici des détails supplémentaires sur la sauvegarde et la restauration.

### <a name="backup"></a>Backup
Le gestionnaire d’état fiable offre la possibilité de créer des sauvegardes cohérentes sans bloquer les opérations de lecture ou d’écriture. Pour ce faire, il utilise un point de contrôle et un mécanisme de persistance de journal.  Le gestionnaire d’état fiable prend des points de contrôle (légers) approximatifs à certains points pour soulager la pression dans le journal des transactions et améliorer les temps de récupération.  Lorsque `BackupAsync` est appelé, le gestionnaire d’état fiable demande à tous les objets fiables de copier leurs derniers fichiers de point de contrôle dans un dossier de sauvegarde local.  Ensuite, le gestionnaire d’état fiable copie tous les enregistrements de journal à partir du pointeur de démarrage vers le dernier enregistrement de journal dans le dossier de sauvegarde.  Comme tous les enregistrements sont inclus dans la sauvegarde et que le gestionnaire d’état fiable conserve les journaux WAL (write-ahead log), toutes les transactions validées (`CommitAsync` a réussi) sont incluses dans la sauvegarde.

Une transaction validée après l’appel de `BackupAsync` peut figurer ou non dans la sauvegarde.  Une fois que le dossier de sauvegarde local a été rempli par la plateforme (c’est-à-dire que la sauvegarde locale est effectuée par le runtime), le rappel de sauvegarde du service est appelé.  Ce rappel est chargé de déplacer le dossier de sauvegarde vers un emplacement externe comme Azure Storage.

### <a name="restore"></a>Restaurer
Le gestionnaire d’état fiable permet de restaurer une sauvegarde à l’aide de l’API `RestoreAsync`.  
La méthode `RestoreAsync` sur `RestoreContext` ne peut être appelée qu’au sein de la méthode `OnDataLossAsync`.
Le booléen retourné par `OnDataLossAsync` indique si le service a été restauré à cet état à partir d’une source externe.
Si la méthode `OnDataLossAsync` retourne la valeur true, Service Fabric recrée tous les autres réplicas à partir de ce réplica principal. Service Fabric s’assure que les réplicas qui vont recevoir l’appel `OnDataLossAsync` effectuent tout d’abord une transition vers le rôle principal, mais ne se voient pas accorder de statut de lecture ou d’écriture.
Pour les responsables de l’implémentation de StatefulService, cela implique que la méthode `RunAsync` n’est pas appelée tant que l’exécution de la méthode `OnDataLossAsync` ne s’est pas terminée correctement.
Ensuite, la méthode `OnDataLossAsync` est appelée sur le nouveau réplica principal.
L’API continuera d’être appelée jusqu’à ce que l’API se termine correctement (en renvoyant true ou false) et termine la reconfiguration concernée.

`RestoreAsync` supprime d’abord tout état existant dans le réplica principal sur lequel elle a été appelée. Le gestionnaire d’état fiable crée ensuite tous les objets fiables qui existent dans le dossier de sauvegarde. Les objets fiables sont alors invités à procéder à une restauration à partir de leurs points de contrôle dans le dossier de sauvegarde. Enfin, le gestionnaire d’état fiable récupère son propre état à partir d’enregistrements de journal dans le dossier de sauvegarde, puis effectue la récupération. Dans le cadre de la récupération, les opérations commençant à partir du point de départ qui ont validé des enregistrements de journal dans le dossier de sauvegarde sont relues dans les objets fiables. Cette étape garantit que l’état récupéré est cohérent.

## <a name="next-steps"></a>Étapes suivantes
  - [Collections fiables](service-fabric-work-with-reliable-collections.md)
  - [Démarrage rapide de Reliable Services](service-fabric-reliable-services-quick-start.md)
  - [Notifications Reliable Services](service-fabric-reliable-services-notifications.md)
  - [Configuration de Reliable Services](service-fabric-reliable-services-configuration.md)
  - [Référence du développeur pour les Collections fiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)
  - [Periodic backup and restore in Azure Service Fabric (Preview)](service-fabric-backuprestoreservice-quickstart-azurecluster.md) (Sauvegarde et restauration périodiques dans Azure Service Fabric (préversion))
