---
title: Migrer des solutions de co-vente de GTM OCP vers Espace partenaires pour Microsoft AppSource
description: Découvrez comment migrer des solutions de co-vente de GTM OCP vers Espace partenaires pour Microsoft AppSource.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: vamahtan
ms.author: vamahtan
ms.date: 12/07/2020
ms.openlocfilehash: 6ad58283ad9078088f241a67426657eb7a538e10
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96781128"
---
# <a name="migration-of-co-sell-solutions-from-ocp-gtm-to-the-commercial-marketplace"></a>Migration de solutions de co-vente de GTM OCP vers le marketplace commercial

Microsoft déplace l’expérience de publication. Le [marketplace commercial](overview.md) propose une publication simplifiée des offres pour la co-vente via les trois canaux de Microsoft en centralisant la création et la gestion des offres dans Espace partenaires, où vous gérez déjà votre relation avec Microsoft.

En tant que partenaire Microsoft inscrit sur le marketplace commercial, vous pouvez :

- publier vos offres de manière centralisée et co-vendre sur les canaux de clients directs, de partenaires et de vendeurs de Microsoft ;
- vous assurer que vos offres se trouvent dans la bonne boutique en ligne ([Microsoft AppSource](https://appsource.microsoft.com) ou [Place de marché Azure](https://azure.microsoft.com)) pour atteindre les millions de clients du cloud qui s’alignent sur les capacités de votre offre ;
- mener votre propre expérience de publication pour co-vendre les offres qui correspondent à vos objectifs commerciaux ;
- aligner la publication de votre offre sur Espace partenaires, où vous gérez déjà votre relation avec Microsoft et les possibilités de co-ventes ;
- déverrouiller [Marketplace Rewards](partner-center-portal/marketplace-rewards.md).

## <a name="prerequisites-to-continue-co-selling-with-microsoft"></a>Conditions préalables pour continuer à co-vendre avec Microsoft

Vérifiez que vous êtes un membre actif du programme Microsoft Partner Network et que vous êtes inscrit au marketplace commercial dans Espace partenaires.

- Rejoignez le programme Microsoft Partner Network [gratuitement](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership). En tant que partenaire, vous aurez accès à des ressources, programmes, outils et connexions exclusifs pour développer votre activité.
- Si vous n’avez pas de compte sur le marketplace commercial, [inscrivez-vous dès maintenant](partner-center-portal/create-account.md) pour continuer à co-vendre avec Microsoft et accéder à l’expérience complète de publication.

## <a name="publishing-updates-for-attaining-co-sell-ready-status"></a>Publication de mises à jour pour obtenir l’état co-sell-ready

Pour que votre solution soit détectable par les vendeurs et les partenaires de Microsoft, elle doit respecter les [exigences de la co-vente](marketplace-co-sell.md). Pour qu’un vendeur Microsoft soit encouragés, votre solution doit répondre aux [exigences d’incitation](marketplace-co-sell.md). Remplissez ces conditions dans l’onglet Co-vente d’Espace partenaires (voir [cette image](#cosell-tab) plus loin dans cet article).

> [!NOTE]
> Sur le marketplace commercial, vos solutions sont appelées « offres » tout au long de l’expérience de publication.

Une fois inscrit sur le marketplace commercial, préparez-vous à migrer vos solutions à partir de GTM OCP.

Procédez comme suit avant d’importer vos solutions à partir de GTM OCP :

1. Accédez à la [liste des éditeurs](https://partner.microsoft.com/dashboard/account/v3/publishers/list) de votre entreprise. Elle comprend le propriétaire du compte, les managers et les développeurs qui ont accès à la publication. En savoir plus sur les [rôles d’utilisateur d’Espace partenaires](/azure/marketplace/partner-center-portal/manage-account#define-user-roles-and-permissions).
2. Demandez à l’un des contacts de la liste d’[ajouter des utilisateurs](https://partner.microsoft.com/dashboard/account/usermanagement) au marketplace commercial en tant que *managers* ou *développeurs*, car seuls ces rôles peuvent modifier et publier des solutions.
3. Collaborez avec vos développeurs pour déplacer vos solutions de votre compte GTM OCP au marketplace commercial.
4. Choisissez l’une des opérations suivantes :
    1. Fusionnez cette solution avec une offre similaire du marketplace commercial.
    1. Migrez cette solution de GTM OCP vers le marketplace commercial.
    1. Abandonnez cette solution.

## <a name="migrate-your-solutions-from-ocp-gtm"></a>Migrer vos solutions à partir de GTM OCP

1. Commencez la migration [ici](https://partner.microsoft.com/solutions/migration#).
2. Sélectionnez la page **Vue d’ensemble**, puis **Cliquez ici pour commencer**.

    :::image type="content" source="media/co-sell-migrate/migration-overview.png" alt-text="Page Vue d’ensemble d’Espace partenaires, onglet Vue d’ensemble.":::

3. Pour commencer la migration, sélectionnez l’onglet **Solutions**, qui affiche toutes les solutions associées à vos ID MPN.

    :::image type="content" source="media/co-sell-migrate/solutions-tab.png" alt-text="Page Vue d’ensemble d’Espace partenaires, onglet Solutions.":::

    > [!NOTE]
    > Cet onglet indique si aucune solution n’est en attente de migration vers le marketplace commercial. Pour continuer à co-vendre avec Microsoft, assurez-vous que les solutions migrées sont publiées sur le marketplace commercial.

    Pour en savoir plus sur l’état de la solution, consultez les info-bulles. Toutes les solutions en attente d’action sont répertoriées sous **Action**.<a name="beginmigration"></a>

4. Sélectionnez **Commencer la migration** (voir l’image ci-dessus) pour la solution que vous souhaitez migrer, puis sélectionnez l’une des options suivantes :

    :::image type="content" source="media/co-sell-migrate/migration-options.png" alt-text="Trois options pour la migration.":::

### <a name="merge-solution-with-a-similar-offer"></a>Fusionner la solution avec une offre similaire

Sélectionnez cette option si la solution est déjà publiée sur le marketplace commercial et que les deux doivent être fusionnées en une seule offre. Cela permet d’éviter de créer des offres en double.

1. Identifiez l’offre existante.
    1. Sélectionnez **Je souhaite fusionner cette solution avec une offre similaire sur le marketplace commercial** (voir l’image **Action requise** [ci-dessus](#beginmigration)).
    1. L’onglet **Action 1** affiche les offres actives sur le marketplace commercial auxquelles votre solution GTM OCP peut être associée. Sélectionnez l’offre active dans la liste, si vous en avez une. S’il n’existe aucune liste d’offres dans laquelle choisir, entrez l’adresse orientée client (URL) de Microsoft AppSource ou de Place de marché Azure.
        [![Onglet Action 1 du processus de fusion.](media/co-sell-migrate/action-1-merge.png)](media/co-sell-migrate/action-1-merge.png#lightbox)
    1. Sélectionnez **Continuer**.
1. Demandez la fusion.
    1. L’onglet **Action 2** indique les instructions pour demander la fusion de votre solution GTM OCP avec celle que vous avez identifiée. Pour demander la fusion, sélectionnez **Enregistrer et contacter le support**, ce qui ouvre la page de support du partenaire dans un navigateur.
    1. Sélectionnez **Fournir les détails du problème** et entrez les informations suivantes :      [![Onglet Action 2 du processus de fusion.](media/co-sell-migrate/action-2-merge.png)](media/co-sell-migrate/action-2-merge.png#lightbox)
    1. Sélectionnez **Envoyer**. Le support technique du partenaire vous contactera dans les deux jours ouvrables.
    1. Le support du partenaire travaillera avec vous pour garantir la réussite de la fusion de cette offre, afin qu’elle soit publiée en tant qu’offre active.

### <a name="migrate-this-solution-from-ocp-gtm"></a>Migrer cette solution à partir de GTM OCP

Sélectionnez cette option si vous avez une solution GTM OCP qui n’a pas encore d’offre correspondante publiée sur le marketplace commercial. Vous devrez publier cette solution sur le marketplace commercial pour poursuivre la co-vente avec Microsoft, et la migration de cette solution vous permettra de gagner du temps en conservant les informations et la nomenclature de GTM OCP. Cette option vous oblige à sélectionner un type d’offre.

1. Sélectionnez **Je souhaite migrer cette solution de GTM OCP vers le marketplace commercial** (voir l’image **Action requise** [ci-dessus](#beginmigration)), puis **Continuer**.
1. Sous l’onglet **Action 1**, sélectionnez le [type d’offre](publisher-guide-by-offer-type.md), puis **Continuer**.

    [![Onglet Action 1 du processus de migration.](media/co-sell-migrate/action-1-migrate.png)](media/co-sell-migrate/action-1-migrate.png#lightbox)

1. Sous l’onglet **Action 2**, sélectionnez le [profil de l’éditeur](partner-center-portal/create-account.md) dans la liste fournie. Si vous n’avez pas de compte d’éditeur, créez-en un dans [Espace partenaires](https://partner.microsoft.com/solutions/migration), et sélectionnez-le ici.

    [![Onglet Action 2 du processus de migration.](media/co-sell-migrate/action-2-migrate.png)](media/co-sell-migrate/action-2-migrate.png#lightbox)

1. Sélectionnez **Créer un brouillon** pour migrer votre solution vers le marketplace commercial en tant que brouillon. Elle ne sera pas encore active.
1. Poursuivez le processus de publication dans Espace partenaires. Pour obtenir de l’aide sur la publication dans Espace partenaires, consultez [Rendre votre offre active sur le marketplace](#make-your-offer-live-in-the-marketplace) ci-dessous.

### <a name="discard-this-solution"></a>Abandonner cette solution

Sélectionnez cette option lorsqu’une solution parmi les solutions GTM OCP n’est plus pertinente. Il vous sera demandé de confirmer l’abandon, et vous pourrez également l’annuler ultérieurement.

1. Sélectionnez **Je souhaite abandonner cette solution** (voir l’image **Action requise** [ci-dessus](#beginmigration)), puis **Continuer**.
2. Sélectionnez **Abandonner**.

    :::image type="content" source="media/co-sell-migrate/migration-discard.png" alt-text="Confirmer l’abandon.":::

3. Pour annuler l’abandon, sélectionnez **Annuler l’abandon**.

    :::image type="content" source="media/co-sell-migrate/migration-discard-undo.png" alt-text="Lien Annuler l’abandon.":::

4. Si vous avez besoin d’une aide supplémentaire, sélectionnez l’onglet **Obtenir de l’aide** pour contacter le support technique du partenaire.

    :::image type="content" source="media/co-sell-migrate/get-support-link.png" alt-text="Lien du support sous l’onglet Obtenir de l’aide.":::

## <a name="make-your-offer-live-in-the-marketplace"></a>Rendre votre offre active sur le marketplace

Si vous avez choisi de migrer votre offre vers le marketplace commercial, celle-ci arrivera en tant que brouillon. Vous devez encore publier votre offre pour la rendre active sur le marketplace commercial. Cela permet de conserver l’état de co-vente, les incitations et le pipeline de référence.

Pour obtenir des instructions détaillées sur les informations que vous devez fournir avant de pouvoir publier votre offre, lisez le [guide de publication](publisher-guide-by-offer-type.md) approprié. Pour obtenir un résumé, lisez ce qui suit.

1. Dans la page **Vue d’ensemble** d’Espace partenaires, sélectionnez le nom de votre brouillon.

    :::image type="content" source="media/co-sell-migrate/offer-overview.png" alt-text="Page Vue d’ensemble de l’offre dans Espace partenaires.":::

<a name="cosell-tab"></a>

2. Renseignez les informations requises dans chaque onglet. Si vous le souhaitez, renseignez la page **Revendre via des CSP** (dans le menu de navigation de gauche ci-dessous) pour revendre par le biais du programme Fournisseur de solutions Cloud (CSP). Les liens **En savoir plus** et des info-bulles vous guident dans les spécifications et les détails.

    :::image type="content" source="media/co-sell-migrate/offer-setup.png" alt-text="Onglets de configuration de la vue d’ensemble de l’offre dans Espace partenaires.":::

3. Certaines des informations relatives aux vendeurs Microsoft ont été copiées à partir de la solution GTM OCP. Renseignez les informations requises restantes dans l’onglet **Co-vendre avec Microsoft** pour que votre offre soit prête pour la co-vente. Lorsque vous avez terminé, sélectionnez **Vérifier et publier**.

    :::image type="content" source="media/co-sell-migrate/co-sell-page.png" alt-text="Onglet Co-vente de l’offre dans Espace partenaires.":::

4. Après avoir examiné toutes les informations envoyées, sélectionnez **Publier** pour soumettre votre ébauche d’offre à l’examen de certification.

    :::image type="content" source="media/co-sell-migrate/co-sell-review-publish.png" alt-text="Onglet Révision de l’offre et bouton Publier dans Espace partenaires.":::

5. Suivez l’état de votre envoi dans l’onglet **Vue d’ensemble**.

    :::image type="content" source="media/publish-status-publisher-signoff.png" alt-text="Barre d’état de la publication de l’offre sous l’onglet Vue d’ensemble dans Espace partenaires.":::

6. Nous vous informerons lorsque l’examen de certification sera terminé. Si nous fournissons des commentaires actionnables, résolvez-les, puis sélectionnez **Publier** pour lancer une nouvelle certification.
7. Une fois que votre offre est certifiée, affichez un aperçu de l’offre à l’aide du lien fourni et effectuez vos ajustements finaux. Quand vous êtes prêt, sélectionnez **Mettre en ligne** (voir le bouton ci-dessus) pour publier votre offre sur la ou les vitrines pertinentes du marketplace commercial. Une fois que votre offre est en ligne, elle conserve l’état de co-vente de la solution GTM OCP d’origine.

## <a name="next-steps"></a>Étapes suivantes

- [Revendre via des partenaires CSP](cloud-solution-providers.md)
- Consulter ces [FAQ](https://partner.microsoft.com/resources/detail/co-sell-requirements-publish-commercial-marketplace-faq-pdf) (PDF)
