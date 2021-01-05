---
title: Comment vérifier et publier une offre sur le marketplace commercial Microsoft
description: Utilisez l’Espace Partenaires pour soumettre votre offre en préversion, afficher un aperçu de votre offre, puis la publier sur le Marketplace Microsoft.
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: mingshen-ms
ms.author: mingshen
ms.date: 10/27/2020
ms.openlocfilehash: b038834df3889a8b4b7f4a749568635e99f8408e
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188749"
---
# <a name="how-to-review-and-publish-an-offer-to-the-commercial-marketplace"></a>Comment vérifier et publier une offre sur le marketplace commercial

Cet article vous montre comment utiliser Espace Partenaires pour soumettre votre offre à la publication, afficher un aperçu de votre offre, puis la publier sur le marketplace commercial. Nous aborderons également la vérification de l’état de publication au fur et à mesure de la procédure de publication. Vous devez avoir déjà créé une offre que vous souhaitez publier.

## <a name="offer-status"></a>État de l’offre

Vous pouvez consulter l’état de votre offre dans l’onglet **Vue d’ensemble** du tableau de bord du marketplace commercial dans [Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace/overview). L’un des indicateurs d’état suivants s’affiche dans la colonne **État** pour chaque offre.

| Statut | Description |
| ------------ | ------------- |
| Brouillon | L’offre a été créée, mais elle n’est actuellement pas publiée. |
| Publication en cours | Le processus de publication de l’offre est en cours. |
| Attention requise | Nous avons découvert un problème critique au cours de la certification ou au cours d’une autre phase de publication. |
| PRÉVERSION | L’offre a été certifiée par Microsoft et doit maintenant faire l’objet d’une vérification finale par l’éditeur. Sélectionnez **Mise en ligne** pour proposer l’offre en direct. |
| En direct | L’offre est également disponible en direct dans la Place de marché. Des clients peuvent la voir et l’acheter. |
| Arrêt de la vente en attente | L’éditeur a sélectionné « Arrêter la vente » sur l’offre ou le plan, mais l’action n’est pas encore terminée. |
| Non disponible dans la Place de marché | Une offre publiée sur la place de marché a été supprimée. |
|||

## <a name="validation-and-publishing-steps"></a>Étapes de validation et de publication

Lorsque vous êtes prêt à soumettre votre offre à la publication, sélectionnez **Vérifier et publier** dans le coin supérieur droit du portail. La page **Vérifier et publier** affiche l’état de chaque page de votre offre, qui peut être l’un des états suivants :

   - **Non démarré** : la page n’est pas démarrée.
   - **Incomplète** : des informations requises sont manquantes dans la page ou des erreurs doivent être corrigées. Vous devez revenir à la page et la mettre à jour.
   - **Complète** : la page est complète. Toutes les données requises ont été fournies et il n’existe aucune erreur.

Si l’un des pages présente un état autre que **Complète**, vous devez corriger le problème sur cette page, puis revenir à la page **Vérifier et publier** pour confirmer que l’état s’affiche comme **Complète**. Certains types d’offres nécessitent un test. Si c’est le cas, vous verrez un champ **Notes pour la certification** dans lequel vous devez fournir des instructions de test à l’équipe de certification et toutes autres notes utiles pour comprendre votre application.

Une fois que toutes les pages sont terminées et que vous avez entré les notes de test correspondantes, sélectionnez **Publier** pour commencer les processus de validation et de publication. Les phases et la séquence globale peuvent varier en fonction du type d’offre que vous publiez. Le tableau suivant présente un seul et même flux de publication. Chaque phase est expliquée plus en détail dans les sections suivantes.

| Phase | Ce qui se produit |
| ------------ | ------------- | ------------- |
| [Validation automatique](#automated-validation-phase) | Nous effectuons un ensemble de validations automatisées. |
| [Certification](#certification-phase) | Nous effectuons des validations manuelles. |
| [Création de la préversion](#preview-creation-phase) | La page de liste de votre offre en préversion est disponible pour toute personne disposant d’un lien d’aperçu. Si votre offre est vendue par le biais de Microsoft (négociable), seul le public que vous avez spécifié sur la page **Preview audience** (Public de la préversion) de votre offre peut acheter et accéder à l’offre pour test. |
| [Approbation de l’éditeur](#publisher-sign-off-phase) | Nous vous envoyons un e-mail vous demandant de regarder votre offre et de l’approuver. |
| [Publier](#publish-phase) | Nous exécutons une série d’étapes pour vérifier que l’offre en préversion est publiée en direct sur la place de marché commerciale. |
|||

## <a name="automated-validation-phase"></a>Phase de validation automatique

La première étape du processus de publication est un ensemble de validations automatiques. Chaque étape de validation correspond à une fonctionnalité que vous avez choisi d’activer lors de la création de votre offre. Chaque contrôle de validation doit être effectué avant que l’offre puisse passer à l’étape suivante du processus de publication.

- **Configuration du flux d’achat de l’offre** (< 10 min)

   Nous faisons en sorte que votre offre puisse être traitée quand elle est achetée par des clients à l’aide du Portail Azure. Cette étape est applicable uniquement aux offres vendues par le biais de Microsoft.

- **Validation des données de la version d’évaluation** (environ 5 min)

   Nous validons les données que vous avez fournies dans la page Configuration technique de l’offre. Nous testons et approuvons les fonctionnalités de la version d'évaluation. Cette étape est applicable uniquement pour les offres avec une version d’évaluation activée.

- **Provisionnement de la version d’évaluation** (environ 30 min)

    Après avoir validé les données et les fonctionnalités de votre version d’évaluation à l’étape précédente, nous déployons et répliquons des instances de votre version d’évaluation afin qu’elles soient prêtes à être utilisées par les clients. Cette étape est applicable uniquement pour les offres avec une version d’évaluation activée.

- **Validation et enregistrement de la gestion des prospects** (< 15 min)

    Nous vérifions que votre système de gestion des prospects peut recevoir des prospects en fonction des détails fournis dans la page **Configuration de l’offre**. Cette étape est applicable uniquement pour les offres avec la gestion des prospects activée.

## <a name="certification-phase"></a>Phase de certification

Les offres soumises à la place de marché commerciale doivent être certifiées avant d’être publiées. Les offres soumises subissent des tests rigoureux, parfois automatisés et parfois manuels. Pour en savoir plus, consultez les [politiques de certification de la place de marché commerciale](/legal/marketplace/certification-policies).

### <a name="types-of-validation-that-take-place-during-certification"></a>Types de validation qui se produisent lors de la certification

Le processus de certification comprend trois niveaux de validation pour chaque offre soumise.

- Éligibilité de l’entreprise de l’éditeur
- Validation du contenu
- Validation technique

#### <a name="publisher-business-eligibility"></a>Éligibilité de l’entreprise de l’éditeur

Chaque type d’offre vérifie un ensemble de critères d’éligibilité de base. Ces critères peuvent inclure l’état MPN de l’éditeur, les compétences détenues, les niveaux de compétence, etc.

#### <a name="content-validation"></a>Validation du contenu

La qualité et la pertinence des informations entrées lors de la création de votre offre sont vérifiées. Ces vérifications passent en revue vos entrées pour les détails de référencement dans la Place de marché, le tarif, la disponibilité, les plans associés, etc. Pour répondre aux critères de référencement de Microsoft AppSource et Place de marché Azure, nous vérifions que votre offre inclut les éléments suivants :

- Un titre qui décrit précisément l’offre
- Des descriptions bien écrites qui fournissent une vue d’ensemble détaillée et une proposition de valeur
- Des captures d’écran et vidéos de qualité
- Une description de la façon dont l’offre utilise les outils et des plateformes Microsoft.

Apprenez-en plus sur les critères de validation du contenu en lisant les [stratégies de référencement générales](/legal/marketplace/certification-policies#100-general).

#### <a name="technical-validation"></a>Validation technique

Pendant la validation technique, l’offre (package ou binaire) fait l’objet des vérifications suivantes.

- Analyse pour détecter les programmes malveillants
- Supervision des appels réseau
- Analyse du package
- Une analyse complète des fonctionnalités réelles de l’offre

L’offre est testée sur différentes plateformes et différentes versions afin de garantir qu’elle est fiable.

### <a name="certification-failure-report"></a>Rapport d’échec de la certification

Si votre offre ne passe pas l’une des vérifications de référencement, techniques ou de politique, ou si vous n’êtes pas qualifié pour soumettre une offre de ce type, un rapport d’échec de certification est généré et vous est envoyé par e-mail.

Ce rapport contient des descriptions de toutes les stratégies ayant échoué, ainsi que des remarques sur les vérifications. Passez en revue ce rapport par e-mail, résolvez tous les problèmes, en mettant à jour votre offre si nécessaire, puis soumettez de nouveau l’offre à l’aide du [portail de la place de marché commerciale](https://partner.microsoft.com/dashboard/commercial-marketplace/offers) dans l’Espace partenaires. (Vous pouvez soumettre l’offre autant de fois que nécessaire jusqu’à ce que la certification réussisse).

## <a name="preview-creation-phase"></a>Phase de création de la préversion

Lors de l’étape de création de la préversion, nous créons une version de votre offre accessible uniquement au public que vous avez spécifié dans la section **Public de la préversion** (Public de la préversion) de votre offre. La préversion de votre offre sera indisponible pour toute personne n’étant pas spécifiée dans le public de la préversion tant que vous n’avez pas publié l’offre en direct.

> [!NOTE]
> N'utilisez pas ce public de préversion pour permettre aux personnes extérieures à votre organisation de visualiser une offre. Utilisez plutôt l'option Private Offer (Offre privée). À ce stade, votre offre n'a pas encore été entièrement testée et validée, et elle n'est pas prête à être distribuée à l'extérieur.

## <a name="cancel-publishing"></a>Annuler la publication

Pour annuler une offre avec l’état **Publication en cours d’exécution** :

1. Sélectionnez le nom de l’offre pour ouvrir la page **Vue d’ensemble de l’offre**.
1. Sélectionnez **Annuler la publication** à partir de l’angle supérieur droit de la page.
1. Confirmez que vous souhaitez arrêter la publication de l’offre.

Si vous souhaitez publier l’offre ultérieurement, vous devrez redémarrer le processus de publication.

> [!NOTE]
> Vous pouvez arrêter la publication d’une offre seulement si elle n’a pas encore atteint l’étape d’approbation de l’éditeur. Après avoir sélectionné **Mise en ligne**, vous n’avez plus la possibilité d’annuler la publication.

## <a name="publisher-sign-off-phase"></a>Phase d’approbation de l’éditeur

Une fois que l’offre est prête à être revue et approuvée, nous vous enverrons un e-mail pour vous demander de consulter et d’approuver la version préliminaire de votre offre. Vous pouvez également actualiser la page **Vue d’ensemble de l’offre** depuis votre navigateur pour voir si votre offre a atteint la phase d’approbation de publication. Si c’est le cas, le bouton **Go Live** et les liens d’aperçu seront disponibles.

La capture d’écran suivante montre la page **Vue d’ensemble de l’offre** pour une offre SaaS. Les étapes de validation que vous verrez sur cette page varient en fonction du type d’offre et des sélections que vous avez apportées lors de la création de l’offre.

![Illustre la page Vue d’ensemble de l’offre pour une offre dans l’Espace partenaires. Le bouton Go Live et les liens d’aperçu s’affichent.](./media/publish-status-publisher-signoff.png)

### <a name="previewing-and-approving-your-offer"></a>Aperçu et approbation de votre offre

> [!IMPORTANT]
> Pour valider le processus d’achat et d’installation de bout en bout, achetez votre offre en version préliminaire. Informez tout d’abord Microsoft avec un [ticket de support](https://aka.ms/marketplacesupport) pour être sûr que nous ne vous imputions pas des frais.

Dans la page **Vue d’ensemble de l’offre**, vous verrez des liens d’aperçu sous le bouton **Mise en ligne**. Il y aura un lien soit pour l’aperçu AppSource, soit pour l’aperçu de la Place de marché Azure, ou les deux, selon les options que vous avez choisies lors de la création de votre offre. Si vous avez choisi de vendre votre offre par le biais de Microsoft, quiconque a été ajouté au public de la préversion peut tester son acquisition et son déploiement pour vérifier qu’elle répond à vos exigences au cours de cette phase.

Une fois que vous avez approuvé votre préversion, sélectionnez **Mise en ligne** pour publier votre offre sur le marketplace commercial. 

Si vous souhaitez apporter des modifications après avoir visualisé un aperçu de l’offre, vous pouvez modifier et soumettre à nouveau votre demande de publication. Si votre offre est déjà en ligne et accessible au public sur le marketplace, toutes les mises à jour que vous apportez ne seront pas mises en service tant que vous n’aurez pas sélectionné **Mise en ligne*. Pour plus d’informations, consultez [Mettre à jour une offre existante dans le marketplace commercial](partner-center-portal/update-existing-offer.md).

## <a name="publish-phase"></a>Phase de publication

Maintenant que vous avez choisi de mettre en service votre offre, ce qui la rend disponible sur la place de marché commerciale, nous effectuons une série de vérifications finales pour garantir que l’offre est configurée exactement comme la préversion.

- **Configuration du flux d’achat de l’offre** (> 10 min)

    Nous faisons en sorte que votre offre puisse être traitée quand elle est achetée par des clients à l’aide du Portail Azure. Cette étape est applicable uniquement aux offres vendues par le biais de Microsoft.

- **Validation des données de la version d’évaluation** (environ 5 min)

    Nous validons les données que vous avez fournies dans la page Configuration technique de l’offre. Nous testons et approuvons les fonctionnalités de la version d'évaluation. Cette étape est applicable uniquement pour les offres avec une version d’évaluation activée.

- **Provisionnement de la version d’évaluation** (environ 30 min)

    Nous déployons et répliquons des instances de votre version d’évaluation afin qu’elles soient prêtes à être utilisées par les clients. Cette étape est applicable uniquement pour les offres avec une version d’évaluation activée.

- **Validation et enregistrement de la gestion des prospects** (> 15 min)

    Nous vérifions que votre système de gestion des prospects peut recevoir des prospects en fonction des détails fournis dans votre page **Configuration de l’offre**. Cette étape est applicable uniquement pour les offres avec la gestion des prospects activée.

- **Dernière publication (> 30 minutes)**

    Nous vérifions que votre offre est publiquement disponible dans la place de marché.

Une fois ces vérifications terminées, votre offre est publiée sur la place de marché.

## <a name="publishing-history"></a>Historique de publication

La page **Historique** dans Espace partenaires affiche les événements de publication de vos offres sur le marketplace commercial. Pour chaque événement, la page affiche l’utilisateur qui a initié l’action, le type d’événement et la date et l’heure de l’événement. Les [étapes de validation et de publication](#validation-and-publishing-steps) sont répertoriées avec la date et l’heure d’achèvement.

Pour afficher l’historique de votre offre :

1.    Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).
2.    Dans le menu de navigation situé à gauche, sélectionnez **Marketplace commercial** > **Vue d’ensemble**.
3.    Sélectionnez votre offre.
4.    Sélectionnez l’onglet **Historique** dans le menu de navigation à gauche. La page fournit des détails sur les événements suivants, en fonction de votre offre :

|Événement    |Description    |
|---------|---------------|
|Offre créée    |L’offre a été créée dans Espace partenaires. Un utilisateur a sélectionné le type d’offre, l’ID d’offre et l’alias d’offre dans **Marketplace commercial** > **Vue d’ensemble**.    |
|Plan créé : *nom du plan*    |Un utilisateur a créé un nouveau plan en entrant l’ID et le nom du plan dans l’onglet **Vue d’ensemble du plan**.</br>*Cet événement s’applique uniquement aux types d’offres qui prennent en charge les plans*.    |
|Plan supprimé    |Un utilisateur a supprimé un plan brouillon qui n’avait pas été publié en sélectionnant **Supprimer le brouillon** dans la page **Vue d’ensemble du plan**.</br>*Cet événement s’applique uniquement aux types d’offres qui prennent en charge les plans*.    |
|Initiation d’un plan de fin de vente : *nom du plan*    |Un utilisateur a initié un plan de fin de vente en sélectionnant **Arrêter la vente** dans la page **Vue d’ensemble du plan**.</br>*Cet événement s’applique uniquement aux types d’offres qui prennent en charge les plans*.    |
|Annulation d’un plan de fin de vente : *nom du plan*    |Un utilisateur a annulé un plan de fin de vente en sélectionnant **Annuler la fin de vente** dans la page **Vue d’ensemble du plan**.</br>*Cet événement s’applique uniquement aux types d’offres qui prennent en charge les plans*.    |
|Offre soumise à la préversion    |Un utilisateur a soumis l’offre à la préversion en sélectionnant **Publier** dans la page **Vérifier et publier**.    |
|Annulation de la soumission initiée à la préversion    |Un utilisateur a demandé l’annulation de la publication de l’offre en préversion en sélectionnant **Annuler la publication** dans la page **Vue d’ensemble de l’offre** après la soumission à la préversion.</br>*Cet événement s’affiche lorsque la demande d’annulation est en cours de traitement*.    |
|Annulation de la soumission à la préversion    |Un utilisateur a annulé la publication de l’offre en préversion en sélectionnant **Annuler la publication** dans la page **Vue d’ensemble de l’offre** après la soumission à la préversion.</br>*Cet événement s’affiche une fois que la demande d’annulation a été traitée*.    |
|Approbation de la mise en ligne    |Un utilisateur a publié l’offre sur le marketplace commercial en sélectionnant **Mise en ligne** dans la page **Vue d’ensemble de l’offre**.    |
|Annulation de la publication initiée sur le marketplace    |Un utilisateur a demandé l’annulation de la publication de l’offre en sélectionnant **Annuler la publication** dans la page **Vue d’ensemble de l’offre** après l’approbation de la mise en ligne.</br>*Cet événement s’affiche lorsque la demande d’annulation est en cours de traitement*.    |
|Annulation de la publication sur le marketplace commercial    |Un utilisateur a annulé la publication de l’offre en sélectionnant **Annuler la publication** dans la page **Vue d’ensemble de l’offre** après l’approbation de la mise en ligne.</br>*Cet événement s’affiche une fois que la demande d’annulation a été traitée*.    |
|Synchronisation des audiences privées    |Un utilisateur a mis à jour et synchronisé l’audience privée en sélectionnant **Synchroniser l’audience privée** dans la page **Vue d’ensemble du plan** dans la page **Tarification et disponibilité du plan**.</br>*Cet événement s’applique uniquement aux types d’offres qui prennent en charge les plans privés*.    |
|Arrêt de la vente du plan    |Un utilisateur a interrompu la vente de l’offre en sélectionnant **Arrêter la vente** dans la page **Vue d’ensemble de l’offre**.    |

> [!NOTE]
> La page Historique n’indique pas quand un brouillon d’offre a été enregistré.

### <a name="filter-options"></a>Options de filtre

Vous pouvez utiliser des filtres pour réduire l’historique complet de votre offre à des événements de publication spécifiques :

1.    Sélectionnez le bouton de filtre situé dans l’angle supérieur droit de la page.
2.    Choisissez un filtre, puis sélectionnez **Appliquer** pour voir quels événements d’historique correspondent aux critères que vous avez sélectionnés.
3.    Sélectionnez **Effacer les filtres** pour revenir à l’historique complet de votre offre.

Il existe quatre filtres :
* Événements
* Utilisateurs
* Date
* Pages

Lorsque vous choisissez le filtre **Pages**, vous pouvez sélectionner n’importe quelle page d’Espace partenaires applicable à votre type d’offre. Lorsqu’il est appliqué, le filtre **Pages** affiche tous les événements **Offre soumise à la préversion** avec les modifications apportées à la page que vous avez sélectionnée.

* Pour toutes les offres, la page **Configuration de l’offre** est incluse par défaut pour chaque événement d’envoi.
* Pour les offres qui prennent en charge les plans, la page **Vue d’ensemble du plan** est incluse pour chaque événement d’envoi.
* Pour les offres qui prennent en charge les versions d’évaluation, la page **Version d’évaluation** est incluse pour chaque événement d’envoi.

### <a name="users"></a>Utilisateurs

Si un événement a été initié par un utilisateur, la page Historique indique l’utilisateur selon les scénarios suivants :

#### <a name="the-event-was-initiated-by-the-publisher"></a>L’événement a été initié par l’éditeur

Les utilisateurs ayant des autorisations de publication pour une offre verront leur nom affiché pour les événements de publication qu’ils initient.

[![The History page displays the name of users with publishing permissions.](./media/review-publish-offer/event-initiated-by-publisher.png)](./media/review-publish-offer/event-initiated-by-publisher.png#lightbox)

#### <a name="the-event-was-initiated-by-microsoft"></a>L’événement a été initié par Microsoft

Vous pouvez accorder des autorisations aux administrateurs Microsoft pour lancer des actions en votre nom ou prendre des actions correctives après une erreur système inattendue. Le nom et le logo de Microsoft s’affichent pour les événements de publication initiés par Microsoft en votre nom.

[![Example of how the History page displays events initiated by Microsoft.](./media/review-publish-offer/event-initiated-by-microsoft.png)](./media/review-publish-offer/event-initiated-by-microsoft.png#lightbox)

#### <a name="the-event-was-initiated-by-an-unidentified-user"></a>L’événement a été initié par un utilisateur non identifié

Les utilisateurs qui ne sont plus associés à un compte voient leur nom supprimé de la colonne **Initié par** après que leurs autorisations de publication leur ont été retirées.

[![Example of how the History page displays events initiated by unidentified users.](./media/review-publish-offer/event-initiated-by-unidentified-user.png)](./media/review-publish-offer/event-initiated-by-unidentified-user.png#lightbox)

#### <a name="validation-and-publishing-steps"></a>Étapes de validation et de publication

Aucun utilisateur n’est affiché pour les processus système qui correspondent aux [étapes de validation et de publication](#validation-and-publishing-steps). Ces événements sont codés par couleur en fonction de l’état d’achèvement de l’événement.

[![Example of how the History page displays validation and publishing steps.](./media/review-publish-offer/validation-and-publishing-event.png)](./media/review-publish-offer/validation-and-publishing-event.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

[Accéder aux rapports analytiques de la Place de marché commerciale dans l’Espace partenaires](partner-center-portal/analytics.md)