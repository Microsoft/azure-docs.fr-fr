---
title: Administration du portail Azure EA
description: Cet article présente les tâches courantes à la charge d’un administrateur dans le portail Azure EA.
author: bandersmsft
ms.author: banders
ms.date: 11/13/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: enterprise
ms.reviewer: boalcsva
ms.custom: contperf-fy21q1
ms.openlocfilehash: e25b2aeee46617bd62a72d204e1a4bdbc31d08ea
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030822"
---
# <a name="azure-ea-portal-administration"></a>Administration du portail Azure EA

Cet article présente les tâches courantes à la charge d’un administrateur dans le portail Azure EA (https://ea.azure.com). Le portail Azure EA est un portail de gestion en ligne qui permet aux clients de gérer le coût de leurs services Azure EA. Pour obtenir des informations préliminaires sur le portail Azure EA, consultez l’article [Bien démarrer avec le portail Azure EA](ea-portal-get-started.md).

## <a name="activate-your-enrollment"></a>Activer votre inscription

Pour activer votre service, l’administrateur d’entreprise initial ouvre le [portail Azure Enterprise](https://ea.azure.com) et se connecte en utilisant l’adresse e-mail indiquée dans son e-mail d’invitation.

Si vous avez été configuré comme administrateur d’entreprise, vous n’avez pas besoin de recevoir l’e-mail d’activation. Accédez au [portail Azure Enterprise](https://ea.azure.com) et connectez-vous avec l’adresse e-mail de votre compte professionnel, scolaire ou Microsoft et votre mot de passe.

Si vous avez plusieurs inscriptions, choisissez celle à activer. Par défaut, seules les inscriptions actives sont affichées. Pour afficher l’historique des inscriptions, désactivez l’option **Active** en haut à droite du portail Azure Enterprise.

Sous **Inscription**, l’état affiche **Active**.

![Exemple d’inscription active](./media/ea-portal-administration/ea-enrollment-status.png)

Seuls les administrateurs d’entreprise Azure existants peuvent créer d’autres administrateurs d’entreprise.

### <a name="create-another-enterprise-administrator"></a>Créer un autre administrateur d’entreprise

Ajouter un autre administrateur d’entreprise :

1. Connectez-vous au [portail Azure Enterprise](https://ea.azure.com).
1. Accédez à **Gérer** > **Détail de l’inscription**.
1. Sélectionnez **+ Ajouter un administrateur** en haut à droite.

Assurez-vous que vous avez l’adresse e-mail de l’utilisateur et la méthode d’authentification par défaut, par exemple l’authentification avec un compte professionnel, scolaire ou Microsoft.

Si vous n’êtes pas l’administrateur d’entreprise, contactez un administrateur d’entreprise pour lui demander de vous ajouter à une inscription. Une fois que vous avez été ajouté à une inscription, vous recevez un e-mail d’activation.

Si votre administrateur d’entreprise ne peut pas vous aider, créez une [demande de support pour le portail Azure Enterprise](https://support.microsoft.com/supportrequestform/cf791efa-485b-95a3-6fad-3daf9cd4027c). Fournissez les informations suivantes :

- Numéro d’inscription
- Adresse e-mail à ajouter et type d’authentification (avec un compte professionnel, scolaire ou Microsoft)
- Approbation reçue par e-mail d’un administrateur d’entreprise existant
  - Si l’administrateur d’entreprise existant n’est pas disponible, contactez votre partenaire ou votre conseiller logiciel pour lui demander de modifier les détails de contact dans l’outil du Centre de gestion des licences en volume (VLSC).

## <a name="create-an-azure-enterprise-department"></a>Créer un service Azure Enterprise

Les administrateurs d’entreprise et les administrateurs de service utilisent les services pour organiser les services Entreprise Azure, et créer des rapports sur l’utilisation par service et par centre de coûts. L’administrateur d’entreprise peut :

- ajouter ou supprimer des services ;
- associer un compte à un service ;
- créer des administrateurs de service ;
- autoriser les administrateurs de service à consulter les tarifs et les coûts.

Un administrateur de service peut ajouter de nouveaux comptes à ses services. Il peut supprimer des comptes de ses services, mais pas de l’inscription.

Pour ajouter un service :

1. Connectez-vous au portail Azure Enterprise.
1. Dans le volet gauche, sélectionnez **Gérer**.
1. Sélectionnez l’onglet **Service**, puis sélectionnez **+ Ajouter un service**.
1. Entrez les informations.
   Le nom du service est le seul champ obligatoire. Il doit comprendre au moins trois caractères.
1. Quand vous avez terminé, sélectionnez **Ajouter**.

## <a name="add-a-department-administrator"></a>Ajouter un administrateur de service

Après avoir créé un service, l’administrateur d’entreprise peut ajouter des administrateurs de service et associer chacun d’eux à un service. Les administrateurs de service peut effectuer les actions suivantes pour leurs services :

- Créer d’autres administrateurs de service
- Consulter et modifier les propriétés d’un service, telles que le nom ou le centre de coûts
- Ajouter des comptes
- Supprimer des comptes
- Télécharger les détails d'utilisation
- Afficher l’utilisation et les frais mensuels <sup>1</sup>

> <sup>1</sup> Un administrateur d’entreprise doit accorder ces autorisations. Si vous avez l’autorisation de consulter les données d’utilisation et les frais mensuels du service, mais que vous ne voyez pas ces informations, contactez votre partenaire.

### <a name="to-add-a-department-administrator"></a>Pour ajouter un administrateur de service

En tant qu’administrateur d’entreprise :

1. Connectez-vous au portail Azure Enterprise.
1. Dans le volet gauche, sélectionnez **Gérer**.
1. Sélectionnez l’onglet **Service**, puis sélectionnez le service.
1. Sélectionnez **+ Ajouter administrateur** et entrez les informations requises.
1. Pour accorder l’accès en lecture seule, définissez l’option **Lecture seule** sur **Oui**, puis sélectionnez **Ajouter**.

![Exemple de boîte de dialogue Ajouter un administrateur de service](./media/ea-portal-administration/ea-create-add-department-admin.png)

### <a name="to-set-read-only-access"></a>Pour accorder l’accès en lecture seule

Vous pouvez accorder l’accès en lecture seule aux administrateurs de service.

- Lorsque vous créez un administrateur de service, définissez l’option de lecture seule sur **Oui**.

- Pour changer l’accès d’un administrateur de service existant :
   1. Sélectionnez un service, puis sélectionnez le symbole de crayon à côté de l’**administrateur de service** à modifier.
   1. Définissez l’option lecture seule ouverte sur **Oui**, puis sélectionnez **Enregistrer**.

Les administrateurs d’entreprise obtiennent automatiquement les autorisations d’administrateur de service.

## <a name="add-an-account"></a>Ajouter un compte

La structure des comptes et des abonnements a un impact sur la façon dont ces éléments sont administrés, ainsi que sur leur mode d’affichage dans vos factures et rapports. Les divisions métier, les équipes fonctionnelles et les emplacements géographiques sont des exemples de structures organisationnelles courantes.

Pour ajouter un compte :

1. Dans le portail Azure Enterprise, sélectionnez **Gérer** dans le volet de navigation de gauche.
1. Sélectionnez l’onglet **Compte**. Dans la page **Compte**, sélectionnez **+ Ajouter un compte**.
1. Sélectionnez un service ou laissez-le comme non attribué, puis sélectionnez le type d’authentification souhaité.
1. Tapez un nom convivial pour identifier le compte dans les rapports.
1. Tapez l’adresse **e-mail du propriétaire du compte** à associer au nouveau compte.
1. Confirmez l’adresse e-mail, puis cliquez sur **Ajouter**.

![Exemple illustrant la liste des comptes et l’option d’ajout de compte](./media/ea-portal-administration/create-ea-add-an-account.png)

Pour ajouter un autre compte, sélectionnez **Ajouter un autre compte**, ou sélectionnez **Ajouter** en bas à droite dans la barre d’outils de gauche.

Pour confirmer la propriété du compte :

1. Connectez-vous au portail Azure Enterprise.
1. Afficher l’état.

   L’état doit passer de **Pending** (En attente) à **Start/End date** (Date de début/fin). La date de début/fin correspond à la date à laquelle l’utilisateur s’est connecté pour la première fois et à la date de fin du contrat.
1. Lorsque le message **Avertissement** s’affiche, le propriétaire du compte doit sélectionner **Continuer** pour activer le compte la première fois qu’il se connecte au portail Azure Enterprise.

## <a name="change-azure-subscription-or-account-ownership"></a>Changer l’abonnement Azure ou la propriété du compte

Les administrateurs d’entreprise peuvent utiliser le portail Azure Enterprise pour transférer la propriété du compte des abonnements sélectionnés ou de tous les abonnements dans une inscription.

Quand vous effectuez un transfert d’abonnement ou de propriété de compte, Microsoft met à jour le propriétaire du compte.

Avant d’effectuer le transfert de propriété, comprenez les stratégies de contrôle d’accès en fonction du rôle Azure (Azure RBAC) :

- Lorsque vous effectuez des transferts d’abonnement ou de propriété de compte entre deux ID d’organisation dans le même locataire, les stratégies Azure RBAC ainsi que les attributions de rôles d’administrateur et de coadministrateur de service existantes sont conservées.
- Les transferts d’abonnement ou de propriété de compte inter-locataires entraînent la perte de vos stratégies Azure RBAC et de vos attributions de rôles.
- Les stratégies et les rôles d’administrateur ne sont pas transférés entre les annuaires. Les administrateurs de service sont mis à jour pour refléter le propriétaire du compte de destination.
- Pour éviter la perte des stratégies Azure RBAC et des attributions de rôles lors du transfert de l’abonnement entre les locataires, veillez à ce que la case **Déplacer les abonnements vers le locataire Azure AD du destinataire** reste **décochée**. Cela permet de conserver les services, les rôles Azure et les stratégies sur le locataire Azure AD actuel, et de transférer uniquement la propriété de facturation pour le compte.  
    :::image type="content" source="./media/ea-portal-administration/unselected-checkbox-move-subscriptions-to-recipients-tenant.png" alt-text="Image représentant la case non cochée permettant de déplacer les abonnements vers le locataire Azure AD" lightbox="./media/ea-portal-administration/unselected-checkbox-move-subscriptions-to-recipients-tenant.png" :::


Avant de changer le propriétaire d’un compte :

1. Dans le portail Azure Enterprise, affichez l’onglet **Compte** et identifiez le compte source. Le compte source doit être actif.
1. Identifiez le compte de destination et assurez-vous qu’il est actif.

Pour transférer la propriété du compte pour tous les abonnements :

1. Connectez-vous au portail Azure Enterprise.
1. Dans la zone de navigation de gauche, sélectionnez **Gérer**.
1. Sélectionnez l’onglet **Compte**, puis pointez sur un compte.
1. Sélectionnez l’icône de changement du propriétaire du compte à droite. L’icône représente une personne.  
    ![Image représentant le symbole de changement du propriétaire du compte](./media/ea-portal-administration/create-ea-create-sub-transfer-account-ownership-of-sub.png)
1. Choisissez le compte de destination vers lequel vous souhaitez effectuer le transfert, puis sélectionnez **Suivant**.
1. Si vous souhaitez transférer la propriété du compte entre des locataires Azure AD, cochez la case **Déplacer les abonnements vers le locataire Azure AD du destinataire**.  
    :::image type="content" source="./media/ea-portal-administration/selected-checkbox-move-subscriptions-to-recipients-tenant.png" alt-text="Image représentant la case cochée permettant de déplacer les abonnements vers le locataire Azure AD" lightbox="./media/ea-portal-administration/selected-checkbox-move-subscriptions-to-recipients-tenant.png" :::
1. Confirmez le transfert et sélectionnez **Envoyer**.

Pour transférer la propriété du compte pour un seul abonnement :

1. Connectez-vous au portail Azure Enterprise.
1. Dans la zone de navigation de gauche, sélectionnez **Gérer**.
1. Sélectionnez l’onglet **Compte**, puis pointez sur un compte.
1. Sélectionnez l’icône de transfert des abonnements à droite. L’icône représente une page.  
    ![Image représentant le symbole de transfert des abonnements](./media/ea-portal-administration/ea-transfer-subscriptions.png)
1. Choisissez le compte de destination vers lequel vous souhaitez transférer l’abonnement, puis sélectionnez **Suivant**.
1. Si vous souhaitez transférer la propriété de l’abonnement entre des locataires Azure AD, cochez la case **Déplacer les abonnements vers le locataire Azure AD du destinataire**.  
    :::image type="content" source="./media/ea-portal-administration/selected-checkbox-move-subscriptions-to-recipients-tenant.png" alt-text="Image représentant la case cochée permettant de déplacer les abonnements vers le locataire Azure AD" lightbox="./media/ea-portal-administration/selected-checkbox-move-subscriptions-to-recipients-tenant.png" :::
1. Confirmez le transfert, puis sélectionnez **Envoyer**.


## <a name="associate-an-account-to-a-department"></a>Associer un compte à un service

Les administrateurs d’entreprise peuvent associer des comptes existants à des services dans le cadre de l’inscription.

### <a name="to-associate-an-account-to-a-department"></a>Pour associer un compte à un service

1. Connectez-vous au portail Azure EA en tant qu’administrateur d’entreprise.
1. Sélectionnez **Gérer** dans le volet de navigation de gauche.
1. Sélectionnez **Service**.
1. Placez le pointeur sur la ligne contenant le compte, puis sélectionnez l’icône du crayon à droite.
1. Sélectionnez le service dans le menu déroulant.
1. Sélectionnez **Enregistrer**.

## <a name="associate-an-existing-account-with-your-pay-as-you-go-subscription"></a>Associer un compte existant à votre abonnement avec paiement à l’utilisation

Si vous avez déjà un compte Microsoft Azure sur le portail Azure, entrez le compte scolaire, professionnel ou Microsoft pour l’associer à votre inscription Accord Entreprise.

### <a name="associate-an-existing-account"></a>Associer un compte existant

1. Sur le portail Azure Enterprise, sélectionnez **Gérer**.
1. Sélectionnez l’onglet **Compte**.
1. Sélectionnez **+ Ajouter un compte**.
1. Entrez le compte professionnel, scolaire ou Microsoft associé au compte Azure existant.
1. Confirmez le compte associé au compte Azure existant.
1. Spécifiez le nom à utiliser pour identifier ce compte dans les rapports.
1. Sélectionnez **Ajouter**.
1. Pour ajouter un compte supplémentaire, vous pouvez sélectionner à nouveau l’option **+ Ajouter un compte**, ou bien revenir à la page d’accueil en sélectionnant le bouton **Administrateur**.
1. Si vous affichez la page **Compte**, le compte que vous venez d’ajouter s’affiche avec l’état **En attente**.

### <a name="confirm-account-ownership"></a>Confirmer la propriété du compte

1. Connectez-vous au compte de messagerie associé au compte professionnel, scolaire ou Microsoft que vous avez fourni.
1. Ouvrez l’e-mail de notification intitulé _« Invitation à activer votre compte sur le service Microsoft Azure de la part de Microsoft Volume Licensing »_ .
1. Cliquez sur le lien **Connectez-vous à Microsoft Azure Enterprise Portal** dans l’invitation.
1. Sélectionnez **Connexion**.
1. Entrez votre compte professionnel, scolaire ou Microsoft et votre mot de passe pour vous connecter et confirmer la propriété du compte.

### <a name="azure-marketplace"></a>Place de marché Azure

Bien que la plupart des abonnements puissent passer de l’environnement de paiement à l’utilisation à un Accord Entreprise Azure, ce n’est pas le cas des services de la place de marché Azure. Pour obtenir une vue unique de tous les abonnements et facturations, nous vous recommandons d’ajouter les services de la place de marché au portail Azure Enterprise.

1. Connectez-vous au portail Azure Enterprise.
1. Sélectionnez **Gérer** dans le volet de navigation de gauche.
1. Sélectionnez l’onglet **EnrollmentTab**.
1. Consultez la section **Détails de l’inscription**.
1. À droite du champ Place de marché Azure, sélectionnez l’icône de crayon pour l’activer. Sélectionnez **Enregistrer**.

Le propriétaire du compte peut désormais acheter des services de la place de marché Azure qui faisaient précédemment partie de l’abonnement avec paiement à l’utilisation.

Après que les nouveaux abonnements de la place de marché Azure sont activés dans votre inscription Azure EA, annulez les services de la place de marché Azure créés dans l’environnement de paiement à l’utilisation. Cette étape est essentielle afin que vos abonnements de la place de marché Azure ne tombent pas dans un état incorrect lors de l’expiration de votre instrument de paiement à l’utilisation.

### <a name="msdn"></a>MSDN

Les abonnements MSDN sont automatiquement convertis en MSDN Dev/Test, et l’offre Azure EA perd tout crédit monétaire existant.

### <a name="azure-in-open"></a>Azure dans Open

Si vous associez un abonnement Azure dans Open à un Accord Entreprise, vous annulez les crédits Azure dans Open non consommés. Par conséquent, nous vous avons recommandé de consommer tout le crédit d’un abonnement Azure dans Open avant d’ajouter le compte à votre Accord Entreprise.  

### <a name="accounts-with-support-subscriptions"></a>Comptes avec abonnements au support

Si votre Accord Entreprise ne dispose pas d’un abonnement au support et que vous ajoutez un compte existant avec un abonnement au support au portail Azure Enterprise, votre abonnement au support MOSA ne sera pas automatiquement transféré. Vous devrez racheter un abonnement au support dans Azure EA durant la période de grâce, soit à la fin du mois suivant.

## <a name="department-spending-quotas"></a>Quotas de dépenses des services

Les clients EA peuvent définir ou changer des quotas de dépenses pour chaque service dans le cadre d’une inscription. Le montant de quota de dépenses est défini pour la période de Paiement anticipé actuelle. À la fin de la période de Paiement anticipé actuelle, le système prolonge le quota de dépenses existant jusqu’à la période de Paiement anticipé suivante, sauf si les valeurs sont mises à jour.

L’administrateur de service peut afficher le quota de dépenses, mais seul l’administrateur d’entreprise peut mettre à jour le montant du quota. L’administrateur d’entreprise et l’administrateur de service recevront des notifications lorsque le quota aura atteint 50 %, 75 %, 90 % et 100 %.

### <a name="enterprise-administrator-to-set-the-quota"></a>Administrateur d’entreprise, pour définir le quota :

 1. Ouvrez le portail Azure EA.
 1. Sélectionnez **Gérer** dans le volet de navigation de gauche.
 1. Sélectionnez l’onglet **Service**.
 1. Sélectionnez le Service.
 1. Sélectionnez le symbole de crayon dans la section Détails du service, ou sélectionnez le symbole **+ Ajouter un service** pour ajouter un quota de dépenses à un nouveau service.
 1. Sous Détails du service, entrez un montant de quota de dépenses dans la devise de l’inscription, dans la zone Quota de dépenses $ (doit être supérieur à 0).
    - Vous pouvez également modifier le nom du service et le centre de coûts à ce stade.
 1. Sélectionnez **Enregistrer**.

Le quota de dépenses du service apparaît maintenant dans la vue de liste des services de l’onglet Service. À la fin du Paiement anticipé actuel, le portail Azure EA conserve les quotas de dépenses pour la période de Paiement anticipé suivante.

Le montant du quota du service est indépendant du Paiement anticipé Azure actuel, et le montant et les alertes du quota s’appliquent uniquement à l’utilisation de la première partie. Le quota de dépenses du service est fourni à titre d’information uniquement et n’applique pas de limites de dépense.

### <a name="department-administrator-to-view-the-quota"></a>Administrateur de service, pour afficher le quota :

1. Ouvrez le portail Azure EA.
1. Sélectionnez **Gérer** dans le volet de navigation de gauche.
1. Sélectionnez l’onglet **Service**, puis consultez la vue de liste des services avec des quotas de dépenses.

Si vous êtes un client indirect, les fonctionnalités de coût doivent être activées par votre partenaire de canal.

## <a name="enterprise-user-roles"></a>Rôles d’utilisateur Entreprise

Le portail Azure EA vous permet de gérer vos coûts et votre utilisation d’Azure EA. Trois rôles principaux existent dans le portail Azure EA :

- Administrateur Contrat Entreprise
- Administrateur de service
- Propriétaire du compte

Chaque rôle a un niveau d’accès et une autorité différents.

Pour plus d’informations sur les rôles d’utilisateur, consultez [Rôles d’utilisateur Entreprise](./understand-ea-roles.md#enterprise-user-roles).

## <a name="add-an-azure-ea-account"></a>Ajouter un compte Azure EA

Le compte Azure EA est une unité d’organisation dans le portail Azure EA. Il est utilisé pour gérer des abonnements, ainsi que pour créer des rapports. Pour accéder aux services Azure et les utiliser, vous devez créer un compte ou vous en faire créer-un.

Pour plus d’informations sur les comptes Azure, consultez [Ajouter un compte](#add-an-account).

## <a name="enterprise-devtest-offer"></a>Offre Enterprise Dev/Test

En tant qu’administrateur d’entreprise Azure, vous pouvez autoriser les propriétaires de comptes de votre organisation à créer des abonnements basés sur l’offre EA Dev/Test. Pour ce faire, sélectionnez la zone Dev/Test du propriétaire du compte dans le portail Azure EA.

Une fois que vous avez coché la case Dev/Test, informez le propriétaire du compte afin qu’il puisse configurer les abonnements Dev/Test EA nécessaires pour ses équipes d’abonnés Dev/Test.

L’offre permet aux abonnés Visual Studio actifs d’exécuter des charges de travail de développement et de test sur Azure à des tarifs Dev/Test spéciaux. Elle fournit un accès à la galerie complète d’images Dev/Test, notamment Windows 8.1 et Windows 10.

### <a name="to-set-up-the-enterprise-devtest-offer"></a>Pour configurer l’offre Enterprise Dev/Test :

1. Connectez-vous en tant qu’administrateur d’entreprise.
1. Sélectionnez **Gérer** dans le volet de navigation de gauche.
1. Sélectionnez l’onglet **Compte**.
1. Sélectionnez la ligne du compte sur lequel vous souhaitez activer l’accès Dev/Test.
1. Sélectionnez le symbole de crayon à droite de la ligne.
1. Cochez la case Dev/Test.
1. Sélectionnez **Enregistrer**.

Lorsqu’un utilisateur est ajouté en tant que propriétaire du compte par le biais du portail Azure EA, tous les abonnements Azure associés au propriétaire du compte qui sont basés sur l’offre PAYG Dev/Test ou les offres de crédit mensuel pour les abonnés Visual Studio sont convertis en l’offre EA Dev/Test. Les abonnements basés sur d’autres types d’offres, tels que PAYG, associés au propriétaire du compte, sont convertis en offres Microsoft Azure Enterprise.

L’offre Dev/Test ne s’applique pas aux clients Azure Government pour le moment.

## <a name="create-a-subscription"></a>Création d’un abonnement

Les propriétaires de compte peuvent consulter et gérer les abonnements. Vous pouvez utiliser des abonnements pour donner aux équipes de votre organisation l’accès à des projets et environnements de développement. Il peut s’agir d’environnements de test, de production, de développement et de préproduction, par exemple.

En créant des abonnements différents pour chacun des environnements d’application, vous sécurisez davantage chaque environnement.

- Vous pouvez également attribuer un compte d’administrateur de service distinct par abonnement.
- Vous pouvez associer plusieurs services aux abonnements.
- Le propriétaire du compte crée des abonnements et attribue un compte d’administrateur de service à chaque abonnement dans son compte.

### <a name="add-a-subscription"></a>Ajouter un abonnement

Aidez-vous des informations suivantes pour ajouter un abonnement.

La première fois que vous ajoutez un abonnement à votre compte, vous êtes invité à accepter le contrat d’abonnement à Microsoft Online (MOSA) et un plan tarifaire. Bien qu’ils ne s’appliquent pas aux clients ayant souscrit un Contrat Entreprise, le MOSA et le plan tarifaire sont requis pour créer votre abonnement. L’avenant à votre inscription Contrat Entreprise Microsoft Azure se substitue aux éléments cités ci-dessus, et votre lien contractuel ne change pas. Lorsque vous y êtes invité, cochez la case qui indique que vous acceptez les conditions.

_Microsoft Azure Enterprise_ est le nom par défaut lorsqu’un abonnement est créé. Vous pouvez modifier le nom de l’abonnement pour le différencier des autres abonnements dans votre inscription et vous assurer qu’il est reconnaissable dans les rapports au niveau de l’entreprise.

Pour ajouter un abonnement :

1. Dans le portail Azure Enterprise, connectez-vous au compte.
1. Sélectionnez l’onglet **Administrateur**, puis sélectionnez **Abonnement** en haut de la page.
1. Vérifiez que vous êtes connecté en tant que propriétaire du compte.
1. Sélectionnez **+ Ajouter l’abonnement**, puis sélectionnez **Achat**.

   La première fois que vous ajoutez un abonnement à un compte, vous devez fournir vos informations de contact. Quand vous ajoutez des abonnements supplémentaires, vos informations de contact sont remplies automatiquement.

1. Sélectionnez **Abonnements**, puis choisissez l’abonnement créé.
1. Sélectionnez **Modifier les détails de l’abonnement**.
1. Modifiez les champs **Nom de l’abonnement** et **Administrateur de service**, puis cochez la case.

   Le nom de l’abonnement apparaît sur les rapports. C’est le nom du projet associé à l’abonnement dans le portail de développement.

Cela peut prendre jusqu’à 24 heures pour que les nouveaux abonnements soient visibles dans la liste des abonnements. Une fois que vous avez créé un abonnement, vous pouvez :

- [Modifier les détails de l’abonnement](https://account.azure.com/Subscriptions)
- [Gérer les services de l’abonnement](https://portal.azure.com/#home)

## <a name="delete-subscription"></a>Supprimer l’abonnement

Pour supprimer un abonnement dans lequel vous êtes le propriétaire du compte :

1. Connectez-vous au portail Azure avec les informations d’identification associées à votre compte.
1. Dans le menu Hub, sélectionnez **Abonnements**.
1. Dans le panneau des abonnements situé en haut à gauche de la page, sélectionnez celui que vous voulez annuler, puis sélectionnez **Annuler l’abonnement** pour ouvrir l’onglet d’annulation.
1. Entrez le nom de l’abonnement et choisissez un motif d’annulation, puis sélectionnez le bouton **Annuler l’abonnement**.

Seuls les administrateurs de compte peuvent annuler des abonnements.

Pour plus d’informations, consultez [Que se passe-t-il après l’annulation de mon abonnement ?](cancel-azure-subscription.md#what-happens-after-subscription-cancellation)

## <a name="delete-an-account"></a>Supprimer un compte

La suppression de compte ne peut être effectuée que pour les comptes actifs sans abonnements actifs.

1. Dans Enterprise Portal, sélectionnez **Gérer** dans le volet de navigation de gauche.
1. Sélectionnez l’onglet **Compte**.
1. Dans la table comptes, sélectionnez le compte que vous souhaitez supprimer.
1. Sélectionnez le symbole X à droite de la ligne de compte.
1. Une fois qu’il n’y a pas d’abonnements actifs sous le compte, sélectionnez **Oui** sous la ligne de compte pour confirmer la suppression du compte.

## <a name="update-notification-settings"></a>Mettre à jour les paramètres de notification

Les administrateurs d’entreprise sont automatiquement inscrits pour recevoir les notifications d’utilisation associées à leur inscription. Chaque administrateur d’entreprise peut modifier l’intervalle des notifications individuelles ou les désactiver complètement.

Les contacts de notification sont affichés dans le portail Azure EA, dans la section **Contact de notification**. La gestion de vos contacts de notification vous permet de vous assurer que les bonnes personnes de votre organisation obtiennent les notifications Azure EA.

Pour afficher les paramètres actuels des notifications :

1. Dans le portail Azure EA, accédez à **Gérer** > **Contact de notification**.
2. Adresse e-mail – Adresse e-mail associée au compte Microsoft, professionnel ou scolaire de l’administrateur d’entreprise qui reçoit les notifications.
3. Fréquence de notification du solde non facturé – Fréquence à laquelle les notifications sont configurées pour être envoyées à chaque administrateur d’entreprise individuel.

Pour ajouter un contact :

1. Sélectionnez **+Ajouter un contact**.
2. Entrez l’adresse e-mail, puis confirmez-la.
3. Sélectionnez **Enregistrer**.

Le nouveau contact de notification s’affiche dans la section **Contact de notification**. Pour changer la fréquence de notification, sélectionnez le contact de notification, puis le symbole de crayon à droite de la ligne sélectionnée. Définissez la fréquence sur **quotidienne**, **hebdomadaire**, **mensuelle** ou **aucune**.

Vous pouvez supprimer les notifications de cycle de vie _La date de fin de la période de couverture approche_ et _La date de désactivation et de déprovisionnement approche_. La désactivation des notifications de cycle de vie supprime les notifications relatives à la période de couverture et à la date de fin du contrat.

## <a name="azure-sponsorship-offer"></a>Offre Azure Sponsorship

L’offre Azure Sponsorship est un compte Microsoft Azure sponsorisé limité. Elle est disponible sur invitation par e-mail pour certains clients sélectionnés par Microsoft. Si vous bénéficiez de l’offre Microsoft Azure Sponsorship, vous recevrez une invitation par e-mail pour votre ID de compte.

Pour plus d’informations, créez une [demande de support pour l’activation de Sponsorship](https://aka.ms/azrsponsorship).

## <a name="conversion-to-work-or-school-account-authentication"></a>Conversion en authentification de compte professionnel ou scolaire

Les utilisateurs d’Azure Enterprise peuvent convertir un compte Microsoft (MSA ou Live ID) en un type d’authentification de compte professionnel ou scolaire (qui utilise Azure Active Directory).

Pour commencer :

1. Ajoutez le compte professionnel ou scolaire au portail Azure EA dans le ou les rôles nécessaires.
1. Si vous recevez des erreurs, le compte n’est peut-être pas valide dans Active Directory.  Azure utilise le nom d’utilisateur principal (UPN), qui n’est pas toujours identique à l’adresse e-mail.
1. Authentifiez-vous sur le portail Azure EA à l’aide du compte professionnel ou scolaire.

### <a name="to-convert-subscriptions-from-microsoft-accounts-to-work-or-school-accounts"></a>Pour convertir des abonnements de comptes Microsoft en comptes professionnels ou scolaires :

1. Connectez-vous au portail de gestion à l’aide du compte Microsoft qui possède les abonnements.
1. Utilisez le transfert de propriété de compte pour passer au nouveau compte.
1. Le compte Microsoft doit à présent être gratuit à partir de tous les abonnements actifs et peut être supprimé.
1. Un compte supprimé reste affiché dans le portail dans un état inactif pour des raisons de facturation historique.  Vous pouvez l’effacer de la vue en cochant une case afin d’afficher les comptes actifs uniquement.

## <a name="account-subscription-ownership-faq"></a>FAQ sur la propriété de l’abonnement de compte

Ce document répond à des questions fréquemment posées relatives à la propriété de l’abonnement de compte.

### <a name="can-i-associate-my-existing-azure-account-to-azure-ea-enrollment"></a>Puis-je associer mon compte Azure existant à l’inscription Azure EA ?

Oui. Tous les abonnements Azure pour lesquels vous êtes propriétaire du compte seront convertis en Accord Entreprise. Les abonnements qui utilisent le crédit mensuel, tels que Visual Studio, AzurePass, MPN et BizSpark, entre autres, sont inclus. Vous perdrez le crédit mensuel lors de la conversion de ce type d’abonnements.

### <a name="how-many-azure-account-owners-can-you-have-per-subscription"></a>Combien de propriétaires de compte Azure pouvez-vous avoir par abonnement ?

Un seul propriétaire de compte est autorisé par abonnement.  Des rôles supplémentaires peuvent être ajoutés à l’aide de l’accès en fonction du rôle ou (Contrôle d’accès (IAM)) sous l’onglet de l’abonnement dans le coin supérieur gauche de la page du [portail Azure](https://portal.azure.com).

### <a name="is-it-possible-to-transfer-subscription-ownership-to-another-account"></a>Est-il possible de transférer la propriété de l’abonnement à un autre compte ?

Oui, vous pouvez transférer la propriété de l’abonnement à un autre compte. Par exemple, si un compte A possède trois abonnements, l’administrateur de l’entreprise peut transférer un abonnement au compte B, l’autre au compte C et l’autre au compte D. Il peut également transférer tous les abonnements au compte E.

Pour transférer des abonnements :

1. Sur le portail Azure Enterprise, sélectionnez **Gérer** > **Compte**.
1. Pointez sur **Compte** tout à droite pour voir les options **Transférer la propriété** (icône représentant une personne) et **Transférer l’abonnement** (icône de liste). Ces options sont visibles uniquement pour les comptes actifs.

### <a name="can-an-azure-account-owner-be-listed-under-more-than-one-department"></a>Un propriétaire de compte Azure peut-il être répertorié sous plusieurs services ?

Non, un propriétaire de compte ne peut être associé qu’à un seul service. La stratégie permet de garantir une supervision et une répartition précises des coûts et des dépenses associés au service auquel il est associé dans le cadre de l’inscription EA dans le portail Azure EA.

### <a name="can-an-azure-account-owner-be-listed-as-a-security-group"></a>Un propriétaire de compte Azure peut-il être répertorié en tant que groupe de sécurité ?

Non, un propriétaire d’abonnement doit être une authentification de compte Microsoft (MSA) ou Azure Active Directory (Azure AD) unique. Pour tenir compte des successions au sein de votre organisation, vous pouvez envisager de créer des comptes génériques et d’utiliser Azure AD pour gérer l’accès aux abonnements.

### <a name="can-an-individual-user-own-multiple-subscriptions"></a>Un utilisateur individuel peut-il avoir plusieurs abonnements ?

Un propriétaire de compte Azure peut créer et gérer un nombre illimité d’abonnements.

### <a name="how-can-i-accessview-all-my-organizations-subscriptions"></a>Comment puis-je accéder à tous les abonnements de mon organisation et les consulter ?

Aujourd’hui, cette opération doit être effectuée via une stratégie. Cela signifie que vous devez exiger que, pour chaque abonnement créé, votre compte soit ajouté à un rôle d’abonnement à l’aide de l’accès en fonction du rôle.

### <a name="where-do-i-go-to-create-a-subscription"></a>Où puis-je créer un abonnement ?

Avant de pouvoir créer un abonnement d’offre Enterprise Azure (EA), votre compte doit être ajouté au rôle de propriétaire du compte par l’administrateur de votre inscription EA dans le portail Azure EA. Vous devez ensuite vous connecter au portail Azure EA pour obtenir votre droit de créer des abonnements de type EA. Nous vous recommandons de créer votre premier abonnement EA à partir du lien « + Ajouter un abonnement » sous l’onglet de l’abonnement sur le portail EA.  Toutefois, une fois que votre compte est autorisé, il peut être plus facile de créer des abonnements dans portal.azure.com sous l’onglet de l’abonnement dans le coin supérieur gauche de la page, où vous pouvez créer et renommer votre abonnement en une seule étape.

### <a name="who-can-create-a-subscription"></a>Qui peut créer un abonnement ?

Pour créer un abonnement de type Enterprise Azure, vous devez être titulaire du rôle de propriétaire du compte sur le [portail EA](https://ea.azure.com).

## <a name="azure-ea-term-glossary"></a>Glossaire des termes Azure EA

- **Compte**  : unité d’organisation dans le portail Azure Enterprise. Elle est utilisée pour gérer des abonnements et pour créer des rapports.
- **Propriétaire du compte** : la personne qui gère les abonnements et les administrateurs de services sur Azure. Elles peuvent afficher les données d’utilisation sur ce compte et ses abonnements associés.
- **Abonnement de modification** : abonnement d’un an ou équivalent dans le cadre de la modification de l’inscription.
- **Acompte** : acompte pour un montant annuel en devises concernant les services Azure, à un tarif réduit.
- **Administrateur de service** : personne identifiée qui gère les services, crée des comptes et des propriétaires de comptes, affiche les détails d’utilisation des services qu’elle gère et peut afficher les coûts quand des autorisations lui sont accordées.
- **Numéro d’inscription** : identificateur unique fourni par Microsoft pour identifier l’inscription spécifique associée à un Accord Entreprise.
- **Administrateur d’entreprise** : personne qui gère les services, les propriétaires de service, les comptes et les propriétaires de comptes sur Azure. Elles ont la possibilité de gérer les administrateurs d’entreprise, ainsi que d’afficher les données d’utilisation, les quantités facturées et les frais non facturés pour tous les comptes et abonnements associés à l’inscription d’entreprise.
- **Contrat Entreprise** : Un contrat de licence Microsoft pour les clients avec des achats centralisés qui souhaitent normaliser leur organisation entière sur la technologie Microsoft et maintenir une infrastructure informatique sur des logiciels Microsoft.
- **Inscription à un contrat Entreprise** : inscription au programme Accord Entreprise fournissant des produits Microsoft en volume à tarif réduit.
- **Compte Microsoft** : service web qui permet aux sites participants d’authentifier un utilisateur avec un ensemble unique d’informations d’identification.
- **Modification d’inscription Microsoft Azure Enterprise (modification de l’inscription)**  : modification signée par une entreprise, qui lui permet d’accéder à Azure dans le cadre de l’inscription de l’entreprise.
- **Portail Azure Enterprise** : portail utilisé par nos clients d’entreprise pour gérer leurs comptes Azure et leurs abonnements associés.
- **Quantité de ressources consommées** : quantité d’un service Azure utilisée durant un mois.
- **Administrateur de services** : personne identifiée qui accède aux abonnements et aux projets de développement sur le portail Azure Enterprise et les gère.
- **Abonnement**: représente un abonnement au portail Azure Enterprise et est un conteneur de services Microsoft Azure gérés par le même administrateur de services.
- **Compte professionnel ou scolaire** : pour les organisations qui ont configuré Azure Active Directory avec une fédération dans le cloud, et dont tous les comptes se trouvent sur un seul locataire.

### <a name="enrollment-statuses"></a>États de l'inscription

- **Nouveau** : Cet état est affecté à une inscription dans les 24 heures qui suivent sa création et est mis à jour en état d’attente dans un délais de 24 heures.
- **Pending** : l’administrateur de l’inscription doit se connecter au portail Azure Enterprise. Une fois la connexion établie, l’inscription passe à l’état Active.
- **Actif** : l’inscription est active et les comptes et les abonnements peuvent être créés dans le portail Azure Enterprise. L’inscription reste active jusqu’à la date de fin de l’Accord Entreprise.
- **Durée prolongée indéterminée** : une durée prolongée indéterminée prend place quand la date de fin d’Accord Entreprise est atteinte. Il permet aux clients Azure EA qui ont opté pour le terme étendu de continuer à utiliser des services Azure indéfiniment à la fin de leur Accord Entreprise.

   Avant que l’inscription Azure EA n’atteigne la date de fin de l’Accord Entreprise, l’administrateur de l’inscription doit prendre l’une des décisions suivantes :

  - Renouveler l’inscription en ajoutant un Paiement anticipé Azure supplémentaire.
  - Transférer vers une nouvelle inscription.
  - Migrer vers le programme d’abonnement Microsoft Online (MOSP).
  - Confirmer la désactivation de tous les services associés à l’inscription.
- **Expiré** : le client Azure EA est exclu du terme prolongé et l’inscription à Azure EA a atteint la date de fin de l’Accord Entreprise. L’inscription va expirer et tous les services associés seront désactivés.
- **Transférée** : Les inscriptions où tous les comptes et services associés ont été transférés vers une nouvelle inscription se voit appliquer l’état Transférée.
  >[!NOTE]
  > Les inscriptions ne sont pas automatiquement transférées si un nouveau numéro d’inscription est généré lors du renouvellement. Vous devez inclure votre numéro d’inscription précédent votre demande de renouvellement pour qu’un transfert automatique puisse avoir lieu.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la façon dont les [réservations de machines virtuelles](ea-portal-vm-reservations.md) peuvent vous aider à réaliser des économies.
- Si vous avez besoin d’aide pour résoudre des problèmes rencontrés avec le portail Azure EA, consultez [Résoudre les problèmes d’accès au portail Azure EA](ea-portal-troubleshoot.md).