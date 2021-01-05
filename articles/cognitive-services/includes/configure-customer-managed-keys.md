---
title: Utiliser le Portail Microsoft Azure pour configurer des clés gérées par le client
titleSuffix: Cognitive Services
description: Découvrez comment utiliser le portail Azure pour configurer des clés gérées par le client avec Azure Key Vault. Les clés gérées par le client vous permettent de créer, faire pivoter, désactiver et révoquer des contrôles d’accès.
services: cognitive-services
author: erindormier
ms.service: cognitive-services
ms.topic: include
ms.date: 05/28/2020
ms.author: egeaney
ms.openlocfilehash: c79846b0a5b675c34e4e7919e9ecd9d591bfefe5
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96356356"
---
## <a name="customer-managed-keys-with-azure-key-vault"></a>Clés gérées par le client avec Azure Key Vault

Vous devez utiliser Azure Key Vault pour stocker les clés gérées par le client. Vous pouvez créer vos propres clés et les stocker dans un coffre de clés, ou utiliser les API d’Azure Key Vault pour générer des clés. La ressource Cognitive Services et le coffre de clés doivent se trouver dans la même région et dans le même locataire Azure Active Directory, mais ils peuvent appartenir à des abonnements différents. Pour plus d’informations sur Azure Key Vault, consultez [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md).

Quand une ressource Cognitive Services est créée, elle est toujours chiffrée à l’aide de clés gérées par Microsoft. Il est impossible d'activer des clés gérées par le client au moment de la création de la ressource. Les clés gérées par le client sont stockées dans Azure Key Vault, et le coffre de clés doit être configuré avec des stratégies d’accès qui accordent des autorisations de clé à l’identité managée associée à la ressource Cognitive Services. L’identité managée est disponible uniquement une fois la ressource créée à l’aide du niveau tarifaire requis pour la CMK.

L’activation des clés gérées par le client active également une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md) affectée par le système, une fonctionnalité d’Azure AD. Une fois activée, l'identité managée affectée par le système est inscrite auprès d'Azure Active Directory. Une fois inscrite, elle a accès au coffre de clés sélectionné lors de la configuration des clés gérées par le client. 

> [!IMPORTANT]
> Si vous désactivez les identités managées affectées par le système, l'accès au coffre de clés est supprimé et toutes les données chiffrées avec les clés client deviennent inaccessibles. Toutes les fonctionnalités dépendant de ces données cessent de fonctionner.

> [!IMPORTANT]
> Les identités managées ne prennent actuellement pas en charge les scénarios entre annuaires. Lorsque vous configurez des clés managées par le client sur le portail Azure, une identité managée est automatiquement affectée. Si, par la suite, vous déplacez l'abonnement, le groupe de ressources ou la ressource d'un répertoire Azure AD vers un autre, l'identité managée associée à la ressource n'est pas transférée vers le nouveau locataire. Par conséquent, les clés gérées par le client peuvent ne plus fonctionner. Pour plus d’informations, consultez **Transfert d’un abonnement entre des répertoires Azure AD** dans [FAQ et problèmes connus en lien avec les identités managées pour ressources Azure](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).  

## <a name="configure-azure-key-vault"></a>Configurer Azure Key Vault

Pour utiliser des clés gérées par le client, deux propriétés doivent être configurées dans le coffre de clés, **Suppression réversible** et **Ne pas vider**. Ces propriétés ne sont pas activées par défaut, mais peuvent être activées à l’aide de PowerShell ou d’Azure CLI sur un coffre de clés nouveau ou existant.

> [!IMPORTANT]
> Si les propriétés **Suppression réversible** et **Ne pas purger** ne sont pas activées et que vous supprimez votre clé, vous ne pourrez pas récupérer les données dans votre ressource Cognitive Service.

Pour savoir comment activer ces propriétés sur un coffre de clés existant, consultez les sections intitulées **Activation de la suppression réversible** et **Activation de la protection contre le vidage** dans l’un des articles suivants :

- [Guide pratique pour utiliser la suppression réversible avec Azure Power​Shell](../../key-vault/general/key-vault-recovery.md).
- [Guide pratique pour utiliser la suppression réversible avec Azure CLI](../../key-vault/general/key-vault-recovery.md).

Seules les clés RSA de taille 2048 sont prises en charge avec le chiffrement Azure Storage. Pour plus d’informations sur les clés, consultez **Clés Key Vault** dans [À propos des clés, des secrets et des certificats Azure Key Vault](../../key-vault/general/about-keys-secrets-certificates.md).

## <a name="enable-customer-managed-keys-for-your-resource"></a>Activer des clés gérées par le client pour votre ressource

Pour activer des clés gérées par le client dans le portail Azure, procédez comme suit :

1. Accédez à votre ressource Cognitive Services.
1. Dans le panneau **Paramètres** de votre ressource Cognitive Services, cliquez sur **Chiffrement**. Sélectionnez l’option **Clés gérées par le client**, comme illustré dans l’illustration suivante.

    ![Capture d’écran montrant comment sélectionner des clés gérées par le client](../media/cognitive-services-encryption/selectcmk.png)

## <a name="specify-a-key"></a>Spécifier une clé

Après avoir activé les clés gérées par le client, vous pourrez spécifier une clé à associer à la ressource Cognitive Services.

### <a name="specify-a-key-as-a-uri"></a>Spécifier une clé en tant qu’URI

Pour spécifier une clé en tant qu’URI, procédez comme suit :

1. Pour localiser l’URI de la clé dans le portail Azure, naviguez jusqu'à votre coffre de clés, puis sélectionnez le paramètre **Clés**. Sélectionnez la clé souhaitée, puis cliquez dessus pour afficher ses versions. Sélectionnez une version de clé pour afficher les paramètres de cette version.
1. Copiez la valeur du champ **Identificateur de clé**, qui fournit l’URI.

    ![Capture d’écran montrant l’URI de la clé du coffre de clés](../media/cognitive-services-encryption/key-uri-portal.png)

1. Dans les paramètres de **chiffrement** de votre compte de stockage, choisissez l’option **Entrer l’URI de la clé**.
1. Collez l’URI que vous avez copié dans le champ **URI de clé**.

   ![Capture d’écran montrant comment entrer l’URI d’une clé](../media/cognitive-services-encryption/ssecmk2.png)

1. Spécifiez l’abonnement qui contient le coffre de clés.
1. Enregistrez vos modifications.

### <a name="specify-a-key-from-a-key-vault"></a>Spécifiez une clé à partir d’un coffre de clés

Pour spécifier une clé à partir d’un coffre de clés, assurez-vous d’abord que vous disposez d’un coffre de clés contenant une clé. Pour spécifier une clé à partir d’un coffre de clés, procédez comme suit :

1. Choisissez l’option **Sélectionner dans le coffre de clés**.
1. Sélectionnez le coffre de clés contenant la clé que vous souhaitez utiliser.
1. Sélectionnez la clé dans le coffre de clés.

   ![Capture d’écran montrant une option de clé gérée par le client](../media/cognitive-services-encryption/ssecmk3.png)

1. Enregistrez vos modifications.

## <a name="update-the-key-version"></a>Mettre à jour la version de la clé

Lors de la création d’une nouvelle version d’une clé, mettez à jour la ressource Cognitive Services afin qu’elle utilise cette nouvelle version. Procédez comme suit :

1. Accédez à votre ressource de Cognitive Services et affichez les paramètres **Chiffrement**.
1. Saisissez l’URI de la nouvelle version de clé. Vous pouvez également sélectionner à nouveau le coffre de clés et la clé pour mettre à jour la version.
1. Enregistrez vos modifications.

## <a name="use-a-different-key"></a>Utiliser une autre clé

Pour modifier la clé utilisée pour le chiffrement, procédez comme suit :

1. Accédez à votre ressource de Cognitive Services et affichez les paramètres **Chiffrement**.
1. Entrez l’URI de la nouvelle clé. Vous pouvez également sélectionner le coffre de clés et choisir une nouvelle clé.
1. Enregistrez vos modifications.

## <a name="rotate-customer-managed-keys"></a>Permuter des clés gérées par le client

Vous pouvez permuter une clé gérée par le client dans Azure Key Vault en fonction de vos stratégies de conformité. Une fois la rotation de la clé effectuée, vous devez mettre à jour la ressource Cognitive Services pour utiliser le nouvel URI de clé. Pour savoir comment mettre à jour la ressource afin d’utiliser une nouvelle version de la clé dans le portail Azure, consultez [Mettre à jour la version de la clé](#update-the-key-version).

La rotation de la clé ne déclenche pas de rechiffrement de données dans la ressource. Aucune autre action n’est requise de la part de l’utilisateur.

## <a name="revoke-access-to-customer-managed-keys"></a>Révoquer l’accès aux clés gérées par le client

Pour révoquer l’accès aux clés gérées par le client, utilisez PowerShell ou Azure CLI. Pour plus d’informations, consultez [Azure Key Vault PowerShell](/powershell/module/az.keyvault//) ou [Interface de ligne de commande Azure Key Vault](/cli/azure/keyvault). La révocation de l’accès bloque l’accès à toutes les données de la ressource Cognitive Services, car Cognitive Services n’a pas accès à la clé de chiffrement.

## <a name="disable-customer-managed-keys"></a>Désactiver les clés gérées par le client

Lorsque vous désactivez les clés gérées par le client, votre ressource Cognitive Services est chiffrée avec des clés gérées par Microsoft. Pour désactiver les clés gérées par le client, procédez comme suit :

1. Accédez à votre ressource de Cognitive Services et affichez les paramètres **Chiffrement**.
1. Désactivez la case à cocher en regard du paramètre **Utiliser votre propre clé**.