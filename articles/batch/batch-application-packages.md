---
title: Déployer des packages d’application sur des nœuds de calcul
description: Utilisez la fonctionnalité de packages d’applications d’Azure Batch pour gérer facilement plusieurs applications et versions pour l’installation sur des nœuds de calcul Batch.
ms.topic: how-to
ms.date: 09/24/2020
ms.custom:
- H1Hack27Feb2017
- devx-track-csharp
- contperf-fy21q1
ms.openlocfilehash: b4c6ab99b0fa93e5fdea4256d6f232cc0fe4218e
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033729"
---
# <a name="deploy-applications-to-compute-nodes-with-batch-application-packages"></a>Déployer des applications sur les nœuds avec des packages d’applications Batch

Des packages d’application peuvent simplifier le code de votre solution Azure Batch et faciliter la gestion des applications que vos tâches exécutent. Les packages d’application vous permettent de charger et gérer plusieurs versions d’applications que vos tâches exécutent, notamment leurs fichiers de prise en charge. Vous pouvez ensuite déployer automatiquement une ou plusieurs de ces applications sur les nœuds de calcul de votre pool.

Les API servant à la création et la gestion des packages d’application font partie de la bibliothèque [Batch Management .NET](/dotnet/api/overview/azure/batch/management). Les API servant à l’installation des packages d’application sur un nœud de calcul font partie de la bibliothèque [Batch .NET](/dotnet/api/overview/azure/batch/client). Des fonctionnalités comparables sont dans les API de Batch disponibles pour d’autres langues.

Cet article explique comment charger et gérer des packages d’application dans le portail Azure. Il montre également comment les installer sur des nœuds de calcul d’un pool au moyen de la bibliothèque [Batch .NET](/dotnet/api/overview/azure/batch/client).

## <a name="application-package-requirements"></a>Configuration requise des packages d’application

Pour utiliser des packages d’application, vous devez [lier un compte de stockage Azure](#link-a-storage-account) à votre compte Batch.

Il existe des restrictions au nombre d’applications et de packages d’application à l’intérieur d’un compte Batch, ainsi qu’à la taille maximale des packages d’application. Pour plus d’informations, consultez [Quotas et limites pour le service Azure Batch](batch-quota-limit.md).

> [!NOTE]
> Les pools Batch créés avant le 5 juillet 2017 ne prennent pas en charge les packages d’applications (sauf s’ils ont été créés après le 10 mars 2016 à l’aide de Microsoft Azure Cloud Services). La fonctionnalité de packages d’application décrite ici remplace la fonctionnalité d’applications Batch disponible dans les versions précédentes du service.

## <a name="understand-applications-and-application-packages"></a>Comprendre les applications et les packages d’application

Dans Azure Batch, une *application* fait référence à un jeu de versions de fichiers binaires automatiquement téléchargeable sur les nœuds de calcul de votre pool. Une application contient un ou plusieurs *packages d’application* qui représentent différentes versions de l’application.

Chaque *package d’application* est un fichier. zip contenant les fichiers binaires d’application et tous les fichiers de prise en charge. Seul le format. zip est pris en charge.

:::image type="content" source="media/batch-application-packages/app_pkg_01.png" alt-text="Diagramme montrant une vue d’ensemble des applications et packages d’applications.":::

Vous pouvez spécifier des packages d’application aux niveaux d’un pool ou d’une tâche.

- **Les packages d’application du pool** sont déployés sur tous les nœuds dans le pool. Les applications sont déployées lorsqu’un nœud rejoint un pool, et lorsqu’il est redémarré ou réinitialisé.
  
    Des packages d’application de pool sont appropriés lorsque tous les nœuds dans un pool exécutent les tâches d’un travail. Vous pouvez spécifier un ou plusieurs packages d’application à déployer lorsque vous créez un pool. Vous pouvez également ajouter ou mettre à jour les packages d’un pool existant. Pour installer un nouveau package dans un pool existant, vous devez redémarrer ses nœuds.

- **Les packages d’application de tâche** sont déployés uniquement sur un nœud de calcul programmé pour exécuter une tâche, juste avant d’exécuter la ligne de commande de la tâche. Si le package d’application spécifié et la version sont déjà sur le nœud, il n’est pas redéployé et le package existant est utilisé.
  
    Les packages d’application de tâche sont utiles dans des environnements de pool partagé où différents travaux s’exécutent sur un pool qui n’est pas supprimé après qu’un travail est terminé. Si votre travail présente moins de tâches que le pool ne contient de nœuds, les packages d’application au niveau des tâches peuvent minimiser le transfert de données, votre application n’étant déployée que sur les nœuds exécutant des tâches.
  
    Les autres scénarios pouvant tirer parti des packages d’application de tâche sont les travaux qui exécutent une application lourde, mais uniquement pour un petit nombre de tâches. Par exemple, des applications de tâches peuvent être utiles pour une étape de pré-traitement ou une tâche de fusion lourdes.

Avec des packages d’application, la tâche de démarrage de votre pool ne doit pas nécessairement spécifier une longue liste de fichiers de ressources à installer sur les nœuds. Vous n’êtes pas obligé de gérer manuellement plusieurs versions de vos fichiers dans le stockage Azure ni sur vos nœuds. En outre, inutile de vous soucier de la génération [d’URL SAP](../storage/common/storage-sas-overview.md) pour fournir l’accès aux fichiers dans votre compte de stockage. Batch fonctionne en arrière-plan avec le stockage Azure pour stocker des packages d’application et les déployer sur les nœuds de calcul.

> [!NOTE]
> La taille totale d’une tâche de démarrage doit être inférieure ou égale à 32 768 caractères, y compris les fichiers de ressources et les variables d’environnement. Si votre tâche de démarrage dépasse cette limite, l’utilisation de packages d’application est une autre option. Vous pouvez également créer un fichier .zip contenant vos fichiers de ressources, le charger en tant que blob sur le service Stockage Azure, puis le décompresser à partir de la ligne de commande de votre tâche de démarrage.

## <a name="upload-and-manage-applications"></a>Téléchargement et gestion des applications

Vous pouvez utiliser le [Portail Azure](https://portal.azure.com) ou les API de Batch Management pour gérer les packages d’application dans votre compte Batch. Les sections suivantes expliquent comment lier un compte de stockage, et comment ajouter et gérer des applications et des packages d’application dans le portail Azure.

### <a name="link-a-storage-account"></a>Lier un compte de stockage

Pour utiliser des packages d’application, vous devez lier un [compte de stockage Azure](accounts.md#azure-storage-accounts) à votre compte Batch. Le service Batch utilisera le compte de stockage associé pour stocker vos packages d’application. Nous vous recommandons de créer un compte de stockage spécifiquement destiné à être utilisé avec votre compte Batch.

Si vous n’avez pas encore configuré de compte de stockage, le portail Azure affiche un avertissement la première fois que vous sélectionnez **Applications** dans votre compte Batch. Pour lier un compte de stockage à votre compte Batch, dans la fenêtre **Avertissement**, sélectionnez **Compte de stockage**, puis sélectionnez à nouveau **Compte de stockage**.

Une fois que vous avez lié les deux comptes, Batch peut déployer automatiquement les packages stockés dans le compte de stockage lié sur vos nœuds de calcul.

> [!IMPORTANT]
> Vous ne pouvez pas utiliser de packages d’application avec des comptes de stockage Azure configurés avec des [règles de pare-feu](../storage/common/storage-network-security.md) ou un **espace de noms hiérarchique** défini sur **Activé**.

Le service Batch utilise un stockage Azure pour stocker vos packages d’application en tant qu’objets blob de blocs. Vous êtes [facturé de façon normale](https://azure.microsoft.com/pricing/details/storage/) pour les données des objets blob de blocs, et la taille de chaque package ne peut pas dépasser la taille maximale d’objet blob de blocs. Pour plus d’informations, consultez la page [Objectifs de performance et évolutivité de Stockage Azure](../storage/blobs/scalability-targets.md). Afin de minimiser les coûts, veillez à prendre en compte la taille et le nombre de vos packages d’application, ainsi qu’à supprimer régulièrement les packages obsolètes.

### <a name="view-current-applications"></a>Affichage des applications en cours

Pour afficher les applications dans votre compte batch, sélectionnez **Applications** dans le menu de navigation à gauche.

:::image type="content" source="media/batch-application-packages/app_pkg_02.png" alt-text="Capture d’écran de l’élément de menu Applications dans le portail Azure.":::

La sélection de cette option de menu a pour effet d’ouvrir la fenêtre **Applications**. Cette fenêtre affiche l’ID de chaque application dans votre compte et les propriétés suivantes :

- **Packages** : nombre de versions associées à cette application.
- **Version par défaut** : Le cas échéant, version de l’application qui sera installée si aucune version n’est spécifiée lors du déploiement de l’application.
- **Autoriser les mises à jour** : Spécifie si les mises à jour et suppressions de package sont autorisées.

Si vous voyez la [structure de fichiers](files-and-directories.md) du package d’application sur un nœud de calcul, accédez à votre compte Batch dans le portail Azure. Sélectionnez **Pools**. Sélectionnez ensuite le pool contenant le nœud de calcul. Sélectionnez le nœud de calcul sur lequel le package d’application est installé, puis ouvrez le dossier **Applications**.

### <a name="view-application-details"></a>Affichage des détails de l’application

Pour afficher les détails d’une application, sélectionnez celle-ci dans la fenêtre **Applications**. Vous pouvez configurer les paramètres suivants pour votre application.

- **Autoriser les mises à jour** : indique si les packages d’application peuvent être [mis à jour ou supprimés](#update-or-delete-an-application-package). La valeur par défaut de ce paramètre est **Oui**. Si la valeur est **Non**, les packages d’application existants ne peuvent pas être mis à jour ou supprimés, mais de nouvelles versions de package d’application peuvent toujours être ajoutées.
- **Version par défaut** : package d’application par défaut à utiliser lors du déploiement de l’application, si aucune version n’est spécifiée.
- **Nom d'affichage** : nom convivial que votre solution Batch peut utiliser quand elle affiche des informations sur l’application. Par exemple, ce nom peut être utilisé dans l’interface utilisateur d’un service que vous fournissez à vos clients par le biais de Batch.

### <a name="add-a-new-application"></a>Ajout d’une application

Pour créer une application, ajoutez un package d’application et spécifiez un ID d’application unique.

Dans votre compte Batch, sélectionnez **Applications**, puis **Ajouter**.

:::image type="content" source="media/batch-application-packages/app_pkg_05.png" alt-text="Capture d’écran du processus de création Nouvelle application dans le portail Azure.":::

Entrez les informations suivantes :

- **ID d’application** : ID de votre nouvelle application.
- **Version** : version du package d’application que vous chargez.
- **Package d'application** : fichier .zip contenant les fichiers binaires de l’application et les fichiers de prise en charge requis pour l’exécution de l’application.

L’**ID d’application** et la **Version** de vous entrez doivent respecter les exigences suivantes :

- Sur des nœuds Windows, l’ID peut contenir n’importe quelle combinaison de caractères alphanumériques, de tirets et de traits de soulignement. Sur des nœuds Linux, seuls les caractères alphanumériques et les traits de soulignement sont autorisés.
- Ne peut pas contenir plus de 64 caractères.
- Doit être unique dans le compte Batch.
- Les ID conservent la casse et ne respectent pas la casse.

Quand vous êtes prêt, sélectionnez **Envoyer**. Une fois le fichier. zip chargé sur votre compte de stockage Azure, le portail affiche une notification. Selon la taille du fichier que vous chargez et la vitesse de votre connexion réseau, cette opération peut prendre un certain temps.

### <a name="add-a-new-application-package"></a>Ajout d’un package d’application

Pour ajouter une version de package d’application pour une application existante, sélectionnez l’application dans la section **Applications** de votre compte Batch, puis sélectionnez **Ajouter**.

Comme vous l’avez fait pour la nouvelle application, spécifiez la **Version** de votre nouveau package, chargez votre fichier. zip dans le champ **Package d’application**, puis sélectionnez **Envoyer**.

### <a name="update-or-delete-an-application-package"></a>Mettre à jour ou supprimer un package d’application

Pour mettre à jour ou supprimer un package d’application existant, sélectionnez l’application dans la section **Applications** de votre compte Batch. Sélectionnez les points de suspension dans la ligne du package d’application que vous souhaitez modifier, puis sélectionnez l’action à effectuer.

:::image type="content" source="media/batch-application-packages/app_pkg_07.png" alt-text="Capture d’écran montrant les options de mise à jour et de suppression pour les packages d’application dans le portail Azure.":::

Si vous sélectionnez **Mettre à jour**, vous pourrez charger un nouveau fichier. zip. Cette opération remplace le fichier. zip précédent que vous avez chargé pour cette version.

Si vous sélectionnez **Supprimer**, vous êtes invité à confirmer la suppression de cette version. Une fois que vous avez sélectionné **OK**, Batch supprime le fichier .zip de votre compte de stockage Azure. Si vous supprimez la version par défaut d’une application, le paramètre **Version par défaut** est supprimé pour cette application.

## <a name="install-applications-on-compute-nodes"></a>Installation d’applications sur des nœuds de calcul

Maintenant que vous avez vu comment gérer les packages d’application au sein du portail Azure, nous pouvons discuter de leur déploiement sur les nœuds de calcul et leur exécution avec des tâches Batch.

### <a name="install-pool-application-packages"></a>Installation des packages d’application de pool

Pour installer un package d’application sur les nœuds de calcul d’un pool, spécifiez au moins une référence de package d’application pour le pool. Les packages d’application que vous spécifiez pour un pool sont installés sur chaque nœud de calcul qui rejoint le pool, ainsi que sur tout nœud redémarré ou réinitialisé.

Dans Batch .NET, spécifiez une ou plusieurs propriétés [CloudPool.ApplicationPackageReferences](/dotnet/api/microsoft.azure.batch.cloudpool.applicationpackagereferences) lorsque vous créez un pool ou pour un pool existant. La classe [ApplicationPackageReference](/dotnet/api/microsoft.azure.batch.applicationpackagereference) spécifie un ID et la version d’une application à installer sur les nœuds de calcul d’un pool.

```csharp
// Create the unbound CloudPool
CloudPool myCloudPool =
    batchClient.PoolOperations.CreatePool(
        poolId: "myPool",
        targetDedicatedComputeNodes: 1,
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

// Specify the application and version to install on the compute nodes
myCloudPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "litware",
        Version = "1.1001.2b" }
};

// Commit the pool so that it's created in the Batch service. As the nodes join
// the pool, the specified application package is installed on each.
await myCloudPool.CommitAsync();
```

> [!IMPORTANT]
> Si un déploiement de package d’application échoue, le service Batch marque le nœud comme [inutilisable](/dotnet/api/microsoft.azure.batch.computenode.state), et aucune tâche n’est planifiée sur ce nœud. Dans ce cas, redémarrez le nœud pour relancer le déploiement du package. Le redémarrage du nœud réactive également la planification des tâches sur celui-ci.

### <a name="install-task-application-packages"></a>Installation des packages d’application de tâche

À l’instar de l’installation sur un pool, vous spécifiez les références de package d’application pour une tâche. Lorsqu’une tâche est planifiée pour s’exécuter sur un nœud, le package est téléchargé et extrait juste avant que la ligne de commande de la tâche soit exécutée. Si une version et un package spécifiés sont déjà installés sur le nœud, le package n’est pas téléchargé et le package existant est utilisé.

Pour installer un package d’application de tâche, configurez la propriété [CloudTask.ApplicationPackageReferences](/dotnet/api/microsoft.azure.batch.cloudtask.applicationpackagereferences) de la tâche :

```csharp
CloudTask task =
    new CloudTask(
        "litwaretask001",
        "cmd /c %AZ_BATCH_APP_PACKAGE_LITWARE%\\litware.exe -args -here");

task.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference
    {
        ApplicationId = "litware",
        Version = "1.1001.2b"
    }
};
```

## <a name="execute-the-installed-applications"></a>Exécution des applications installées

Les packages que vous avez spécifiés pour un pool ou une tâche sont téléchargés et extraits dans un répertoire nommé au sein du `AZ_BATCH_ROOT_DIR` du nœud. Batch crée également une variable d’environnement qui contient le chemin d’accès au répertoire nommé. Vos lignes de commande de tâche utilisent cette variable d’environnement lors du référencement de l’application sur le nœud.

Sur des nœuds Windows, le format de la variable est le suivant :

```
Windows:
AZ_BATCH_APP_PACKAGE_APPLICATIONID#version
```

Sur des nœuds Linux, le format est légèrement différent. Les points (.), les tirets (-) et les signes dièse (#) sont aplatis en traits de soulignement dans la variable d’environnement. En outre, notez que la casse de l’ID d’application est conservée. Par exemple :

```
Linux:
AZ_BATCH_APP_PACKAGE_applicationid_version
```

`APPLICATIONID` et `version` sont des valeurs qui correspondent à la version de l’application et du package que vous avez spécifiées pour le déploiement. Par exemple, si vous spécifiez que la version 2.7 de l’application *blender* doit être installée sur des nœuds Windows, vos lignes de commande de tâche utilisent cette variable d’environnement pour accéder aux fichiers de l’application :

```
Windows:
AZ_BATCH_APP_PACKAGE_BLENDER#2.7
```

Sur des nœuds Linux, spécifiez la variable d’environnement au format suivant. Aplatissez les points (.), les tirets (-) et les signes dièse (#) en traits de soulignement et conservez la casse de l’ID d’application :

```
Linux:
AZ_BATCH_APP_PACKAGE_blender_2_7
```

Lorsque vous téléchargez un package d’application, vous pouvez spécifier une version par défaut à déployer sur vos nœuds de calcul. Si vous avez spécifié une version par défaut pour une application, vous pouvez omettre le suffixe de version lorsque vous faites référence à l’application. Vous pouvez spécifier la version d’application par défaut sur le Portail Azure, dans la fenêtre **Applications** tel qu’illustré dans [Télécharger et gérer des applications](#upload-and-manage-applications).

Par exemple, si vous définissez « 2.7 » comme version par défaut pour l’application *blender*, et si vos tâches référencent la variable d’environnement suivante, vos nœuds Windows exécutent la version 2.7 :

`AZ_BATCH_APP_PACKAGE_BLENDER`

L’extrait de code suivant montre un exemple de ligne de commande de tâche qui lance la version par défaut de l’application *blender* :

```csharp
string taskId = "blendertask01";
string commandLine =
    @"cmd /c %AZ_BATCH_APP_PACKAGE_BLENDER%\blender.exe -args -here";
CloudTask blenderTask = new CloudTask(taskId, commandLine);
```

> [!TIP]
> Pour plus d’informations sur les paramètres d’environnement de nœud de calcul, consultez [Paramètres d’environnement des tâches](jobs-and-tasks.md#environment-settings-for-tasks).

## <a name="update-a-pools-application-packages"></a>Mise à jour des packages d’applications d’un pool

Si un pool existant a déjà été configuré avec un package d’application, vous pouvez spécifier un nouveau package pour le pool. La procédure est la suivante :

- Le service Batch installe le package nouvellement spécifié sur tous les nouveaux nœuds rejoignant le pool, ainsi que sur tout nœud actuel redémarré ou réinitialisé.
- Les nœuds de calcul qui sont déjà dans le pool lorsque vous mettez à jour les références du package n’installent pas automatiquement le nouveau package d’application. Ces nœuds de calcul doivent être redémarrés ou réinitialisés pour recevoir le nouveau package.
- Lorsqu’un nouveau package est déployé, les variables d’environnement créées reflètent les références du nouveau package d’application.

Dans cet exemple, le pool existant comporte la version 2.7 de l’application *blender* configurée comme l’une de ses propriétés [CloudPool.ApplicationPackageReferences](/dotnet/api/microsoft.azure.batch.cloudpool.applicationpackagereferences). Pour mettre à jour les nœuds du pool avec la version 2.76b, spécifiez une nouvelle classe [ApplicationPackageReference](/dotnet/api/microsoft.azure.batch.applicationpackagereference) avec la nouvelle version, puis validez la modification.

```csharp
string newVersion = "2.76b";
CloudPool boundPool = await batchClient.PoolOperations.GetPoolAsync("myPool");
boundPool.ApplicationPackageReferences = new List<ApplicationPackageReference>
{
    new ApplicationPackageReference {
        ApplicationId = "blender",
        Version = newVersion }
};
await boundPool.CommitAsync();
```

Maintenant que la nouvelle version a été configurée, le service Batch installe la version 2.76b sur tout nouveau nœud rejoignant le pool. Pour installer la version 2.76b sur des nœuds qui figurent déjà dans le pool, redémarrez ou réinitialisez ces derniers. Notez que les nœuds redémarrés conservent les fichiers des déploiements précédents du package.

## <a name="list-the-applications-in-a-batch-account"></a>Liste des applications dans un compte Batch

Vous pouvez lister les applications et leurs packages dans un compte Batch à l’aide de la méthode [ApplicationOperations.ListApplicationSummaries](/dotnet/api/microsoft.azure.batch.applicationoperations.listapplicationsummaries).

```csharp
// List the applications and their application packages in the Batch account.
List<ApplicationSummary> applications = await batchClient.ApplicationOperations.ListApplicationSummaries().ToListAsync();
foreach (ApplicationSummary app in applications)
{
    Console.WriteLine("ID: {0} | Display Name: {1}", app.Id, app.DisplayName);

    foreach (string version in app.Versions)
    {
        Console.WriteLine("  {0}", version);
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

- [L’API REST Batch](/rest/api/batchservice) prend également en charge l’utilisation de packages d’application. Par exemple, consultez l’élément [applicationPackageReferences](/rest/api/batchservice/pool/add#applicationpackagereference) pour savoir comment spécifier les packages à installer, et [applications](/rest/api/batchservice/application) pour savoir comment obtenir les informations sur l’application.
- Découvrez comment [gérer les quotas et les comptes Azure Batch avec Batch Management .NET](batch-management-dotnet.md)par programme. La bibliothèque [Batch Management .NET](/dotnet/api/overview/azure/batch/management) peut activer les fonctionnalités de création et de suppression de comptes pour votre application ou service Batch.
