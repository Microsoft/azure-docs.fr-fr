---
title: Présentation des licences basées sur le groupe - Azure Active Directory | Microsoft Docs
description: Apprenez-en plus sur les licences basées sur les groupes Azure Active Directory, leur fonctionnement et les bonnes pratiques
services: active-directory
keywords: Gestion des licences Azure AD
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.topic: conceptual
ms.workload: identity
ms.date: 10/29/2018
ms.author: ajburnle
ms.reviewer: krbain
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0490334c759da6ef7ba7ff2535f5f561cdb7a9bf
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369809"
---
# <a name="what-is-group-based-licensing-in-azure-active-directory"></a>À quoi correspondent les licences basées sur les groupes dans Azure Active Directory ?

Les services cloud payants Microsoft, comme Microsoft 365, Enterprise Mobility + Security, Dynamics 365 et d’autres produits similaires, requièrent des licences. Ces licences sont affectées à chaque utilisateur qui a besoin d’accéder à ces services. Pour gérer les licences, les administrateurs utilisent l’un des portails de gestion (Office ou Azure) et des applets de commande PowerShell. Azure Active Directory (Azure AD) est l’infrastructure de base qui prend en charge la gestion de tous les services cloud Microsoft. Azure AD stocke des informations sur les états d’affectation de licence pour les utilisateurs.

Jusqu’à présent, les licences ne pouvaient être affectées qu’au niveau de chaque utilisateur, ce qui pouvait rendre difficile la gestion à grande échelle. Par exemple, pour ajouter ou supprimer des licences utilisateur en raison d’évolutions de l’organisation, par exemple l’arrivée ou le départ d’utilisateurs dans l’organisation ou dans un service, l’administrateur devait souvent écrire un script PowerShell complexe. Ce script permet d’effectuer des appels individuels au service cloud.

Pour relever ces défis, Azure AD inclut maintenant une gestion des licences par groupe. Vous pouvez affecter une ou plusieurs licences de produits à un groupe. Azure AD permet de garantir que les licences sont affectées à tous les membres du groupe. Tous les nouveaux membres qui rejoignent le groupe se voient affecter les licences appropriées. Lorsqu’ils quittent le groupe, ces licences sont supprimées. Ceci élimine toute nécessité d’automatiser la gestion des licences avec PowerShell pour refléter les évolutions de la structure de l’organisation et des services utilisateur par utilisateur.

## <a name="licensing-requirements"></a>Exigences en termes de licence
Vous devez disposer d’une des licences suivantes pour utiliser les licences basées sur le groupe :

- Abonnement d’essai ou payant pour Azure AD Premium P1 et versions ultérieures

- Édition payante ou d’évaluation d’Office 365 E3, d’Office 365 A3, d’Office 365 GCC G3, d’Office 365 E3 pour GCCH ou d’Office 365 E3 pour DOD et versions ultérieures

### <a name="required-number-of-licenses"></a>Nombre de licences requis
Pour tous les groupes auxquels une licence est attribuée, chaque membre unique doit également disposer d’une licence. Même si vous n’êtes pas obligé d’attribuer une licence à chaque membre du groupe, vous devez disposer de suffisamment de licences pour inclure tous les membres. Par exemple, si vous avez 1 000 membres uniques dans des groupes sous licence dans votre locataire, vous devez disposer d’au moins 1 000 licences pour respecter le contrat de licence.

## <a name="features"></a>Fonctionnalités

Voici les principales fonctionnalités de la gestion des licences par groupe :

- Les licences peuvent être affectées à n’importe quel groupe de sécurité dans Azure AD. Les groupes de sécurité peuvent être synchronisés en local à l’aide d’Azure AD Connect. Vous pouvez également créer des groupes de sécurité directement dans Azure AD (ils sont également appelé groupes cloud purs) ou automatiquement avec la fonctionnalité de groupe dynamique d’Azure AD.

- Lorsqu’une licence de produit est affectée à un groupe, l’administrateur peut désactiver un ou plusieurs plans de services dans le produit. En règle générale, cette affectation a lieu lorsque l’organisation n’est pas encore prête à utiliser un service inclus dans un produit. Par exemple, l’administrateur peut affecter Microsoft 365 à un service, mais désactiver temporairement le service Yammer.

- Tous les services de cloud computing Microsoft nécessitant des licences au niveau des utilisateurs sont pris en charge. Cette prise en charge comprend tous les produits Microsoft 365, Enterprise Mobility + Security et Dynamics 365.

- Pour le moment, les licences basées sur les groupes sont uniquement accessibles via le [portail Azure](https://portal.azure.com). Si vous utilisez principalement d'autres portails pour la gestion des utilisateurs et des groupes, comme le [Centre d'administration Microsoft 365](https://admin.microsoft.com), vous pouvez continuer à le faire. Vous devez toutefois utiliser le portail Azure pour gérer les licences au niveau du groupe.

- Azure AD gère automatiquement les modifications de licences qui résultent de modifications de l’appartenance aux groupes. En règle générale, les modifications de licence sont effectives quelques minutes après une modification d’appartenance.

- Un utilisateur peut être membre de plusieurs groupes dans lesquels des stratégies de licences sont spécifiées. Un utilisateur peut également disposer de licences affectées directement, en dehors de groupes. L’état utilisateur qui en résulte est une combinaison de toutes les licences de produits et services affectées. Si une même licence est affectée à un utilisateur à partir de plusieurs sources, la licence n’est consommée qu’une seule fois.

- Dans certains cas, des licences ne peuvent pas être affectées à un utilisateur. Par exemple, les licences disponibles dans le client peuvent ne pas être suffisantes, ou des services conflictuels peuvent avoir été affectés simultanément. Les administrateurs ont accès aux informations sur les utilisateurs pour lesquels Azure AD n’a pas pu traiter entièrement les licences de groupes. Ils peuvent prendre des mesures correctives en fonction de ces informations.

## <a name="your-feedback-is-welcome"></a>Vos commentaires sont les bienvenus.

Si vous avez des commentaires ou des demandes de fonctionnalités, partagez-les avec nous via le [forum d’administrateur Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=162510).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur d’autres scénarios de gestion des licences par le biais des licences basées sur les groupes, consultez :

* [Affectation de licences à un groupe dans Azure Active Directory](../enterprise-users/licensing-groups-assign.md)
* [Identification et résolution des problèmes de licence pour un groupe dans Azure Active Directory](../enterprise-users/licensing-groups-resolve-problems.md)
* [Migration des utilisateurs individuels sous licence vers les licences basées sur les groupes dans Azure Active Directory](../enterprise-users/licensing-groups-migrate-users.md)
* [Guide pratique pour migrer des utilisateurs entre des licences de produit à l’aide de la gestion de licences basée sur des groupes dans Azure Active Directory](../enterprise-users/licensing-groups-change-licenses.md)
* [Azure Active Directory group-based licensing additional scenarios (Autres scénarios de licence basée sur le groupe Azure Active Directory)](../enterprise-users/licensing-group-advanced.md)
* [Exemples PowerShell pour les licences basées sur les groupes dans Azure Active Directory](../enterprise-users/licensing-ps-examples.md)
