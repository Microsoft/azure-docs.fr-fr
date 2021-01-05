---
title: Utiliser une identité managée affectée par l’utilisateur de machine virtuelle Linux pour accéder à Azure Resource Manager
description: Ce didacticiel vous guide tout au long de l’utilisation d’une identité managée affectée par l’utilisateur sur une machine virtuelle Linux, pour accéder à Azure Resource Manager.
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
ms.date: 12/01/2020
ms.author: barclayn
ROBOTS: NOINDEX,NOFOLLOW
ms.collection: M365-identity-device-management
ms.openlocfilehash: 247b3c1b4341eff11069a6af324fff5cf1fba62c
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546588"
---
# <a name="tutorial-use-a-user-assigned-managed-identity-on-a-linux-vm-to-access-azure-resource-manager"></a>Tutoriel : Utiliser une identité managée affectée par l’utilisateur sur une machine virtuelle Linux pour accéder à Azure Resource Manager

Ce didacticiel explique comment créer une identité managée affectée par l’utilisateur, comment l’attribuer à une machine virtuelle Linux, puis l’utiliser pour accéder à l’API d’Azure Resource Manager. Les identités managées pour ressources Azure sont gérées automatiquement par Azure. Elles permettent l’authentification auprès de services prenant en charge Azure AD authentication, sans devoir nécessairement incorporer des informations d’identification à votre code. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une identité managée attribuée par l’utilisateur
> * Attribuer l’identité managée affectée par l’utilisateur à une machine virtuelle Linux 
> * Accorder à l’identité managée affectée par l’utilisateur l’accès à un groupe de ressources dans Azure Resource Manager 
> * Obtenir un jeton d’accès par l’identité managée affectée par l’utilisateur et l’utiliser pour appeler Azure Resource Manager 

## <a name="prerequisites"></a>Prérequis

- Compréhension des identités managées. Si vous n’êtes pas familiarisé de la fonctionnalité identités managées pour ressources Azure, consultez cette [Vue d’ensemble](overview.md). 
- Un compte Azure. [Inscrivez-vous pour obtenir un compte gratuit](https://azure.microsoft.com/free/).
- Vous avez également besoin d’une machine virtuelle Linux. Si vous devez créer une machine virtuelle pour ce tutoriel, vous pouvez suivre les instructions de l’article intitulé [Créer une machine virtuelle Linux avec le portail Azure](../../virtual-machines/linux/quick-create-portal.md#create-virtual-machine)
- Pour exécuter les exemples de scripts, vous avez deux options :
    - Utiliser [Azure Cloud Shell](../../cloud-shell/overview.md), que vous pouvez ouvrir en utilisant le bouton **Essayer** dans le coin supérieur droit des blocs de code.
    - Exécuter les scripts localement en installant la dernière version d’[Azure CLI](/cli/azure/install-azure-cli), puis en vous connectant à Azure à l’aide d’[az login](/cli/azure/reference-index#az-login).

## <a name="create-a-user-assigned-managed-identity"></a>Créer une identité managée attribuée par l’utilisateur

Créez une identité managée affectée par l’utilisateur en utilisant la commande [az identity create](/cli/azure/identity#az-identity-create). Le paramètre `-g` spécifie le groupe de ressources où l’identité managée affectée par l’utilisateur est créée, et le paramètre `-n` spécifie son nom. N’oubliez pas de remplacer les valeurs des paramètres `<RESOURCE GROUP>` et `<UAMI NAME>` par vos propres valeurs :
    
[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```azurecli-interactive
az identity create -g <RESOURCE GROUP> -n <UAMI NAME>
```

La réponse contient les détails de l’identité managée affectée par l’utilisateur qui a été créée, comme dans l’exemple suivant. Notez la valeur `id` pour votre identité managée affectée par l’utilisateur, car elle est utilisée à l’étape suivante :

```json
{
"clientId": "73444643-8088-4d70-9532-c3a0fdc190fz",
"clientSecretUrl": "https://control-westcentralus.identity.azure.net/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAMI NAME>/credentials?tid=5678&oid=9012&aid=12344643-8088-4d70-9532-c3a0fdc190fz",
"id": "/subscriptions/<SUBSCRIPTON ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAMI NAME>",
"location": "westcentralus",
"name": "<UAMI NAME>",
"principalId": "9012",
"resourceGroup": "<RESOURCE GROUP>",
"tags": {},
"tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533bl",
"type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

## <a name="assign-an-identity-to-your-linux-vm"></a>Affecter une identité à votre machine virtuelle Linux

Une identité managée affectée par l’utilisateur peut être utilisée par les clients sur plusieurs ressources Azure. Utilisez les commandes suivantes pour attribuer l’identité managée affectée par l’utilisateur à une seule machine virtuelle. Utilisez la propriété `Id` retournée à l’étape précédente pour le paramètre `-IdentityID`.

Attribuez l’identité managée affectée par l’utilisateur à votre machine virtuelle Linux à l’aide de la commande [az vm identity assign](/cli/azure/vm). N’oubliez pas de remplacer les valeurs des paramètres `<RESOURCE GROUP>` et `<VM NAME>` par vos propres valeurs. Utilisez la propriété `id` retournée à l’étape précédente comme valeur du paramètre `--identities`.

```azurecli-interactive
az vm identity assign -g <RESOURCE GROUP> -n <VM NAME> --identities "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<UAMI NAME>"
```

## <a name="grant-access-to-a-resource-group-in-azure-resource-manager"></a>Accorder l’accès à un groupe de ressources dans Azure Resource Manager

Les identités managées pour ressources Azure fournissent des identités que votre code peut utiliser pour demander des jetons d’accès afin de s’authentifier auprès d’API de ressources qui prennent en charge l’authentification Azure AD. Dans ce tutoriel, votre code accède à l’API Azure Resource Manager.  

Avant que ce code puisse accéder à l’API, vous devez accorder à l’identité l’accès à une ressource dans Azure Resource Manager. Dans le cas présent, le groupe de ressources dans lequel la machine virtuelle est contenue. Mettez à jour la valeur de `<SUBSCRIPTION ID>` et `<RESOURCE GROUP>` en fonction de votre environnement. En outre, remplacez `<UAMI PRINCIPALID>` par la propriété `principalId` retournée par la commande `az identity create` dans [Créer une identité managée affectée par l’utilisateur](#create-a-user-assigned-managed-identity) :

```azurecli-interactive
az role assignment create --assignee <UAMI PRINCIPALID> --role 'Reader' --scope "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESOURCE GROUP> "
```

La réponse contient les détails de l’affectation de rôle qui a été créée, comme dans l’exemple suivant :

```json
{
  "id": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Authorization/roleAssignments/b402bd74-157f-425c-bf7d-zed3a3a581ll",
  "name": "b402bd74-157f-425c-bf7d-zed3a3a581ll",
  "properties": {
    "principalId": "f5fdfdc1-ed84-4d48-8551-999fb9dedfbl",
    "roleDefinitionId": "/subscriptions/<SUBSCRIPTION ID>/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7",
    "scope": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>"
  },
  "resourceGroup": "<RESOURCE GROUP>",
  "type": "Microsoft.Authorization/roleAssignments"
}

```

## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-call-resource-manager"></a>Obtenir un jeton d’accès à l’aide de l’identité de la machine virtuelle et l’utiliser pour appeler Gestionnaire des ressources 

Pour la suite de ce didacticiel, nous allons utiliser la machine virtuelle que nous avons créée précédemment.

Pour effectuer cette procédure, vous avez besoin d’un client SSH. Si vous utilisez Windows, vous pouvez utiliser le client SSH dans le [Sous-système Windows pour Linux](/windows/wsl/about). 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le portail, accédez à **Machines virtuelles**, puis à la machine virtuelle Linux. Ensuite, dans **Vue d’ensemble**, cliquez sur **Connecter**. Copiez la chaîne permettant de se connecter à votre machine virtuelle.
3. Connectez-vous à la machine virtuelle à l’aide du client SSH de votre choix. Si vous utilisez Windows, vous pouvez utiliser le client SSH dans le [Sous-système Windows pour Linux](/windows/wsl/about). Si vous avez besoin d’aide pour configurer les clés de votre client SSH, consultez [Comment utiliser les clés SSH avec Windows sur Azure](~/articles/virtual-machines/linux/ssh-from-windows.md), ou [Comment créer et utiliser une paire de clés publique et privée SSH pour les machines virtuelles Linux dans Azure](~/articles/virtual-machines/linux/mac-create-ssh-keys.md).
4. Dans la fenêtre du terminal, à l’aide de CURL, envoyez une requête au point de terminaison d’identité du service IMDS (Instance Metadata Service) Azure pour obtenir un jeton d’accès à Azure Resource Manager.  

   La requête CURL permettant d’acquérir un jeton d’accès est illustrée dans l’exemple suivant. Veillez à remplacer `<CLIENT ID>` par la propriété `clientId` retournée par la commande `az identity create` dans [Créer une identité managée affectée par l’utilisateur](#create-a-user-assigned-managed-identity) : 
    
   ```bash
   curl -H Metadata:true "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com/&client_id=<UAMI CLIENT ID>"   
   ```
    
    > [!NOTE]
    > La valeur du paramètre `resource` doit correspondre exactement à ce qui est attendu par Azure AD. Lorsque vous utilisez l’ID de ressource Resource Manager, vous devez inclure la barre oblique de fin à l’URI. 
    
    La réponse inclut le jeton d’accès dont vous avez besoin pour accéder à Azure Resource Manager. 
    
    Exemple de réponse :  

    ```bash
    {
    "access_token":"eyJ0eXAiOi...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://management.azure.com",
    "token_type":"Bearer"
    } 
    ```

5. Utilisez le jeton d’accès pour accéder à Azure Resource Manager et lisez les propriétés du groupe de ressources dont vous avez précédemment accordé l’accès à votre identité managée affectée par l’utilisateur. Veillez à remplacer `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` par les valeurs que vous avez spécifiées auparavant, et `<ACCESS TOKEN>` par le jeton retourné à l’étape précédente.

    > [!NOTE]
    > L’URL respecte la casse, par conséquent veillez à ce que l’usage des minuscules et des majuscules soit strictement identique à celui que vous avez précédemment suivi lorsque vous avez nommé le groupe de ressources, que la majuscule « G » soit également bien respectée dans `resourceGroups`.  

    ```bash 
    curl https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>?api-version=2016-09-01 -H "Authorization: Bearer <ACCESS TOKEN>" 
    ```

    La réponse contient les informations particulières du groupe de ressources, comme dans l’exemple suivant : 

    ```bash
    {
    "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/DevTest",
    "name":"DevTest",
    "location":"westus",
    "properties":{"provisioningState":"Succeeded"}
    } 
    ```
    
## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez découvert comment créer une identité managée affectée par l’utilisateur, puis l’attacher à une machine virtuelle Linux pour accéder à l’API Azure Resource Manager.  Pour en savoir plus sur Azure Resource Manager, consultez :

> [!div class="nextstepaction"]
>[Azure Resource Manager](../../azure-resource-manager/management/overview.md)
