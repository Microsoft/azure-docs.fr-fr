---
title: Résilience et récupération d'urgence Azure App Configuration
description: Découvrez comment implémenter la résilience et la récupération d'urgence avec Azure App Configuration.
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 07/09/2020
ms.openlocfilehash: 7910e2092259081aade799fc662052e5a1375e25
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96930481"
---
# <a name="resiliency-and-disaster-recovery"></a>Résilience et reprise d’activité après sinistre

Actuellement, Azure App Configuration est un service régional. Chaque magasin de configuration est créé dans une région Azure spécifique. Une panne au niveau régional affecte tous les magasins de cette région. App Configuration n’offre pas de basculement automatique vers une autre région. Cet article fournit des instructions générales sur l’utilisation de plusieurs magasins de configurations dans différentes régions Azure afin d’augmenter la résilience géographique de votre application.

## <a name="high-availability-architecture"></a>Architecture de haute disponibilité

Pour obtenir une redondance entre régions, vous devez créer plusieurs magasins App Configuration dans différentes régions. Avec cette configuration, votre application a au moins un magasin de configuration supplémentaire vers lequel basculer si le magasin principal devient inaccessible. Le diagramme suivant illustre la topologie entre votre application et ses magasins de configuration principal et secondaire :

![Stockages géoredondants](./media/geo-redundant-app-configuration-stores.png)

Votre application charge en parallèle sa configuration à partir des magasins principal et secondaire. Cette opération augmente les chances d’obtenir les données de configuration. Vous êtes responsable de la synchronisation des données dans les deux magasins. Les sections suivantes expliquent comment configurer la résilience géographique pour votre application.

## <a name="failover-between-configuration-stores"></a>Basculement entre les magasins de configuration

Techniquement, votre application n’effectue pas de basculement. Elle tente d’extraire simultanément le même jeu de données de configuration à partir de deux magasins App Configuration. Organisez votre code afin qu’il charge d’abord les données à partir du magasin secondaire, puis du magasin principal. Cette approche garantit que les données de configuration du magasin principal sont prioritaires quand elles sont disponibles. L’extrait de code suivant montre comment implémenter cette organisation dans .NET Core :

#### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((hostingContext, config) =>
        {
            var settings = config.Build();
            config.AddAzureAppConfiguration(settings["ConnectionString_SecondaryStore"], optional: true)
                  .AddAzureAppConfiguration(settings["ConnectionString_PrimaryStore"], optional: true);
        })
        .UseStartup<Startup>();
    
```

#### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
            webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();
                config.AddAzureAppConfiguration(settings["ConnectionString_SecondaryStore"], optional: true)
                    .AddAzureAppConfiguration(settings["ConnectionString_PrimaryStore"], optional: true);
            })
            .UseStartup<Startup>());
```
---

Notez que le paramètre `optional` est passé dans la fonction `AddAzureAppConfiguration`. Quand il est défini sur `true`, ce paramètre empêche l’arrêt de l’application si la fonction ne peut pas charger les données de configuration.

## <a name="synchronization-between-configuration-stores"></a>Synchronisation entre les magasins de configuration

Il est important que vos magasins de configuration géoredondants disposent tous du même jeu de données. Il existe deux moyens de parvenir à cet objectif :

### <a name="backup-manually-using-the-export-function"></a>Sauvegarde manuelle à l’aide de la fonction d’exportation

Vous pouvez utiliser la fonction **Export** d’App Configuration pour copier des données à la demande du magasin principal vers le magasin secondaire. Cette fonction est disponible via le portail Azure et l’interface CLI.

Dans le portail Azure, vous pouvez envoyer (push) une modification vers un autre magasin de configuration en suivant ces étapes.

1. Accédez à l’onglet **Importer/Exporter** et sélectionnez **Exporter** > **Configuration de l’application** > **Cible** > **Sélectionner une ressource**.

1. Dans le nouveau panneau qui s’ouvre, spécifiez l’abonnement, le groupe de ressources et le nom de la ressource de votre magasin secondaire, puis cliquez sur **Appliquer**.

1. L’interface utilisateur est mise à jour pour vous permettre de choisir les données de configuration à exporter vers votre magasin secondaire. Vous pouvez conserver la valeur de temps par défaut et définir les deux options **Étiquette d’expédition** et **Étiquette** sur la même valeur. Sélectionnez **Appliquer**. Répétez cette opération pour toutes les étiquettes de votre magasin principal.

1. Répétez les étapes précédentes chaque fois que votre configuration change.

Le processus d’exportation peut également être effectué à l’aide de l’interface de ligne de commande Azure. La commande suivante montre comment exporter toutes les configurations du magasin principal vers le magasin secondaire :

```azurecli
    az appconfig kv export --destination appconfig --name {PrimaryStore} --dest-name {SecondaryStore} --label * --preserve-labels -y
```

### <a name="backup-automatically-using-azure-functions"></a>Sauvegarde automatique avec Azure Functions

Le processus de sauvegarde peut être automatisé avec Azure Functions. Ce processus tire parti de l’intégration à Azure Event Grid dans App Configuration. Une fois configuré, App Configuration publie des événements sur Event Grid pour toute modification apportée aux valeurs de clés dans un magasin de configuration. Ainsi, une application Azure Functions peut écouter ces événements et sauvegarder les données en conséquence. Pour plus d’informations, consultez le tutoriel [Guide pratique pour sauvegarder automatiquement des magasins App Configuration](./howto-backup-config-store.md).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à optimiser votre application pour obtenir une résilience géographique pendant l’exécution pour App Configuration. Vous pouvez aussi incorporer des données de configuration à partir d’App Configuration au moment de la génération ou du déploiement. Pour plus d’informations, consultez [Intégration à un pipeline CI/CD](./integrate-ci-cd-pipeline.md).
