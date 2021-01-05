---
title: Authentifier Azure Spring Cloud auprès de Key Vault dans GitHub Actions
description: Guide pratique pour utiliser le coffre de clés avec le workflow CI/CD pour Azure Spring Cloud avec GitHub Actions
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: how-to
ms.date: 09/08/2020
ms.custom: devx-track-java
ms.openlocfilehash: 995d10b3c7064e462500e0bec4d5d8aa010afe64
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90888777"
---
# <a name="authenticate-azure-spring-cloud-with-key-vault-in-github-actions"></a>Authentifier Azure Spring Cloud auprès de Key Vault dans GitHub Actions

**Cet article s’applique à :** ✔️ Java ✔️ C#

Le coffre de clés est un emplacement sécurisé pour stocker les clés. Au sein des entreprises, les utilisateurs doivent stocker les informations d’identification des environnements CI/CD dans l’étendue qu’ils contrôlent. La clé permettant d’accéder aux informations d’identification dans le coffre de clés doit être limitée à l’étendue des ressources.  Elle a uniquement accès à l’étendue du coffre de clés, et non à l’ensemble de l’étendue Azure. Elle est assimilable à une clé qui peut ouvrir uniquement un coffre-fort, non à une clé principale qui peut ouvrir toutes les portes d’un bâtiment. C’est un moyen d’obtenir une clé avec une autre clé, ce qui est utile dans un workflow CI/CD. 

## <a name="generate-credential"></a>Générer les informations d’identification
Pour générer une clé d’accès au coffre de clés, exécutez la commande ci-dessous sur votre ordinateur local :
```
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.KeyVault/vaults/<KEY_VAULT> --sdk-auth
```
L’étendue spécifiée par le paramètre `--scopes` limite l’accès de la clé à la ressource.  Elle ne peut accéder qu’au coffre-fort.

Avec les résultats :
```
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
}
```
Enregistrez ensuite les résultats dans des **secrets** GitHub comme décrit dans [Configurer votre dépôt GitHub et vous authentifier auprès d’Azure](./spring-cloud-howto-github-actions.md#set-up-github-repository-and-authenticate).

## <a name="add-access-policies-for-the-credential"></a>Ajouter des stratégies d’accès pour les informations d’identification
Les informations d’identification que vous avez créées ci-dessus peuvent obtenir uniquement des informations générales sur le coffre de clés, et non le contenu qu’il stocke.  Pour obtenir les secrets stockés dans le coffre de clés, vous devez définir des stratégies d’accès pour les informations d’identification.

Accédez au tableau de bord **Key Vault** dans le portail Azure, cliquez sur le menu **Contrôle d’accès**, puis ouvrez l’onglet **Attributions de rôles**. Sélectionnez **Applications** en guise de **Type** et `This resource` comme **étendue**.  Vous devez voir les informations d’identification que vous avez créées à l’étape précédente :

 ![Définir une stratégie d’accès](./media/github-actions/key-vault1.png)

Copiez le nom des informations d’identification, par exemple, `azure-cli-2020-01-19-04-39-02`. Ouvrez le menu **Stratégies d’accès** et cliquez sur le lien **+Ajouter une stratégie d’accès**.  Sélectionnez `Secret Management` comme **Modèle**, puis sélectionnez **Principal**. Collez le nom des informations d’identification dans la zone d’entrée **Principal**/**Sélectionner** :

 ![Sélectionnez](./media/github-actions/key-vault2.png)

 Cliquez sur le bouton **Ajouter** dans la boîte de dialogue **Ajouter une stratégie d’accès**, puis cliquez sur **Enregistrer**.

## <a name="generate-full-scope-azure-credential"></a>Générer des informations d’identification Azure à étendue complète
Il s’agit de la clé principale pour ouvrir toutes les portes du bâtiment. La procédure est similaire à l’étape précédente, mais ici, nous changeons l’étendue pour générer la clé principale :

```
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID> --sdk-auth
```

Voici de nouveau les résultats :
```
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
}
```
Copiez la chaîne JSON entière.  Retournez au tableau de bord **Key Vault**. Ouvrez le menu **Secrets**, puis cliquez sur le bouton **Générer/importer**. Entrez le nom du secret, par exemple `AZURE-CREDENTIALS-FOR-SPRING`. Collez la chaîne d’informations d’identification JSON dans la zone d’entrée **Valeur**. Vous pouvez remarquer que la zone d’entrée Valeur est un champ de texte d’une ligne, plutôt qu’une zone de texte multiligne.  Vous pouvez y coller la chaîne JSON complète.

 ![Informations d’identification d’étendue complète](./media/github-actions/key-vault3.png)

## <a name="combine-credentials-in-github-actions"></a>Combiner les informations d’identification dans GitHub Actions
Définissez les informations d’identification utilisées quand le pipeline CI/CD s’exécute :

```
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}           # Strong box key you generated in the first step
    - uses: Azure/get-keyvault-secrets@v1.0
      with:
        keyvault: "<Your Key Vault Name>"
        secrets: "AZURE-CREDENTIALS-FOR-SPRING"           # Master key to open all doors in the building
      id: keyvaultaction
    - uses: azure/login@v1
      with:
        creds: ${{ steps.keyvaultaction.outputs.AZURE-CREDENTIALS-FOR-SPRING }}
    - name: Azure CLI script
      uses: azure/CLI@v1
      with:
        azcliversion: 2.0.75
        inlineScript: |
          az extension add --name spring-cloud             # Spring CLI commands from here
          az spring-cloud list

```

## <a name="next-steps"></a>Étapes suivantes
* [GitHub Actions avec Spring Cloud](./spring-cloud-howto-github-actions.md)
