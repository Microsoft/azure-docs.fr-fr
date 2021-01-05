---
title: Modifier les paramètres d’application Azure IoT Central | Microsoft Docs
description: Découvrez comment gérer en tant qu’administrateur votre application Azure IoT Central en changeant son nom et son URL et en chargeant une image, puis supprimer l’application.
author: viv-liu
ms.author: viviali
ms.date: 12/19/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: ece77c2091e7c7c2aa09516bd4d3022bb6fcdfcc
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97796956"
---
# <a name="change-iot-central-application-settings"></a>Modifier les paramètres d’application IoT Central



Cet article explique comment, en tant qu’administrateur, vous pouvez gérer une application en changeant son nom et son URL et en chargeant une image, puis supprimer une application dans votre application Azure IoT Central.

Pour accéder à la section **Administration** et l’utiliser, vous devez avoir le rôle **Administrateur** dans l’application Azure IoT Central. Si vous créez une application Azure IoT Central, le rôle **Administrateur** vous est automatiquement attribué pour cette application.

## <a name="change-application-name-and-url"></a>Modifier le nom et l’URL de l’application

À la page **Paramètres d’application**, vous pouvez modifier le nom et l’URL de votre application, puis sélectionner **enregistrer**.

![Page Paramètres de l’application](media/howto-administer/image0-a.png)

Si votre administrateur crée un thème personnalisé pour votre application, cette page contient une option permettant de masquer le **Nom de l’application** dans l’interface utilisateur. Cette option est utile si le nom de l’application figure dans le logo de l’application inclus dans le thème personnalisé. Pour plus d’informations, consultez [Personnaliser l’interface utilisateur d’Azure IoT Central](./howto-customize-ui.md).

> [!Note]
> Si vous changez votre URL, l’ancienne peut être prise par un autre client Azure IoT Central. Dans ce cas, vous ne pouvez plus l’utiliser. Quand vous changez votre URL, l’ancienne ne fonctionne plus et vous devez indiquer à vos utilisateurs la nouvelle URL à utiliser.

## <a name="delete-an-application"></a>Supprimer une application

Utilisez le bouton **Supprimer** pour supprimer définitivement votre application IoT Central. Cette action supprime définitivement toutes les données qui lui sont associées.

> [!Note]
> Pour pouvoir supprimer une application, vous devez aussi disposer des autorisations permettant de supprimer des ressources de l’abonnement Azure que vous avez choisi lors de la création de l’application. Pour en savoir plus, consultez [Utiliser le contrôle d’accès en fonction du rôle pour gérer l’accès aux ressources de votre abonnement Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="manage-programmatically"></a>Gérer par programme

Des packages de kit de développement logiciel (SDK) Azure Resource Manager d’IoT Central sont disponibles pour Node, Python, C#, Ruby, Java et Go. Vous pouvez utiliser ces packages pour créer, lister, mettre à jour ou supprimer des applications IoT Central. Les packages incluent des ressources d’assistance pour gérer l’authentification et les erreurs.

Vous trouverez des exemples montrant comment utiliser les kits de développement logiciel (SDK) Azure Resource Manager à l’adresse [https://github.com/Azure-Samples/azure-iot-central-arm-sdk-samples](https://github.com/Azure-Samples/azure-iot-central-arm-sdk-samples).

Pour plus d’informations, consultez les packages et dépôts GitHub suivants :

| Langage | Référentiel | Package |
| ---------| ---------- | ------- |
| Nœud | [https://github.com/Azure/azure-sdk-for-node](https://github.com/Azure/azure-sdk-for-node) | [https://www.npmjs.com/package/azure-arm-iotcentral](https://www.npmjs.com/package/azure-arm-iotcentral)
| Python |[https://github.com/Azure/azure-sdk-for-python](https://github.com/Azure/azure-sdk-for-python) | [https://pypi.org/project/azure-mgmt-iotcentral](https://pypi.org/project/azure-mgmt-iotcentral)
| C# | [https://github.com/Azure/azure-sdk-for-net](https://github.com/Azure/azure-sdk-for-net) | [https://www.nuget.org/packages/Microsoft.Azure.Management.IotCentral](https://www.nuget.org/packages/Microsoft.Azure.Management.IotCentral)
| Ruby | [https://github.com/Azure/azure-sdk-for-ruby](https://github.com/Azure/azure-sdk-for-ruby) | [https://rubygems.org/gems/azure_mgmt_iot_central](https://rubygems.org/gems/azure_mgmt_iot_central)
| Java | [https://github.com/Azure/azure-sdk-for-java](https://github.com/Azure/azure-sdk-for-java) | [https://search.maven.org/search?q=a:azure-mgmt-iotcentral](https://search.maven.org/search?q=a:azure-mgmt-iotcentral)
| Go | [https://github.com/Azure/azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go) | [https://github.com/Azure/azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à administrer votre application Azure IoT Central, nous vous recommandons d’en apprendre davantage sur la [gestion des utilisateurs et des rôles](howto-manage-users-roles.md) dans Azure IoT Central.