---
title: Synchronisation SCIM avec Active Directory
description: Guide architectural sur la réalisation d'une synchronisation SCIM avec Azure Active Directory.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/10/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: b95aac504bc6ee72c353faecad25384e2dc90840
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96172416"
---
# <a name="scim-synchronization-with-azure-active-directory"></a>Synchronisation SCIM avec Active Directory

SCIM (System for Cross-domain Identity Management) est un protocole ouvert standard permettant d’automatiser l’échange d’informations d’identité d’utilisateur entre les domaines d’identité et les systèmes informatiques. SCIM garantit que les employés ajoutés au système HCM (Human Capital Management) disposent automatiquement de comptes créés dans Azure Active Directory (Azure AD) ou Windows Server Active Directory. Les profils et attributs utilisateur sont synchronisés entre les deux systèmes, mettant à jour et supprimant des utilisateurs en fonction de l’état de l’utilisateur ou de la modification du rôle.

La spécification SCIM est une définition normalisée de deux points de terminaison : un point de terminaison/Users et un point de terminaison/Groups. Il utilise des verbes REST communs pour créer, mettre à jour et supprimer des objets. Il utilise également un schéma prédéfini pour les attributs courants tels que le nom de groupe, le nom d’utilisateur, le prénom, le nom et l'adresse e-mail. Les applications proposant une API REST SCIM 2.0 peuvent réduire ou éliminer les difficultés liées à l’utilisation d’une API ou de produits de gestion des utilisateurs propriétaires. Par exemple, tout client SCIM conforme peut effectuer une requête HTTP POST d’objet JSON sur le point de terminaison/Users afin de créer une nouvelle entrée d’utilisateur. Sans devoir utiliser une API légèrement différente pour les mêmes actions de base, les applications conformes à la norme SCIM peuvent instantanément tirer parti des clients, des outils et du code existants. 

## <a name="use-when"></a>Contexte d'utilisation : 

Vous souhaitez approvisionner automatiquement les informations utilisateur à partir d’un système HCM pour Azure AD et Windows Server Active Directory, puis pour cibler les systèmes si nécessaire. 

![Diagramme architectural](./media/authentication-patterns/scim-auth.png)


## <a name="components-of-system"></a>Composants du système 

* **Système HCM** : Applications et technologies qui permettent de gérer les processus et pratiques de gestion du capital humain (HCM) et automatisent les processus RH tout au long du cycle de vie des employés. 

* **Service d’approvisionnement Azure AD** : Utilise le protocole SCIM 2.0 pour l’approvisionnement automatique. Le service se connecte au point de terminaison SCIM de l’application, et il utilise le schéma d’objet utilisateur SCIM et les API REST pour automatiser le provisionnement et le déprovisionnement des utilisateurs et des groupes.  

* **Azure AD** : Le référentiel d’utilisateurs utilisé pour la gestion du cycle de vie des identités et de leurs droits. 

* **Système cible** : Application ou système disposant d’un point de terminaison SCIM et qui fonctionne avec l’approvisionnement Azure AD pour activer l’approvisionnement automatique des utilisateurs et groupes.  

## <a name="implement-scim-with-azure-ad"></a>Implémenter SCIM avec Azure AD 

* [Fonctionnement de l’approvisionnement dans Azure AD ](../app-provisioning/how-provisioning-works.md)

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise dans le portail Azure](../app-provisioning/configure-automatic-user-provisioning-portal.md)

* [Créer un point de terminaison SCIM et configurer l’attribution des utilisateurs à l’aide d’Azure AD](../app-provisioning/use-scim-to-provision-users-and-groups.md)

* [Conformité du protocole SCIM 2.0 du service d’approvisionnement Azure AD](../app-provisioning/application-provisioning-config-problem-scim-compatibility.md)