---
title: Chiffrer la source de votre application au repos
description: Chiffrez les données de votre application dans le Stockage Azure et déployez-la en tant que fichier de package.
ms.topic: article
ms.date: 03/06/2020
ms.openlocfilehash: 6e4cf16118e748e3ee1d1ff4899730463565ec70
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92108029"
---
# <a name="encrypt-your-application-data-at-rest-using-customer-managed-keys"></a>Chiffrer vos données d’application au repos avec des clés gérées par le client

Le chiffrement des données au repos de votre application de fonction nécessite un compte de stockage Azure et un coffre de clés Azure. Ces services sont utilisés lorsque vous exécutez votre application à partir d’un package de déploiement.

  - [Le stockage Azure permet un chiffrement au repos](../storage/common/storage-service-encryption.md). Vous pouvez utiliser des clés fournies par le système ou vos propres clés gérées par vous-même. C’est dans ce cas que vos données d’application sont stockées lorsqu’elles ne sont pas exécutées dans une application de fonction dans Azure.
  - L’[exécution à partir d’un package de déploiement](run-functions-from-deployment-package.md) est une fonctionnalité de déploiement d’App Service. Elle vous permet de déployer le contenu de votre site à partir d’un compte de stockage Azure, à l’aide d’une URL de signature d’accès partagé (SAS).
  - Les [références Key Vault](../app-service/app-service-key-vault-references.md) sont une fonctionnalité de sécurité d’App Service. Elle vous permet d’importer des secrets au moment de l’exécution comme des paramètres d’application. Utilisez cette valeur pour chiffrer l’URL SAS de votre compte de stockage Azure.

## <a name="set-up-encryption-at-rest"></a>Configurer le chiffrement au repos

### <a name="create-an-azure-storage-account"></a>Création d'un compte Azure Storage

Tout d’abord, [créez un compte de stockage Azure](../storage/common/storage-account-create.md) et [chiffrez-le avec des clés gérées par le client](../storage/common/customer-managed-keys-overview.md). Une fois le compte de stockage créé, utilisez l’[Explorateur Stockage Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md) pour charger les fichiers de package.

Ensuite, utilisez l’Explorateur Stockage pour [générer une URL SAS](../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows#generate-a-sas-in-storage-explorer). 

> [!NOTE]
> Enregistrez cette URL SAS, car vous en aurez besoin plus tard pour permettre un accès sécurisé au package de déploiement au moment de l’exécution.

### <a name="configure-running-from-a-package-from-your-storage-account"></a>Configurer l’exécution à partir d’un package de votre compte de stockage
  
Une fois que vous avez chargé votre fichier dans le stockage Blob et que vous avez une URL SAS pour le fichier, définissez le paramètre d’application `WEBSITE_RUN_FROM_PACKAGE` sur l’URL SAS. Pour ce faire, l’exemple suivant utilise l’interface de ligne de commande Azure :

```
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings WEBSITE_RUN_FROM_PACKAGE="<your-SAS-URL>"
```

L’ajout de ce paramètre d’application entraîne le redémarrage de votre application de fonction. Une fois l’application redémarrée, accédez-y et vérifiez qu’elle a démarré correctement à l’aide du package de déploiement. Si l’application n’a pas démarré correctement, consultez le [guide de résolution des problèmes d’exécution à partir des packages](run-functions-from-deployment-package.md#troubleshooting).

### <a name="encrypt-the-application-setting-using-key-vault-references"></a>Chiffrer le paramètre d’application à l’aide des références Key Vault

À présent, vous pouvez remplacer la valeur du paramètre d’application `WEBSITE_RUN_FROM_PACKAGE` par une référence Key Vault à l’URL encodée au format SAS. De cette façon, l’URL SAS reste chiffrée dans Key Vault, ce qui fournit une couche supplémentaire de sécurité.

1. Exécutez la commande [`az keyvault create`](/cli/azure/keyvault#az-keyvault-create) suivante pour créer une instance Key Vault.       

    ```azurecli    
    az keyvault create --name "Contoso-Vault" --resource-group <group-name> --location eastus    
    ```    

1. Suivez [ces instructions pour accorder à votre application l’accès](../app-service/app-service-key-vault-references.md#granting-your-app-access-to-key-vault) à votre coffre de clés :

1. Utilisez la commande [`az keyvault secret set`](/cli/azure/keyvault/secret#az-keyvault-secret-set) suivante pour ajouter votre URL externe comme clé secrète dans votre coffre de clés :   

    ```azurecli    
    az keyvault secret set --vault-name "Contoso-Vault" --name "external-url" --value "<SAS-URL>"    
    ```    

1.  Utilisez la commande [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) suivante pour créer le paramètre d’application `WEBSITE_RUN_FROM_PACKAGE` avec une valeur servant de référence Key Vault à l’URL externe :

    ```azurecli    
    az webapp config appsettings set --settings WEBSITE_RUN_FROM_PACKAGE="@Microsoft.KeyVault(SecretUri=https://Contoso-Vault.vault.azure.net/secrets/external-url/<secret-version>"    
    ```

    `<secret-version>` se trouvera dans la sortie de la commande `az keyvault secret set` précédente.

La mise à jour de ce paramètre d’application entraîne le redémarrage de votre application de fonction. Une fois que l’application a redémarré, accédez-y pour vérifier qu’elle a démarré correctement à l’aide de la référence Key Vault.

## <a name="how-to-rotate-the-access-token"></a>Rotation du jeton d’accès

Il est recommandé d’effectuer régulièrement la rotation de la clé SAS de votre compte de stockage. Pour garantir que l’application de fonction ne perdra pas son accès, vous devez également mettre à jour l’URL SAS dans Key Vault.

1. Pour effectuer la rotation de la clé SAS, accédez à votre compte de stockage dans le portail Azure. Sous **Paramètres** > **Clés d’accès**, cliquez sur l’icône de rotation des clés SAS.

1. Copiez la nouvelle URL SAS et utilisez la commande suivante pour définir l’URL SAS mise à jour dans votre coffre de clés :

    ```azurecli    
    az keyvault secret set --vault-name "Contoso-Vault" --name "external-url" --value "<SAS-URL>"    
    ``` 

1. Mettez à jour la référence du coffre de clés dans le paramètre d’application en fonction de la version du nouveau secret :

    ```azurecli    
    az webapp config appsettings set --settings WEBSITE_RUN_FROM_PACKAGE="@Microsoft.KeyVault(SecretUri=https://Contoso-Vault.vault.azure.net/secrets/external-url/<secret-version>"    
    ```

    `<secret-version>` se trouvera dans la sortie de la commande `az keyvault secret set` précédente.

## <a name="how-to-revoke-the-function-apps-data-access"></a>Révocation de l’accès aux données de l’application de fonction

Il existe deux méthodes pour révoquer l’accès de l’application de fonction au compte de stockage. 

### <a name="rotate-the-sas-key-for-the-azure-storage-account"></a>Effectuer la rotation de la clé SAS du compte de stockage Azure

Si la clé SAS du compte de stockage a fait l’objet d’une rotation, l’application de fonction n’aura plus accès au compte de stockage, mais elle continuera à s’exécuter avec la dernière version téléchargée du fichier de package. Redémarrez l’application de fonction pour effacer la dernière version téléchargée.

### <a name="remove-the-function-apps-access-to-key-vault"></a>Supprimer l’accès de l’application de fonction à Key Vault

Vous pouvez révoquer l’accès de l’application de fonction aux données de site en désactivant son accès à Key Vault. Pour ce faire, supprimez la stratégie d’accès de l’identité de l’application de fonction. Il s’agit de l’identité que vous avez créée précédemment lors de la configuration des références de coffre de clés.

## <a name="summary"></a>Résumé

Vos fichiers d’application sont maintenant chiffrés au repos dans votre compte de stockage. Lorsque votre application de fonction démarre, elle récupère l’URL SAS à partir de votre coffre de clés. Enfin, l’application de fonction charge les fichiers d’application à partir du compte de stockage. 

Si vous devez révoquer l’accès de l’application de fonction à votre compte de stockage, vous pouvez soit révoquer l’accès au coffre de clés, soit effectuer une rotation des clés de compte de stockage pour rendre l’URL SAS non valide.

## <a name="frequently-asked-questions"></a>Forum Aux Questions (FAQ)

### <a name="is-there-any-additional-charge-for-running-my-function-app-from-the-deployment-package"></a>L’exécution de mon application de fonction à partir du package de déploiement engendre-t-elle des frais supplémentaires ?

Non, seulement le coût associé au compte de stockage Azure et les frais de sortie applicables.

### <a name="how-does-running-from-the-deployment-package-affect-my-function-app"></a>De quelle façon mon application de fonction est-elle affectée par une exécution à partir du package de déploiement ?

- Lorsque vous exécutez votre application à partir du package de déploiement, `wwwroot/` passe en lecture seule. Votre application reçoit une erreur lorsqu’elle tente d’écrire des données dans ce répertoire.
- Les formats TAR et GZIP ne sont pas pris en charge.
- Cette fonctionnalité n’est pas compatible avec le cache local.

## <a name="next-steps"></a>Étapes suivantes

- [Références Key Vault pour App Service](../app-service/app-service-key-vault-references.md)
- [Chiffrement du stockage Azure pour les données au repos](../storage/common/storage-service-encryption.md)