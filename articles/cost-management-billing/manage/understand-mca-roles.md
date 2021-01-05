---
title: Rôles de facturation pour des Contrats clients Microsoft - Azure
description: Découvrez-en plus sur les rôles de facturation des comptes de facturation Azure dans le cadre des Contrats client Microsoft.
author: amberbhargava
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: f0ace05d6a4df65282bd56cca8e7895b0dc24edf
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88686545"
---
# <a name="understand-microsoft-customer-agreement-administrative-roles-in-azure"></a>Présentation des rôles d'administrateur Azure dans le cadre des Contrats client Microsoft

Pour gérer votre compte de facturation dans le cadre d'un Contrat client Microsoft, utilisez les rôles décrits dans les sections suivantes. Ces rôles s'ajoutent aux rôles intégrés dont Azure doit contrôler l'accès aux ressources. Pour plus d’informations, voir [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md).

Cet article s'applique à un compte de facturation associé à un contrat client Microsoft. Vérifiez que vous avez accès à un Contrat client Microsoft.

## <a name="billing-role-definitions"></a>Définitions des rôles de facturation

Le tableau suivant décrit les rôles de facturation que vous utilisez pour gérer votre compte de facturation, vos profils de facturation et vos sections de facture.

|Role|Description|
|---|---|
|Propriétaire du compte de facturation|Gérer tout le contenu du compte de facturation|
|Collaborateur du compte de facturation|Gérer tout le contenu du compte de facturation, à l'exception des autorisations|
|Lecteur du compte de facturation|Affichage en lecture seule de tout ce qui se trouve sur le compte de facturation|
|Propriétaire du profil de facturation|Gérer tout le contenu du profil de facturation|
|Collaborateur du profil de facturation|Gérer tout le contenu du profil de facturation, à l'exception des autorisations|
|Lecteur du profil de facturation|Affichage en lecture seule de tout ce qui se trouve sur le profil de facturation|
|Gestionnaire de factures|Consulter et payer les factures pour le profil de facturation|
|Propriétaire de section de facture|Gérer tout le contenu de la section de facture|
|Contributeur de section de facture|Gérer tout le contenu de la section de facture, à l'exception des autorisations|
|Lecteur de section de facture|Affichage en lecture seule de tout le contenu de la section de facture|
|Créateur de l'abonnement Azure|Créer des abonnements Azure|

## <a name="billing-account-roles-and-tasks"></a>Rôles et tâches liés au compte de facturation

Un compte de facturation vous permet de gérer la facturation de votre organisation. Vous utilisez un compte de facturation pour organiser les coûts, superviser les frais et les factures, et contrôler l'accès à la facturation pour votre organisation. Pour plus d'informations, consultez [Présentation du compte de facturation](../understand/mca-overview.md#your-billing-account).

Les tableaux suivants indiquent le rôle dont vous avez besoin pour accomplir des tâches dans le contexte du compte de facturation.

### <a name="manage-billing-account-permissions-and-properties"></a>Gérer les autorisations et les propriétés du compte de facturation

|Tâche|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation|
|---|---|---|---|
|Afficher les autorisations existantes associées au compte de facturation|✔|✔|✔|
|Autoriser d'autres utilisateurs à consulter et gérer le compte de facturation|✔|✘|✘|
|Afficher les propriétés du compte de facturation, comme le nom de la société, son adresse, etc.|✔|✔|✔|

### <a name="manage-billing-profiles-for-billing-account"></a>Gérer les profils de facturation associés au compte de facturation

|Tâche|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation|
|---|---|---|---|
|Afficher tous les profils de facturation du compte|✔|✔|✔|

### <a name="manage-invoices-for-billing-account"></a>Gérer les factures liées au compte de facturation

|Tâche|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation|
|---|---|---|---|
|Afficher toutes les factures du compte|✔|✔|✔|
|Télécharger les factures, les fichiers d'utilisation et de frais Azure, les grilles tarifaires et les documents fiscaux du compte.|✔|✔|✔|

### <a name="manage-invoice-sections-for-billing-account"></a>Gérer les sections de facture associées au compte de facturation

|Tâche|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation|
|---|---|---|---|
|Afficher toutes les sections de facture du compte|✔|✔|✔|

### <a name="manage-transactions-for-billing-account"></a>Gérer les transactions associées au compte de facturation

|Tâche|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation|
|---|---|---|---|
|Afficher toutes les transactions de facturation associées au compte|✔|✔|✔|
|Afficher tous les produits achetés pour le compte|✔|✔|✔|

### <a name="manage-subscriptions-for-billing-account"></a>Gérer les abonnements associés au compte de facturation

|Tâche|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation|
|---|---|---|---|
|Afficher tous les abonnements Azure du compte de facturation|✔|✔|✔|

## <a name="billing-profile-roles-and-tasks"></a>Rôles et tâches liés au profil de facturation

Un profil de facturation vous permet de gérer vos factures et vos modes de paiement. Une facture mensuelle est générée pour les abonnements Azure et autres produits achetés par le biais du profil de facturation. Les modes de paiement vous permettent de payer la facture. Pour plus d'informations, consultez [Présentation des profils de facturation](../understand/mca-overview.md#billing-profiles).

Les tableaux suivants indiquent le rôle dont vous avez besoin pour accomplir des tâches dans le contexte du profil de facturation.

### <a name="manage-billing-profile-permissions-properties-and-policies"></a>Gérer les autorisations, les propriétés et les stratégies des profils de facturation

|Tâche|Propriétaire du profil de facturation|Collaborateur du profil de facturation|Lecteur du profil de facturation|Gestionnaire de factures|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation
|---|---|---|---|---|---|---|---|
|Afficher les autorisations existantes associées au profil de facturation|✔|✔|✔|✔|✔|✔|✔|
|Autoriser d'autres utilisateurs à consulter et gérer le profil de facturation|✔|✘|✘|✘|✘|✘|✘|
|Afficher les propriétés du profil de facturation, comme le numéro du bon de commande, la préférence de facturation par e-mail, etc.|✔|✔|✔|✔|✔|✔|✔|
|Mettre à jour les propriétés du profil de facturation |✔|✔|✘|✘|✘|✘|✘|
|Afficher les stratégies appliquées au profil de facturation, comme l'activation des achats de réservations Azure, des achats sur la Place de marché Azure, etc.|✔|✔|✔|✔|✔|✔|✔|
|Appliquer des stratégies au profil de facturation |✔|✔|✘|✘|✘|✘|✘|

### <a name="manage-invoices-for-billing-profile"></a>Gérer les factures associées au profil de facturation

|Tâche|Propriétaire du profil de facturation|Collaborateur du profil de facturation|Lecteur du profil de facturation|Gestionnaire de factures|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation
|---|---|---|---|---|---|---|---|
|Afficher toutes les factures associées au profil de facturation|✔|✔|✔|✔|✔|✔|✔|
|Télécharger les factures, les fichiers d'utilisation et de frais Azure, les grilles tarifaires et les documents fiscaux associés au profil de facturation|✔|✔|✔|✔|✔|✔|✔|

### <a name="manage-invoice-sections-for-billing-profile"></a>Gérer les sections de facture associées au profil de facturation

|Tâche|Propriétaire du profil de facturation|Collaborateur du profil de facturation|Lecteur du profil de facturation|Gestionnaire de factures|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation
|---|---|---|---|---|---|---|---|
|Afficher toutes les sections de facture associées au profil de facturation|✔|✔|✔|✔|✔|✔|✔|
|Créer une nouvelle section de facture pour le profil de facturation|✔|✔|✘|✘|✘|✘|✘|

### <a name="manage-transactions-for-billing-profile"></a>Gérer les transactions associées au profil de facturation

|Tâche|Propriétaire du profil de facturation|Collaborateur du profil de facturation|Lecteur du profil de facturation|Gestionnaire de factures|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation
|---|---|---|---|---|---|---|---|
|Afficher toutes les transactions de facturation associées au profil de facturation|✔|✔|✔|✔|✔|✔|✔|

### <a name="manage-payment-methods-for-billing-profile"></a>Gérer les modes de paiement associés au profil de facturation

|Tâche|Propriétaire du profil de facturation|Collaborateur du profil de facturation|Lecteur du profil de facturation|Gestionnaire de factures|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation
|---|---|---|---|---|---|---|---|
|Afficher les modes de paiement associés au profil de facturation|✔|✔|✔|✔|✔|✔|✔|
|Suivre le solde de crédits Azure du profil de facturation|✔|✔|✔|✔|✔|✔|✔|

### <a name="manage-subscriptions-for-billing-profile"></a>Gérer les abonnements associés au profil de facturation

|Tâche|Propriétaire du profil de facturation|Collaborateur du profil de facturation|Lecteur du profil de facturation|Gestionnaire de factures|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation
|---|---|---|---|---|---|---|---|
|Afficher tous les abonnements Azure associés au profil de facturation|✔|✔|✔|✔|✔|✔|✔|

## <a name="invoice-section-roles-and-tasks"></a>Rôles et tâches liés aux sections de facture

Une section de facture vous permet d'organiser les coûts sur votre facture. Vous pouvez créer une section pour organiser vos coûts par service, par environnement de développement, ou en fonction des besoins de votre organisation. Autorisez d'autres utilisateurs à créer des abonnements Azure pour la section. Les frais d'utilisation et les achats liés aux abonnements apparaissent ensuite dans la section de la facture. Pour plus d'informations, consultez [Présentation des sections de facture](../understand/mca-overview.md#invoice-sections).

Les tableaux suivants indiquent le rôle dont vous avez besoin pour accomplir des tâches dans le contexte des sections de facture.

### <a name="manage-invoice-section-permissions-and-properties"></a>Gérer les autorisations et les propriétés des sections de facture

|Tâches|Propriétaire de section de facture|Contributeur de section de facture|Lecteur de section de facture|Créateur de l'abonnement Azure|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation |
|---|---|---|---|---|---|---|---|
|Afficher toutes les autorisations associées à la section de facture|✔|✔|✔|✔|✔|✔|✔|
|Autoriser d'autres utilisateurs à consulter et gérer la section de facture|✔|✘|✘|✘|✘|✘|✘|
|Afficher les propriétés de la section de facture|✔|✔|✔|✔|✔|✔|✔|
|Mettre à jour les propriétés de la section de facture|✔|✔|✘|✘|✘|✘|✘|

### <a name="manage-products-for-invoice-section"></a>Gérer les produits associés à la section de facture

|Tâches|Propriétaire de section de facture|Contributeur de section de facture|Lecteur de section de facture|Créateur de l'abonnement Azure|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation
|---|---|---|---|---|---|---|---|
|Afficher tous les produits achetés dans la section de facture|✔|✔|✔|✘|✔|✔|✔|
|Gérer la facturation des produits de la section de facture (annuler, désactiver le renouvellement automatique, etc.)|✔|✔|✘|✘|✘|✘|✘|
|Modifier la section de facture associée aux produits|✔|✔|✘|✘|✘|✘|✘|

### <a name="manage-subscriptions-for-invoice-section"></a>Gérer les abonnements associés à la section de facture

|Tâches|Propriétaire de section de facture|Contributeur de section de facture|Lecteur de section de facture|Créateur de l'abonnement Azure|Propriétaire du compte de facturation|Collaborateur du compte de facturation|Lecteur du compte de facturation
|---|---|---|---|---|---|---|---|
|Afficher tous les abonnements Azure associés à la section de facture|✔|✔|✔|✘|✔|✔|✔|
|Modifier la section de facture associée aux abonnements|✔|✔|✘|✘|✘|✘|✘|
|Demander la propriété de facturation sur les abonnements d'utilisateurs d'autres comptes de facturation|✔|✔|✘|✘|✘|✘|✘|

## <a name="subscription-billing-roles-and-tasks"></a>Rôles et tâches liés à la facturation des abonnements

Les tableaux suivants indiquent le rôle dont vous avez besoin pour accomplir des tâches dans le contexte d'un abonnement.

|Tâches|Propriétaire de section de facture|Contributeur de section de facture|Lecteur de section de facture|Créateur de l'abonnement Azure|
|---|---|---|---|---|
|Créer des abonnements Azure|✔|✔|✘|✔|
|Mettre à jour le centre de coûts associé à l'abonnement|✔|✔|✘|✘|
|Modifier la section de facture associée à l'abonnement|✔|✔|✘|✘|

## <a name="manage-billing-roles-in-the-azure-portal"></a>Gérer les rôles de facturation sur le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Recherchez **Gestion des coûts + facturation**.

   ![Capture d’écran montrant la recherche dans le portail Azure](./media/understand-mca-roles/billing-search-cost-management-billing.png)

3. Sélectionnez **Contrôle d'accès (IAM)** dans une étendue, comme un compte de facturation, un profil de facturation ou une section de facture, à laquelle vous souhaitez accorder l'accès.

4. La page Contrôle d'accès (IAM) répertorie les utilisateurs et les groupes attribués à chaque rôle pour cette étendue.

   ![Capture d'écran présentant la liste des administrateurs du compte de facturation](./media/understand-mca-roles/billing-list-admins.png)

5. Pour accorder l'accès à un utilisateur, sélectionnez **Ajouter** en haut de la page. Dans la liste déroulante Rôle, sélectionnez un rôle. Entrez l’adresse e-mail de l’utilisateur auquel vous souhaitez accorder l’accès. Sélectionnez **Enregistrer** pour attribuer le rôle.

   ![Capture d'écran illustrant l'ajout d'un administrateur à un compte de facturation](./media/understand-mca-roles/billing-add-admin.png)

6. Pour supprimer l'accès d'un utilisateur, sélectionnez l'utilisateur doté du rôle que vous souhaitez supprimer. Sélectionnez Supprimer.

   ![Capture d'écran illustrant la suppression d'un administrateur d'un compte de facturation](./media/understand-mca-roles/billing-remove-admin.png)

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Vérifier l'accès à un contrat client Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique
Si vous avez toujours besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur votre compte de facturation :

- [Prise en main de votre compte de facturation dans le cadre d'un Contrat client Microsoft](../understand/mca-overview.md)
- [Créer un abonnement Azure associé à votre compte de facturation dans le cadre d'un Contrat client Microsoft](create-subscription.md)
