---
title: Approvisionner un utilisateur à la demande en utilisant Azure Active Directory
description: Forcer la synchronisation
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: how-to
ms.date: 10/01/2020
ms.author: mimart
ms.reviewer: arvinh
ms.openlocfilehash: f7cbe9e9f81b3b71ee7da2feac2908c36f1777e5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91629010"
---
# <a name="on-demand-provisioning"></a>Approvisionnement à la demande
L’approvisionnement à la demande vous permet d’approvisionner un utilisateur dans une application en quelques secondes. Entre autres choses, vous pouvez utiliser cette fonctionnalité pour :

* Résoudre rapidement des problèmes de configuration.
* Valider des expressions que vous avez définies.
* Utiliser des filtres d’étendue.

## <a name="how-to-use-on-demand-provisioning"></a>Utilisation de l’approvisionnement à la demande

1. Connectez-vous au **portail Azure**.
1. Accédez à **tous les services** > **Applications d’entreprise**.
1. Sélectionnez votre application, puis accédez à la page de configuration de l’approvisionnement.
1. Configurez l’approvisionnement en fournissant vos informations d’identification d’administrateur.
1. Sélectionnez **Approvisionner à la demande**.
1. Recherchez un utilisateur par prénom, nom, nom d’affichage, nom d’utilisateur principal ou adresse e-mail.
   > [!NOTE]
   > Pour l’application de provisionnement RH cloud (Workday/SuccessFactors à AD/Azure AD), la valeur d’entrée est différente. Pour le scénario Workday, indiquez le « WID » de l’utilisateur dans Workday. Pour le scénario SuccessFactors, indiquez le « personIdExternal » de l’utilisateur dans SuccessFactors. 
 
1. Sélectionnez **Approvisionner** au bas de la page.

:::image type="content" source="media/provision-on-demand/on-demand-provision-user.jpg" alt-text="Capture d’écran montrant l’interface utilisateur du portail Azure pour approvisionner un utilisateur à la demande.":::

## <a name="understand-the-provisioning-steps"></a>Comprendre les étapes d’approvisionnement

Le processus d’approvisionnement à la demande tente d’afficher les étapes que le service d’approvisionnement effectue lors de l’approvisionnement d’un utilisateur. Il existe généralement cinq étapes pour approvisionner un utilisateur. Une ou plusieurs de ces étapes, expliquées dans les sections suivantes, s’affichent au cours de l’expérience d’approvisionnement à la demande.

### <a name="step-1-test-connection"></a>Étape 1 : Tester la connexion

Le service d’approvisionnement tente d’autoriser l’accès à l’application cible en faisant une requête pour un « utilisateur test ». Le service d’approvisionnement attend une réponse indiquant que le service est autorisé à poursuivre les étapes d’approvisionnement. Cette étape s’affiche uniquement en cas d’échec. Elle n’apparaît pas pendant l’expérience d’approvisionnement à la demande lorsque l’étape aboutit.

#### <a name="troubleshooting-tips"></a>Conseils de dépannage

* Vérifiez que vous avez fourni à l’application cible des informations d’identification valides, telles que le jeton secret et l’URL du locataire. Les informations d’identification requises varient selon l’application. Pour obtenir des didacticiels de configuration détaillés, consultez la [liste des didacticiels](../saas-apps/tutorial-list.md). 
* Assurez-vous que l’application cible prend en charge le filtrage sur les attributs correspondants définis dans le volet **Mappages d’attributs**. Vous devrez peut-être vérifier la documentation de l’API fournie par le développeur de l’application pour comprendre les filtres pris en charge.
* Pour les applications SCIM (System for Cross-domain Identity Management), vous pouvez utiliser un outil tel que Postman. Ces outils vous permettent de vous assurer que l’application répond aux demandes d’autorisation comme le service d’approvisionnement de Azure Active Directory (Azure AD) l’attend. Examinez un [exemple de demande](./use-scim-to-provision-users-and-groups.md#request-3).

### <a name="step-2-import-user"></a>Étape 2 : Importer l'utilisateur

Ensuite, le service d’approvisionnement récupère l’utilisateur à partir du système source. Les attributs utilisateur que le service récupère sont utilisés ultérieurement pour effectuer les tâches suivantes :

* évaluer si l’utilisateur est dans l’étendue d’approvisionnement ;
* vérifier si un utilisateur existant est présent dans le système cible ;
* déterminer les attributs d’utilisateur à exporter vers le système cible.

#### <a name="view-details"></a>Afficher les détails


La section **Afficher les détails** affiche les propriétés de l’utilisateur qui ont été importées à partir du système source (par exemple, Azure AD).

#### <a name="troubleshooting-tips"></a>Conseils de dépannage

* L’importation de l’utilisateur peut échouer lorsque l’attribut correspondant est manquant sur l’objet utilisateur dans le système source. Pour résoudre ce problème, essayez l’une des méthodes suivantes :

  * Mettez à jour l’objet utilisateur avec une valeur pour l’attribut correspondant.
  * Modifiez l’attribut correspondant dans votre configuration d’approvisionnement.

* Si un attribut que vous attendez ne figure pas dans la liste importée, assurez-vous que l’attribut a une valeur sur l’objet utilisateur dans le système source. Actuellement, le service d’approvisionnement ne prend pas en charge l’approvisionnement des attributs Null.
* Assurez-vous que la page **Mappage d’attributs** de votre configuration d’approvisionnement contient l’attribut que vous attendez.

### <a name="step-3-determine-if-user-is-in-scope"></a>Étape 3 : Déterminer si l'utilisateur se trouve dans l'étendue

Ensuite, le service d’approvisionnement détermine si l’utilisateur se trouve dans l’[étendue](./how-provisioning-works.md#scoping) pour l’approvisionnement. Le service vérifie des aspects tels que les suivants :

* si l’utilisateur est affecté à l’application ;
* si l’étendue est définie sur **Synchroniser les utilisateurs affectés** ou **Synchroniser tous les utilisateurs** ;
* filtres d’étendue définis dans votre configuration d’approvisionnement.  

#### <a name="view-details"></a>Afficher les détails

La section **Afficher les détails** affiche les conditions d’étendue qui ont été évaluées. Vous pourriez voir une ou plusieurs des propriétés suivantes :

* **Actif dans le système source** indique que la propriété `IsActive` de l’utilisateur est définie sur la **true** dans Azure AD.
* **Affecté à l’application** indique que l’utilisateur est affecté à l’application dans Azure AD.
* **Scope sync all (Étendue tout synchroniser)** indique que la configuration de l’étendue autorise tous les utilisateurs et les groupes du locataire.
* **User has required role (Rôle obligatoire de l’utilisateur)** indique que l’utilisateur dispose des rôles nécessaires pour être approvisionné dans l’application. 
* Les **Filtres d’étendue** s’affichent également si vous avez défini des filtres d’étendue pour votre application. Le filtre sera affiché au format suivant : {titre de filtre d’étendue} {attribut de filtre d’étendue} {opérateur de filtre d’étendue} {valeur de filtre d’étendue}.

#### <a name="troubleshooting-tips"></a>Conseils de dépannage

* Assurez-vous que vous avez défini un rôle d’étendue valide. Par exemple, évitez d’utiliser l’opérateur [Greater_Than](./define-conditional-rules-for-provisioning-user-accounts.md#create-a-scoping-filter) (Supérieur à) avec une valeur non entière.
* Si l’utilisateur n’a pas le rôle nécessaire, consultez les [conseils pour l’approvisionnement d’utilisateurs affectés au rôle d’accès par défaut](./application-provisioning-config-problem-no-users-provisioned.md#provisioning-users-assigned-to-the-default-access-role).

### <a name="step-4-match-user-between-source-and-target"></a>Étape 4 : Faire correspondre l'utilisateur entre la source et la cible

Dans cette étape, le service tente de faire correspondre l’utilisateur qui a été récupéré à l’étape d’importation avec un utilisateur dans le système cible.

#### <a name="view-details"></a>Afficher les détails

La page **Afficher les détails** affiche les propriétés des utilisateurs qui ont été mis en correspondance dans le système cible. Les propriétés que vous voyez dans le volet contextuel varient comme suit :

* Si aucun utilisateur ne correspond au système cible, aucune propriété n’est visible.
* Si un utilisateur correspond au système cible, voyez les propriétés de l’utilisateur correspondant dans le système cible.
* Si plusieurs utilisateurs sont mis en correspondance, vous voyez les propriétés des deux utilisateurs correspondants.
* Si plusieurs attributs correspondants font partie de vos mappages d’attributs, chaque attribut correspondant est évalué de manière séquentielle, et les utilisateurs correspondants pour cet attribut sont affichés.

#### <a name="troubleshooting-tips"></a>Conseils de dépannage

* Il se peut que le service d’approvisionnement ne puisse pas être mettre en correspondance un utilisateur dans le système source de manière unique avec un utilisateur dans la cible. Pour résoudre ce problème, assurez-vous que l’attribut correspondant est unique.
* Assurez-vous que l’application cible prend en charge le filtrage sur l’attribut défini en tant qu’attribut correspondant.  

### <a name="step-5-perform-action"></a>Étape 5 : Exécuter l’action

Enfin, le service d’approvisionnement effectue une action telle que créer, mettre à jour, supprimer ou ignorer l’utilisateur.

Voici un exemple de ce que vous pouvez voir une fois l’approvisionnement à la demande d’un utilisateur effectué avec succès :

:::image type="content" source="media/provision-on-demand/success-on-demand-provision.jpg" alt-text="Capture d’écran montrant l’interface utilisateur du portail Azure pour approvisionner un utilisateur à la demande.":::

#### <a name="view-details"></a>Afficher les détails

La section **Afficher les détails** affiche les attributs qui ont été modifiés dans l’application cible. Cet affichage représente la sortie finale de l’activité du service d’approvisionnement et des attributs qui ont été exportés. Si cette étape échoue, les attributs affichés représentent les attributs que le service d’approvisionnement a tenté de modifier.

#### <a name="troubleshooting-tips"></a>Conseils de dépannage

* Les défaillances d’exportation des modifications peuvent varier considérablement. Pour les défaillances courantes, consultez la [documentation sur les journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md#error-codes).

## <a name="frequently-asked-questions"></a>Forum aux questions

* **Devez-vous désactiver l’approvisionnement pour utiliser l’approvisionnement à la demande ?** Pour les applications qui utilisent un jeton de porteur à long terme ou un nom d’utilisateur et un mot de passe pour l’autorisation, aucune étape supplémentaire n’est requise. Les applications qui utilisent OAuth pour l’autorisation nécessitent actuellement que le travail d’approvisionnement soit arrêté avant d’utiliser l’approvisionnement à la demande. Les applications telles que G suite, Box, Workplace by Facebook et Slack appartiennent à cette catégorie. Le travail est en cours pour permettre un approvisionnement à la demande pour toutes les applications, sans qu’il soit nécessaire d’arrêter les travaux d’approvisionnement.

* **Quelle est la durée de l’approvisionnement à la demande ?** L’approvisionnement à la demande prend généralement moins de 30 secondes.

## <a name="known-limitations"></a>Limitations connues

Il existe actuellement quelques limitations connues de l’approvisionnement à la demande. Publiez vos [suggestions et commentaires](https://aka.ms/appprovisioningfeaturerequest) afin que nous puissions mieux déterminer les améliorations à apporter à l’étape suivante.

> [!NOTE]
> Les limitations suivantes sont spécifiques de la fonctionnalité d’approvisionnement à la demande. Pour savoir si une application prend en charge l’approvisionnement de groupes, de suppressions ou d’autres fonctionnalités, consultez le didacticiel de cette application.

* L’application Amazon Web Services (AWS) ne prend pas en charge le provisionnement à la demande. 
* L’approvisionnement à la demande de groupes et de rôles n’est pas prise en charge.
* L’approvisionnement à la demande prend en charge la désactivation des utilisateurs dont l’affectation a été annulée à partir de l’application. En revanche, il ne prend pas en charge la désactivation ou la suppression d’utilisateurs qui ont été désactivés ou supprimés d’Azure AD. Ces utilisateurs n’apparaissent pas lorsque vous les recherchez.

## <a name="next-steps"></a>Étapes suivantes

* [Résolution des problèmes d’approvisionnement](./application-provisioning-config-problem.md)