---
title: Tutoriel `:` Utiliser une identité managée pour accéder à Azure Key Vault - Windows - Azure AD
description: Ce didacticiel vous guide tout au long de l’utilisation d’une identité managée attribuée par le système de machine virtuelle Windows pour accéder à Azure Key Vault.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/10/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 668d3cb044512220ff7afbc165c77da704a9a5d7
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107513"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-key-vault"></a>Tutoriel : Utiliser une identité managée de machine virtuelle Windows attribuée par le système pour accéder à Azure Key Vault 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Ce tutoriel vous indique comment une machine virtuelle Windows peut utiliser une identité managée affectée par le système pour accéder à [Azure Key Vault](../../key-vault/general/overview.md). En agissant comme un amorçage, Key Vault permet à votre application cliente d’utiliser ensuite un secret pour accéder à des ressources non sécurisées par Azure Active Directory (AD). Les identités de service managées sont automatiquement managées par Azure et vous permettent de vous authentifier auprès de services qui prennent en charge l’authentification Azure AD sans inclure des informations d’authentification dans votre code.

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Accorder à votre machine virtuelle l’accès à un secret stocké dans Key Vault
> * Obtenir un jeton d’accès à l’aide d’une identité de machine virtuelle et l’utiliser pour récupérer un secret de Key Vault 

## <a name="prerequisites"></a>Prérequis

- Compréhension des identités managées. Si vous n’êtes pas familiarisé de la fonctionnalité identités managées pour ressources Azure, consultez cette [Vue d’ensemble](overview.md). 
- Un compte Azure. [Inscrivez-vous pour obtenir un compte gratuit](https://azure.microsoft.com/free/).
- Des autorisations « Propriétaire » avec l’étendue appropriée (votre abonnement ou groupe de ressources) pour effectuer les étapes de création de ressource et de gestion de rôles nécessaires. Si vous avez besoin d’aide concernant l’attribution de rôle, consultez [Utiliser le contrôle d’accès en fonction du rôle pour gérer l’accès aux ressources d’un abonnement Azure](../../role-based-access-control/role-assignments-portal.md).
- Vous avez également besoin d’une machine virtuelle Windows sur laquelle les identités managées attribuées par le système sont activées.
  - Si vous devez créer une machine virtuelle pour ce tutoriel, vous pouvez suivre les instructions de l’article intitulé [Créer une machine virtuelle avec une identité affectée par le système activée](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity)

## <a name="create-a-key-vault"></a>Créer un coffre de clés  

Cette section montre comment accorder à votre machine virtuelle l’accès à un secret stocké dans un coffre de clés. À l’aide d’identités managées pour ressources Azure, votre code peut obtenir des jetons d’accès pour vous authentifier sur des ressources prenant en charge l’authentification Azure AD.Toutefois, tous les services Azure ne prennent pas en charge l’authentification Azure AD. Pour utiliser des identités managées pour les ressources Azure avec ces services, stockez les informations d’identification des services dans Azure Key Vault, puis utilisez des identités managées de machine virtuelle afin d’accéder à Key Vault pour récupérer les informations d’identification.

Tout d’abord, nous devons créer un Key Vault et accorder son accès à l’identité managée attribuée par le système de machine virtuelle.

1. Ouvrez le [portail](https://portal.azure.com/) Azure.
1. En haut de la barre de navigation de gauche, sélectionnez **Créer une ressource**.  
1. Dans la zone **Rechercher dans la Place de marché**, tapez **Coffre de clés**, puis appuyez sur **Entrer**.  
1. Sélectionnez **Coffre de clés** dans les résultats.
1. Sélectionnez **Créer**
1. Entrez un **Nom** pour le nouveau Key Vault.

    ![Écran Créer un coffre de clés](./media/msi-tutorial-windows-vm-access-nonaad/create-key-vault.png)

1. Renseignez toutes les informations obligatoires en prenant soin de choisir l’abonnement et le groupe de ressources où vous avez créé la machine virtuelle que vous utilisez pour ce tutoriel.
1. Sélectionner **Vérifier + créer**
1. Sélectionnez **Créer**

### <a name="create-a-secret"></a>Créer un secret

Ajoutez ensuite un secret au coffre de clés afin de pouvoir le récupérer ultérieurement à l’aide du code en cours d’exécution dans votre machine virtuelle. Dans le cadre de ce tutoriel, nous utilisons PowerShell, mais les mêmes concepts s’appliquent à tout code s’exécutant dans cette machine virtuelle.

1. Accédez au coffre de clés que vous venez de créer.
1. Sélectionnez **Secret**, puis cliquez sur **Ajouter**.
1. Sélectionnez **Générer/Importer**.
1. Dans l’écran **Créer un secret**, dans les **Options de chargement** laissez **Manuel** sélectionné.
1. Entrez un nom et une valeur pour le secret.  Vous pouvez choisir la valeur de votre choix. 
1. Laissez les champs pour la date d’activation et la date d’expiration vides, puis pour **Activé**, laissez la valeur **Oui**. 
1. Cliquez sur **Créer** pour créer le secret.

   ![Créer un secret](./media/msi-tutorial-windows-vm-access-nonaad/create-secret.png)

## <a name="grant-access"></a>Accorder l'accès

L’identité managée utilisée par la machine virtuelle doit disposer d’un accès pour lire le secret que nous allons stocker dans le coffre de clés.

1. Accédez au coffre de clés que vous venez de créer.
1. Dans le menu de gauche, sélectionnez **Stratégie d’accès**.
1. Sélectionnez **Ajouter une stratégie d’accès**.

   ![écran de création d’une stratégie d’accès pour les coffres de clés](./media/msi-tutorial-windows-vm-access-nonaad/key-vault-access-policy.png)

1. Dans la section **Ajouter une stratégie d’accès** sous **Configurer à partir du modèle (facultatif)** , choisissez **Gestion des secrets** dans le menu déroulant.
1. Choisissez **Sélectionner le principal**, et dans la zone de recherche entrez le nom de la machine virtuelle créée précédemment.  Sélectionnez la machine virtuelle dans la liste des résultats, puis choisissez **Sélectionner**.
1. Sélectionnez **Ajouter**
1. Sélectionnez **Enregistrer**.


## <a name="access-data"></a>Accéder aux données  

Cette section montre comment obtenir un jeton d’accès à l’aide d’une identité de machine virtuelle et comment l’utiliser pour récupérer le secret dans le coffre de clés. Si PowerShell 4.3.1 ou version ultérieure n’est pas installé, vous devez [télécharger et installer la dernière version](/powershell/azure/).

Tout d’abord, nous utilisons l’identité managée attribuée par le système de machine virtuelle pour obtenir un jeton d’accès permettant de s’authentifier auprès de Key Vault :
 
1. Dans le portail, accédez à **Machines virtuelles** et accédez à votre machine virtuelle Windows. Puis, dans **Vue d’ensemble**, cliquez sur **Connecter**.
2. Entrez le **Nom d’utilisateur** et le **Mot de passe** que vous avez ajoutés lorsque vous avez créé la **machine virtuelle Windows**.  
3. Maintenant que vous avez créé une **Connexion au Bureau à distance** avec la machine virtuelle, ouvrez PowerShell dans la session à distance.  
4. Dans PowerShell, appelez la requête web sur le client pour obtenir le jeton de l’hôte local dans le port spécifique pour la machine virtuelle.  

La requête PowerShell :

```powershell
$Response = Invoke-RestMethod -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -Method GET -Headers @{Metadata="true"} 
```

Vous pouvez voir ci-dessous à quoi ressemble la réponse :

![demande avec une réponse de jeton](./media/msi-tutorial-windows-vm-access-nonaad/token.png)

Ensuite, extrayez le jeton d’accès de la réponse.  

```powershell
   $KeyVaultToken = $Response.access_token
```

Enfin, utilisez la commande Invoke-WebRequest de PowerShell pour récupérer le secret que vous avez créé précédemment dans Key Vault et transmettre le jeton d’accès vers l’en-tête d’autorisation.  Vous avez besoin de l’URL de votre Key Vault qui se trouve dans la section **Bases** de la page **Vue d’ensemble** de Key Vault.  

```powershell
Invoke-RestMethod -Uri https://<your-key-vault-URL>/secrets/<secret-name>?api-version=2016-10-01 -Method GET -Headers @{Authorization="Bearer $KeyVaultToken"}
```

La réponse aura l’aspect suivant : 

```powershell
  value       id                                                                                    attributes
  -----       --                                                                                    ----------
  'My Secret' https://mi-lab-vault.vault.azure.net/secrets/mi-test/50644e90b13249b584c44b9f712f2e51 @{enabled=True; created=16…
```

Après avoir récupéré le secret à partir de Key Vault, vous pouvez l’utiliser pour vous authentifier sur un service qui requiert un nom et un mot de passe.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer les ressources, accédez au [portail Azure](https://portal.azure.com), sélectionnez **Groupes de ressources**, puis localisez et sélectionnez le groupe de ressources qui a été créé dans ce tutoriel (par exemple `mi-test`). Ensuite, utilisez la commande **Supprimer le groupe de ressources**.

Vous pouvez également effectuer cette opération par le biais de [PowerShell ou de l’interface CLI](../../azure-resource-manager/management/delete-resource-group.md)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à utiliser une identité managée de machine virtuelle Windows attribuée par le système pour accéder à Azure Key Vault.  Pour en savoir plus sur Azure Key Vault, consultez :

> [!div class="nextstepaction"]
>[Azure Key Vault](../../key-vault/general/overview.md)