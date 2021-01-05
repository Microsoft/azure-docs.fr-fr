---
title: Questions fréquentes (FAQ) sur la transition du portail Cloud Partner vers l’Espace partenaires - Place marché commerciale Microsoft
description: Réponses aux questions fréquemment posées sur la transition des offres du portail Cloud Partner vers l’Espace partenaires.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 8346edd0f6016f1751a392213e5d6321f875baf2
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96455658"
---
# <a name="frequently-asked-questions-about-transitioning-from-the-cloud-partner-portal-to-partner-center"></a>Questions fréquentes (FAQ) sur la transition du portail Cloud Partner vers l’Espace partenaires

Le portail Cloud Partner a fait l’objet d’une transition vers l’Espace partenaires. L’Espace partenaires offre une expérience simplifiée et intégrée pour la publication de nouvelles offres sur [Microsoft AppSource](https://appsource.microsoft.com/) ou la [Place de marché Azure](https://azuremarketplace.microsoft.com/) et pour la gestion des offres existantes qui ont été migrées à partir du portail Cloud Partner. Toutes les fonctions du portail Cloud Partner sont disponibles dans l’Espace partenaires. Cet article répond aux questions fréquentes sur ce sujet.

## <a name="what-does-the-transition-to-partner-center-mean-for-me"></a>Que signifie pour moi la transition vers l’Espace partenaires ?

Vous pouvez continuer à travailler dans l’Espace partenaires :

| Domaine<img src="" width=200px> | Modifications |
| --- | --- |
| Compte | Vous n’avez pas besoin de créer un nouveau compte Espace partenaires ; vous pouvez utiliser vos informations d’identification existantes du portail Cloud Partner pour vous connecter à l’Espace partenaires, où vous allez maintenant gérer votre compte, les utilisateurs, les autorisations et la facturation. Le contrat d’éditeur et les informations sur le profil de l’entreprise sont migrés vers votre nouveau compte Espace partenaires, ainsi que les informations de profil de paiement, les comptes d’utilisateur et les autorisations, ainsi que les offres actives. En savoir plus dans [Gérer votre compte Place de marché commerciale dans l’Espace partenaires](partner-center-portal/manage-account.md). |
| Expérience de publication d’offre et de gestion des offres | Nous avons déplacé vos données d’offre du portail Cloud Partner vers l’Espace partenaires. Maintenant, vous accédez à vos offres dans l’Espace partenaires, qui offre une expérience utilisateur améliorée et une interface intuitive. Découvrez comment [Mettre à jour une offre existante dans la place de marché commerciale](partner-center-portal/update-existing-offer.md). |
| Disponibilité de vos offres sur la Place de marché commerciale | Aucun changement. Si votre offre est active dans la place de marché commerciale, elle restera active. |
| Nouveaux achats et déploiements | Aucun changement. Vos clients peuvent continuer à acheter et à déployer vos offres sans interruption. |
| Paiements | Les achats et les déploiements continuent à vous être payés normalement. En savoir plus sur [Recevoir un paiement dans la Place de marché commerciale](/partner-center/marketplace-get-paid?context=/azure/marketplace/context/context). |
| Intégrations d’API avec des [API du portail Cloud Partner existantes](cloud-partner-portal-api-overview.md) | Les API du portail Cloud Partner existantes sont toujours prises en charge et vos intégrations existantes fonctionnent toujours. En savoir plus dans [Les API REST du portail Cloud Partner seront-elles prises en charge ?](#are-the-cloud-partner-portal-rest-apis-still-supported) |
| Analytics | Vous pouvez continuer à surveiller les ventes, évaluer les performances et optimiser vos offres sur la Place de marché commerciale en affichant l’analytique dans l’Espace partenaires. Il existe des différences entre l’affichage des rapports analytiques dans CPP et Espace partenaires. Par exemple, la fonctionnalité **Insights Vendeurs** dans CPP a un onglet **Commandes et utilisation** qui affiche les données pour les offres basées sur l’utilisation et celles non basées sur l’utilisation, tandis que, dans l’Espace partenaires, la page **Commandes** a un onglet distinct pour les offres SaaS. En savoir plus dans [Accéder aux rapports analytiques de la Place de marché commerciale dans l’Espace partenaires](partner-center-portal/analytics.md). |
|||

## <a name="do-i-need-to-create-a-new-account-to-manage-my-offers-in-partner-center"></a>Dois-je créer un nouveau compte pour gérer mes offres dans l’Espace partenaires ?

Non, votre compte est préservé. Cela signifie que si vous êtes un partenaire existant, vous pouvez utiliser vos informations d’identification de compte Portail Cloud Partner existantes pour vous connecter à l’Espace partenaires. Ne créez pas de compte ou les offres créées dans le portail Cloud Partner et déplacées vers l’Espace partenaires n’y seront pas associées.

## <a name="what-pages-in-partner-center-correspond-to-pages-i-used-in-the-cloud-partner-portal"></a>Quelles pages de l’Espace partenaires correspondent aux pages que j’utilisais dans le portail Cloud Partner ?

Voici les liens de l’Espace partenaires pour les pages couramment utilisées dans le portail Cloud Partner. Si vous avez enregistré les liens du portail Cloud Partner en tant que signets, vous devriez les mettre à jour.

| Page du portail Microsoft Cloud Partner <img src="" width=100px>| Lien de la page du portail Cloud Partner | Lien de la page de l’Espace partenaire |
| --- | --- | --- |
| Page Toutes les offres | [https://cloudpartner.azure.com/#alloffers](https://cloudpartner.azure.com/#alloffers) | [https://partner.microsoft.com/dashboard/commercial-marketplace/overview](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) |
| Page Tous les éditeurs | [https://cloudpartner.azure.com/#publishers](https://cloudpartner.azure.com/#publishers) | [https://partner.microsoft.com/dashboard/account/v3/publishers/list](https://partner.microsoft.com/dashboard/account/v3/publishers/list) |
| Profil de publication | [https://cloudpartner.azure.com/#profile](https://cloudpartner.azure.com/#profile) | [https://partner.microsoft.com/dashboard/account/management](https://partner.microsoft.com/dashboard/account/management) |
| Page Utilisateurs | [https://cloudpartner.azure.com/#users](https://cloudpartner.azure.com/#users) | [https://partner.microsoft.com/dashboard/account/usermanagement](https://partner.microsoft.com/dashboard/account/usermanagement) |
| Page Historique | [https://cloudpartner.azure.com/#history](https://cloudpartner.azure.com/#history) | La fonctionnalité d’historique n’est pas encore prise en charge dans l’Espace partenaires. |
| Tableau de bord Insights | [https://cloudpartner.azure.com/#insights](https://cloudpartner.azure.com/#insights) | [https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary) |
| Rapport de paiements | [https://cloudpartner.azure.com/#insights/payout](https://cloudpartner.azure.com/#insights/payout) | [https://partner.microsoft.com/dashboard/payouts/reports/incentivepayments](https://partner.microsoft.com/dashboard/payouts/reports/incentivepayments) |
|||

## <a name="payout-report-differences"></a>Différences dans le rapport de revenu

Voici les différences dans le rapport de revenu entre l’ancien Portail Cloud Partner et l’actuel Espace partenaires :

| Portail des partenaires cloud | Espace partenaire |
| --- | --- |
| **Lien** : https://cloudpartner.azure.com/ | **Lien** : https://partner.microsoft.com/dashboard/payouts/reports/transactionhistory et https://partner.microsoft.com/dashboard/payouts/reports/incentivepayments |
| **Navigation** : Rapports de paiement fournis dans le paiement des insights | **Navigation** : Rapports de paiement fournis dans l’Espace partenaires – Icône de paiement |
| **Étendue** :<ul><li>La transaction par ligne est visible, quand l’encaissement est en cours, quand elle est terminée et quand elle est payée.</li><li>Rapports : montre toutes les lignes une fois que le bon de commande est créé, notamment l’encaissement et la facturation en cours ainsi que l’état de l’encaissement et les lignes qui ne peuvent pas encore être payés.</li></ul> | **Étendue** :<ul><li>Montre les lignes une fois qu’elles sont considérées comme des revenus éligibles.</li><li>Les clients paient d’abord Microsoft, puis les éditeurs de logiciels indépendants peuvent voir le rapport de paiement commencer.</li><li>Le rapport de revenu ne montre pas l’encaissement ni la facturation en cours.</li></ul> |
| **Transaction non prête pour le paiement** : Facturation en cours | **Transaction non prête pour le paiement** : Estimation du prochain paiement : Le paiement est dans l’état non traité. |
| **Statut de paiement** : n/a | **Statut de paiement** :<ul><li>Non traité : Le revenu est éligible au paiement.</li><li>Ensuite : Le revenu sera envoyé à l’éditeur à l’occasion du prochain paiement mensuel.</li><li>Envoyé : Le paiement a été envoyé à votre banque.</li></ul> |
|||

## <a name="what-about-offers-i-published-in-the-cloud-partner-portal"></a>Qu’en est-il des offres que j’ai publiées dans le portail Cloud Partner ?

Les offres ont été déplacées vers l’Espace partenaires. Vous pourrez y accéder une fois que vous vous serez connecté à l’Espace partenaires, à l’exception des offres Dynamics Nav Managed Service et Cortana Intelligence. Si votre offre était active sur la Place de marché commerciale, elle continuera à être active et vos clients pourront toujours l’acheter et la déployer sans interruption. Pour plus d’informations, consultez la question suivante **Quelles offres ont été déplacées vers l’Espace partenaires ?** .

## <a name="what-offers-were-moved-to-partner-center"></a>Quelles offres ont été déplacées vers l’Espace partenaires ?

Tous les types d’offres précédemment pris en charge dans le portail Cloud Partner sont pris en charge dans l’Espace partenaires, à l’exception des offres Dynamics Nav Managed Service et Cortana Intelligence.

Pour les types d’offres pris en charge dans l’Espace partenaires, toutes les offres ont été déplacées, quel que soit leur état. Les offres de type brouillon, plus répertoriées et en préversion uniquement ont également été déplacées.

| Type d’offre <img src="" width=150px>| Déplacé vers l’Espace partenaires ? <img src="" width=100px>| Étapes suivantes |
| --- | --- | --- |
| SaaS | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Planifier une offre SaaS pour la place de marché commerciale](plan-saas-offer.md). |
| Machine virtuelle | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Planifier une offre de machine virtuelle](marketplace-virtual-machines.md). |
| Application Azure | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Créer une offre d’application Azure](create-new-azure-apps-offer.md). |
| Dynamics 365 Business Central | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Créer une offre Dynamics 365 Business Central](partner-center-portal/create-new-business-central-offer.md). |
| Dynamics 365 for Customer Engagement et PowerApps | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Créer une offre Dynamics 365 for Customer Engagement et PowerApps](partner-center-portal/create-new-customer-engagement-offer.md). |
| Dynamics 365 for Operations | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Créer une offre Dynamics 365 for Operations](partner-center-portal/create-new-operations-offer.md). |
| Application Power BI | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Créer une application Power BI pour AppSource](partner-center-portal/create-power-bi-app-offer.md). |
| Module IoT Edge | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Créer, configurer et publier une offre de module IoT Edge dans la Place de marché Azure](partner-center-portal/azure-iot-edge-module-creation.md). |
| Conteneur | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Créer une offre de conteneur Azure](./create-azure-container-offer.md). |
| Service de conseil | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Créer une offre de services de conseil](./create-consulting-service-offer.md). |
| Service managé | Oui | Connectez-vous à l’Espace partenaires pour créer de nouvelles offres et gérer les offres créées dans le portail Cloud Partner. En savoir plus dans [Créer une offre de service managé](partner-center-portal/create-new-managed-service-offer.md). |
| Dynamics Nav Managed Service | Non | Microsoft a fait évoluer Dynamics NAV Managed Service en [Dynamics 365 Business Central](/dynamics365/business-central/). Nous avons donc supprimé les offres actives Dynamics NAV Managed Service dans AppSource. Ces offres ne sont plus détectables par les clients et n’ont pas été déplacées vers l’Espace partenaires. Pour que vos offres soient disponibles dans AppSource, transformez-les en offres Dynamics 365 Business Central et envoyez-les dans [l’Espace partenaires](https://partner.microsoft.com/). En savoir plus dans [Créer une offre Dynamics 365 Business Central](partner-center-portal/create-new-business-central-offer.md). |
| Cortana Intelligence | Non | Microsoft a fait évoluer la feuille de route pour Cortana Intelligence. Nous avons donc supprimé les offres actives Cortana Intelligence dans AppSource. Ces offres ne sont plus détectables par les clients et n’ont pas été déplacées vers l’Espace partenaires. Pour que vos offres soient disponibles sur la place de marché commerciale, transformez-les en offres SaaS (Software as a Service) et envoyez-les dans [l’Espace partenaires](https://partner.microsoft.com/). En savoir plus dans [Liste de vérification de la création d’offre SaaS dans l’Espace partenaires](./plan-saas-offer.md). |

## <a name="i-cant-find-my-cloud-partner-portal-offers-in-partner-center"></a>Je ne trouve pas mes offres du portail Cloud Partner dans l’Espace partenaires

Ce que vous voyez dans l’Espace partenaires dépend des programmes auxquels vous êtes inscrit, des comptes auxquels vous appartenez, des rôles utilisateur et des autorisations qui vous ont été attribués. De nombreux programmes de l’Espace partenaires sont disponibles et vous pourriez être inscrit à plusieurs d’entre eux. Vous pouvez également avoir accès à plusieurs comptes avec les mêmes informations d’identification utilisateur.

Les offres que vous avez créées dans le portail Cloud Partner sont disponibles dans l’Espace partenaires sous le programme **Place de marché commerciale** et sous le compte utilisé pour créer les offres. Procédez comme suit pour vérifier que vous affichez le bon programme et le bon compte. Pour obtenir d’autres conseils de dépannage, consultez [Gérer votre compte Espace partenaires](/partner-center/partner-center-account-setup).

### <a name="access-the-right-program-in-partner-center"></a>Accéder au programme approprié dans l’Espace partenaires

1. Connectez-vous à [l’Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) avec les informations d’identification utilisées pour vous connecter au portail Cloud Partner. Le volet de navigation sur la gauche affiche les options associées aux programmes auxquels vous êtes inscrit.

    Exemple : supposons que vous ayez accès à trois programmes : le programme MPN, le programme Références et le programme Place de marché commerciale. Lorsque vous vous connectez à l’Espace partenaires, ces trois programmes s’affichent dans le volet de navigation.

2. Sélectionnez **Place de marché commerciale** > **Vue d’ensemble** pour accéder à vos offres.

    Si vous ne voyez pas le programme Place de marché commerciale dans le volet de navigation à gauche, le compte est peut-être erroné. Suivez la procédure décrite ci-après pour accéder au compte approprié.

    [![Screenshot that shows the Partner Center Overview menu](media/cpp-pc-faq/overview-menu.png "Affiche le menu Vue d’ensemble de l’Espace partenaires")](media/cpp-pc-faq/overview-menu.png#lightbox)

### <a name="access-the-right-account-in-partner-center"></a>Accéder au compte approprié dans l’Espace partenaires

Si vous êtes membre de plusieurs comptes, un bouton de sélection de compte marqué par deux flèches s’affiche dans le menu de navigation à gauche de l’Espace partenaires. Sélectionnez le bouton de sélection de compte pour afficher la liste de tous les comptes auxquels vous appartenez. Sélectionnez un compte dans la liste pour y accéder et voir l’ensemble des programmes et des informations sur ce compte. Si le bouton de sélection de compte ne s’affiche pas dans le menu de navigation, cela signifie que vous êtes membre d’un seul compte.

[![Screenshot shows the Partner Center account picker button.](media/cpp-pc-faq/picker-button.png "Affiche le bouton de sélection de compte de l’Espace partenaires")](media/cpp-pc-faq/picker-button.png#lightbox)

## <a name="how-do-i-create-new-offers"></a>Comment créer de nouvelles offres ?

Accédez au programme Place de marché commerciale dans [l’Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) pour créer des offres. Dans la page Vue d’ensemble, sélectionnez **+ Nouvelle offre**.

[![Screenshot shows the Partner Center Overview menu.](media/cpp-pc-faq/new-offer.png "Affiche le menu Vue d’ensemble de l’Espace partenaires")](media/cpp-pc-faq/new-offer.png#lightbox)

## <a name="i-cant-sign-in-and-need-to-open-a-support-ticket"></a>Je ne parviens pas à me connecter et je dois ouvrir un ticket de support

Si vous ne pouvez pas vous connecter à votre compte, vous pouvez ouvrir un [ticket de support](https://partner.microsoft.com/support/v2/?stage=1) ici.

## <a name="where-are-instructions-for-using-partner-center"></a>Où se trouvent les instructions d’utilisation de l’Espace partenaires ?

Accédez à la [documentation du marketplace commercial](index.yml), puis développez **Portail du marketplace commercial dans Espace partenaires**. Pour afficher des articles d’aide sur la création d’offres dans l’Espace partenaires, développez **Créer une offre**.

## <a name="what-are-the-publishing-and-offer-management-differences"></a>Quelles sont les différences en matière de publication et de gestion des offres ?

Voici quelques différences entre le portail Cloud Partner et l’Espace partenaires.

### <a name="modular-publishing-capabilities"></a>Fonctionnalités de publication modulaires

L’Espace partenaires fournit une option de publication modulaire qui vous permet de sélectionner les modifications que vous souhaitez publier au lieu de toujours publier toutes les mises à jour à la fois. Par exemple, l’écran suivant montre que les seules modifications sélectionnées à publier sont les modifications apportées à **Propriétés** et **Référencement de l’offre**. Les modifications que vous apportez dans la page Préversion ne sont pas publiées.

[![Screenshot shows the Partner Center Review and publish page.](media/cpp-pc-faq/review-page.png "Affiche la page Révision et publication de l’Espace partenaires")](media/cpp-pc-faq/review-page.png#lightbox)

Les mises à jour que vous ne publiez pas sont enregistrées en tant que brouillons. Continuez à utiliser la préversion de votre offre afin de la vérifier avant de la rendre accessible au public.

### <a name="enhanced-preview-options"></a>Options d’aperçu améliorées

L’Espace partenaires comprend une [fonctionnalité de comparaison](partner-center-portal/update-existing-offer.md#compare-changes-to-your-offer) avec des options de filtrage améliorées. Cela vous permet d’effectuer des comparaisons entre la préversion et la version active de l’offre.

[![Screenshot shows the Partner Center compare feature.](media/cpp-pc-faq/compare.png "Affiche la fonctionnalité de comparaison de l’Espace partenaires")](media/cpp-pc-faq/compare.png#lightbox)

### <a name="branding-and-navigation-changes"></a>Changements dans la personnalisation et la navigation

Vous remarquerez certaines modifications de personnalisation. Par exemple, les *Références SKU* sont appelées *Plans* dans l’Espace partenaires :

[![Screenshot shows the Partner Center Plans page.](media/cpp-pc-faq/plans.png "Affiche la page Plans de l’Espace partenaires")](media/cpp-pc-faq/plans.png#lightbox)

De plus, les informations que vous fournissiez précédemment dans les pages **Place de marché** ou **Détails de la vitrine** (Service de conseil, application Power BI) du portail Cloud Partner sont maintenant recueillies dans la page **Référencement de l’offre** dans l’Espace partenaires :

[![La capture d’écran montre la page de référencement d’offre de l’Espace partenaires.](media/cpp-pc-faq/offer-listing.png](media/cpp-pc-faq/offer-listing.png#lightbox)

Les informations que vous fournissiez précédemment pour les références SKU dans une seule page du portail Cloud Partner peuvent maintenant être recueillies dans plusieurs pages de l’Espace partenaires :

- Page de configuration du plan
- Page de référencement du plan
- Page de disponibilité du plan
- Page de configuration technique du plan, comme illustré ici :

[![Affiche la page de configuration technique de l’Espace partenaires.](media/cpp-pc-faq/technical-configuration.png)](media/cpp-pc-faq/technical-configuration.png#lightbox)

Votre ID d’offre est maintenant affiché dans la barre de navigation gauche de l’offre :

![Affiche l’emplacement de l’ID de l’offre de l’Espace partenaires](media/cpp-pc-faq/offer-id.png)

### <a name="stop-selling-an-offer"></a>Arrêter la vente d’une offre

Vous pouvez demander à [arrêter la vente d’une offre](partner-center-portal/update-existing-offer.md#stop-selling-an-offer-or-plan) sur la Place de marché directement à partir du portail de l’Espace partenaires. L’option est disponible dans la page **Vue d’ensemble de l’offre** de votre offre.

[![Screenshot shows the Partner Center page to stop selling an offer.](media/cpp-pc-faq/stop-sell.png "Affiche la page de l’Espace partenaires pour arrêter la vente d’une offre")](media/cpp-pc-faq/stop-sell.png#lightbox)
<br><br>

## <a name="are-the-cloud-partner-portal-rest-apis-still-supported"></a>Les API REST du portail Cloud Partner sont-elles toujours prises en charge ?

Les API du portail Cloud Partner sont intégrées à l’Espace partenaires et continueront à fonctionner. La transition vers l’Espace partenaires introduit de petites modifications. Passez en revue le tableau suivant pour vous assurer que votre code continue à fonctionner dans l’Espace partenaires.

| API <img src="" width=100px>| Description de la modification | Impact |
| --- | --- | --- |
| POST Publish, GoLive, Cancel | Pour les offres migrées, l’en-tête de réponse aura un format différent, mais continuera à fonctionner de la même façon, indiquant un chemin relatif pour récupérer l’état de l’opération. | Lors de l’envoi d’une des requêtes POST correspondantes pour une offre, l’en-tête d’emplacement aura l’un des deux formats suivants en fonction de l’état de migration de l’offre : <ul><li>Offres non migrées : `/api/operations/{PublisherId}${offerId}$2$preview?api-version=2017-10-31`</li><li>Offres migrées : `/api/publishers/{PublisherId}/offers/{offereId}/operations/408a4835-0000-1000-0000-000000000000?api-version=2017-10-31`</li></ul>|
| Opération GET | Pour les offres qui prenaient auparavant en charge le champ « notification-email » dans la réponse, ce champ est déprécié et n’est plus retourné pour les offres migrées. | Pour les offres migrées, nous n’enverrons plus de notifications à la liste des e-mails spécifiés dans les requêtes. Au lieu de cela, le service d’API s’alignera sur le processus de notification par e-mail dans l’Espace partenaires pour envoyer des e-mails. Plus précisément, les notifications de progression de l’opération seront envoyées à l’adresse e-mail définie dans la section Coordonnées du vendeur de vos paramètres de compte dans l’Espace partenaires.<br><br>Vérifiez que l’adresse e-mail définie dans la section Coordonnées du vendeur dans [Paramètres du compte](https://partner.microsoft.com/dashboard/account/management) de l’Espace partenaires est correcte pour recevoir les notifications. |
|||