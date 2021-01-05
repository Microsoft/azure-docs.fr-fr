---
title: Organiser votre facture en fonction de vos besoins – Azure
description: Découvrez comment organiser les coûts sur votre facture. Vous pouvez personnaliser votre compte de facturation en créant des profils de facturation et des sections de facture.
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 6424fc0ff49566fad949b3fba4718acb2bad4cd3
ms.sourcegitcommit: d95cab0514dd0956c13b9d64d98fdae2bc3569a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91362767"
---
# <a name="organize-costs-by-customizing-your-billing-account"></a>Organiser les coûts en personnalisant votre compte de facturation

Votre compte de facturation pour le contrat client Microsoft vous permet d’organiser vos coûts en fonction de vos besoins, que ce soit par département, par projet ou par environnement de développement.

Cet article explique comment utiliser le Portail Azure pour organiser vos coûts. Il s’applique à un compte de facturation associé à un contrat client Microsoft. [Vérifiez que vous avez accès à un contrat client Microsoft](#check-access-to-a-microsoft-customer-agreement).

## <a name="structure-your-account-with-billing-profiles-and-invoice-sections"></a>Structurer votre compte à l’aide de profils de facturation et de sections de facture

Dans le compte de facturation d’un contrat client Microsoft, vous utilisez des profils de facturation et des sections de facture pour organiser vos coûts.

![Capture d’écran montrant la hiérarchie de facturation d’un contrat client Microsoft](./media/mca-section-invoice/mca-hierarchy.png)

### <a name="billing-profile"></a>Profil de facturation

Un profil de facturation représente une facture et les informations de facturation associées, telles que les modes de paiement et l’adresse de facturation. Une facture mensuelle est générée au début du mois pour chaque profil de facturation associé à votre compte. La facture contient les frais pour l’utilisation d’Azure et les autres achats du mois précédent.

Un profil de facturation est automatiquement créé en même temps que votre compte de facturation lorsque vous vous inscrivez à Azure. Vous pouvez créer des profils de facturation supplémentaires pour organiser vos coûts avec plusieurs factures mensuelles.

> [!IMPORTANT]
>
> La création de profils de facturation supplémentaires peut avoir un impact sur le coût global. Pour plus d’informations, consultez [Éléments à prendre en compte lors de l’ajout de nouveaux profils de facturation](#things-to-consider-when-adding-new-billing-profiles).

### <a name="invoice-section"></a>Section de facture

Une section de facture représente un regroupement de coûts dans votre facture. Une section de facture est créée automatiquement pour chaque profil de facturation dans votre compte. Vous pouvez créer des sections supplémentaires pour organiser vos coûts en fonction de vos besoins. Chaque section de facture s’affiche sur la facture avec les frais engagés pour le mois en vigueur.

L’image ci-dessous montre une facture avec deux sections de facture : Ingénierie et Marketing. Le résumé et le détail des frais pour chaque section s’affichent dans la facture. Les prix indiqués dans l’image sont fournis à titre d’exemple uniquement et ne représentent pas les prix réels des services Azure.

![Image présentant une facture avec des sections](./media/mca-section-invoice/mca-invoice-with-sections.png)

## <a name="billing-account-structure-for-common-scenarios"></a>Structure de compte de facturation pour les scénarios courants

Cette section décrit des scénarios courants pour organiser les coûts et les structures de compte de facturation correspondantes :

|Scénario  |Structure  |
|---------|---------|
|Jack s’inscrit à Azure et il a besoin d’une facture mensuelle unique. | Un profil de facturation et une section de facture. Cette structure est automatiquement configurée pour Jack lorsqu’il s’inscrit à Azure et elle ne nécessite aucune étape supplémentaire. |

![Graphique d’informations pour un scénario avec facturation simple.](./media/mca-section-invoice/organize-billing-scenario1.png)

|Scénario  |Structure  |
|---------|---------|
|Contoso est une petite entreprise qui a besoin d’une facture mensuelle unique, mais qui regroupe les coûts en fonction du service concerné : marketing ou ingénierie.  | Un profil de facturation pour Contoso et une section de facture pour chaque service marketing et ingénierie. |

![Graphique d’informations pour un scénario avec facturation multiple.](./media/mca-section-invoice/organize-billing-scenario2.png)

|Scénario  |Structure  |
|---------|---------|
|Fabrikam est une entreprise de taille moyenne nécessitant des factures distinctes pour ses services d’ingénierie et de marketing. Pour le service d’ingénierie, elle souhaite regrouper les coûts en fonction des environnements : production et développement.  | Un profil de facturation pour chaque service marketing et ingénierie. Pour le service ingénierie, une section de facture pour chaque environnement de production et de développement. |

![Graphique d’informations pour un scénario avec facturation multiple, avec facturation séparée pour les environnements de production et de développement.](./media/mca-section-invoice/organize-billing-scenario3.png)

## <a name="create-a-new-invoice-section"></a>Créer une nouvelle section de facture

Pour créer une section de facture, vous devez être **propriétaire de profil de facturation** ou **contributeur de profil de facturation**. Pour plus d’informations, consultez [Gérer les sections de facture associées au profil de facturation](understand-mca-roles.md#manage-invoice-sections-for-billing-profile).

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Recherchez **Gestion des coûts + facturation**.

   ![Capture d’écran montrant la recherche dans le portail pour gestion des coûts + facturation](./media/mca-section-invoice/search-cmb.png)

3. Sélectionnez **Profils de facturation** dans le volet gauche. Dans la liste, sélectionnez un profil de facturation. La nouvelle section s’affiche sur la facture du profil de facturation sélectionné.

   [![Capture d’écran montrant la liste des profils de facturation.](./media/mca-section-invoice/mca-select-profile.png)](./media/mca-section-invoice/mca-select-profile-zoomed-in.png#lightbox)

4. Sélectionnez **Sections de facture** dans le volet gauche, puis sélectionnez **Ajouter** en haut de la page.

   [![Capture d’écran montrant l’ajout de sections de facture](./media/mca-section-invoice/mca-list-invoice-sections.png)](./media/mca-section-invoice/mca-list-invoice-sections-zoomed-in.png#lightbox)

5. Entrez un nom pour la section de facture.

   [![Capture d’écran montrant la page de création de la section de facture](./media/mca-section-invoice/mca-create-invoice-section.png)](./media/mca-section-invoice/mca-create-invoice-section-zoomed-in.png#lightbox)

6. Sélectionnez **Create** (Créer).

## <a name="create-a-new-billing-profile"></a>Créer un nouveau profil de facturation

Pour créer un profil de facturation, vous devez être **propriétaire du compte de facturation** ou **collaborateur du compte de facturation**. Pour plus d’informations, consultez [Gérer les profils de facturation associés au compte de facturation](understand-mca-roles.md#manage-billing-profiles-for-billing-account).

> [!IMPORTANT]
>
> La création de profils de facturation supplémentaires peut avoir un impact sur le coût global. Pour plus d’informations, consultez [Éléments à prendre en compte lors de l’ajout de nouveaux profils de facturation](#things-to-consider-when-adding-new-billing-profiles).

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Recherchez **Gestion des coûts + facturation**.

   ![Capture d’écran montrant la recherche dans le portail pour gestion des coûts + facturation](./media/mca-section-invoice/search-cmb.png)

3. Sélectionnez **Profils de facturation** dans le volet gauche, puis sélectionnez **Ajouter** en haut de la page.

   [![Capture d’écran montrant la liste des profils de facturation, avec l’option Ajouter sélectionnée.](./media/mca-section-invoice/mca-list-profiles.png)](./media/mca-section-invoice/mca-list-profiles-zoomed-in.png#lightbox)

    > [!Note]
    >
    > Si vous ne voyez pas le bouton Ajouter dans la page Profil de facturation, la fonctionnalité n’est pas disponible pour votre compte. Actuellement, elle est disponible uniquement pour les comptes qui ont été configurés lors de l’utilisation d’un représentant Microsoft.

4. Remplissez le formulaire et cliquez sur **Créer**.

   [![Capture d’écran montrant la page de création du profil de facturation](./media/mca-section-invoice/mca-add-profile.png)](./media/mca-section-invoice/mca-add-profile-zoomed-in.png#lightbox)

    |Champ  |Définition  |
    |---------|---------|
    |Nom     | Nom d’affichage qui vous aide à identifier facilement le profil de facturation dans le Portail Azure.  |
    |Numéro de BDC    | Numéro de bon de commande facultatif. Le numéro de bon de commande s’affiche sur les factures générées pour le profil de facturation. |
    |Adresse de facturation   | L’adresse de facturation s’affiche sur les factures générées pour le profil de facturation. |
    |Facture par e-mail   | Cochez la case de facture par e-mail pour recevoir les factures pour ce profil de facturation par e-mail. Si vous ne choisissez pas cette option, vous pouvez afficher et télécharger les factures dans le Portail Azure.|

5. Sélectionnez **Create** (Créer).

## <a name="link-charges-to-invoice-sections-and-billing-profiles"></a>Lier des frais à des sections de facture et à des profils de facturation

Une fois que vous avez personnalisé votre compte de facturation en fonction de vos besoins, vous pouvez lier les abonnements et d’autres produits au profil de facturation et à la section de facture de votre choix.

### <a name="link-a-new-subscription"></a>Lier un nouvel abonnement

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Recherchez **Abonnements**.

   [![Capture d’écran montrant la recherche d’abonnement dans le Portail Azure.](./media/mca-section-invoice/search-subscriptions.png)](./media/mca-section-invoice/search-subscriptions.png#lightbox)

3. En haut de la page, sélectionnez **Ajouter**.

   ![Capture d’écran montrant le bouton Ajouter dans la vue Abonnements pour un nouvel abonnement.](./media/mca-section-invoice/subscription-add.png)

4. Si vous avez accès à plusieurs comptes de facturation, sélectionnez le compte de facturation de votre contrat client Microsoft.

   ![Capture d’écran montrant le volet Créer un abonnement.](./media/mca-section-invoice/mca-create-azure-subscription.png)

5. Sélectionnez le profil de facturation sur lequel l’utilisation de l’abonnement sera facturé. Les frais d’utilisation d’Azure et d’autres achats pour cet abonnement seront portés sur la facture du profil de facturation sélectionné.

6. Sélectionnez la section de facture pour lier les frais de l’abonnement. Les frais s’affichent sous cette section sur la facture du profil de facturation.

7. Sélectionnez un plan Azure et entrez un nom convivial pour votre abonnement.

9. Cliquez sur **Créer**.  

### <a name="link-existing-subscriptions-and-products"></a>Lier des abonnements et des produits existants

Si vous avez des abonnements Azure existants ou d’autres produits tels que la Place de marché Azure et les ressources App Source, vous pouvez les déplacer de leur section de facture existante vers une autre section de facture pour réorganiser vos coûts.

> [!IMPORTANT]
>
> Les abonnements et autres produits peuvent uniquement être déplacés entre les sections de facture qui appartiennent au même profil de facturation. Le déplacement d’abonnements et de produits entre les sections de facture dans différents profils de facturation n’est pas pris en charge.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Recherchez **Gestion des coûts + facturation**.

   ![Capture d’écran montrant la recherche de « gestion des coûts + facturation » dans le Portail Azure.](./media/mca-section-invoice/search-cmb.png)

3. Pour lier un abonnement à une nouvelle section de facture, sélectionnez **Abonnements Azure** dans la partie gauche de l’écran. Pour d’autres produits tels que la Place de marché Azure et les ressources App Source, sélectionnez **Charges périodiques**.

   [![Capture d’écran montrant l’option de modification de la section de facture](./media/mca-section-invoice/mca-select-change-invoice-section.png)](./media/mca-section-invoice/mca-select-change-invoice-section.png#lightbox)

4. Dans la page, cliquez sur les points de suspension (trois points) pour l’abonnement ou le produit que vous souhaitez lier à une nouvelle section de facture. Sélectionnez **Modifier la section de facture**.

5. Sélectionnez la nouvelle section de facture dans la liste déroulante. La liste déroulante affiche uniquement les sections de facture associées au même profil de facturation que la section de facture existante.

    [![Capture d’écran montrant comment sélectionner une nouvelle section de facture](./media/mca-section-invoice/mca-select-new-invoice-section.png)](./media/mca-section-invoice/mca-select-new-invoice-section-zoomed-in.png#lightbox)

6. Sélectionnez **Enregistrer**.

## <a name="things-to-consider-when-adding-new-billing-profiles"></a>Éléments à prendre en compte lors de l’ajout de nouveaux profils de facturation

### <a name="azure-usage-charges-may-be-impacted"></a>Les frais d’utilisation d’Azure peuvent être affectés

Dans votre compte de facturation d’un contrat client Microsoft, l’utilisation d’Azure est agrégée chaque mois pour chaque profil de facturation. Pour les ressources Azure avec une tarification hiérarchisée, les prix sont déterminés en fonction de l’utilisation pour chaque profil de facturation séparément. L’utilisation n’est pas agrégée entre les profils de facturation lors du calcul du prix. Cela peut avoir un impact sur le coût global de l’utilisation d’Azure pour les comptes avec plusieurs profils de facturation.

Examinons un exemple illustrant la façon dont les coûts varient pour deux scénarios. Les prix utilisés dans ces scénarios sont fournis à titre d’exemple uniquement et ne représentent pas les prix réels des services Azure.

#### <a name="you-only-have-one-billing-profile"></a>Vous n’avez qu’un seul profil de facturation.

Supposons que vous utilisez le stockage d’objets blob de blocs Azure, qui coûte 0,00184 USD par Go pour les 50 premiers téraoctets (To), puis 0,00177 USD par Go pour les 450 téraoctets (To) suivants. Vous avez utilisé 100 To dans les abonnements qui sont facturés à votre profil de facturation. Voici les montants qui vous seraient facturés.

|  Niveau de tarification (USD) |Quantité | Montant (USD)|
|---------|---------|---------|
|1,84 par To pour les 50 premiers To/mois    | 50 To        | 92,0   |
|1,77 par To pour les 450 To suivants/mois    |  50 To         | 88,5   |
|Total     |     100 To  | 180,5

Le montant total des frais pour l’utilisation de 100 To de données dans ce scénario est de **180,5** USD.

#### <a name="you-have-multiple-billing-profiles"></a>Vous avez plusieurs profils de facturation.

À présent, supposons que vous avez créé un autre profil de facturation et utilisé 50 Go via des abonnements facturés sur le premier profil de facturation et 50 Go via des abonnements facturés sur le second profil de facturation. Voici les montants qui vous seraient facturés.

`Charges for the first billing profile`

|  Niveau de tarification (USD) |Quantité | Montant (USD)|
|---------|---------|---------|
|1,84 par To pour les 50 premiers To/mois    | 50 To        | 92,0  |
|1,77 par To pour les 450 To suivants/mois    |  0 To         | 0.0  |
|Total     |     50 To  | 92,0

`Charges for the second billing profile`

|  Niveau de tarification (USD) |Quantité | Montant (USD)|
|---------|---------|---------|
|1,84 par To pour les 50 premiers To/mois    | 50 To        | 92,0  |
|1,77 par To pour les 450 To suivants/mois    |  0 To         | 0.0  |
|Total     |     50 To  | 92,0

Le montant total pour l’utilisation de 100 To de données dans ce scénario est de **184,0** (92,0 * 2) USD.

### <a name="azure-reservation-benefits-might-not-apply-to-all-subscriptions"></a>Les avantages des réservations Azure peuvent ne pas s’appliquer à tous les abonnements

Les réservations Azure avec étendue partagée s’appliquent aux abonnements dans un profil de facturation unique et ne sont pas partagées entre les profils de facturation.

![Graphique d’informations pour l’application d’une réservation pour différentes structures de compte de facturation](./media/mca-section-invoice/mca-reservations-benefits-by-bg.png)

Dans l’image ci-dessus, Contoso a deux abonnements. L’avantage de la réservation Azure s’applique différemment en fonction de la structure du compte de facturation. Dans le scénario de gauche, l’avantage de la réservation s’applique aux deux abonnements facturés sur le profil de facturation d’ingénierie. Dans le scénario de droite, l’avantage de la réservation s’applique uniquement à l’abonnement 1, car il s’agit du seul abonnement facturé sur le profil de facturation d’ingénierie.

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Vérifier l'accès à un contrat client Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez toujours besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.

## <a name="next-steps"></a>Étapes suivantes

- [Créer un abonnement Azure associé dans le cadre de votre Contrat client Microsoft](create-subscription.md)
- [Gérer les rôles de facturation dans le portail Azure](understand-mca-roles.md#manage-billing-roles-in-the-azure-portal)
- [Obtenir la propriété de facturation des abonnements des utilisateurs d’autres comptes de facturation](mca-request-billing-ownership.md)
