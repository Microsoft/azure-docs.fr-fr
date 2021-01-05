---
title: Configurer une identité de labo dans Azure DevTest Labs
description: Apprenez à configurer une identité de labo dans Azure DevTest.
ms.topic: article
ms.date: 08/20/2020
ms.openlocfilehash: a652eb5751f9b723911a1c1baaaaf9860febc5b6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88719559"
---
# <a name="configure-a-lab-identity"></a>Configurer une identité de labo

La gestion des informations d’identification dans votre code pour s’authentifier auprès des services cloud constitue un défi courant lors de la génération d’applications cloud. La sécurisation de ces informations d’identification est une tâche importante. Dans l’idéal, elles ne s’affichent jamais sur les stations de travail de développement et ne sont jamais archivées dans le contrôle de code source. Azure Key Vault permet de stocker en toute sécurité les informations d'identification, les secrets et les clés, mais votre code doit s'authentifier auprès de Key Vault pour les récupérer. 

La fonctionnalité des identités managées dédiées aux ressources Azure disponible dans Azure Active Directory (Azure AD) résout ce problème. Elle fournit aux services Azure une identité système administrée automatiquement dans Azure AD. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, y compris Key Vault, sans avoir d’informations d’identification dans votre code. Découvrez-en plus sur les [identités managées sur Azure](../active-directory/managed-identities-azure-resources/overview.md). 

Il existe deux types d’identités administrées : 

## <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système  

Une  **identité managée attribuée par le système**  est directement activée sur une instance de service Azure. Lorsque l’identité est activée, Azure crée une identité pour l’instance dans le locataire Azure AD approuvé par l’abonnement de l’instance. Une fois l’identité créée, ses informations d’identification sont provisionnées sur l’instance. Le cycle de vie d’une identité affectée par le système est directement lié à l’instance de service Azure sur laquelle elle est activée. Si l’instance est supprimée, Azure efface automatiquement les informations d’identification et l’identité dans Azure AD. 

### <a name="scenarios-for-using-labs-system-assigned-identity"></a>Scénarios d'utilisation de l'identité attribuée par le système du labo  

Chaque instance de DevTest Labs est créée avec une identité attribuée par le système qui reste valide pendant toute la durée de vie du labo. L'identité attribuée par le système est utilisée aux fins suivantes :  

- Tous les déploiements basés sur [Azure Resource Manager](devtest-lab-create-environment-from-arm.md) utilisés pour lancer un environnement multimachines virtuelles et/ou PaaS (Platform as a service) seront exécutés à l'aide de l'identité attribuée par le système du labo.  
- Le chiffrement des disques du labo à l'aide d'une clé gérée par le client est pris en charge via l'identité attribuée par le système du labo. En fournissant un accès explicite à l'identité du labo pour accéder à votre jeu de chiffrement de disque, le labo peut chiffrer tous les disques des machines virtuelles en votre nom. Découvrez-en plus sur l'[activation du chiffrement des disques](encrypt-disks-customer-managed-keys.md) de votre labo à l'aide d'une clé gérée par le client.  

### <a name="configure-identity"></a>Configurer une identité

Cette section explique comment configurer la stratégie d'identité du labo.

> [!NOTE]
> Pour les labos créés avant le 10/08/2020, l'identité attribuée par le système sera désactivée. En tant que propriétaire de labo, vous pouvez l'activer, au cas où vous auriez l'intention d'utiliser des labos aux fins énumérées dans la section précédente.  
>
> Pour les nouveaux labos créés après le 10/08/2020, l'identité attribuée par le système du labo est activée par défaut, et le propriétaire n'a aucun moyen de la désactiver pendant le cycle de vie du labo.  

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez **DevTest Labs**.
1. Dans la liste des laboratoires, sélectionnez le laboratoire souhaité.
1. Sélectionnez **Configuration et stratégies** -> **Identité (Préversion)** . 

> [!div class="mx-imgBorder"]
> ![Configurer l'identité](./media/configure-lab-identity/configure-identity.png)

## <a name="user-assigned-managed-identity"></a>Identité managée affectée par l’utilisateur  

Une identité gérée attribuée par l'utilisateur est créée en tant que ressource Azure autonome. Via un processus de création, Azure crée une identité dans le locataire Azure AD approuvé par l’abonnement en cours d’utilisation. Une fois l’identité créée, elle peut être affectée à une ou plusieurs instances de service Azure. Le cycle de vie d’une identité affectée par l’utilisateur est géré séparément de celui des instances de service Azure auxquelles elle est affectée. 

DevTest Labs prend en charge les identités attribuées par les utilisateurs pour les machines virtuelles et les environnements basés sur Azure Resource Manager.  Pour plus d'informations, voir les rubriques suivantes :

- [Ajouter une identité attribuée par l'utilisateur pour déployer des environnements lab Azure Resource Manager](use-managed-identities-environments.md)
- [Ajouter une identité attribuée par l'utilisateur pour déployer des machines virtuelles lab](enable-managed-identities-lab-vms.md)

## <a name="next-steps"></a>Étapes suivantes

Consultez [Configurer la gestion des coûts](devtest-lab-configure-cost-management.md)