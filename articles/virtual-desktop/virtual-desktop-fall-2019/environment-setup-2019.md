---
title: Environnement Windows Virtual Desktop (classique) – Azure
description: Éléments de base d’un environnement Windows Virtual Desktop (classique).
author: Heidilohr
ms.topic: conceptual
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 4fa281f6435013e9feb6808aab04ffa69c5f96c4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88008608"
---
# <a name="windows-virtual-desktop-classic-environment"></a>Environnement Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../environment-setup.md).

Windows Virtual Desktop est un service qui offre aux utilisateurs un accès facile et sécurisé à leurs postes de travail virtualisés et à leurs applications distantes. Cette rubrique fournit des informations supplémentaires sur la structure générale de l’environnement Windows Virtual Desktop.

## <a name="tenants"></a>Locataires

Le locataire Windows Virtual Desktop est l’interface principale pour la gestion de votre environnement Windows Virtual Desktop. Chaque locataire Windows Virtual Desktop doit être associé à l’Azure Active Directory contenant les utilisateurs qui se connecteront à l’environnement. À partir du locataire Windows Virtual Desktop, vous pouvez commencer à créer des pools hôtes pour exécuter des charges de travail de vos utilisateurs.

## <a name="host-pools"></a>Pools hôtes

Un pool hôte est une collection de machines virtuelles Azure qui s’inscrivent auprès de Windows Virtual Desktop en tant qu’hôtes de session lorsque vous exécutez l’agent Windows Virtual Desktop. Pour que l’expérience utilisateur soit cohérente, toutes les machines virtuelles hôtes de session dans un pool hôte doivent provenir de la même image.

Il existe deux types de pools hôtes :

- Personnel, où chaque hôte de session est affecté à des utilisateurs individuels.
- Mis en pool, où les hôtes de session peuvent accepter des connexions de tout utilisateur autorisé à un groupe d’applications au sein du pool hôte.

Vous pouvez définir des propriétés supplémentaires sur le pool hôte pour modifier son comportement d’équilibrage de charge, le nombre de sessions que chaque hôte de session peut prendre, et ce que l’utilisateur peut faire sur les hôtes de session dans le pool hôte quand il est connecté à leurs sessions Windows Virtual Desktop. Vous contrôlez les ressources publiées pour les utilisateurs au travers de groupes d’applications.

## <a name="app-groups"></a>Groupes d’applications

Un groupe d’applications est un regroupement logique d’applications installées sur des hôtes de session dans le pool hôte. Il existe deux types de groupes d’applications :

- RemoteApp, où les utilisateurs accèdent aux applications distantes que vous sélectionnez individuellement et publiez sur le groupe d’applications.
- Bureau, où les utilisateurs accèdent à l’intégralité du bureau.

Par défaut, un groupe de l’application Bureau est créé automatiquement chaque fois que vous créez un pool hôte. Vous pouvez supprimer ce groupe d’application à tout moment. En revanche, vous ne pouvez pas créer d’autre groupe d’applications de bureau dans le pool hôte tant que celui-ci contient déjà groupe d’applications de bureau. Pour publier des applications distantes, vous devez créer un groupe d’applications RemoteApp. Vous pouvez créer plusieurs groupes d’applications RemoteApp en fonction des différents scénarios de travail. Les différents groupes d’applications RemoteApp peuvent également contenir des applications distantes se chevauchant.

Pour publier des ressources pour des utilisateurs, vous devez affecter ceux-ci à des groupes d’applications. Lors de l’affectation d’utilisateurs à des groupes d’applications, tenez compte des considérations suivantes :

- Un utilisateur ne peut pas être affecté à la fois à un groupe d’applications Bureau et à un groupe d’applications RemoteApp au sein du même pool hôte.
- Un utilisateur peut être affecté à plusieurs groupes d’applications au sein du même pool hôte, dont les flux sont cumulés.

## <a name="tenant-groups"></a>Groupes de locataires

Dans Windows Virtual Desktop, c’est au niveau du locataire Windows Virtual Desktop qu’ont lieu la majeure partie des opérations d’installation et de configuration. Le locataire Windows Virtual Desktop contient les pools hôtes, les groupes d’applications et les affectations d’utilisateurs de groupe d’applications. Toutefois, il peut y avoir des situations où vous devez gérer plusieurs locataires Windows Virtual Desktop à la fois, en particulier si vous êtes un fournisseur de services cloud (CSP) ou un partenaire d’hébergement. Dans ces situations, vous pouvez utiliser un groupe de locataires Windows Virtual Desktop personnalisé pour y placer les locataires Windows Virtual Desktop de chaque client et gérer les accès de manière centralisée. Toutefois, si vous ne gérez qu’un seul locataire Windows Virtual Desktop, le concept de groupe de locataires ne s’applique pas et vous pouvez continuer à exploiter et gérer votre locataire existant dans le groupe de locataires par défaut.

## <a name="end-users"></a>Utilisateurs finaux

Une fois que vous avez affecté des utilisateurs à leurs groupes d’applications, ces utilisateurs peuvent se connecter à un déploiement Windows Virtual Desktop avec n’importe lequel des clients Windows Virtual Desktop.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur l’accès délégué et sur la manière d’attribuer des rôles aux utilisateurs en consultant [Accès délégué dans Windows Virtual Desktop](delegated-access-virtual-desktop-2019.md).

Pour apprendre à configurer votre locataire Windows Virtual Desktop, consultez [Créer un locataire dans Windows Virtual Desktop](tenant-setup-azure-active-directory.md).

Pour savoir comment vous connecter à un Windows Virtual Desktop, voir les articles suivants :

- [Se connecter à partir de Windows 10 ou Windows 7](connect-windows-7-10-2019.md)
- [Se connecter à partir d’un navigateur web](connect-web-2019.md)
