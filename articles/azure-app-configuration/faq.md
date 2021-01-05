---
title: Azure App Configuration – Questions fréquentes (FAQ)
description: Lisez les réponses à la foire aux questions (FAQ) sur Azure App Configuration, telles que la façon dont le service diffère d’Azure Key Vault.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 02/19/2020
ms.author: alkemper
ms.openlocfilehash: 4e19574e5848d1ee86d13aa02a9cf583b92eff02
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929563"
---
# <a name="azure-app-configuration-faq"></a>Azure App Configuration – Questions fréquentes (FAQ)

Cet article répond aux questions fréquentes sur Azure App Configuration.

## <a name="how-is-app-configuration-different-from-azure-key-vault"></a>En quoi App Configuration est-il différent d’Azure Key Vault ?

App Configuration permet aux développeurs de gérer les paramètres d’application et de contrôler la disponibilité des fonctionnalités. Il vise à simplifier de nombreuses tâches impliquant l’utilisation de données de configuration complexes.

App Configuration prend en charge :

- Les espace de noms hiérarchiques
- L’étiquetage
- Le requêtes étendues
- La récupération par lots
- Les opérations de gestion spécialisées
- Une interface utilisateur de gestion des fonctionnalités

App Configuration et Key Fault sont complémentaires. D’ailleurs, il est conseillé d’utiliser les deux côte à côte dans la plupart des déploiements d’applications.

## <a name="should-i-store-secrets-in-app-configuration"></a>Dois-je stocker les secrets dans App Configuration ?

Même si App Configuration offre une sécurité renforcée, Key Vault reste le meilleur emplacement de stockage pour les secrets d’application. Key Vault propose un chiffrement au niveau du matériel, des stratégies d’accès précises, ainsi que des opérations de gestion, comme la rotation des certificats.

Vous pouvez créer des valeurs App Configuration qui font référence à des secrets stockés dans Key Vault. Pour plus d’informations, consultez [Utiliser des références Key Vault dans une application ASP.NET Core](./use-key-vault-references-dotnet-core.md).

## <a name="does-app-configuration-encrypt-my-data"></a>Mes données sont-elles chiffrées par App Configuration ?

Oui. App Configuration chiffre toutes les valeurs de clés qu’il renferme, de même que les communications réseau. Les noms de clés et les étiquettes servent d’index pour récupérer les données de configuration et ne sont pas chiffrés.

## <a name="where-does-data-stored-in-app-configuration-reside"></a>Où résident les données stockées dans App Configuration ? 

Les données client stockées dans App Configuration résident dans la région où le magasin App Configuration du client a été créé. App Configuration peut répliquer des données vers des [régions jumelées](../best-practices-availability-paired-regions.md) pour la résilience des données, mais il ne réplique pas ni ne déplace les données client en dehors de leur zone géographique, comme défini par [la résidence des données dans Azure](https://azure.microsoft.com/global-infrastructure/data-residency/). Les clients et les utilisateurs finaux peuvent déplacer leurs données client, les copier ou y accéder à partir de n’importe quel emplacement dans le monde.

## <a name="how-is-app-configuration-different-from-azure-app-service-settings"></a>En quoi App Configuration est-il différent des paramètres Azure App Service ?

Azure App Service vous permet de définir des paramètres d’application pour chaque instance App Service. Ces paramètres sont transmis en tant que variables d’environnement au code de l’application. Vous pouvez associer un paramètre à un emplacement de déploiement spécifique, si vous le souhaitez. Pour plus d’informations, consultez [Configuration des paramètres de l’application](../app-service/configure-common.md#configure-app-settings).

En revanche, Azure App Configuration vous permet de définir des paramètres qui peuvent être partagés entre plusieurs applications. Cela comprend les applications qui s’exécutent dans App Service, de même que d'autres plateformes. Votre code d'application accède à ces paramètres via les fournisseurs de configuration pour .NET et Java, par le biais du kit de développement logiciel (SDK) Azure, ou directement via les API REST.

Vous pouvez également importer et exporter des paramètres entre App Service et App Configuration. Cela vous permet de configurer rapidement un nouveau magasin App Configuration en fonction des paramètres App Service existants. Vous pouvez aussi partager la configuration avec une application existante qui s’appuie sur les paramètres App Service.

## <a name="are-there-any-size-limitations-on-keys-and-values-stored-in-app-configuration"></a>Existe-t-il des limitations de taille sur les clés et les valeurs stockées dans App Configuration ?

Il existe une limite de 10 Ko pour un seul élément de clé-valeur.

## <a name="how-should-i-store-configurations-for-multiple-environments-test-staging-production-and-so-on"></a>Comment dois-je stocker les configurations pour plusieurs environnements (test, intermédiaire, production, etc) ?

Vous contrôlez qui peut accéder à App Configuration au niveau de chaque magasin. Utilisez un magasin distinct pour chaque environnement qui nécessite des autorisations différentes. Il s’agit de l’approche qui offre la meilleure isolation sur le plan de la sécurité.

Si vous n’avez pas besoin d’un isolement de sécurité entre les environnements, vous pouvez utiliser des étiquettes pour différencier les valeurs de configuration. La rubrique [Utilisation d’étiquettes pour permettre différentes configurations selon les environnements](./howto-labels-aspnet-core.md) fournit un exemple complet.

## <a name="what-are-the-recommended-ways-to-use-app-configuration"></a>Quels sont les modes d’utilisation recommandés d’App Configuration ?

Prenez connaissance des [bonnes pratiques](./howto-best-practices.md).

## <a name="how-much-does-app-configuration-cost"></a>Quel est le coût d’App Configuration ?

Il existe deux niveaux tarifaires :

- Niveau Gratuit
- Niveau standard.

Si vous avez créé un magasin avant l’introduction du niveau Standard, il passe automatiquement au niveau Gratuit lors de la mise à la disposition générale. Vous pouvez choisir de passer au niveau Standard ou de rester au niveau Gratuit.

En revanche, vous ne pouvez pas passer un magasin du niveau Standard au niveau Gratuit. Au niveau Gratuit, vous pouvez créer un magasin, puis importer des données de configuration dans ce magasin.

## <a name="which-app-configuration-tier-should-i-use"></a>Quel niveau App Configuration dois-je utiliser ?

Les deux niveaux App Configuration offrent des fonctionnalités de base, notamment des paramètres de configuration, indicateurs de fonctionnalité, références Key Vault, opérations de gestion de base, métriques et journaux.

Voici quelques considérations relatives au choix d'un niveau.

- **Ressources par abonnement** : Une ressource se compose d’un magasin de configuration unique. Au niveau Gratuit, chaque abonnement est limité à un magasin de configuration. Au niveau Standard, les abonnements ne sont pas limités en termes de magasins de configuration.
- **Stockage par ressource** : Au niveau Gratuit, chaque magasin de configuration est limité à 10 Mo de stockage. Au niveau Standard, chaque magasin de configuration peut utiliser jusqu’à 1 Go de stockage.
- **Historique des révisions** : App Configuration stocke l'historique des différentes modifications apportées aux clés. Au niveau Gratuit, cet historique est stocké pendant sept jours. Au niveau Standard, cet historique est stocké pendant 30 jours.
- **Quota de requêtes** : Au niveau Gratuit, les magasins sont limités à 1 000 requêtes par jour. Lorsqu’un magasin atteint 1 000 requêtes, il renvoie le code d’état HTTP 429 pour toutes les requêtes jusqu’à minuit UTC.

    Au niveau Standard, les magasins sont limités à 20 000 requêtes par heure. Lorsque le quota est épuisé, le code d’état HTTP 429 est renvoyé pour toutes les requêtes jusqu’à la fin de l’heure.

- **Contrat de niveau de service** : Le niveau Standard présente un contrat de niveau de service garantissant une disponibilité de 99,9 %. Le niveau Gratuit ne présente aucun contrat de niveau de service.
- **Fonctionnalités de sécurité** : Les deux niveaux incluent des fonctionnalités de sécurité de base, parmi lesquelles le chiffrement avec des clés gérées par Microsoft, l’authentification via HMAC ou Azure Active Directory, la prise en charge d’Azure RBAC, l’identité managée et les étiquettes de service. Le niveau Standard offre des fonctionnalités de sécurité plus avancées, notamment la prise en charge d'Azure Private Link et le chiffrement avec des clés managées par le client.
- **Coût** : Au niveau Standard, les magasins génèrent des frais d’utilisation quotidiens. Chaque jour, les 200 000 premières requêtes sont comprises dans les frais quotidiens. Au-delà de l'allocation quotidienne, les requêtes génèrent aussi des frais de dépassement. Au niveau Gratuit, l'utilisation d'un magasin ne génère pas de frais.

## <a name="can-i-upgrade-a-store-from-the-free-tier-to-the-standard-tier-can-i-downgrade-a-store-from-the-standard-tier-to-the-free-tier"></a>Puis-je passer un magasin du niveau Gratuit au niveau Standard ? Puis-je passer un magasin du niveau Standard au niveau Gratuit ?

Vous pouvez à tout moment passer du niveau Gratuit au niveau Standard.

En revanche, vous ne pouvez pas passer un magasin du niveau Standard au niveau Gratuit. Au niveau Gratuit, vous pouvez créer un magasin, puis [importer des données de configuration dans ce magasin](howto-import-export-data.md).

## <a name="are-there-any-limits-on-the-number-of-requests-made-to-app-configuration"></a>Est-ce qu’il y a des limites au nombre de requêtes adressées à App Configuration ?

Les magasins de configuration dans le niveau Gratuit sont limités à 1 000 requêtes par jour. Les magasins de configuration dans le niveau Standard peuvent subir une limitation temporaire lorsque le taux de requêtes dépasse 20 000 requêtes par heure.

Lorsqu’un magasin atteint sa limite, il renvoie le code d’état HTTP 429 pour toutes les requêtes effectuées jusqu’à l’expiration de la période. L’en-tête `retry-after-ms` dans la réponse indique un délai d’attente suggéré (en millisecondes) avant d’effectuer une nouvelle tentative de requête.

Si votre application rencontre régulièrement des réponses du code d’état HTTP 429, envisagez de la reconcevoir pour réduire le nombre de requêtes effectuées. Pour plus d’informations, consultez [Réduire les requêtes adressées à la App Configuration](./howto-best-practices.md#reduce-requests-made-to-app-configuration)

## <a name="my-application-receives-http-status-code-429-responses-why"></a>Mon application reçoit des réponses de code d’état HTTP 429. Pourquoi ?

Vous allez recevoir une réponse de code d’état HTTP 429 dans les circonstances suivantes :

* Dépassement du nombre limite de requêtes quotidiennes pour un magasin dans le niveau Gratuit.
* Limitation temporaire en raison d’un taux de requêtes élevé pour un magasin au niveau Standard.
* Utilisation excessive de la bande passante.
* Tentative de création ou de modification d’une clé lorsque le quota de stockage est dépassé.

Examinez le corps de la réponse 429 connaître pour la raison spécifique de l’échec de la requête.

## <a name="how-can-i-receive-announcements-on-new-releases-and-other-information-related-to-app-configuration"></a>Comment puis-je recevoir des annonces sur les nouvelles versions et d’autres informations relatives à App Configuration ?

Abonnez-vous à notre [référentiel d’annonces GitHub](https://github.com/Azure/AppConfiguration-Announcements).

## <a name="how-can-i-report-an-issue-or-give-a-suggestion"></a>Comment puis-je signaler un problème ou faire une suggestion ?

Vous pouvez nous contacter directement sur [GitHub](https://github.com/Azure/AppConfiguration/issues).

## <a name="next-steps"></a>Étapes suivantes

* [À propos d’Azure App Configuration](./overview.md)
