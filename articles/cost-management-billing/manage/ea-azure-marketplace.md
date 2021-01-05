---
title: Place de marché Azure
description: Décrit comment les clients EA peuvent utiliser la Place de marché Azure
author: bandersmsft
ms.reviewer: baolcsva
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: conceptual
ms.date: 09/03/2020
ms.author: banders
ms.openlocfilehash: 0f2d3c830f27eec9f521e6f79ac8dce3bce818e9
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/03/2020
ms.locfileid: "89442562"
---
# <a name="azure-marketplace"></a>Place de marché Azure

Cet article explique comment les clients et partenaires EA peuvent afficher les frais de la Place de marché et activer les achats sur la Place de marché Azure.

## <a name="azure-marketplace-for-ea-customers"></a>Place de marché Azure pour les clients EA

Pour les clients titulaires d’un Contrat Entreprise Direct, les frais de la Place de marché Azure sont visibles sur le portail Azure Enterprise. Les achats et consommations de la Place de marché Azure sont facturés hors Paiement anticipé Azure, tous les trimestres ou tous les mois, et à terme échu.

Les clients indirects peuvent consulter leurs abonnements Place de marché Azure sur la page **Gérer les abonnements** du portail Azure Enterprise, mais la tarification est masquée. Les clients doivent contacter leur fournisseur de services de concession de licences (LSP) pour obtenir des informations sur les frais de la Place de marché Azure.

Les nouveaux achats récurrents mensuels ou annuels de la Place de marché Azure sont facturés en totalité dans la période au cours de laquelle les articles de la Place de marché Azure ont été achetés. Ces articles sont renouvelés automatiquement dans la période suivante, le même jour que l’achat d’origine.

Les frais récurrents mensuels continuent d’être renouvelés le premier jour de chaque mois calendaire. Les frais annuels sont renouvelés à l’anniversaire de la date d’achat.

Certains services de revendeur tiers sur la Place de marché Azure utilisent maintenant votre solde de Paiement anticipé Azure Contrat Entreprise (EA). Ces services étaient précédemment facturés séparément, c’est-à-dire en dehors de Paiement anticipé Azure EA. Paiement anticipé Azure EA pour ces services de la Place de marché Azure permet de simplifier la gestion des achats et des paiements clients. Pour obtenir la liste complète des services qui consomment désormais Paiement anticipé Azure, consultez la [mise à jour du 6 mars 2018 sur le site web d’Azure](https://azure.microsoft.com/updates/azure-marketplace-third-party-reseller-services-now-use-azure-monetary-commitment/).

### <a name="partners"></a>Partenaires

Les LSP peuvent télécharger une liste de prix spécifique à la Place de marché Azure en accédant à la grille tarifaire dans le portail Azure Enterprise. Sélectionnez le lien vers la **liste des prix de la Place de marché** dans le coin supérieur droit. La liste des prix de la Place de marché Azure répertorie tous les services disponibles et leurs prix.

Pour télécharger la liste de prix :

1. Dans le portail Azure Enterprise, sélectionnez **Rapports** > **Grille tarifaire**.
1. Dans le coin supérieur droit, recherchez le lien vers la liste des prix de la Place de marché Azure sous votre nom d’utilisateur.
1. Cliquez sur le lien avec le bouton droit et sélectionnez **Enregistrer la cible sous**.
1. Dans la fenêtre d’**enregistrement**, remplacez le titre du document par `AzureMarketplacePricelist.zip`, ce qui remplacera le fichier .xlsx par un fichier .zip.
1. Une fois le téléchargement terminé, vous disposez d’un fichier zip avec des listes de prix spécifiques des pays.
1. Les LSP doivent se référer au fichier de pays pour la tarification spécifique du pays. Les LSP peuvent utiliser l'onglet **Notifications** pour connaître les références (SKU) nouvelles ou retirées.
1. Les changements de prix sont peu fréquents. Les LSP sont informés par e-mail des augmentations de prix et des modifications des taux de change 30 jours à l'avance.
1. Les LSP reçoivent une facture par inscription, par ISV et par trimestre.

### <a name="enabling-azure-marketplace-purchases"></a>Activation des achats de la Place de marché Azure

Les administrateurs d’entreprise peuvent désactiver ou activer des achats de la Place de marché Azure pour tous les abonnements Azure dans le cadre de cette inscription. Si l’administrateur d’entreprise désactive les achats et que des abonnements Azure incluent déjà des abonnements de Place de marché Azure, ceux-ci ne sont pas annulés ni affectés.

Bien que les clients puissent convertir leurs abonnements Azure directs en Azure EA en les associant à leur inscription dans le portail Azure Enterprise, cette action ne convertit pas automatiquement les abonnements enfants.

Pour activer les achats de la Place de marché Azure :

1. Connectez-vous au portail Azure Enterprise en tant qu’administrateur d’entreprise.
1. Accédez à **Gérer**.
1. Sous **Détails de l’inscription**, sélectionnez l’icône du crayon en regard de la ligne **Place de marché Azure**.
1. Basculez sur **Activé/désactivé** ou sur **BYOL SKUs Only** (Références SKU BYOL uniquement) en fonction des besoins.
1. Sélectionnez **Enregistrer**.

> [!NOTE]
> BYOL (apportez votre propre licence) et l’option gratuite uniquement limitent l’achat et l’acquisition de références SKU de la Place de marché Azure aux références SKU BYOL et gratuites uniquement.

### <a name="services-billed-hourly-for-azure-ea"></a>Services facturés à l’heure pour Azure EA

Les services suivants sont facturés à l'heure dans le cadre d'un Contrat Entreprise par rapport au tarif mensuel dans MOSP :

- Application Delivery Network (ADN)
- Pare-feu d’applications web

### <a name="azure-remoteapp"></a>Azure RemoteApp

Si vous disposez d’un Contrat Entreprise, vous payez Azure RemoteApp en fonction du niveau de prix de ce Contrat Entreprise. Il n'y a pas de frais supplémentaires. Le prix standard inclut une durée initiale de 40 heures. Le niveau de prix illimité couvre une durée initiale de 80 heures. L’application distante cesse d’émettre de l’utilisation au-delà des 80 heures.

## <a name="azure-marketplace-faq"></a>FAQ Place de marché Azure

Cette section explique comment votre Paiement anticipé Azure peut s’appliquer à certains services de revendeurs tiers sur la Place de marché Azure.

### <a name="what-changed-with-azure-marketplace-services-and-azure-ea-prepayment"></a>Qu’est-ce qui a changé avec les services de la place de marché Azure et Paiement anticipé Azure EA ?

Depuis le 1er mars 2018, certains services de revendeurs tiers consomment Paiement anticipé Azure EA. À l’exception des instances de machines virtuelles réservées (RI) Azure, les services étaient auparavant facturés séparément, en dehors de Paiement anticipé Azure EA.

Nous avons développé Paiement anticipé Azure pour inclure certains des services de la Place de marché Azure publiés par des tiers et les plus fréquemment achetés. Paiement anticipé Azure EA pour ces services de la Place de marché Azure permet de simplifier la gestion de vos achats et paiements.

### <a name="why-did-we-make-this-change"></a>Pourquoi avons-nous procédé à ce changement ?

Les clients recherchent continuellement de nouveaux moyens pour tirer parti du Paiement anticipé Azure. Ce changement a souvent été demandé par les clients, et il a eu un impact sur une grande partie des clients de la Place de marché Azure.

### <a name="how-do-you-benefit"></a>Quels sont les avantages pour vous ?

Vous bénéficiez d’une expérience de facturation plus simple et de la possibilité d’utiliser votre Paiement anticipé Azure EA. Comme ces services sont inclus dans votre Paiement anticipé Azure, votre Paiement anticipé Azure EA prend de la valeur.

### <a name="what-azure-marketplace-services-use-azure-ea-prepayment-and-how-do-i-know"></a>Quels services de la Place de marché Azure utilisent Paiement anticipé Azure EA et comment savoir ?

Quand vous achetez un service qui utilise Paiement anticipé Azure, la Place de marché Azure affiche une clause de non-responsabilité. Certains services publiés par Red Hat, SUSE, Autodesk et Oracle sont pris en charge. Actuellement, des services portant le même nom et publiés par d’autres parties ne sont pas déduits de Paiement anticipé Azure. Une liste complète est disponible à la fin de cette FAQ.

### <a name="what-if-my-azure-ea-prepayment-runs-out"></a>Que se passe-t-il si mon Paiement anticipé Azure EA tombe à zéro ?

Si vous avez consommé la totalité de votre Paiement anticipé Azure et êtes maintenant en dépassement, les frais liés à ces services apparaissent sur votre prochaine facture de dépassement, avec les autres services de consommation. Avant le changement survenu le 1er mars 2018, ces frais étaient facturés avec d'autres services de la Place de marché Azure.

### <a name="why-dont-all-azure-marketplaces-consume-azure-ea-prepayment"></a>Pourquoi toutes les Places de marché Azure ne consomment-elles pas Paiement anticipé Azure EA ?

Nous nous efforçons d’offrir la meilleure expérience client liée à Paiement anticipé Azure EA. Cette modification concerne un grand nombre de clients et une part importante du total des dépenses dans la Place de marché Azure. D’autres services peuvent être ajoutés à l’avenir.

### <a name="how-does-this-impact-indirect-enrollment-and-partners"></a>Comment cela affecte-t-il l’inscription et les partenaires indirects ?

Il n’y a aucun impact sur nos clients ou partenaires d’inscription indirects. Ces services sont soumis aux mêmes fonctionnalités de majoration partenaire que les autres services de consommation. La seule modification est qu’ils apparaissent sur une facture différente, et que le paiement des frais provient du Paiement anticipé Azure EA du client.

### <a name="is-there-a-list-of-azure-marketplace-services-that-consume-azure-ea-prepayment"></a>Existe-t-il une liste des services de la Place de marché Azure qui consomment Paiement anticipé Azure EA ?

Des offres spécifiques de la Place de marché Azure peuvent utiliser des fonds Paiement anticipé Azure. Consultez la section consacrée aux [services tiers qui utilisent Paiement anticipé Azure](https://azure.microsoft.com/updates/azure-marketplace-third-party-reseller-services-now-use-azure-monetary-commitment) pour obtenir une liste complète des produits participant à ce programme.


## <a name="next-steps"></a>Étapes suivantes

- Obtenez davantage d’informations sur la [tarification](ea-pricing-overview.md).