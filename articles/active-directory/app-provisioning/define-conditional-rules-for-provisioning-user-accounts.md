---
title: Approvisionner des applications avec filtres d’étendue | Microsoft Docs
description: Découvrez comment utiliser des filtres d’étendue pour empêcher les objets dans les applications qui prennent en charge l’approvisionnement automatisé des utilisateurs d’être approvisionnés si un objet n’est pas conforme à vos besoins.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: how-to
ms.date: 06/08/2020
ms.author: kenwith
ms.openlocfilehash: 88d004836d5311fc3a971df81a3dc6e6ab605ca9
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861389"
---
# <a name="attribute-based-application-provisioning-with-scoping-filters"></a>Approvisionnement d’applications basé sur les attributs avec filtres d’étendue
L’objectif de cet article est d’expliquer comment utiliser des filtres d’étendue pour définir des règles basées sur des attributs qui déterminent quels utilisateurs sont approvisionnés pour une application.

## <a name="scoping-filter-use-cases"></a>Cas d’utilisation du filtre d’étendue

Un filtre d’étendue permet au service d’approvisionnement Azure Active Directory (Azure AD) d’inclure ou d’exclure tous les utilisateurs dont un attribut correspond à une valeur spécifique. Par exemple, lors d’un approvisionnement d’utilisateurs depuis Azure AD vers une application SaaS utilisée par une équipe de vente, vous pouvez spécifier que seuls les utilisateurs dont l’attribut « Département » est « Ventes » doivent être sujets à l’approvisionnement.

Les filtres d’étendue peuvent être utilisés différemment en fonction du type du connecteur d’approvisionnement :

* **Approvisionnement sortant d’Azure AD vers des applications SaaS**. Quand Azure AD est le système source, [les affectations d’utilisateur et de groupe](../manage-apps/assign-user-or-group-access-portal.md) sont les méthodes les plus courantes pour déterminer les utilisateurs sujets à un approvisionnement. Ces affectations sont également utilisées pour activer l’authentification unique et fournissent une méthode unique pour gérer l’accès et l’approvisionnement. Les filtres d’étendue peuvent être utilisés si vous le souhaitez, en plus des affectations ou à leur place, afin de filtrer les utilisateurs selon des valeurs d’attribut.

    >[!TIP]
    > Vous pouvez désactiver l’approvisionnement basé sur des affectations pour une application d’entreprise en changeant le menu [Étendue](../app-provisioning/user-provisioning.md#how-do-i-set-up-automatic-provisioning-to-an-application) sous les paramètres d’approvisionnement sur **Synchroniser tous les utilisateurs et groupes**. 

* **Approvisionnement entrant des applications HCM vers Azure AD et Active Directory**. Quand une [application HCM telle que Workday](../saas-apps/workday-tutorial.md) est le système source, l’utilisation de filtres d’étendue est la principale méthode pour déterminer les utilisateurs sujets à un approvisionnement de l’application HCM vers Active Directory ou Azure AD.

Par défaut, les connecteurs d’approvisionnement Azure AD ne disposent pas de filtres d’étendue basés sur les attributs. 

## <a name="scoping-filter-construction"></a>Construction d’un filtre d’étendue

Un filtre d’étendue se compose d’une ou plusieurs *clauses*. Les clauses déterminent quels utilisateurs sont autorisés à traverser le filtre d’étendue en évaluant les attributs de chaque utilisateur. Par exemple, l’une de vos clauses peut exiger que l’attribut « État » d’un utilisateur soit égal à « New York », afin que seuls les utilisateurs de New York soient approvisionnés dans l’application. 

Une seule clause définit une condition unique pour une seule valeur d’attribut. Si plusieurs clauses sont créées dans un seul filtre d’étendue, ils sont évalués ensemble en utilisant la logique « ET ». Cela signifie que chacune des clauses doit être considérée « true » pour qu’un utilisateur soit approvisionné.

Enfin, plusieurs filtres d’étendue peuvent être créés pour une seule application. Si plusieurs filtres d’étendue sont présents, ils sont évalués ensemble en utilisant la logique « OU ». Cela signifie que si toutes les clauses présentes dans un seul des filtres d’étendue configurés sont considérées « true », l’utilisateur est approvisionné.

Chaque utilisateur ou groupe traité par le service d’approvisionnement Azure AD est toujours évalué individuellement par rapport à chaque filtre d’étendue.

Par exemple, considérez le filtre d’étendue suivant :

![Filtre d’étendue](./media/define-conditional-rules-for-provisioning-user-accounts/scoping-filter.PNG) 

D’après ce filtre d’étendue, les utilisateurs doivent satisfaire aux critères suivants pour être approvisionnés :

* Ils doivent être de New York.
* Ils doivent travailler dans le service Ingénierie.
* Leur ID d’employé de leur société doit être compris entre 1 000 000 et 2 000 000.
* Leur poste ne doit être null ou vide.

## <a name="create-scoping-filters"></a>Créer des filtres d’étendue
Les filtres d’étendue sont configurés comme parties des mappages d’attributs pour chaque connecteur d’approvisionnement d’utilisateur Azure AD. La procédure suivante suppose que l’approvisionnement automatique soit déjà configuré pour [l’une des applications prises en charge](../saas-apps/tutorial-list.md)et que vous lui ajoutez un filtre d’étendue.

### <a name="create-a-scoping-filter"></a>Créer un filtre d’étendue
1. Dans le [Portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory** > **Applications d’entreprise** > **Toutes les applications**.

2. Sélectionnez l’application pour laquelle vous avez configuré l’approvisionnement automatique. Par exemple, « ServiceNow ».

3. Sélectionnez l’onglet **Approvisionnement**.

4. Dans la section **Mappages** , sélectionnez le mappage pour lequel vous souhaitez configurer un filtre d’étendue : par exemple, « Synchroniser les utilisateurs Azure Active Directory avec ServiceNow ».

5. Sélectionnez le menu **Étendue de l’objet source**.

6. Sélectionnez **Ajouter un filtre d’étendue**.

7. Définissez une clause en sélectionnant un **nom d’attribut** source, un **opérateur** et une **valeur d’attribut** pour effectuer la comparaison. Les opérateurs suivants sont pris en charge :

   a. **EQUALS**. La clause renvoie « true » si l’attribut évalué correspond exactement à la valeur de chaîne d’entrée (respecte la casse).

   b. **NOT EQUALS**. La clause renvoie « true » si l’attribut évalué ne correspond pas à la valeur de chaîne d’entrée (respecte la casse).

   c. **IS TRUE**. La clause renvoie « true » si l’attribut évalué contient une valeur booléenne True.

   d. **IS FALSE**. La clause renvoie « true » si l’attribut évalué contient une valeur booléenne True.

   e. **IS NULL**. La clause renvoie « true » si l’attribut évalué est vide.

   f. **IS NOT NULL**. La clause renvoie « true » si l’attribut évalué n’est pas vide.

   g. **REGEX MATCH**. La clause renvoie « true » si l’attribut évalué correspond à un modèle d’expression régulière. Exemple : ([1-9][0-9]) comprend tout nombre compris entre 10 et 99.

   h. **NOT REGEX MATCH**. La clause renvoie « true » si l’attribut évalué ne correspond pas à un modèle d’expression régulière.
   
   i. **Greater_Than.** La clause renvoie « true » si l’attribut évalué est supérieur à la valeur. La valeur indiquée dans le filtre d’étendue doit être un nombre entier et l’attribut de l’utilisateur doit être un nombre entier [0, 1, 2,...]. 
   
   j. **Greater_Than_OR_EQUALS.** La clause renvoie « true » si l’attribut évalué est supérieur ou égal à la valeur. La valeur indiquée dans le filtre d’étendue doit être un nombre entier et l’attribut de l’utilisateur doit être un nombre entier [0, 1, 2,...]. 
   
   k. **Includes.** La clause renvoie « true » si l’attribut évalué contient la valeur de chaîne (respecte la casse) comme décrit [ici](/dotnet/api/system.string.contains). 


>[!IMPORTANT] 
> - Le filtre IsMemberOf n’est pas pris en charge actuellement.
> - Les opérateurs EGALS et NOT EGALS ne sont pas pris en charge pour les attributs à valeurs multiples

9. Si vous le souhaitez, répétez les étapes 7 et 8 pour ajouter d’autres clauses d’étendues.

10. Dans **Titre du filtre d’étendue**, saisissez un nom pour votre filtre d’étendue.

11. Sélectionnez **OK**.

12. Sélectionnez **OK** à nouveau sur l’écran **Filtres d’étendue**. Si vous le souhaitez, répétez les étapes 6 et 11 pour ajouter d’autres filtres d’étendue.

13. Sélectionnez **Enregistrer** sur l’écran de **mappage d’attributs**. 

>[!IMPORTANT] 
> L’enregistrement d’un nouveau filtre d’étendue déclenche une nouvelle synchronisation complète pour l’application, avec une comparaison de tous les utilisateurs dans le système source par rapport au nouveau filtre d’étendue. Si un utilisateur dans l’application répondait précédemment aux exigences d’un approvisionnement, mais que ce n’est plus le cas maintenant, son compte est désactivé ou déprovisionné dans l’application. Pour remplacer ce comportement par défaut, consultez [Ignorer la suppression des comptes d’utilisateurs qui sortent de l’étendue](../app-provisioning/skip-out-of-scope-deletions.md).


## <a name="common-scoping-filters"></a>Filtres d’étendue communs
| Attribut cible| Opérateur | Valeur | Description|
|----|----|----|----|
|userPrincipalName|REGEX MATCH|.\*@domain.com |Tous les utilisateurs dont userPrincipal a le domaine @domain.com sont concernés par l’approvisionnement|
|userPrincipalName|NOT REGEX MATCH|.\*@domain.com|Tous les utilisateurs dont userPrincipal a le domaine @domain.com ne sont pas concernés par l’approvisionnement|
|department|EQUALS|sales|Tous les utilisateurs du service commercial sont concernés par l’approvisionnement|
|workerID|REGEX MATCH|(1[0-9][0-9][0-9][0-9][0-9][0-9])| Tous les employés dont les workerID sont compris entre 1 000 000 et 2 000 000 sont concernés par l’approvisionnement.|

## <a name="related-articles"></a>Articles connexes
* [Automatisation de l’approvisionnement et de l’annulation de l’approvisionnement des utilisateurs pour les applications SaaS](../app-provisioning/user-provisioning.md)
* [Personnaliser les mappages d’attributs pour l’approvisionnement des utilisateurs](../app-provisioning/customize-application-attributes.md)
* [Écrire des expressions pour les mappages d’attributs](functions-for-customizing-application-data.md)
* [Notifications d’approvisionnement de comptes](../app-provisioning/user-provisioning.md)
* [Utiliser SCIM pour activer la configuration automatique des utilisateurs et des groupes d’Azure Active Directory sur des applications](../app-provisioning/use-scim-to-provision-users-and-groups.md)
* [Liste des didacticiels sur l’intégration des applications SaaS](../saas-apps/tutorial-list.md)
