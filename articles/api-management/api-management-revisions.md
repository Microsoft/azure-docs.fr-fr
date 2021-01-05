---
title: Révisions dans Gestion des API Azure | Microsoft Docs
description: Découvrez le concept des révisions dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: johndowns
ms.service: api-management
ms.topic: article
ms.date: 06/12/2020
ms.author: jodowns
ms.custom: fasttrack-new
ms.openlocfilehash: 9a20a7966daff372cf5c0abc9b7b1dbbfd459838
ms.sourcegitcommit: dc68a2c11bae2e9d57310d39fbed76628233fd7f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2020
ms.locfileid: "91403236"
---
# <a name="revisions-in-azure-api-management"></a>Révisions dans Gestion des API Azure

Les révisions vous permettent d’apporter des modifications à vos API de manière contrôlée et sécurisée. Lorsque vous souhaitez apporter des modifications, créez une nouvelle révision. Vous pouvez ensuite modifier et tester l’API, sans perturber vos consommateurs d’API. Lorsque vous êtes prêt, vous rendez votre révision actuelle. Ce faisant, vous pouvez publier une entrée dans le journal des modifications pour informer vos consommateurs d’API des changements. Le journal des modifications est publié sur le portail des développeurs.

> [!NOTE]
> Le portail des développeurs n’est pas disponible dans le niveau de consommation.

Avec les révisions, vous pouvez :

- Apporter des modifications en toute sécurité à vos définitions et stratégies d’API, sans affecter votre API de production.
- Essayer des modifications avant de les publier.
- Documenter les modifications que vous apportez pour permettre aux développeurs de comprendre les nouveautés.
- Procéder à une restauration en cas de problèmes.

[Prenez en main les révisions en suivant notre procédure pas à pas.](./api-management-get-started-revise-api.md)

## <a name="accessing-specific-revisions"></a>Accès à des révisions spécifiques

Chaque révision de votre API est accessible à l’aide d’une URL spécialement formée. Ajoutez `;rev={revisionNumber}` à la fin de votre URL d’API, avant la chaîne de requête, pour accéder à une révision spécifique de cette API. Par exemple, vous pouvez utiliser cette URL pour accéder à la révision 3 de l’API `customers` :

`https://apis.contoso.com/customers;rev=3?customerId=123`

Par défaut, chaque révision présente les mêmes paramètres de sécurité que la révision actuelle. Vous pouvez modifier délibérément les stratégies d’une révision spécifique pour appliquer une sécurité différente à chaque révision. Par exemple, vous souhaiterez peut-être ajouter une [stratégie de filtrage d’adresses IP](./api-management-access-restriction-policies.md#RestrictCallerIPs) pour empêcher les appelants externes d’accéder à une révision en cours de développement.

Une révision peut être mise hors connexion, ce qui la rend inaccessible aux appelants, même s’ils essaient d’accéder à la révision via son URL. Vous pouvez marquer une révision comme hors connexion à l’aide du portail Azure. Si vous utilisez PowerShell, vous pouvez utiliser la cmdlet `Set-AzApiManagementApiRevision` et définir l’argument `Path` sur `$null`.

> [!NOTE]
> Nous vous suggérons de procéder aux révisions hors connexion lorsque vous ne les utilisez pas à des fins de test.

## <a name="current-revision"></a>Révision actuelle

Une seule révision peut être définie en tant que révision *actuelle*. Cette révision sera utilisée pour toutes les requêtes d’API ne spécifiant pas de numéro de révision explicite dans l’URL. Vous pouvez restaurer une révision précédente en définissant cette révision comme actuelle.

Vous pouvez définir une révision comme actuelle à l’aide du portail Azure. Si vous utilisez PowerShell, vous pouvez utiliser la cmdlet `New-AzApiManagementApiRelease`.

## <a name="revision-descriptions"></a>Descriptions des révisions

Lorsque vous créez une révision, vous pouvez définir une description à des fins de suivi personnel. Les descriptions ne sont pas lues par les utilisateurs de votre API.

Lorsque vous définissez une révision comme actuelle, vous pouvez également spécifier une note de journal des modifications public. Le journal des modifications est disponible dans le portail des développeurs pour permettre à vos utilisateurs d’API de le consulter. Vous pouvez modifier votre note de journal des modifications à l’aide de la cmdlet PowerShell `Update-AzApiManagementApiRelease`.

## <a name="versions-and-revisions"></a>Versions et révisions

Les versions et les révisions constituent des fonctionnalités distinctes. Chaque version peut présenter plusieurs révisions, tout comme il peut exister une API sans version. Vous pouvez utiliser des révisions sans utiliser de versions ou inversement. En général, les versions sont utilisées pour distinguer les versions d’API avec changements cassants, tandis que les révisions peuvent être utilisées pour les changements mineurs et non cassants apportés à une API.

Si vous pensez que votre révision présente des changements cassants ou si vous souhaitez formellement faire évoluer votre révision en version bêta/test, vous pouvez créer une version à partir d’une révision. À l’aide du portail Azure, cliquez sur « Créer une version à partir de la révision » dans le menu contextuel de révision sous l’onglet Révisions.
