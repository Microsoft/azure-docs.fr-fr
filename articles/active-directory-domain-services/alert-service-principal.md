---
title: Résoudre les alertes liées aux principaux de service dans Azure AD Domain Services | Microsoft Docs
description: Découvrez comment résoudre les alertes liées à la configuration des principaux de service pour Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: f168870c-b43a-4dd6-a13f-5cfadc5edf2c
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 00ab5c85a477c9c4080acf252cbbde9d4ce816a9
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620237"
---
# <a name="known-issues-service-principal-alerts-in-azure-active-directory-domain-services"></a>Problèmes connus : Alertes liées aux principaux de service dans Azure AD Domain Services

[Les principaux de service](../active-directory/develop/app-objects-and-service-principals.md) sont des applications que la plateforme Azure utilise pour gérer, mettre à jour et tenir à jour un domaine managé Azure AD DS (Azure Active Directory Domain Services). Si un principal de service est supprimé, les fonctionnalités dans le domaine managé Azure AD DS sont affectées.

Cet article vous aide à résoudre les alertes liées à la configuration des principaux de service.

## <a name="alert-aadds102-service-principal-not-found"></a>Alerte AADDS102 : Principal de service introuvable

### <a name="alert-message"></a>Message d’alerte

*Un principal de service requis pour que les services de domaine Azure AD fonctionnent correctement a été supprimé de votre annuaire Azure AD. Cette configuration affecte la capacité de Microsoft à surveiller, gérer, mettre à jour et synchroniser votre domaine géré.*

Si un principal de service requis est supprimé, la plateforme Azure ne peut pas effectuer de tâches de gestion automatisées. Le domaine managé risque de ne pas appliquer correctement les mises à jour ou de ne pas effectuer de sauvegardes.

### <a name="check-for-missing-service-principals"></a>Vérifier les principaux de service manquants

Pour vérifier quel est le principal de service manquant à recréer, procédez ainsi :

1. Sur le portail Azure, sélectionnez **Azure Active Directory** dans le volet de navigation gauche.
1. Sélectionnez **Applications d’entreprise**. Choisissez *Toutes les application* dans le menu déroulant **Type d’application**, puis sélectionnez **Appliquer**.
1. Recherchez chacun des ID d’applications suivants. Si aucune application existante n’est trouvée, suivez les étapes de *Résolution* pour créer le principal de service ou réinscrivez l’espace de noms.

    | ID de l'application | Résolution |
    | :--- | :--- |
    | 2565bd9d-da50-47d4-8b85-4c97f669dc36 | [Recréer un principal de service manquant](#recreate-a-missing-service-principal) |
    | 443155a6-77f3-45e3-882b-22b3a8d431fb | [Réinscrire l’espace de noms Microsoft.AAD](#re-register-the-microsoft-aad-namespace) |
    | abba844e-bc0e-44b0-947a-dc74e5d09022 | [Réinscrire l’espace de noms Microsoft.AAD](#re-register-the-microsoft-aad-namespace) |
    | d87dcbc6-a371-462e-88e3-28ad15ec4e64 | [Réinscrire l’espace de noms Microsoft.AAD](#re-register-the-microsoft-aad-namespace) |

### <a name="recreate-a-missing-service-principal"></a>Recréer un principal de service manquant

Si l’ID d’application *2565bd9d-da50-47d4-8b85-4c97f669dc36* est manquant dans votre annuaire Azure AD, effectuez les étapes suivantes avec Azure AD PowerShell. Pour plus d’informations, consultez [Azure AD PowerShell](/powershell/azure/active-directory/install-adv2).

1. Si nécessaire, installez le module Azure AD PowerShell, puis importez-le de la façon suivante :

    ```powershell
    Install-Module AzureAD
    Import-Module AzureAD
    ```

1. Maintenant, recréez le principal de service à l’aide de l’applet de commande [New-AzureAdServicePrincipal][New-AzureAdServicePrincipal] :

    ```powershell
    New-AzureAdServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
    ```

L’intégrité du domaine managé se met automatiquement à jour dans les deux heures, et l’alerte est supprimée.

### <a name="re-register-the-microsoft-aad-namespace"></a>Réinscrire l’espace de noms Microsoft AAD

Si l’ID d’application *443155a6-77f3-45e3-882b-22b3a8d431fb*, *abba844e-bc0e-44b0-947a-dc74e5d09022* ou *d87dcbc6-a371-462e-88e3-28ad15ec4e64* est manquant dans votre annuaire Azure AD, effectuez les étapes suivantes pour réinscrire le fournisseur de ressources *Microsoft.AAD* :

1. Dans le portail Azure, recherchez et sélectionnez **Abonnements**.
1. Choisissez l’abonnement associé à votre domaine managé.
1. Dans la navigation de gauche, choisissez **Fournisseurs de ressources**.
1. Recherchez *Microsoft.AAD*, puis sélectionnez **Réinscrire**.

L’intégrité du domaine managé se met automatiquement à jour dans les deux heures, et l’alerte est supprimée.

## <a name="alert-aadds105-password-synchronization-application-is-out-of-date"></a>Alerte AADDS105 : La synchronisation du mot de passe est obsolète

### <a name="alert-message"></a>Message d’alerte

*Le principal du service dont l’ID d’application est « d87dcbc6-a371-462e-88e3-28ad15ec4e64 » a été supprimé, puis recréé. Cette nouvelle création laisse des autorisations incohérentes sur les ressources Azure AD Domain Services nécessaires pour traiter votre domaine managé. La synchronisation des mots de passe dans votre domaine managé pourrait en être affectée.*

Azure AD DS synchronise automatiquement les comptes d’utilisateur et les informations d’identification à partir d’Azure AD. En cas de problème avec l’application Azure AD utilisée pour ce processus, la synchronisation des informations d’identification entre Azure AD DS et Azure AD échoue.

### <a name="resolution"></a>Résolution

Pour recréer l’application Azure AD utilisée pour la synchronisation des informations d’identification, effectuez les étapes suivantes avec Azure AD PowerShell. Pour plus d’informations, consultez [Installer Azure AD PowerShell](/powershell/azure/active-directory/install-adv2).

1. Si nécessaire, installez le module Azure AD PowerShell, puis importez-le de la façon suivante :

    ```powershell
    Install-Module AzureAD
    Import-Module AzureAD
    ```

2. Maintenant, supprimez l’ancienne application et l’ancien objet à l’aide des applets de commande PowerShell suivantes :

    ```powershell
    $app = Get-AzureADApplication -Filter "IdentifierUris eq 'https://sync.aaddc.activedirectory.windowsazure.com'"
    Remove-AzureADApplication -ObjectId $app.ObjectId
    $spObject = Get-AzureADServicePrincipal -Filter "DisplayName eq 'Azure AD Domain Services Sync'"
    Remove-AzureADServicePrincipal -ObjectId $spObject
    ```

Une fois que vous avez supprimé les deux applications, la plateforme Azure les recrée automatiquement et tente de reprendre la synchronisation de mot de passe. L’intégrité du domaine managé se met automatiquement à jour dans les deux heures, et l’alerte est supprimée.

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez toujours des problèmes, [formulez une demande de support Azure][azure-support] pour bénéficier d’une aide supplémentaire.

<!-- INTERNAL LINKS -->
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md

<!-- EXTERNAL LINKS -->
[New-AzureAdServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal
