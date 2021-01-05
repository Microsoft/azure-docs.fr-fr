---
title: Comprendre les rôles d’administrateur pour Accord Entreprise dans Azure
description: Découvrez les rôles d’administrateur d’entreprise dans Azure. Vous pouvez affecter cinq rôles d’administration distincts.
author: bandersmsft
ms.reviewer: adwise
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: conceptual
ms.date: 12/10/2020
ms.author: banders
ms.custom: contperf-fy21q1
ms.openlocfilehash: e8420f68fba40ddb61d06770f9f5bc2fafe5616a
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560523"
---
# <a name="managing-azure-enterprise-agreement-roles"></a>Administration des rôles de l’Accord Entreprise Azure

Pour faciliter la gestion de l’utilisation et des dépenses d’une organisation, les clients Azure disposant d’un Accord Entreprise peuvent affecter cinq rôles d’administrateur distincts :

- Administrateur d’entreprise
- Administrateur d’entreprise (lecture seule)<sup>1</sup>
- Administrateur de service
- Administrateur de service (lecture seule)
- Propriétaire de compte<sup>2</sup>

<sup>1</sup> Le contact de facturation du contrat EA sera sous ce rôle.

<sup>2</sup> Le contact de facturation ne peut pas être ajouté ou modifié sur le portail Azure Enterprise Portal et sera ajouté à l’inscription EA en fonction de l’utilisateur qui est configuré en tant que contact de facturation au niveau du contrat. Pour modifier le contact de facturation, une demande doit être effectuée par l’intermédiaire d’un conseiller partenaire/logiciel auprès du centre régional des opérations (ROC).

Le premier administrateur d’inscription configuré pendant le provisionnement de l’inscription détermine le type d’authentification du compte de contact de facturation. Quand le contact de facturation est ajouté au portail EA en tant qu’administrateur en lecture seule, il reçoit l’authentification de compte Microsoft. 

Par exemple, si le type d’authentification initiale est défini sur Mixte, le Contrat Entreprise sera ajouté en tant que compte Microsoft et le contact de facturation aura des privilèges d’administrateur Contrat Entreprise en lecture seule. Si l’administrateur Contrat Entreprise n’approuve pas l’autorisation de compte Microsoft pour un contact de facturation existant, il peut supprimer l’utilisateur en question et demander au client de rajouter l’utilisateur en tant qu’administrateur en lecture seule avec un compte professionnel ou scolaire défini uniquement au niveau de l’inscription dans le portail EA.

Ces rôles sont spécifiques à la gestion des Contrats Entreprise Azure et s’ajoutent aux rôles intégrés dont Azure doit contrôler l’accès aux ressources. Pour plus d’informations, voir [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md).

## <a name="azure-enterprise-portal-hierarchy"></a>Hiérarchie du portail Azure Enterprise

La hiérarchie du portail Azure Enterprise se compose des éléments suivants :

- Le **portail Azure Enterprise** : un portail de gestion en ligne qui vous permet de gérer les coût de vos services Azure EA. Vous pouvez :

  - créer une hiérarchie Azure EA avec des services, des comptes et des abonnements ;
  - rapprocher les coûts des services consommés, télécharger des rapports d’utilisation et consulter les tarifs ;
  - créer des clés API pour votre inscription.

- **Les services** vous aident à segmenter les coûts en regroupements logiques. Les services vous permettent de définir un budget ou un quota au niveau du service.

- Les **comptes** sont des unités d’organisation dans le portail Azure Enterprise. Vous pouvez utiliser des comptes pour gérer les abonnements et les rapports d’accès.

- Les **abonnements** sont la plus petite unité du portail Azure Enterprise. Ce sont les conteneurs des services Azure qui sont gérés par l’administrateur de service.

Le diagramme ci-dessous illustre des hiérarchies Azure EA simples.

![Diagramme de hiérarchies Azure EA simples](./media/understand-ea-roles/ea-hierarchies.png)

## <a name="enterprise-user-roles"></a>Rôles d’utilisateur Entreprise

Les rôles d’utilisateur d’administration suivants font partie de l’inscription de votre entreprise :

- Administrateur d’entreprise
- Administrateur de service
- Propriétaire du compte
- Administrateur de services fédérés
- Contact de notification

Les rôles fonctionnent dans deux portails différents pour effectuer des tâches. Vous utilisez le [portail Azure Enterprise](https://ea.azure.com) pour gérer la facturation et les coûts, et le [portail Azure](https://portal.azure.com) pour gérer les services Azure.

Les rôles utilisateur sont associés à un compte d’utilisateur. Pour valider l’authenticité de l’utilisateur, chaque utilisateur doit avoir un compte professionnel, scolaire ou Microsoft valide. Assurez-vous que chaque compte est associé à une adresse e-mail activement supervisée. Les notifications de compte sont envoyées à l’adresse e-mail.

Quand vous configurez des utilisateurs, vous pouvez attribuer plusieurs comptes au rôle d’administrateur d’entreprise. Toutefois, un seul compte peut contenir le rôle de propriétaire du compte. Vous pouvez également attribuer les rôles d’administrateur d’entreprise et de propriétaire du compte à un seul compte.

### <a name="enterprise-administrator"></a>Administrateur d’entreprise

Les utilisateurs ayant ce rôle ont le niveau d’accès le plus élevé. Ils peuvent effectuer les tâches suivantes :

- Gérer les comptes et les propriétaires de compte.
- Gérer d’autres administrateurs d’entreprise.
- Gérer les administrateurs de service.
- Gérer les contacts de notification.
- Consulter l’utilisation pour tous les comptes.
- Consulter les frais non facturés pour tous les comptes.
- Affichez et gérez tous les ordres de réservation et les réservations qui s’appliquent à l’Accord Entreprise.
  - L’administrateur d’entreprise (en lecture seule) peut afficher les ordres de réservation et les réservations. Il ne peut pas les gérer.

Il peut y avoir plusieurs administrateurs d’entreprise dans une inscription Entreprise. Vous pouvez accorder un accès en lecture seule aux administrateurs d’entreprise. Tous les administrateurs d’entreprise héritent le rôle Administrateur de service.

### <a name="department-administrator"></a>Administrateur de service

Les utilisateurs ayant ce rôle peuvent effectuer les tâches suivantes :

- Créer et gérer des services.
- Créer des propriétaires de compte.
- Consulter les détails de l’utilisation des services qu’ils gèrent.
- Consulter les coûts, s’ils ont les autorisations appropriées.

Il peut y avoir plusieurs administrateurs de service dans chaque inscription Entreprise.

Vous pouvez accorder un accès en lecture seule aux administrateurs de service lorsque vous modifiez ou créez un administrateur de service. Définissez l’option de lecture seule sur **Yes** (Oui).

### <a name="account-owner"></a>Propriétaire du compte

Les utilisateurs ayant ce rôle peuvent effectuer les tâches suivantes :

- Créer et gérer des abonnements.
- Gérer les administrateurs de service.
- Consulter l’utilisation des abonnements.

Chaque compte nécessite un compte professionnel, scolaire ou Microsoft unique. Pour plus d’informations sur les rôles d’administrateur dans le portail Azure Enterprise, consultez [Comprendre les rôles d’administrateur Accord Entreprise Azure dans Azure](understand-ea-roles.md).

### <a name="service-administrator"></a>Administrateur de services fédérés

Le rôle d’administrateur de service est autorisé à gérer les services dans le portail Azure et à attribuer le rôle de coadministrateur à des utilisateurs.

### <a name="notification-contact"></a>Contact de notification

Le contact de notification reçoit des notifications d’utilisation liées à l’inscription.

Les sections suivantes décrivent les limitations et les capacités de chaque rôle.

## <a name="user-limit-for-admin-roles"></a>Limite utilisateur pour les rôles d’administrateur

|Role| Limite utilisateur|
|---|---|
|Administrateur d’entreprise|Illimité|
|Administrateur d’entreprise (lecture seule)|Illimité|
|Administrateur de service|Illimité|
|Administrateur de service (lecture seule)|Illimité|
|Propriétaire du compte|1 par compte<sup>3</sup>|

<sup>3</sup> Chaque compte nécessite un compte Microsoft unique ou un compte professionnel ou scolaire.

## <a name="organization-structure-and-permissions-by-role"></a>Structure de l’organisation et autorisations par rôle

|Tâches| Administrateur d’entreprise|Administrateur d’entreprise (lecture seule)|Administrateur de service|Administrateur de service (lecture seule)|Propriétaire du compte| Partenaire|
|---|---|---|---|---|---|---|
|Voir les administrateurs d’entreprise|✔|✔|✘|✘|✘|✔|
|Ajouter ou supprimer des administrateurs d’entreprise|✔|✘|✘|✘|✘|✘|
|Voir les contacts de notification<sup>4</sup> |✔|✔|✘|✘|✘|✔|
|Ajouter ou supprimer des contacts de notification<sup>4</sup> |✔|✘|✘|✘|✘|✘|
|Créer et gérer des services |✔|✘|✘|✘|✘|✘|
|Voir les administrateurs de service|✔|✔|✔|✔|✘|✔|
|Ajouter ou supprimer des administrateurs de service|✔|✘|✔|✘|✘|✘|
|Voir les comptes de l’inscription |✔|✔|✔<sup>5</sup>|✔<sup>5</sup>|✘|✔|
|Ajouter des comptes à l’inscription et changer le propriétaire du compte|✔|✘|✔<sup>5</sup>|✘|✘|✘|
|Créer et gérer des abonnements et des autorisations d’abonnement|✘|✘|✘|✘|✔|✘|

- <sup>4</sup> Les contacts de notification reçoivent des communications par e-mail à propos du Contrat Entreprise Azure.
- <sup>5</sup> La tâche est limitée aux comptes de votre service.

## <a name="add-a-new-enterprise-administrator"></a>Ajouter un nouvel administrateur d’entreprise

Les administrateurs d’entreprise disposent de la majorité des privilèges lors de la gestion d’une inscription Azure EA. L’administrateur Azure EA initial a été créé lors de la configuration du Contrat Entreprise. Toutefois, vous pouvez ajouter ou supprimer de nouveaux administrateurs à tout moment. Les nouveaux administrateurs peuvent être ajoutés uniquement par des administrateurs existants. Pour plus d’informations sur l’ajout d’administrateurs d'entreprise supplémentaires, consultez [Créer un autre administrateur d’entreprise](ea-portal-administration.md#create-another-enterprise-administrator). Pour plus d’informations sur les rôles et les tâches d’un profil de facturation, consultez [Tâches et rôles du profil de facturation](understand-mca-roles.md#billing-profile-roles-and-tasks).

## <a name="update-account-owner-state-from-pending-to-active"></a>Mettre à jour l’état propriétaire du compte de En attente à Actif

Lorsque de nouveaux propriétaires de compte sont ajoutés pour la première fois à une inscription Azure EA, leur état est _En attente_. Quand un nouveau propriétaire de compte reçoit l’e-mail de bienvenue et d’activation, il peut se connecter pour activer son compte. Une fois qu’il active son compte, l’état du compte est mis à jour de _En attente_ à _Actif_. Le propriétaire du compte doit lire le message « Avertissement », puis sélectionner **Continuer**. Un nouvel utilisateur peut être invité à entrer son prénom et son nom pour créer un compte de commerce. Dans ce cas, il doit ajouter les informations nécessaires pour continuer. Le compte est alors activé.

## <a name="add-a-department-admin"></a>Ajouter un administrateur de service

Une fois qu’un administrateur Azure EA a créé un service, l’administrateur d’entreprise Azure peut ajouter des administrateurs de service et associer chacun d’eux à un service. Un administrateur de service peut créer des comptes. Ces comptes sont nécessaires pour créer des abonnements Azure EA.

Pour plus d’informations sur l’ajout d’un administrateur de service, consultez [Créer un administrateur de service Azure EA](ea-portal-administration.md#add-a-department-administrator).

## <a name="usage-and-costs-access-by-role"></a>Accès à l’utilisation et aux coûts par rôle

|Tâches| Administrateur d’entreprise|Administrateur d’entreprise (lecture seule)|Administrateur de service|Administrateur de service (lecture seule) |Propriétaire du compte| Partenaire|
|---|---|---|---|---|---|---|
|Afficher le solde du crédit, y compris Paiement anticipé Azure|✔|✔|✘|✘|✘|✔|
|Voir les quotas de dépenses des services|✔|✔|✘|✘|✘|✔|
|Définir les quotas de dépenses des services|✔|✘|✘|✘|✘|✘|
|Voir la grille tarifaire du Contrat Entreprise de l’organisation|✔|✔|✘|✘|✘|✔|
|Voir les détails relatifs à l’utilisation et aux coûts|✔|✔|✔<sup>6</sup>|✔<sup>6</sup>|✔<sup>7</sup>|✔|
|Gérer les ressources dans le portail Azure|✘|✘|✘|✘|✔|✘|

- <sup>6</sup> L’administrateur d’entreprise doit activer la stratégie d’**affichage des frais pour l’administrateur de service** dans Enterprise Portal. L’administrateur de service peut ensuite consulter le détail des coûts pour le service.
- <sup>7</sup> L’administrateur d’entreprise doit activer la stratégie d’**affichage des frais pour le propriétaire du compte** dans Enterprise Portal. Le propriétaire du compte peut ensuite consulter le détail des coûts pour le compte.

## <a name="see-pricing-for-different-user-roles"></a>Voir les tarifs des différents rôles d’utilisateur

Vous pouvez voir différents tarifs dans le portail Azure en fonction de votre rôle d’administrateur et de la manière dont les stratégies d’affichage des frais sont définies par l’administrateur d’entreprise. Les deux stratégies d’Enterprise Portal qui affectent les tarifs que vous voyez dans le portail Azure sont les suivantes :

- Affichage des frais pour l’administrateur de service
- Affichage des frais pour le propriétaire du compte

Pour découvrir comment définir ces stratégies, consultez [Gérer l’accès aux informations de facturation pour Azure](manage-billing-access.md).

Le tableau suivant montre la relation entre les rôles d’administrateur Contrat Entreprise, la stratégie d’affichage des frais, le rôle Azure du portail Azure et les tarifs que vous voyez dans le portail Azure. L’administrateur d’entreprise voit toujours les détails relatifs à l’utilisation en fonction des tarifs Contrat Entreprise de l’organisation. Toutefois, l’administrateur de service et le propriétaire du compte voient différents tarifs en fonction de la stratégie d’affichage des frais et de leur rôle Azure. Le rôle Administrateur de service listé dans le tableau suivant fait référence aux rôles Administrateur de service et Administrateur de service (lecture seule).

|Rôle d’administrateur Contrat Entreprise|Stratégie d’affichage des frais pour le rôle|Rôle Azure|Affichage des tarifs|
|---|---|---|---|
|Propriétaire du compte OU Administrateur de service|✔ Activée|Propriétaire|Tarifs Contrat Entreprise de l’organisation|
|Propriétaire du compte OU Administrateur de service|✘ Désactivée|Propriétaire|Tarifs au détail|
|Propriétaire du compte OU Administrateur de service|✔ Activée |Aucun|Aucun tarif|
|Propriétaire du compte OU Administrateur de service|✘ Désactivée |Aucun|Aucun tarif|
|None|Non applicable |Propriétaire|Tarifs au détail|

Vous définissez le rôle d’administrateur Contrat Entreprise et les stratégies d’affichage des frais dans Enterprise Portal. Vous pouvez mettre à jour le rôle Azure dans le portail Azure. Pour plus d’informations, consultez [Gérer l’accès à l’aide de RBAC et du portail Azure](../../role-based-access-control/role-assignments-portal.md).



## <a name="next-steps"></a>Étapes suivantes

- [Gérer l’accès aux informations de facturation pour Azure](manage-billing-access.md)
- [Manage access using RBAC and the Azure portal](../../role-based-access-control/role-assignments-portal.md) (Gérer les accès à l’aide du contrôle d’accès en fonction du rôle et du Portail Azure)
- [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md)
