---
title: API d’accès conditionnel et PowerShell – Azure Active Directory
description: Utilisation de PowerShell et des API d’accès conditionnel Azure AD pour gérer des stratégies telles que du code
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 09/10/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: videor, jeevanb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6d4dd6e64053af41d66e3accbb7fc9b21d951fc6
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860211"
---
# <a name="conditional-access-programmatic-access"></a>Accès conditionnel : Accès par programme

De nombreuses organisations ont exprimé leur besoin de gérer le plus possible leurs environnements, par exemple le code. À l’aide de Microsoft Graph vous pouvez traiter des stratégies d’accès conditionnel comme n’importe quel autre morceau de code dans votre environnement.

Microsoft Graph fournit un modèle de programmabilité unifié que votre organisation peut utiliser pour interagir avec les données dans Microsoft 365, Windows 10 et Enterprise Mobility + Security. Pour plus d’informations sur Microsoft Graph, consultez l’article [Vue d’ensemble de Microsoft Graph](/graph/overview).

![Image indiquant les principales ressources et relations qui font partie du graphe](./media/howto-conditional-access-apis/microsoft-graph.png)

Les exemples suivants sont fournis tel que, sans prise en charge. Vous pouvez utiliser ces exemples comme bases d’outils dans votre organisation. 

Un grand nombre des exemples suivants utilisent des outils tels que les [identités managées](../managed-identities-azure-resources/overview.md), [Logic Apps](../../logic-apps/logic-apps-overview.md), [OneDrive](https://www.microsoft.com/microsoft-365/onedrive/online-cloud-storage), [Teams](https://www.microsoft.com/microsoft-365/microsoft-teams/group-chat-software/) et [Azure Key Vault](../../key-vault/general/overview.md).

## <a name="configure"></a>Configurer

### <a name="powershell"></a>PowerShell

Pour de nombreux administrateurs, PowerShell est déjà un outil de script reconnu. L’exemple suivant montre comment utiliser le [module PowerShell Azure AD](https://www.powershellgallery.com/packages/AzureAD) pour gérer les stratégies d’accès conditionnel.

- [Configurer les stratégies d’accès conditionnel avec les commandes Azure AD PowerShell](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/01-configure/powershell)

### <a name="graph-api"></a>API Graph

Cet exemple montre les options de base disponibles en matière de création, de lecture, de mise à jour et de suppression, dans les API Graph d’accès conditionnel. L’exemple comprend également des modèles JSON que vous pouvez utiliser pour créer quelque exemples de stratégies.

- [Configurer les stratégies d’accès conditionnel avec les appels d’API Microsoft Graph](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/01-configure/graphapi)

### <a name="configure-using-templates"></a>Configurer à l’aide de modèles

Utilisez les API d’accès conditionnel pour déployer des stratégies d’accès conditionnel dans votre environnement de pré-production au moyen d’un modèle.

- [Configurer les stratégies d’accès conditionnel avec les modèles d’API Microsoft Graph](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/01-configure/templates)

## <a name="test"></a>Test

Cet exemple modélise des pratiques de déploiement plus sûres, avec des workflows d’approbation qui peuvent copier des stratégies d’accès conditionnel dans un environnement, tel que la préproduction, vers un autre, comme votre environnement de production.

- [Promouvoir les stratégies d’accès conditionnel à partir des environnements de test](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/02-test)

## <a name="deploy"></a>Déployer

Cet exemple fournit un mécanisme qui sert à mettre en œuvre progressivement des stratégies d’accès conditionnel de déploiement intermédiaire pour votre population d’utilisateurs, ce qui vous permet de gérer l’impact de la prise en charge et de détecter rapidement les problèmes.

- [Déployer les stratégies d’accès conditionnel dans des environnements de production avec des workflows d’approbation](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/03-deploy)

## <a name="monitor"></a>Monitor

Cet exemple fournit un mécanisme permettant de superviser les modifications de stratégie d’accès conditionnel au fil du temps, et capable également de déclencher des alertes lorsque des stratégies clés sont modifiées.

- [Superviser les stratégies d’accès conditionnel déployées pour détecter les modifications et déclencher des alertes](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/04-monitor)

## <a name="manage"></a>Gérer

### <a name="backup-and-restore"></a>Sauvegarde et restauration

Automatisez la sauvegarde et la restauration des stratégies d’accès conditionnel avec des approbations dans Teams à l’aide de cet exemple.

- [Gérer le processus de sauvegarde et de restauration des stratégies d’accès conditionnel à l’aide des appels d’API Microsoft Graph](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/05-manage/01-backup-restore)

### <a name="emergency-access-accounts"></a>Comptes d’accès d’urgence

Plusieurs administrateurs peuvent créer des stratégies d’accès conditionnel, et parfois oublier d’ajouter vos [comptes d’accès d’urgence](../roles/security-emergency-access.md) en exclusion à ces stratégies. Cet exemple vérifie que toutes les stratégies sont mises à jour pour inclure vos comptes d’accès d’urgence désignés.

- [Gérer l’affectation de comptes d’accès d’urgence à des stratégies d’accès conditionnel à l’aide des appels d’API Microsoft Graph](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/05-manage/02-emergency-access)

### <a name="contingency-planning"></a>Planification des impondérables

Les choses ne fonctionnent pas toujours comme vous le souhaitez, et lorsque cela se produit, vous avez besoin d’un moyen qui vous permet de revenir à un état dans lequel le travail peut se poursuivre. L’exemple suivant vous permet de restaurer vos stratégies dans un plan d’urgence fiable connu, et de désactiver d’autres stratégies d’accès conditionnel.

- [Gérer l’activation des stratégies d’urgence d’accès conditionnel à l’aide des appels d’API Microsoft Graph](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/05-manage/03-contingency)

## <a name="community-contribution"></a>Contribution de la communauté

Ces exemples sont disponibles sur notre [dépôt GitHub](https://github.com/Azure-Samples/azure-ad-conditional-access-apis). Nous sommes heureux d’accueillir les contributions de la communauté par l’intermédiaire des sections Issues et Pull Requests sur GitHub.

## <a name="next-steps"></a>Étapes suivantes

- [Overview of Microsoft Graph (Vue d’ensemble de Microsoft Graph)](/graph/overview)

- [API d’accès conditionnel](/graph/api/resources/conditionalaccesspolicy)

- [API d’emplacement nommé](/graph/api/resources/namedlocation)
