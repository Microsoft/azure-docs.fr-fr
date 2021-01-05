---
title: Corriger des erreurs d’annuaire incompatible dans Azure AD Domain Services | Microsoft Docs
description: Découvrez ce que signifie une erreur d’annuaire incompatible et comment la résoudre dans Azure AD Domain Services
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 40eb75b7-827e-4d30-af6c-ca3c2af915c7
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: ee8174114f1b892210e8ee9173ce0eb1d09c7e31
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619298"
---
# <a name="resolve-mismatched-directory-errors-for-existing-azure-active-directory-domain-services-managed-domains"></a>Résoudre des erreurs d’annuaire incompatible pour des domaines managés Azure Active Directory Domain Services existants

Si un domaine managé Azure Active Directory Domain Services (Azure AD DS) affiche une erreur de locataire incompatible, vous ne pouvez pas administrer le domaine managé jusqu’à ce qu’elle soit résolue. Cette erreur se produit si le réseau virtuel Azure sous-jacent est déplacé vers un autre annuaire Azure AD.

Cet article explique pourquoi l’erreur se produit et comment la résoudre.

## <a name="what-causes-this-error"></a>Quelle est la cause de cette erreur ?

Une erreur d’annuaire incompatible se produit quand un domaine managé Azure AD DS et un réseau virtuel appartiennent à deux locataires Azure AD différents. Par exemple, vous pouvez avoir un domaine managé nommé *aaddscontoso.com* qui s’exécute dans le locataire Azure AD de Contoso. Toutefois, le réseau virtuel Azure pour le domaine managé fait partie du locataire Azure AD Fabrikam.

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) sert à limiter l’accès aux ressources. Lorsque vous activez Azure AD DS dans un locataire Azure AD, les hachages des informations d’identification sont synchronisés avec le domaine managé. Pour effectuer cette opération, vous devez être un administrateur de locataire pour l’annuaire Azure AD et l’accès aux informations d’identification doit être contrôlé.

Pour déployer des ressources sur un réseau virtuel Azure et contrôler le trafic, vous devez disposer de privilèges d’administrateur sur le réseau virtuel sur lequel vous déployez le domaine managé.

Pour qu’Azure RBAC fonctionne de manière cohérente et sécurise l’accès à toutes les ressources utilisées par Azure AD DS, le domaine managé et le réseau virtuel doivent appartenir au même locataire Azure AD.

Les règles suivantes s’appliquent aux déploiements :

- Un annuaire Azure AD peut avoir plusieurs abonnements Azure.
- Un abonnement Azure peut avoir plusieurs ressources telles que des réseaux virtuels.
- Un seul domaine managé est activé pour un annuaire Azure AD.
- Un domaine managé peut être activé sur un réseau virtuel appartenant à l’un des abonnements Azure du même locataire Azure AD.

### <a name="valid-configuration"></a>Configuration valide

Dans l’exemple de scénario de déploiement suivant, le domaine managé Contoso est activé dans le locataire Azure AD Contoso. Le domaine managé est déployé sur un réseau virtuel appartenant à un abonnement Azure détenu par le locataire Azure AD Contoso.

Le domaine managé et le réseau virtuel appartiennent tous deux au même locataire Azure AD. Cet exemple de configuration est valide et entièrement pris en charge.

![Configuration valide de locataire Azure AD DS avec la partie domaine managé et réseau virtuel du même locataire Azure AD](./media/getting-started/valid-tenant-config.png)

### <a name="mismatched-tenant-configuration"></a>Configuration de locataire incompatible

Dans cet exemple de scénario de déploiement, le domaine managé Contoso est activé dans le locataire Azure AD Contoso. Toutefois, le domaine managé est déployé dans un réseau virtuel appartenant à un abonnement Azure détenu par le locataire Azure AD Fabrikam.

Le domaine managé et le réseau virtuel appartiennent à deux locataires Azure AD différents. Cet exemple de configuration correspond à un locataire incompatible et n’est pas pris en charge. Le réseau virtuel doit être déplacé vers le même locataire Azure AD que le domaine managé.

![Configuration de locataire incompatible](./media/getting-started/mismatched-tenant-config.png)

## <a name="resolve-mismatched-tenant-error"></a>Résoudre l’erreur de locataire incompatible

Les deux options suivantes permettent de résoudre l’erreur d’annuaire incompatible :

* Tout d’abord, [supprimez le domaine managé](delete-aadds.md) de votre annuaire Azure AD. Ensuite, [créez un domaine managé de remplacement](tutorial-create-instance.md) dans le même annuaire Azure AD que le réseau virtuel que vous souhaitez utiliser. Quand vous êtes prêt, joignez au domaine managé recréé toutes les machines précédemment jointes au domaine supprimé.
* [Déplacez l’abonnement Azure](../cost-management-billing/manage/billing-subscription-transfer.md) contenant le réseau virtuel vers le même annuaire Azure AD que le domaine managé.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la résolution des problèmes liés à Azure AD DS, consultez le [guide de résolution des problèmes](troubleshoot.md).
