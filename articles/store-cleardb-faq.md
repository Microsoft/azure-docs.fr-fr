---
title: FAQ sur les bases de données MySQL ClearDB avec Azure App Service
description: Réponses aux questions courantes sur l’utilisation de bases de données MySQL ClearDB avec Azure App Service.
documentationcenter: php
services: mysql
author: sunbuild
manager: yochayk
tags: mysql
ms.service: multiple
ms.topic: article
ms.date: 10/27/2016
ms.author: sumuth
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: references_regions
ms.openlocfilehash: 25cc4dfaabee58a47cc33fe4be8dc7375bc804d2
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96491640"
---
# <a name="faq-for-cleardb-mysql-databases-with-azure-app-service"></a>FAQ sur les bases de données MySQL ClearDB avec Azure App Service
Ce FAQ répond aux questions courantes sur l’utilisation et l’achat de bases de données MySQL ClearDB pour Azure Web Apps.

> [!IMPORTANT]
> À partir du 13 juin 2018, ClearDB a transféré les clients basés sur Azure actuellement facturés par Microsoft vers un modèle de facturation directe avec ClearDB. Les informations contenues dans cet article sont désormais obsolètes. Vous ne pourrez plus ni créer ni mettre à niveau une base de données ClearDB qui a été créée dans Azure.
>
> Pour plus d’informations, également concernant les étapes suivantes, consultez [Modifications apportées aux plans de service ClearDB](https://w2.cleardb.net/important-change-of-billing-notice-for-all-azure-cleardb-service-plans/).

## <a name="what-options-do-i-have-for-mysql-on-azure"></a>Quelles sont les options MySQL sur Azure ?
Pour obtenir les dernières informations sur ce service, consultez [ClearDB](https://w2.cleardb.net/). ClearDB est un service d’hébergement MySQL qui gère l’infrastructure MySQL pour vous. 

Vous avez plusieurs autres possibilités pour héberger MySQL dans Azure :
* [Azure Database pour MySQL](https://azure.microsoft.com/services/mysql/)
* [Cluster MySQL s’exécutant sur une machine virtuelle Azure](https://github.com/azure/azure-quickstart-templates/tree/master/mysql-replication)
* [Instance unique de MySQL s’exécutant sur une machine virtuelle Azure](/previous-versions/azure/virtual-machines/windows/classic/mysql-2008r2?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)


## <a name="do-i-need-a-credit-card-for-the-web-app--mysql-template-in-the-azure-marketplace"></a>Ai-je besoin d’une carte de crédit pour le modèle Application web + MySQL dans Azure Marketplace ?
Cela dépend du type d’abonnement que vous utilisez. Voici quelques types d’abonnement couramment utilisés :

* [Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/) : nécessite une carte de crédit. Quand vous achetez une base de données MySQL payante, votre carte de crédit est débitée.
* [Essai gratuit](https://azure.microsoft.com/pricing/free-trial/) : inclut des crédits à utiliser avec les services Microsoft Azure mais ne permet pas d’acheter des ressources tierces. Pour acheter des services tiers ou une base de données MySQL payante, vous devez utiliser un abonnement avec une carte de crédit activée. Pour Web Apps, vous pouvez créer une base de données MySQL ClearDB GRATUITE.
* [Abonnement MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits/) et **Dev MSDN/Tester le paiement à l’utilisation** : semblable à l’essai gratuit, un abonnement MSDN nécessite une carte de crédit pour acheter une solution MySQL payante à partir de ClearDB.
* [Contrat Entreprise](https://azure.microsoft.com/pricing/enterprise-agreement/) : les clients Contrat Entreprise sont facturés conformément à leur contrat, chaque trimestre, à hauteur de tous leurs achats sur la Place de marché Azure (tiers), sur une facture distincte et cumulée. Vous êtes facturé hors engagement monétaire pour tous les achats Marketplace. Notez que, à ce jour, Azure Store n’est pas disponible pour les clients inscrits en Azerbaïdjan, Croatie, Norvège et à Porto Rico. 

## <a name="why-was-i-charged-350-for-a-web-app--mysql-from-the-azure-marketplace"></a>Pourquoi ai-je été facturé 3,50 USD pour une application Web + MySQL dans Azure Marketplace ?
L’option de base de données par défaut est Titan, qui coûte 3,50 USD. Nous n’indiquons pas le coût lors de la création de la base de données et vous pouvez acheter par erreur une base de données que vous ne souhaitiez pas. Nous essayons de trouver un moyen d’améliorer l’expérience, mais en attendant, vous devez vérifier tous vos niveaux tarifaires sélectionnés pour l’application web et la base de données avant de cliquer sur **Créer** et commencer le déploiement des ressources.

## <a name="i-am-running-mysql-on-my-own-azure-virtual-machine-can-i-connect-my-azure-web-app-to-my-database"></a>J’exécute MySQL sur ma propre machine virtuelle Azure. Puis-je connecter mon application web Azure à ma base de données ?
Oui. Vous pouvez connecter votre application web à votre base de données tant que votre machine virtuelle Azure a octroyé l’accès à distance à votre application web. Pour plus d’informations, consultez la page [Installer MySQL sur une machine virtuelle](/previous-versions/azure/virtual-machines/windows/classic/mysql-2008r2?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).

## <a name="in-which-countriesregions-are-cleardb-premium-mysql-clusters-supported"></a>Dans quels pays/régions les clusters Premium MySQL ClearDB sont-ils pris en charge ?
Les clusters Premium MySQL ClearDB sont disponibles dans toutes les régions Azure du monde, à l’exception des régions suivantes : Inde, Australie, Brésil Sud et Chine.

## <a name="can-i-create-a-new-cluster-prior-to-creating-a-database-with-cleardb-premium-cluster-solution"></a>Puis-je créer un nouveau cluster avant de créer une base de données avec la solution de cluster premium ClearDB ?
Non, la création de clusters ClearDB vides n’est pas prise en charge. Le portail Azure vous permet de créer des bases de données dans un cluster, ce qui peut éventuellement créer un cluster en même temps.

## <a name="will-i-be-warned-if-i-try-to-delete-a-cleardb-mysql-database-that-is-in-use-by-one-of-my-applications"></a>Suis-je averti si j’essaie de supprimer une base de données MySQL ClearDB en cours d’utilisation par une de mes applications ?
Non, Azure ne vous avertit pas si vous supprimez un achat Marketplace dont dépend votre application.

## <a name="which-regions-can-i-create-cleardb-databases-in"></a>Dans quelles régions puis-je créer des bases de données ClearDB ?
Azure Marketplace n’est pas disponible pour les clients inscrits en Azerbaïdjan, Croatie, Norvège ou à Porto Rico. ClearDB n’est pas disponible dans ces régions.

## <a name="what-pricing-tier-should-i-choose-for-a-production-web-app-and-database"></a>Quel niveau tarifaire dois-je choisir pour une application web de production et une base de données ?
Utilisez le niveau tarifaire De base ou un niveau supérieur pour Web Apps. Pour ClearDB, nous vous recommandons d’utiliser un plan Saturne ou Jupiter. Examinez les fonctionnalités et les limitations de chaque niveau tarifaire pour [Web Apps](https://azure.microsoft.com/pricing/details/app-service/) et les [bases de données MySQL ClearDB](https://w2.cleardb.net/important-change-of-billing-notice-for-all-azure-cleardb-service-plans/) pour choisir celui qui correspond à vos besoins.

## <a name="how-do-i-upgrade-my-cleardb-database-from-one-plan-to-another"></a>Comment dois-je faire pour mettre à niveau ma base de données ClearDB d’un plan vers un autre ?
Dans le [portail Azure](https://portal.azure.com), vous pouvez faire monter en puissance une base de données d’hébergement partagé ClearDB. (pour plus d’informations, consultez cet [article](https://blogs.msdn.microsoft.com/appserviceteam/2016/10/06/upgrade-your-cleardb-mysql-database-in-azure-portal/)). Actuellement, nous ne prenons pas en charge la mise à niveau des clusters ClearDB Premium dans le portail Azure.

## <a name="i-cant-see-my-cleardb-database-in-azure-portal"></a>Ma base de données ClearDB ne s’affiche pas dans le portail Azure.
Si vous avez créé une base de données ClearDB dans Azure Classic, vous ne pourrez pas voir votre base de données dans le [portail Azure](https://portal.azure.com). Il n’existe aucune solution de contournement pour ce scénario.

## <a name="who-do-i-contact-for-support-when-my-database-is-down"></a>Qui dois-je contacter pour obtenir de l’aide quand ma base de données est en panne ?
Contactez le [support ClearDB](https://www.cleardb.com/developers/help/support) pour tout problème lié à la base de données. Préparez-vous à fournir des informations sur votre abonnement Azure.

## <a name="can-i-create-additional-users-for-my-cleardb-mysql-database-cluster-solution"></a>Puis-je créer des utilisateurs supplémentaires pour ma solution de cluster de base de données ClearDB MySQL ?
Non. Vous ne pouvez pas créer des utilisateurs supplémentaires, mais vous pouvez créer des bases de données sur votre cluster de base de données ClearDB.  

## <a name="can-basicpro-series-databases-be-upgraded-in-place-similar-to-planetary-plans-today-on-cleardb-portal"></a>Les bases de données Basic/Pro peuvent-elles être mises à niveau sur place comme les plans Planetary actuellement sur le portail de ClearDB ?
Oui, les bases de données Basic peuvent être mises à niveau sur place (Basic 60 à Basic 500). Les bases de données Pro peuvent être mises à niveau sur place (Pro 125 à Pro 1000), à l’exception de Pro 60. Actuellement, nous ne prenons pas en charge la mise à niveau de la base de données Pro 60. 

## <a name="when-i-migrate-my-resources-from-one-subscription-to-another-does-my-cleardb-mysql-database-get-migrated-as-well"></a>Lorsque je migre mes ressources d’un abonnement à un autre, ma base de données ClearDB MySQL migre-t-elle également ?
Lorsque vous effectuez la migration de ressources entre les différents abonnements, certaines [limitations](azure-resource-manager/management/move-support-resources.md) s’appliquent. Une base de données ClearDB MySQL est un service tiers et ne peut donc pas être migrée lors d’une migration d’abonnement Azure. Si vous ne gérez pas la migration de votre base de données MySQL avant la migration des ressources Azure, vos bases de données ClearDB MySQL peuvent être désactivées. Commencez par migrer manuellement vos bases de données, puis effectuez la migration des abonnements Azure pour votre application web. 

## <a name="i-hit-the-spending-limit-on-my-subscription-i-removed-the-limit-and-my-app-service-is-online-however-the-database-is-not-accessible-how-do-i-re-enable-the-cleardb-database"></a>J’ai atteint la limite de dépense de mon abonnement. J’ai supprimé la limite et mon App Service est en ligne, mais la base de données n’est pas accessible. Comment ré-activer la base de données ClearDB ?
Contactez [l’assistance de ClearDB](https://www.cleardb.com/developers/help/support) pour réactiver la base de données. Préparez-vous à fournir les informations sur votre abonnement Azure et le nom de la base de données.

## <a name="can-i-transfer-a-cleardb-database-from-a-credit-card-subscription-to-an-ea-subscription"></a>Puis-je transférer une base de données ClearDB d’un abonnement avec carte de crédit vers un abonnement Contrat Entreprise ?
Les bases de données ClearDB existantes utilisent la carte de crédit associée aux abonnements existants. Pour utiliser un abonnement Contrat Entreprise, vous devez faire migrer vos données vers une nouvelle base de données :

* Achetez une nouvelle base de données ClearDB avec votre abonnement Contrat Entreprise.
* Faites migrer vos données vers votre nouvelle base de données.
* Mettez à jour votre application pour utiliser la nouvelle base de données.
* Supprimez votre ancienne base de données ClearDB.

Quand vous créez une application web avec MySQL (ClearDB) ou une base de données MySQL (ClearDB), l’abonnement que vous choisissez détermine votre mode de paiement pour le service. Avec un abonnement Contrat Entreprise, nous ne bloquons pas l’achat de services tiers tels que ClearDB dans le portail Azure. Les abonnements Contrat Entreprise sont facturés hors engagement monétaire, tous les trimestres et à terme échu. Les clients Contrat Entreprise doivent configurer un mode de paiement, comme une carte de crédit, pour payer tous les services Marketplace tiers.

## <a name="where-can-i-see-the-charges-for-cleardb-resources-in-an-ea-subscription"></a>Où apparaissent les frais liés aux ressources ClearDB dans un abonnement Contrat Entreprise ?
Pour les clients titulaires d’un Contrat Entreprise Direct, les frais Azure Marketplace sont visibles sur Enterprise Portal. Notez que tous les achats et consommations Marketplace sont facturés hors engagement monétaire, tous les trimestres et à terme échu. Les clients Contrat Entreprise doivent payer directement les fournisseurs de services tiers. Pour cela, ils peuvent autoriser un mode de paiement, tel qu’une carte de crédit, avec leur compte Contrat Entreprise.

Les clients titulaires d’un Contrat Entreprise Indirect peuvent consulter leurs abonnements Azure Marketplace à la page **Gérer les abonnements** d’Enterprise Portal, mais la tarification est masquée. Les clients doivent contacter leur fournisseur de services de concession de licences (LSP) pour obtenir des informations sur les frais Marketplace.

L’accès à Azure Marketplace pour les services tiers peut être géré par les administrateurs de votre inscription Azure au Contrat Entreprise. Ils peuvent désactiver ou réactiver l’accès aux achats tiers depuis le Store dans Gérer les comptes et les abonnements sous la section Comptes dans Enterprise Portal.

## <a name="who-do-i-contact-for-questions-about-my-bill-for-cleardb-services-in-my-ea-subscription"></a>Qui dois-je contacter pour toute question sur ma facture des services ClearDB dans mon abonnement Contrat Entreprise ?
Contactez le [support technique](https://aka.ms/AzureEntSupport) pour toute question sur la facturation liée à votre inscription Contrat Entreprise. L’équipe du support du portail Contrat Entreprise peut répondre à votre question ou trouver une solution à votre problème.

## <a name="more-information"></a>Informations complémentaires
[FAQ de Azure Marketplace](https://azure.microsoft.com/marketplace/faq/)