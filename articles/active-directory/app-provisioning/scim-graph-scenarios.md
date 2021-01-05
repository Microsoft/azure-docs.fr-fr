---
title: Utilisez SCIM, Microsoft Graph et Azure AD pour approvisionner les utilisateurs et enrichir les applications avec des données
description: Utiliser SCIM et Microsoft Graph ensemble pour approvisionner des utilisateurs et enrichir votre application avec les données dont elle a besoin.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: reference
ms.date: 04/26/2020
ms.author: kenwith
ms.reviewer: arvinh, celested
ms.openlocfilehash: ba8dec19443bbafa300144b1aa42b3973cce9e6d
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97655150"
---
# <a name="using-scim-and-microsoft-graph-together-to-provision-users-and-enrich-your-application-with-the-data-it-needs"></a>Utiliser SCIM et Microsoft Graph ensemble pour provisionner des utilisateurs et enrichir votre application avec les données dont elle a besoin.

**Public cible :** Cet article est destiné aux développeurs qui créent des applications intégrées à Azure Active Directory (Azure AD). Si vous envisagez d’utiliser des applications déjà intégrées à Azure AD, telles que Zoom, ServiceNow et Dropbox, vous pouvez ignorer cet article et passer en revue les [didacticiels](../saas-apps/tutorial-list.md) spécifiques aux applications ou passer en revue la [manière dont le service de provisionnement fonctionne](./how-provisioning-works.md).

**Scénarios courants**

Azure AD fournit un service prêt à l’emploi pour l’approvisionnement et une plateforme extensible sur laquelle créer vos applications. L’arbre de décision décrit comment un développeur utilise [SCIM](https://aka.ms/scimoverview) et le [Microsoft Graph](/graph/overview) pour automatiser l’approvisionnement. 

> [!div class="checklist"]
> * Créer automatiquement des utilisateurs dans mon application
> * Supprimer automatiquement des utilisateurs de mon application quand ils ne doivent plus y avoir accès
> * Intégrer mon application à plusieurs fournisseurs d’identité pour le provisionnement
> * Enrichir mon application avec des données provenant de services Microsoft tels que Teams, Outlook et Office.
> * Créer, mettre à jour et supprimer automatiquement des utilisateurs et des groupes dans Azure AD et Active Directory

![Arbre de décision SCIM Graph](./media/user-provisioning/scim-graph.png)

## <a name="scenario-1-automatically-create-users-in-my-app"></a>Scénario 1 : Créer automatiquement des utilisateurs dans mon application
Aujourd’hui, les administrateurs informatiques approvisionnent les utilisateurs en créant manuellement des comptes d’utilisateur ou en chargeant régulièrement des fichiers CSV dans l’application. Le processus prend du temps pour les clients et ralentit l’adoption de mon application. J’ai besoin uniquement des informations utilisateur de base telles que le nom, l’e-mail et le nom d’utilisateur principal pour créer un utilisateur. 

**Recommandation** : 
* Si vos clients utilisent différents fournisseurs et que vous ne souhaitez pas maintenir un moteur de synchronisation pour intégrer chacun d’eux, prenez en charge un point de terminaison [/Users](https://aka.ms/scimreferencecode) conforme SCIM. Vos clients peuvent facilement utiliser ce point de terminaison pour l’intégrer au service de provisionnement Azure AD et créer automatiquement des comptes d’utilisateur quand ils ont besoin d’un accès. Vous pouvez créer le point de terminaison une fois et il est compatible avec tous les fournisseurs d’identité. Consultez l’exemple de demande ci-dessous pour savoir comment créer un utilisateur avec SCIM.
* Si vous avez besoin de données utilisateur qui se trouvent sur l’objet utilisateur dans Azure AD et d’autres données de Microsoft, envisagez de créer un point de terminaison SCIM pour l’approvisionnement des utilisateurs et d’appeler Microsoft Graph pour obtenir le reste des données. 

```json
POST /Users
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"],
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "userName": "BillG",
    "active": true,
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "Bill Gates",
        "familyName": "Gates",
        "givenName": "Bill"
    },
    "roles": []
}
```

## <a name="scenario-2-automatically-remove-users-from-my-app"></a>Scénario 2 : Supprimer automatiquement des utilisateurs de mon application
Les clients qui utilisent mon application ont la sécurité à cœur et ont des exigences de gouvernance pour supprimer les comptes lorsque les employés n’en ont plus besoin. Comment puis-je automatiser le déprovisionnement à partir de mon application ?

**Recommandation :** prendre en charge un point de terminaison /Users compatible avec SCIM. Le service de provisionnement Azure AD envoie des demandes de désactivation et de suppression lorsque l’utilisateur ne doit plus avoir d’accès. Nous vous recommandons de prendre en charge à la fois la désactivation et la suppression des utilisateurs. Consultez les exemples ci-dessous pour savoir à quoi ressemble une demande de désactivation et de suppression. 

Désactiver l’utilisateur
```json
PATCH /Users/5171a35d82074e068ce2 HTTP/1.1
{
    "Operations": [
        {
            "op": "Replace",
            "path": "active",
            "value": false
        }
    ],
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ]
}
```
Supprimer un utilisateur
```json
DELETE /Users/5171a35d82074e068ce2 HTTP/1.1
```

## <a name="scenario-3-automate-managing-group-memberships-in-my-app"></a>Scénario 3 : Automatiser la gestion des appartenances aux groupes dans mon application
Mon application s’appuie sur des groupes pour accéder à diverses ressources, et les clients veulent réutiliser les groupes qu’ils possèdent dans Azure AD. Comment puis-je importer des groupes à partir d’Azure AD et les tenir à jour à mesure que les appartenances changent ?  

**Recommandation :** prendre en charge un point de terminaison [/Groups](https://aka.ms/scimreferencecode) compatible avec SCIM. Le service de provisionnement Azure AD prend en charge la création de groupes et la gestion des mises à jour d’appartenance dans votre application. 

## <a name="scenario-4-enrich-my-app-with-data-from-microsoft-services-such-as-teams-outlook-and-onedrive"></a>Scénario 4 : Enrichir mon application avec des données provenant de services Microsoft tels que Teams, Outlook et OneDrive
Mon application est intégrée à Microsoft Teams et s’appuie sur les données de message. En outre, nous stockons des fichiers pour les utilisateurs dans OneDrive. Comment puis-je enrichir mon application avec les données de ces services et de Microsoft ?

**Recommandation :** [Microsoft Graph](/graph/) est votre point d’entrée pour accéder aux données Microsoft. Chaque charge de travail expose les API avec les données dont vous avez besoin. Microsoft Graph peut être utilisé avec [le provisionnement SCIM](./use-scim-to-provision-users-and-groups.md) pour les scénarios ci-dessus. Vous pouvez utiliser SCIM pour provisionner des attributs d’utilisateur de base dans votre application lors de l’appel à Graph pour récupérer d’autres données dont vous avez besoin. 

## <a name="scenario-5-track-changes-in-microsoft-services-such-as-teams-outlook-and-azure-ad"></a>Scénario 5 : Suivre les modifications apportées dans des services Microsoft, tels que Teams, Outlook et Azure AD
Je dois pouvoir suivre les modifications apportées aux messages Teams et Outlook et réagir en temps réel. Comment puis-je faire en sorte que ces modifications soient envoyées à mon application ?

**Recommandation :** Microsoft Graph fournit des [notifications de modifications](/graph/webhooks) et le [suivi des modifications](/graph/delta-query-overview) pour différentes ressources. Notez les limitations suivantes des notifications de modifications :
- Si un récepteur d’événements accuse réception d’un événement, mais échoue pour une raison quelconque, l’événement peut être perdu.
- L’ordre dans lequel les modifications sont reçues n’est pas obligatoirement chronologique.
- Les notifications de modifications ne contiennent pas toujours de [données de ressources](/graph/webhooks-with-resource-data). Pour les raisons ci-dessus, les développeurs utilisent souvent des notifications de modifications avec le suivi des modifications pour les scénarios de synchronisation. 

## <a name="scenario-6-provision-users-and-groups-in-azure-ad"></a>Scénario 6 : Provisionner des utilisateurs et groupes dans Azure AD
Mon application crée des informations sur un utilisateur dont les clients ont besoin dans Azure AD. Il peut s’agir d’une application RH qui gère l’embauche, d’une application de communication qui crée des numéros de téléphone pour les utilisateurs ou de toute autre application qui génère des données qui seraient précieuses dans Azure AD. Comment remplir l’enregistrement utilisateur dans Azure AD avec ces données ? 

**Recommandation** : Microsoft Graph expose les points de terminaison/Users et /Groups que vous pouvez intégrer pour provisionner des utilisateurs dans Azure AD. Notez qu’Azure Active Directory ne prend pas en charge la réécriture de ces utilisateurs dans Active Directory. 

> [!NOTE]
> Microsoft dispose d’un service de provisionnement qui extrait les données des applications RH, telles que Workday et SuccessFactors. Ces intégrations sont créées et gérées par Microsoft. Pour l’intégration d’une nouvelle application RH à notre service, vous pouvez envoyer une demande sur [UserVoice](https://feedback.azure.com/forums/374982-azure-active-directory-application-requests). 

## <a name="related-articles"></a>Articles connexes

- [Consultez la documentation Microsoft Graph sur la synchronisation](/graph/api/resources/synchronization-overview?view=graph-rest-beta)
- [Intégrer une application SCIM personnalisée à Azure AD](use-scim-to-provision-users-and-groups.md)