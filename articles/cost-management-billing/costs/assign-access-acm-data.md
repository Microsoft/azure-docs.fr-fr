---
title: Attribuer une autorisation d’accès aux données Azure Cost Management
description: Cet article vous explique comment affecter une autorisation aux données Azure Cost Management pour différentes étendues d’accès.
author: bandersmsft
ms.author: banders
ms.date: 07/24/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.custom: secdec18
ms.openlocfilehash: 660b5751e3b3cbc632331e99d797af3392a8aea4
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "91371964"
---
# <a name="assign-access-to-cost-management-data"></a>Affecter une autorisation d’accès aux données Cost Management

Pour la plupart des utilisateurs disposant d’un Contrat Entreprise Azure, une combinaison d’autorisations accordées dans le portail Azure et dans le portail Entreprise permet de définir le niveau d’accès d’un utilisateur aux données d’Azure Cost Management. Pour les utilisateurs titulaires d’autres types de comptes Azure, la définition d’un niveau d’accès de l’utilisateur aux données d’Azure Cost Management est plus simple en utilisant le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Cet article vous explique comment affecter un accès aux données Cost Management. Une fois la combinaison d’autorisations attribuée, l’utilisateur voit les données dans Cost Management en fonction de son étendue d’accès et de l’étendue qu’il a sélectionnée dans le portail Azure.

L’étendue sélectionnée par l’utilisateur est utilisée dans Cost Management pour regrouper les données et contrôler l’accès aux informations sur les coûts. Quand les utilisateurs emploient des étendues, ils n’effectuent pas plusieurs sélections. À la place, ils sélectionnent une étendue plus grande qui englobe les étendues enfants, puis appliquent un filtre pour voir ce qui les intéresse. Il est important de bien comprendre le fonctionnement du regroupement des données, car certaines personnes ne doivent pas accéder à l’étendue parente qui englobe les étendues enfants.

Regardez la vidéo [Cost Management - Contrôle d’accès](https://www.youtube.com/watch?v=_uQzQ9puPyM) pour en savoir plus sur l’attribution de l’accès permettant de voir les frais et les coûts du contrôle d’accès en fonction du rôle Azure (Azure RBAC). Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/_uQzQ9puPyM]

## <a name="cost-management-scopes"></a>Étendues de Cost Management

Azure Cost Management prend en charge divers types de comptes Azure. Pour accéder à la liste complète des types de comptes pris en charge, voir [Comprendre les données de Cost Management](understand-cost-mgt-data.md). Le type de compte détermine les étendues disponibles.

### <a name="azure-ea-subscription-scopes"></a>Portées d’abonnement Azure EA

Pour afficher les données de coût relatives aux abonnements Azure EA, un utilisateur doit disposer au moins d’un accès en lecture à une ou plusieurs des étendues suivantes.

| **Portée** | **Définition** | **Accès obligatoire pour afficher les données** | **Paramètre de Contrat Entreprise (EA) prérequis** | **Regroupe les données dans** |
| --- | --- | --- | --- | --- |
| <sup>Premier</sup> compte de facturation | [https://ea.azure.com](https://ea.azure.com/) | Administrateur d'entreprise | None | Abonnements du contrat entreprise |
| department | [https://ea.azure.com](https://ea.azure.com/) | Administrateur de service | Activation de l’option d’**affichage des frais pour l’administrateur de service** | Abonnements appartenant à un compte d’inscription lié au service |
| <sup>Deuxième</sup> compte d’inscription | [https://ea.azure.com](https://ea.azure.com/) | Propriétaire du compte | Activation de l’option d’**affichage des frais pour le propriétaire du compte** | Abonnements à partir du compte d’inscription |
| Groupe d’administration | [https://portal.azure.com](https://portal.azure.com/) | Lecteur Cost Management (ou Lecteur) | Activation de l’option d’**affichage des frais pour le propriétaire du compte** | Abonnements en dessous du groupe d’administration |
| Abonnement | [https://portal.azure.com](https://portal.azure.com/) | Lecteur Cost Management (ou Lecteur) | Activation de l’option d’**affichage des frais pour le propriétaire du compte** | Ressources/groupes de ressources compris dans l’abonnement |
| Resource group | [https://portal.azure.com](https://portal.azure.com/) | Lecteur Cost Management (ou Lecteur) | Activation de l’option d’**affichage des frais pour le propriétaire du compte** | Ressources comprises dans le groupe de ressources |

<sup>1</sup> Le compte de facturation est souvent désigné sous le terme de Contrat Entreprise ou Compte d’inscription.

<sup>2</sup> Le compte d’inscription est souvent désigné sous le terme de propriétaire du compte.


## <a name="other-azure-account-scopes"></a>Autres étendues de compte Azure

Pour afficher les données de coûts d’autres abonnements Azure, un utilisateur doit disposer au moins d’un accès en lecture à une ou plusieurs des étendues suivantes :

- Compte Azure
- Groupe d’administration
- Resource group

Plusieurs étendues sont disponibles après l’intégration de clients à un contrat client Microsoft par des partenaires. Les clients CSP peuvent ensuite utiliser les fonctionnalités de Cost Management si elles sont activées par leur partenaire CSP. Pour plus d’informations, consultez [Prise en main d’Azure Cost Management pour les partenaires](get-started-partners.md).

## <a name="enable-access-to-costs-in-the-azure-portal"></a>Activer l’accès aux coûts dans le portail Azure

L’étendue de service nécessite que l’option **Les administrateurs de service peuvent afficher les frais** soit définie sur **Activée**. Configurez l’option dans le portail Azure ou le portail EA. Toutes les autres étendues nécessitent que l’option **Les propriétaires de comptes peuvent afficher les frais** soit définie sur **Activée**.

Pour activer une option dans le portail Azure :

1. Connectez-vous au portail Azure sur https://portal.azure.com avec un compte administrateur d’entreprise.
1. Sélectionnez l’élément de menu **Gestion des coûts + Facturation**.
1. Sélectionnez **Étendues de facturation** pour voir la liste des étendues et comptes de facturation disponibles.
1. Sélectionnez votre **Compte de facturation** dans la liste des comptes de facturation disponibles.
1. Sous **Paramètres**, sélectionnez l’élément de menu **Stratégies**, puis configurez le paramètre.  
    ![Stratégies d’étendue de facturation avec les options d’affichage des frais](./media/assign-access-acm-data/azure-portal-policies-view-charges.png)

Une fois les options d’affichage des frais activées, la plupart des étendues réclament également une configuration des autorisations avec contrôle d’accès en fonction du rôle Azure (Azure RBAC) sur le Portail Azure.

## <a name="enable-access-to-costs-in-the-ea-portal"></a>Activer l’accès aux coûts dans le portail EA

Pour l’étendue du service, l’option **Affichage des frais pour l’administrateur de service** doit être **activée** sur le portail EA. Configurez l’option dans le portail Azure ou le portail EA. Pour toutes les autres étendues, l’option d’**affichage des frais pour le propriétaire du compte** doit être **activée** dans le portail EA.

Pour activer une option dans le portail EA :

1. Connectez-vous au portail EA sur [https://ea.azure.com](https://ea.azure.com) avec un compte administrateur d’entreprise.
2. Sélectionnez **Gérer** dans le volet gauche.
3. Pour les étendues de Cost Management auxquelles vous souhaitez fournir l’accès, activez l’option d’**affichage des frais pour l’administrateur de service** et/ou d’**affichage des frais pour le propriétaire du compte**.  
    ![Onglet Inscription montrant les options d’affichage des frais pour l’administrateur de service et pour le propriétaire du compte](./media/assign-access-acm-data/ea-portal-enrollment-tab.png)

Une fois les options d’affichage des frais activées, la plupart des étendues réclament également une configuration des autorisations avec contrôle d’accès en fonction du rôle Azure (Azure RBAC) sur le Portail Azure.

## <a name="enterprise-administrator-role"></a>Rôle d’administrateur d’entreprise

Par défaut, un administrateur d’entreprise a accès au compte de facturation (Contrat Entreprise/compte d’inscription) et à toutes les autres étendues, qui sont des étendues enfants. L’administrateur d’entreprise affecte aux autres utilisateurs une autorisation d’accès aux étendues. Pour garantir la continuité de l’activité, le fait d’avoir toujours deux utilisateurs avec un accès d’administrateur d’entreprise est une bonne pratique. Les sections suivantes sont des exemples de procédures pas à pas, où l’administrateur d’entreprise affecte aux autres utilisateurs une autorisation d’accès aux étendues.

## <a name="assign-billing-account-scope-access"></a>Affecter une autorisation d’accès à l’étendue du compte de facturation

L’accès à l’étendue du compte de facturation nécessite une autorisation d’administrateur d’entreprise dans le portail EA. L’administrateur d’entreprise peut voir les coûts pour l’ensemble des inscriptions EA ou pour plusieurs inscriptions. Aucune action n’est nécessaire dans le portail Azure pour l’étendue du compte de facturation.

1. Connectez-vous au portail EA sur [https://ea.azure.com](https://ea.azure.com) avec un compte administrateur d’entreprise.
2. Sélectionnez **Gérer** dans le volet gauche.
3. Sous l’onglet **Inscription**, sélectionnez l’inscription à gérer.  
    ![sélectionner votre inscription dans le portail EA](./media/assign-access-acm-data/ea-portal.png)
4. Sélectionnez **+ Ajouter un administrateur**.
5. Dans la zone Ajouter un administrateur, sélectionnez le type d’authentification, puis tapez l’adresse e-mail de l’utilisateur.
6. Si l’utilisateur doit avoir un accès en lecture seule aux données de coût et d’utilisation, sous **Lecture seule**, sélectionnez **Oui**.  Sinon, sélectionnez **Non**.
7. Sélectionnez **Ajouter** pour créer le compte.  
    ![exemple d’informations indiquées dans la zone Ajouter un administrateur](./media/assign-access-acm-data/add-admin.png)

Un délai pouvant aller jusqu’à 30 minutes est parfois nécessaire pour que le nouvel utilisateur puisse accéder aux données dans Cost Management.

### <a name="assign-department-scope-access"></a>Affecter une autorisation d’accès à l’étendue de service

L’accès à l’étendue de service nécessite un accès d’administrateur de service (affichage des frais pour l’administrateur de service) dans le portail EA. L’administrateur de service peut voir les données de coût et d’utilisation associées à un ou plusieurs services. Les données du service incluent tous les abonnements appartenant à un compte d’inscription et liés au service. Aucune action n’est nécessaire dans le portail Azure.

1. Connectez-vous au portail EA sur [https://ea.azure.com](https://ea.azure.com) avec un compte administrateur d’entreprise.
2. Sélectionnez **Gérer** dans le volet gauche.
3. Sous l’onglet **Inscription**, sélectionnez l’inscription à gérer.
4. Sélectionnez l’onglet **Service**, puis sélectionnez **Ajouter un administrateur**.
5. Dans la zone d’ajout d’un administrateur de service, sélectionnez le type d’authentification, puis tapez l’adresse e-mail de l’utilisateur.
6. Si l’utilisateur doit avoir un accès en lecture seule aux données de coût et d’utilisation, sous **Lecture seule**, sélectionnez **Oui**.  Sinon, sélectionnez **Non**.
7. Sélectionnez les services auxquels vous souhaitez accorder une autorisation d’administration.
8. Sélectionnez **Ajouter** pour créer le compte.  
    ![entrer les informations requises dans la zone d’ajout d’un administrateur de service](./media/assign-access-acm-data/add-depart-admin.png)

## <a name="assign-enrollment-account-scope-access"></a>Affecter une autorisation d’accès à l’étendue du compte d’inscription

L’accès à l’étendue du compte d’inscription nécessite un accès de propriétaire de compte (affichage des frais pour le propriétaire du compte) dans le portail EA. Le propriétaire du compte peut voir les coûts et les données d’utilisation associés aux abonnements créés à partir de ce compte d’inscription. Aucune action n’est nécessaire dans le portail Azure.

1. Connectez-vous au portail EA sur [https://ea.azure.com](https://ea.azure.com) avec un compte administrateur d’entreprise.
2. Sélectionnez **Gérer** dans le volet gauche.
3. Sous l’onglet **Inscription**, sélectionnez l’inscription à gérer.
4. Sélectionnez l’onglet **Compte**, puis sélectionnez **Ajouter un compte**.
5. Dans la zone Ajouter un compte, sélectionnez le **Service** auquel le compte doit être associé, ou laissez ce dernier non affecté.
6. Sélectionnez le type d’authentification, puis tapez le nom du compte.
7. Tapez l’adresse e-mail de l’utilisateur, puis indiquez éventuellement le centre de coûts.
8. Sélectionnez **Ajouter** pour créer le compte.  
    ![entrer les informations requises dans la zone Ajouter un compte pour un compte d’inscription](./media/assign-access-acm-data/add-account.png)

Après les étapes ci-dessus, le compte d’utilisateur devient un compte d’inscription, capable de créer des abonnements, sur Enterprise Portal. L’utilisateur peut accéder aux données de coût et d’utilisation des abonnements qu’il crée.

## <a name="assign-management-group-scope-access"></a>Affecter une autorisation d’accès à l’étendue du groupe d’administration

L’accès permettant de voir l’étendue du groupe d’administration nécessite au minimum une autorisation de lecteur Cost Management (ou lecteur). Vous pouvez configurer les autorisations d’accès à un groupe d’administration sur le Portail Azure. L’autorisation d’administrateur des accès utilisateur (ou propriétaire) du groupe d’administration est nécessaire au minimum pour donner accès à d’autres utilisateurs. Par ailleurs, pour des comptes Azure EA, vous devez également avoir activé le paramètre d’**affichage des frais pour le propriétaire du compte** dans le portail EA.

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Dans la barre latérale, sélectionnez **Tous les services**, recherchez _groupes d’administration_, puis sélectionnez **groupes d’administration**.
3. Sélectionnez le groupe d’administration dans la hiérarchie.
4. En regard du nom de votre groupe d’administration, sélectionnez **Détails**.
5. Dans le volet gauche, sélectionnez **Contrôle d’accès (IAM)** .
6. Sélectionnez **Ajouter**.
7. Sous **Rôle**, sélectionnez **Lecteur Cost Management**.
8. Sous **Attribuer l’accès à**, sélectionnez **Utilisateur, groupe ou application Azure AD**.
9. Pour affecter une autorisation d’accès, recherchez, puis sélectionnez l’utilisateur.
10. Sélectionnez **Enregistrer**.  
    ![exemple d’informations figurant dans la zone Ajouter des autorisations d’un groupe d’administration](./media/assign-access-acm-data/add-permissions.png)

## <a name="assign-subscription-scope-access"></a>Affecter une autorisation d’accès à l’étendue d’abonnement

L’accès à un abonnement requiert au minimum l’autorisation de lecteur Cost Management (ou lecteur). Vous pouvez configurer les autorisations d’accès à un abonnement sur le Portail Azure. L’autorisation d’administrateur des accès utilisateur (ou propriétaire) de l’abonnement est nécessaire au minimum pour donner accès à d’autres utilisateurs. Par ailleurs, pour des comptes Azure EA, vous devez également avoir activé le paramètre d’**affichage des frais pour le propriétaire du compte** dans le portail EA.

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Dans la barre latérale, sélectionnez **Tous les services**, recherchez _abonnements_, puis sélectionnez **Abonnements**.
3. Sélectionnez votre abonnement.
4. Dans le volet gauche, sélectionnez **Contrôle d’accès (IAM)** .
5. Sélectionnez **Ajouter**.
6. Sous **Rôle**, sélectionnez **Lecteur Cost Management**.
7. Sous **Attribuer l’accès à**, sélectionnez **Utilisateur, groupe ou application Azure AD**.
8. Pour affecter une autorisation d’accès, recherchez, puis sélectionnez l’utilisateur.
9. Sélectionnez **Enregistrer**.

## <a name="assign-resource-group-scope-access"></a>Affecter une autorisation d’accès à l’étendue du groupe de ressources

L’accès à un groupe de ressources requiert au minimum l’autorisation de lecteur Cost Management (ou lecteur). Vous pouvez configurer les autorisations d’accès à un groupe de ressources sur le Portail Azure. L’autorisation d’administrateur des accès utilisateur (ou propriétaire) du groupe de ressources est nécessaire au minimum pour donner accès à d’autres utilisateurs. Par ailleurs, pour des comptes Azure EA, vous devez également avoir activé le paramètre d’**affichage des frais pour le propriétaire du compte** dans le portail EA.

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Dans la barre latérale, sélectionnez **Tous les services**, recherchez _groupes de ressources_, puis sélectionnez **Groupes de ressources**.
3. Sélectionnez votre groupe de ressources.
4. Dans le volet gauche, sélectionnez **Contrôle d’accès (IAM)** .
5. Sélectionnez **Ajouter**.
6. Sous **Rôle**, sélectionnez **Lecteur Cost Management**.
7. Sous **Attribuer l’accès à**, sélectionnez **Utilisateur, groupe ou application Azure AD**.
8. Pour affecter une autorisation d’accès, recherchez, puis sélectionnez l’utilisateur.
9. Sélectionnez **Enregistrer**.

## <a name="cross-tenant-authentication-issues"></a>Problèmes d’authentification multilocataires

Actuellement, Azure Cost Management assure une prise en charge limitée de l’authentification multilocataires. Dans certaines circonstances, lorsque vous tentez de vous authentifier entre locataires, vous pouvez recevoir une erreur **Accès refusé** dans l’analyse des coûts. Ce problème peut se produire si vous configurez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour l’abonnement d’un autre locataire et que vous essayez de voir les données de coût.

*Pour contourner le problème* : Après avoir configuré Azure RBAC inter-locataires, patientez une heure. Ensuite, essayez d’afficher les coûts dans l’analyse des coûts ou d’accorder l’accès de la gestion des coûts aux utilisateurs figurant dans les deux locataires.  


## <a name="next-steps"></a>Étapes suivantes

- Si vous n’avez pas encore effectué le premier guide de démarrage rapide relatif à Cost Management, lisez-le à partir de [Démarrer l’analyse des coûts](quick-acm-cost-analysis.md).
