---
title: Migrer les espaces de noms Azure Service Bus Standard vers Premium
description: Guide pour permettre la migration d’espaces de noms Standard Service Bus Azure existants vers le niveau Premium
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 1ed09a077f086390c658e6650171c552b361008d
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "85340753"
---
# <a name="migrate-existing-azure-service-bus-standard-namespaces-to-the-premium-tier"></a>Migrer des espaces de noms Standard Service Bus existants vers le niveau Premium

Auparavant, Azure Service Bus offrait des espaces de noms uniquement pour le niveau Standard. Les espaces de noms sont des configurations multi-locataires optimisées pour les environnements à faible débit et les environnements de développement. Le niveau Premium offre des ressources dédiées par espace de noms pour une latence prévisible et un débit accru à un prix fixe. Le niveau Premium est optimisé pour les environnements à haut débit et de production qui nécessitent des fonctionnalités d’entreprise supplémentaires.

Cet article explique comment migrer des espaces de noms de niveau Standard existants vers le niveau Premium.  

>[!WARNING]
> La migration s’adresse aux espaces de noms Standard Service Bus qui doivent être mis à niveau vers le niveau Premium. L'outil de migration ne prend pas en charge la rétrogradation.

Quelques points à noter :

- Cette migration est censée se produire sur place, ce qui signifie que les applications émettrices et réceptrices existantes **n'exigent aucun changement de code ou de configuration**. La chaîne de connexion existante pointe automatiquement vers le nouvel espace de noms Premium.
- L'espace de noms **Premium** ne doit contenir **aucune entité** pour que la migration réussisse.
- Toutes les **entités** dans l'espace de noms Standard sont **copiées** vers l'espace de noms Premium pendant le processus de migration.
- La migration prend en charge **1 000 entités par unité de messagerie** sur le niveau Premium. Pour déterminer le nombre d'unités de messagerie dont vous avez besoin, commencez par le nombre d'entités dont vous disposez sur votre espace de noms Standard actuel.
- Vous ne pouvez pas migrer directement du **niveau De base** vers le **niveau Premium**, mais vous pouvez le faire indirectement en migrant d’abord du niveau De base à Standard, puis de Standard à Premium à l’étape suivante.

## <a name="migration-steps"></a>Étapes de la migration

Certaines conditions sont associées au processus de migration. Familiarisez-vous avec les étapes suivantes pour réduire les risques d'erreurs. Ces étapes décrivent le processus de migration, et les détails étape par étape sont énumérés dans les sections suivantes.

1. Créez un espace de noms Premium.
1. Associez les espaces de noms Standard et Premium.
1. Synchronisez (par copier-coller) les entités du niveau Standard avec l'espace de noms Premium.
1. Validez la migration.
1. Videz les entités de l’espace de noms Standard en utilisant l’espace de noms post-migration.
1. Supprimez l’espace de noms Standard.

>[!IMPORTANT]
> Une fois la migration validée, accédez à l'ancien espace de noms Standard et videz les files d'attente et les abonnements. Une fois les messages vidés, ils peuvent être envoyés vers le nouvel espace de noms Premium pour y être traités par les applications réceptrices. Une fois les files d'attente et les abonnements vidés, nous vous recommandons de supprimer l'ancien espace de noms Standard.

### <a name="migrate-by-using-the-azure-cli-or-powershell"></a>Migrer en utilisant l’interface de ligne de commande Azure (CLI) ou PowerShell

Pour migrer votre espace de noms Standard Service Bus vers Premium à l'aide de l'outil Azure CLI ou PowerShell, procédez comme suit.

1. Créez un espace de noms Premium Service Bus. Vous pouvez référencer les [modèles Azure Resource Manager](service-bus-resource-manager-namespace.md) ou [utilisez le portail Azure](service-bus-create-namespace-portal.md). Veillez à sélectionner **Premium** pour le paramètre **serviceBusSku**.

1. Définissez les variables d'environnement suivantes pour simplifier les commandes de migration.

   ```
   resourceGroup = <resource group for the standard namespace>
   standardNamespace = <standard namespace to migrate>
   premiumNamespaceArmId = <Azure Resource Manager ID of the premium namespace to migrate to>
   postMigrationDnsName = <post migration DNS name entry to access the standard namespace>
   ```

    >[!IMPORTANT]
    > L'alias/nom post-migration (post_migration_dns_name) sera utilisé pour accéder à l'ancien espace de noms Standard post-migration. Utilisez cette variable pour vider les files d'attente et les abonnements, puis supprimez l'espace de noms.

1. Associez les espaces de noms Standard et Premium, puis lancez la synchronisation à l'aide de la commande suivante :

    ```azurecli-interactive
    az servicebus migration start --resource-group $resourceGroup --name $standardNamespace --target-namespace $premiumNamespaceArmId --post-migration-name $postMigrationDnsName
    ```

1. Vérifiez l'état de la migration à l'aide de la commande suivante :

    ```azurecli-interactive
    az servicebus migration show --resource-group $resourceGroup --name $standardNamespace
    ```

    La migration est considérée comme terminée lorsque les valeurs suivantes s'affichent :

    * MigrationState = "Active"
    * pendingReplicationsOperationsCount = 0
    * provisioningState = "Succeeded"

    Cette commande affiche également la configuration de la migration. Vérifiez que les valeurs sont correctement définies. Vérifiez également l'espace de noms Premium dans le portail pour vous assurer que toutes les files d'attente et rubriques ont été créées et correspondent à celles qui existaient dans l'espace de noms Standard.

1. Validez la migration en exécutant la commande complète suivante :

   ```azurecli-interactive
   az servicebus migration complete --resource-group $resourceGroup --name $standardNamespace
   ```

### <a name="migrate-by-using-the-azure-portal"></a>Migration en utilisant le portail Azure

La migration à l’aide du portail Azure suit le même flux logique que la migration avec les commandes. Suivez ces étapes pour migrer à l’aide du portail Azure.

1. Dans le menu **Navigation** du volet de gauche, sélectionnez **Migrer vers Premium**. Cliquez sur le bouton **Commencer** pour passer à la page suivante.
    ![Page d’accueil de migration][]

1. Exécutez l’**installation**.
   ![Configurer l'espace de noms][]
   1. Créez et attribuez l'espace de noms Premium vers lequel migrer l'espace de noms Standard existant.
        ![Créer un espace de noms - créer un espace de noms Premium][]
   1. Choisissez un **nom de post- migration**. Vous utiliserez ce nom pour accéder à l'espace de noms Standard une fois la migration terminée.
        ![Configurer un espace de noms - choisir le nom de la post-migration][]
   1. Sélectionnez **Suivant** pour continuer.
1. Synchronisez les entités entre les espaces de noms Standard et Premium.
    ![Configurer un espace de noms - synchroniser les entités - démarrer][]

   1. Sélectionnez **Démarrer la synchronisation** pour lancer la synchronisation des entités.
   1. Sélectionnez **Oui** dans la boîte de dialogue pour confirmer et démarrer la synchronisation.
   1. Attendez la fin de la synchronisation. Le statut est disponible dans la barre d'état.
        ![Configurer l'espace de noms - synchroniser les entités - avancement][]
        >[!IMPORTANT]
        > Si vous avez besoin d'interrompre la migration pour une raison quelconque, veuillez consulter la section FAQ de ce document.
   1. Une fois la synchronisation terminée, sélectionnez **Suivant** en bas de la page.

1. Examinez les changements sur la page Résumé. Sélectionnez **Terminer la migration** pour changer d'espace de noms et terminer la migration.
    ![Changer d'espace de noms - changer de menu][]  
    La page de confirmation s'affiche lorsque la migration est terminée.
    ![Changer d'espace de noms - réussite][]

## <a name="caveats"></a>Mises en garde

Certaines des fonctionnalités fournies par le niveau Azure Service Bus Standard ne sont pas prises en charge par le niveau Azure Service Bus Premium. Cela est intentionnel, car le niveau Premium offre des ressources dédiées pour un débit et une latence prévisibles.

Voici la liste des fonctionnalités non prises en charge par le niveau Premium et la manière dont elles sont traitées –

### <a name="express-entities"></a>Entités Express

   Les entités Express qui n’enregistrent aucune donnée de message dans le stockage ne sont pas prises en charge au niveau Premium. Les ressources dédiées ont permis d’améliorer considérablement le débit tout en garantissant une persistance des données, telle qu’on peut l’attendre de tout système de messagerie d’entreprise.

   Durant la migration, toutes les entités Express dans votre espace de noms Standard sont créées sur l’espace de noms Premium en tant qu’entités non-Express.

   Si vous utilisez des modèles Azure Resource Manager (ARM), veillez à supprimer l’indicateur « enableExpress » de la configuration du déploiement afin que vos flux de travail automatisés s’exécutent sans erreur.

### <a name="partitioned-entities"></a>Entités partitionnées

   Les entités partitionnées étaient prises en charge au niveau Standard pour offrir une meilleure disponibilité dans une configuration multilocataire. Avec l’approvisionnement de ressources dédiées disponibles par espace de noms au niveau Premium, cela n’est plus nécessaire.

   Pendant la migration, toute entité partitionnée dans l’espace de noms Standard est créée sur l’espace de noms Premium en tant qu’entité non partitionnée.

   Si votre modèle ARM définit « enablePartitioning » sur « true » pour une file d’attente ou une rubrique spécifiques, le répartiteur l’ignore.

## <a name="faqs"></a>FAQ

### <a name="what-happens-when-the-migration-is-committed"></a>Que se passe-t-il lorsque la migration est validée ?

Une fois la migration validée, la chaîne de connexion qui pointait vers l'espace de noms Standard pointera vers l'espace de noms Premium.

Les applications émettrices et réceptrices se déconnecteront de l'espace de noms Standard et se reconnecteront automatiquement à l'espace de noms Premium.

### <a name="what-do-i-do-after-the-standard-to-premium-migration-is-complete"></a>Que dois-je faire une fois la migration du niveau Standard vers Premium terminée ?

La migration du niveau Standard vers Premium garantit que les métadonnées de l'entité, notamment les rubriques, les abonnements et les filtres, sont copiées de l'espace de noms Standard vers l'espace de noms Premium. Les données du message qui ont été validées dans l'espace de noms Standard ne sont pas copiées de l'espace de noms Standard vers l'espace de noms Premium.

L'espace de noms Standard peut contenir des messages qui ont été envoyés et validés pendant la migration. Videz manuellement ces messages de l’espace de noms Standard et envoyez-les manuellement vers l'espace de noms Premium. Pour vider manuellement les messages, utilisez une application console ou un script qui vide les entités d'espace de noms Standard en utilisant le nom DNS post-migration que vous avez spécifié dans les commandes de migration. Envoyez ces messages à l'espace de noms Premium afin qu'ils puissent être traités par les destinataires.

Une fois les messages vidés, supprimez l'espace de noms Standard.

>[!IMPORTANT]
> Une fois les messages de l’espace de noms Standard vidés, supprimez l'espace de noms Standard. Cette étape est importante car la chaîne de connexion qui se référait initialement à l'espace de noms Standard se réfère maintenant à l'espace de noms Premium. Vous n'aurez plus besoin de l'espace de noms Standard. La suppression de l'espace de noms Standard que vous avez migré vous facilitera la tâche par la suite.

### <a name="how-much-downtime-do-i-expect"></a>Combien durera le temps d'arrêt ?

Le processus de migration vise à réduire le temps d'arrêt prévu au niveau des applications. Le temps d'arrêt est réduit grâce à la chaîne de connexion que les applications émettrices et réceptrices utilisent pour pointer vers le nouvel espace de noms Premium.

Le temps d’arrêt de l'application est limité au délai nécessaire à la mise à jour de l'entrée DNS afin qu’elle pointe vers l'espace de noms Premium. Le temps d'arrêt est d'environ 5 minutes.

### <a name="do-i-have-to-make-any-configuration-changes-while-doing-the-migration"></a>Dois-je modifier ma configuration lors de la migration ?

Non, aucun changement de code ou de configuration n'est nécessaire pour effectuer la migration. La chaîne de connexion utilisée par les applications émettrices et réceptrices pour accéder à l'espace de noms Standard est automatiquement mappée pour servir d'alias à l'espace de noms Premium.

### <a name="what-happens-when-i-abort-the-migration"></a>Que se passe-t-il lorsque j'annule la migration ?

La migration peut être interrompue en utilisant la commande `Abort` ou à l’aide du portail Azure.

#### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az servicebus migration abort --resource-group $resourceGroup --name $standardNamespace
```

#### <a name="azure-portal"></a>Portail Azure

![Abandonner le flux - abandonner la synchronisation][]
![Abandonner le flux - abandon terminé][]

Lorsque le processus de migration est abandonné, il interrompt le processus de copie des entités (rubriques, abonnements et filtres) de l'espace de noms Standard vers l'espace de noms Premium et interrompt aussi le jumelage.

La chaîne de connexion n'est pas mise à jour pour pointer vers l'espace de noms Premium. Vos applications existantes continuent de fonctionner comme avant le début de la migration.

Cependant, ni les entités de l'espace de noms Premium ni l'espace de nom Premium ne sont supprimés. Supprimez les entités manuellement si vous choisissez de ne pas effectuer la migration.

>[!IMPORTANT]
> Si vous décidez d'abandonner la migration, supprimez l'espace de noms Premium que vous aviez préparé pour la migration afin que les ressources ne vous soient pas facturées.

#### <a name="i-dont-want-to-have-to-drain-the-messages-what-do-i-do"></a>Je ne souhaite pas vider les messages. Que faire ?

Certains messages sont envoyés par les applications émettrices et validés pour être stockés dans l'espace de noms Standard pendant la migration et juste avant que cette migration ne soit validée.

Pendant la migration, les données/charges utiles réelles du message ne sont pas copiées de l’espace de noms Standard vers l'espace de noms Premium. Les messages doivent être manuellement vidés puis envoyés vers l'espace de noms Premium.

Cependant, si vous pouvez migrer au cours d'une fenêtre de maintenance planifiée et que vous ne souhaitez pas vider et envoyer les messages manuellement, procédez comme suit :

1. Arrêtez les applications émettrices. Les applications réceptrices traiteront les messages qui se trouvent actuellement dans l'espace de noms Standard et videront la file d'attente.
1. Une fois les files d'attente et les abonnements dans l'espace de noms Standard vides, suivez la procédure décrite précédemment pour migrer l'espace de noms Standard vers l'espace de noms Premium.
1. Une fois la migration terminée, vous pouvez redémarrer les applications émettrices.
1. Les expéditeurs et les destinataires se connecteront désormais automatiquement à l'espace de noms Premium.

    >[!NOTE]
    > Vous n'avez pas besoin d'arrêter les applications réceptrices pour la migration.
    >
    > Une fois la migration terminée, les applications réceptrices se déconnecteront de l'espace de noms Standard et se connecteront automatiquement à l'espace de noms Premium.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [différences entre les messageries Standard et Premium](./service-bus-premium-messaging.md).
* En savoir plus sur les [Aspects liés à la haute disponibilité et à la géo-reprise d'activité après sinistre pour Service Bus Premium](service-bus-outages-disasters.md#protecting-against-outages-and-disasters---service-bus-premium).

[Page d’accueil de migration]: ./media/service-bus-standard-premium-migration/1.png
[Configurer l'espace de noms]: ./media/service-bus-standard-premium-migration/2.png
[Créer un espace de noms - créer un espace de noms Premium]: ./media/service-bus-standard-premium-migration/3.png
[Configurer un espace de noms - choisir le nom de la post-migration]: ./media/service-bus-standard-premium-migration/4.png
[Configurer un espace de noms - synchroniser les entités - démarrer]: ./media/service-bus-standard-premium-migration/5.png
[Configurer l'espace de noms - synchroniser les entités - avancement]: ./media/service-bus-standard-premium-migration/8.png
[Changer d'espace de noms - changer de menu]: ./media/service-bus-standard-premium-migration/9.png
[Changer d'espace de noms - réussite]: ./media/service-bus-standard-premium-migration/12.png

[Abandonner le flux - abandonner la synchronisation]: ./media/service-bus-standard-premium-migration/abort1.png
[Abandonner le flux - abandon terminé]: ./media/service-bus-standard-premium-migration/abort3.png
