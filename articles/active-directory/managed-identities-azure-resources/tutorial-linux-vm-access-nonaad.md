---
title: Tutoriel `:` Utiliser une identité managée pour accéder à Azure Key Vault - Linux - Azure AD
description: Ce didacticiel vous guide tout au long du processus consistant à utiliser une identité managée attribuée par le système de machine virtuelle Linux pour accéder à Azure Resource Manager.
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
ms.openlocfilehash: 5151f97386ebb6b06be2320505771dc8f47d59a0
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107530"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-key-vault"></a>Tutoriel : Utiliser une identité managée de machine virtuelle Linux attribuée par le système pour accéder à Azure Key Vault 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Ce tutoriel vous indique comment une machine virtuelle Linux peut utiliser une identité managée affectée par le système pour accéder à [Azure Key Vault](../../key-vault/general/overview.md). En agissant comme un amorçage, Key Vault permet à votre application cliente d’utiliser ensuite un secret pour accéder à des ressources non sécurisées par Azure Active Directory (AD). Les identités de service managées sont automatiquement managées par Azure et vous permettent de vous authentifier auprès de services qui prennent en charge l’authentification Azure AD sans inclure des informations d’authentification dans votre code.

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Accorder à votre machine virtuelle l’accès à un secret stocké dans Key Vault 
> * Obtenir un jeton d’accès à l’aide d’une identité de machine virtuelle et l’utiliser pour récupérer la clé secrète de Key Vault 
 
## <a name="prerequisites"></a>Prérequis

- Connaissance de base des identités managées. Si vous n’êtes pas familiarisé de la fonctionnalité identités managées pour ressources Azure, consultez cette [Vue d’ensemble](overview.md). 
- Un compte Azure. [Inscrivez-vous pour obtenir un compte gratuit](https://azure.microsoft.com/free/).
- Des autorisations « Propriétaire » avec l’étendue appropriée (votre abonnement ou groupe de ressources) pour effectuer les étapes de création de ressource et de gestion de rôles nécessaires. Si vous avez besoin d’aide concernant l’attribution de rôle, consultez [Utiliser le contrôle d’accès en fonction du rôle pour gérer l’accès aux ressources d’un abonnement Azure](../../role-based-access-control/role-assignments-portal.md).
- Vous avez également besoin d’une machine virtuelle Linux sur laquelle les identités managées attribuées par le système sont activées.
  - Si vous devez créer une machine virtuelle pour ce tutoriel, vous pouvez suivre les instructions de l’article intitulé [Créer une machine virtuelle Linux avec le portail Azure](../../virtual-machines/linux/quick-create-portal.md#create-virtual-machine)


## <a name="create-a-key-vault"></a>Créer un coffre de clés  

Cette section montre comment accorder à votre machine virtuelle l’accès à un secret stocké dans un coffre de clés. À l’aide d’identités managées pour ressources Azure, votre code peut obtenir des jetons d’accès pour vous authentifier sur des ressources prenant en charge l’authentification Azure AD.Toutefois, tous les services Azure ne prennent pas en charge l’authentification Azure AD. Pour utiliser des identités managées pour les ressources Azure avec ces services, stockez les informations d’identification des services dans Azure Key Vault, puis utilisez des identités managées de machine virtuelle afin d’accéder à Key Vault pour récupérer les informations d’identification.

Tout d’abord, nous devons créer un Key Vault et accorder son accès à l’identité managée attribuée par le système de machine virtuelle.

1. Ouvrez le [portail](https://portal.azure.com/) Azure.
1. En haut de la barre de navigation de gauche, sélectionnez **Créer une ressource**.  
1. Dans la zone **Rechercher dans la Place de marché**, tapez **Coffre de clés**, puis appuyez sur **Entrer**.  
1. Sélectionnez **Coffre de clés** dans les résultats.
1. Sélectionnez **Créer**
1. Entrez un **Nom** pour le nouveau Key Vault.

    ![Écran Créer un coffre de clés](./media/tutorial-linux-vm-access-nonaad/create-key-vault.png)

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

   ![Créer un secret](./media/tutorial-linux-vm-access-nonaad/create-secret.png)

## <a name="grant-access"></a>Accorder l'accès

L’identité managée utilisée par la machine virtuelle doit disposer d’un accès pour lire le secret que nous allons stocker dans le coffre de clés.

1. Accédez au coffre de clés que vous venez de créer.
1. Dans le menu de gauche, sélectionnez **Stratégie d’accès**.
1. Sélectionnez **Ajouter une stratégie d’accès**.

   ![écran de création d’une stratégie d’accès pour les coffres de clés](./media/tutorial-linux-vm-access-nonaad/key-vault-access-policy.png)

1. Dans la section **Ajouter une stratégie d’accès** sous **Configurer à partir du modèle (facultatif)** , choisissez **Gestion des secrets** dans le menu déroulant.
1. Choisissez **Sélectionner le principal**, et dans la zone de recherche entrez le nom de la machine virtuelle créée précédemment.  Sélectionnez la machine virtuelle dans la liste des résultats, puis choisissez **Sélectionner**.
1. Sélectionnez **Ajouter**
1. Sélectionnez **Enregistrer**.

## <a name="access-data"></a>Accéder aux données

Pour effectuer cette procédure, vous avez besoin d’un client SSH.  Si vous utilisez Windows, vous pouvez utiliser le client SSH dans le [Sous-système Windows pour Linux](/windows/wsl/about). Si vous avez besoin d’aide pour configurer les clés de votre client SSH, consultez [Comment utiliser les clés SSH avec Windows sur Azure](../../virtual-machines/linux/ssh-from-windows.md), ou [Comment créer et utiliser une paire de clés publique et privée SSH pour les machines virtuelles Linux dans Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).
 
1. Dans le portail, accédez à votre machine virtuelle Linux et dans **Vue d’ensemble**, cliquez sur **Connexion**. 
2. **Connectez-vous** à la machine virtuelle à l’aide du client SSH de votre choix. 
3. Dans la fenêtre de terminal, envoyez une requête aux identités managées locales pour le point de terminaison des ressources Azure en vue d’obtenir un jeton d’accès pour Azure Key Vault.  
 
    Vous trouverez la requête CURL pour le jeton d’accès ci-dessous.  
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -H Metadata:true  
    ```
    La réponse inclut le jeton d’accès dont vous avez besoin pour accéder au Gestionnaire des ressources. 
    
    Réponse :  
    
    ```bash
    {"access_token":"eyJ0eXAi...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://vault.azure.net",
    "token_type":"Bearer"} 
    ```
    
    Vous pouvez utiliser ce jeton d’accès pour vous authentifier sur Azure Key Vault.  La requête CURL suivante montre comment lire un secret de Key Vault à l’aide de CURL et de l’API REST de Key Vault.  Vous avez besoin de l’URL de votre Key Vault qui se trouve dans la section **Bases** de la page **Vue d’ensemble** de Key Vault.  Vous avez également besoin du jeton d’accès que vous avez obtenu lors de l’appel précédent. 
        
    ```bash
    curl 'https://<YOUR-KEY-VAULT-URL>/secrets/<secret-name>?api-version=2016-10-01' -H "Authorization: Bearer <ACCESS TOKEN>" 
    ```
    
    La réponse aura l’aspect suivant : 
    
    ```bash
    {"value":"p@ssw0rd!","id":"https://mytestkeyvault.vault.azure.net/secrets/MyTestSecret/7c2204c6093c4d859bc5b9eff8f29050","attributes":{"enabled":true,"created":1505088747,"updated":1505088747,"recoveryLevel":"Purgeable"}} 
    ```
    
Après avoir récupéré le secret à partir de Key Vault, vous pouvez l’utiliser pour vous authentifier sur un service qui requiert un nom et un mot de passe.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer les ressources, accédez au [portail Azure](https://portal.azure.com), sélectionnez **Groupes de ressources**, puis localisez et sélectionnez le groupe de ressources qui a été créé dans ce tutoriel (par exemple `mi-test`). Ensuite, utilisez la commande **Supprimer le groupe de ressources**.

Vous pouvez également effectuer cette opération par le biais de [PowerShell ou de l’interface CLI](../../azure-resource-manager/management/delete-resource-group.md)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à utiliser une identité managée de machine virtuelle Linux attribuée par le système pour accéder à Azure Key Vault.  Pour en savoir plus sur Azure Key Vault, consultez :

> [!div class="nextstepaction"]
>[Azure Key Vault](../../key-vault/general/overview.md)