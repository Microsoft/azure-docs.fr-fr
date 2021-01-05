---
title: Chiffrement des données au repos du service Commandes personnalisées
titleSuffix: Azure Cognitive Services
description: Chiffrement des données au repos des Commandes personnalisées
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2020
ms.author: sausin
ms.openlocfilehash: 0f932eed2f1d58e8470a24ea595e21712deb7f03
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021896"
---
# <a name="custom-commands-encryption-of-data-at-rest"></a>Chiffrement des données au repos des Commandes personnalisées

Le service Commandes personnalisées chiffre automatiquement vos données lors de leur conservation dans le cloud. Le chiffrement du service Commandes personnalisées protège vos données et vous aide à répondre aux engagements de votre entreprise en matière de sécurité et de conformité.

> [!NOTE]
> Le service Commandes personnalisées n’active pas automatiquement le chiffrement pour les ressources LUIS associées à votre application. Si nécessaire, vous devez activer le chiffrement pour votre ressource LUIS à partir d’[ici](./../LUIS/luis-encryption-of-data-at-rest.md).

## <a name="about-cognitive-services-encryption"></a>À propos du chiffrement de Cognitive Services
Les données sont chiffrées et déchiffrées à l'aide du chiffrement [AES 256 bits](https://en.wikipedia.org/wiki/FIPS_140-2) certifié [FIPS 140-2](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). Le chiffrement et le déchiffrement sont transparents, ce qui signifie que le chiffrement et l’accès sont gérés automatiquement. Vos données étant sécurisées par défaut, vous n’avez pas besoin de modifier votre code ou vos applications pour tirer parti du chiffrement.

## <a name="about-encryption-key-management"></a>À propos de la gestion des clés de chiffrement

Lorsque vous utilisez le service Commandes personnalisées, le service de reconnaissance vocale stocke les données suivantes dans le cloud :
* Configuration JSON derrière l’application Commandes personnalisées
* Clés de création et de prédiction LUIS

Par défaut, votre abonnement utilise des clés de chiffrement gérées par Microsoft. Toutefois, vous pouvez également gérer votre abonnement avec vos propres clés de chiffrement. Les clés gérées par le client (CMK), également appelées BYOK (Bring Your Own Key), offrent plus de flexibilité pour créer, permuter, désactiver et révoquer des contrôles d’accès. Vous pouvez également effectuer un audit sur les clés de chiffrement utilisées pour protéger vos données.


> [!IMPORTANT]
> Les clés gérées par le client sont uniquement des ressources disponibles créées après le 27 mai 2020. Pour utiliser CMK avec les services de reconnaissance vocale, vous devez créer une ressource de reconnaissance vocale. Une fois la ressource créée, vous pouvez utiliser Azure Key Vault pour configurer votre identité managée.

Pour demander la possibilité d’utiliser des clés gérées par le client, complétez et envoyez le Formulaire de demande de clé gérée par le client. Comptez environ 3 à 5 jours ouvrables pour obtenir des nouvelles sur le statut de votre demande. Selon la demande, vous pouvez être placé dans une file d’attente et approuvé lorsque de l’espace devient disponible. Une fois que vous avez approuvé l’utilisation de CMK avec les services de reconnaissance vocale, vous devez créer une ressource de reconnaissance vocale à partir du portail Azure.
   > [!NOTE]
   > **Les clés gérées par le client (CMK) sont prises en charge uniquement pour le service Commandes personnalisées.**
   >
   >  **Les services Custom Speech et Custom Voice ne prennent toujours en charge que la stratégie BYOS.**  [En savoir plus](speech-encryption-of-data-at-rest.md)
   >
   > Si vous utilisez la ressource de reconnaissance vocale donnée pour accéder à ces services, vous devez respecter les exigences de conformité en configurant explicitement la stratégie BYOS.


## <a name="customer-managed-keys-with-azure-key-vault"></a>Clés gérées par le client avec Azure Key Vault

Vous devez utiliser Azure Key Vault pour stocker les clés gérées par le client. Vous pouvez créer vos propres clés et les stocker dans un coffre de clés, ou utiliser les API d’Azure Key Vault pour générer des clés. La ressource de reconnaissance vocale et le coffre de clés doivent se trouver dans la même région et dans le même locataire Azure Active Directory, mais ils peuvent être dans des abonnements différents. Pour plus d’informations sur Azure Key Vault, consultez [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md).

Quand une nouvelle ressource de reconnaissance vocale est créée et utilisée pour approvisionner l’application Commandes personnalisées, les données sont toujours chiffrées à l’aide de clés gérées par Microsoft. Il est impossible d'activer des clés gérées par le client au moment de la création de la ressource. Les clés gérées par le client sont stockées dans Azure Key Vault, et le coffre de clés doit être configuré avec des stratégies d’accès qui accordent des autorisations de clé à l’identité managée associée à la ressource Cognitive Services. L’identité managée est disponible uniquement une fois la ressource créée à l’aide du niveau tarifaire requis pour la CMK.

L’activation des clés gérées par le client active également une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md) affectée par le système, une fonctionnalité d’Azure AD. Une fois activée, l'identité managée affectée par le système est inscrite auprès d'Azure Active Directory. Une fois inscrite, elle a accès au coffre de clés sélectionné lors de la configuration des clés gérées par le client. 

> [!IMPORTANT]
> Si vous désactivez les identités managées affectées par le système, l'accès au coffre de clés est supprimé et toutes les données chiffrées avec les clés client deviennent inaccessibles. Toutes les fonctionnalités dépendant de ces données cessent de fonctionner.

> [!IMPORTANT]
> Les identités managées ne prennent actuellement pas en charge les scénarios entre répertoires. Lorsque vous configurez des clés managées par le client sur le portail Azure, une identité managée est automatiquement affectée. Si, par la suite, vous déplacez l'abonnement, le groupe de ressources ou la ressource d'un répertoire Azure AD vers un autre, l'identité managée associée à la ressource n'est pas transférée vers le nouveau locataire. Par conséquent, les clés gérées par le client peuvent ne plus fonctionner. Pour plus d’informations, consultez **Transfert d’un abonnement entre des répertoires Azure AD** dans [FAQ et problèmes connus en lien avec les identités managées pour ressources Azure](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).  

## <a name="configure-azure-key-vault"></a>Configurer Azure Key Vault

Pour utiliser des clés gérées par le client, deux propriétés doivent être configurées dans le coffre de clés, **Suppression réversible** et **Ne pas vider**. Ces propriétés ne sont pas activées par défaut, mais peuvent être activées à l’aide de PowerShell ou d’Azure CLI sur un coffre de clés nouveau ou existant.

> [!IMPORTANT]
> Si les propriétés **Suppression réversible** et **Ne pas purger** ne sont pas activées et que vous supprimez votre clé, vous ne pourrez pas récupérer les données dans votre ressource Cognitive Service.

Pour savoir comment activer ces propriétés sur un coffre de clés existant, consultez les sections intitulées **Activation de la suppression réversible** et **Activation de la protection contre le vidage** dans l’un des articles suivants :

- [Guide pratique pour utiliser la suppression réversible avec Azure Power​Shell](../../key-vault/general/key-vault-recovery.md).
- [Guide pratique pour utiliser la suppression réversible avec Azure CLI](../../key-vault/general/key-vault-recovery.md).

Seules les clés RSA de taille 2048 sont prises en charge avec le chiffrement Azure Storage. Pour plus d’informations sur les clés, consultez **Clés Key Vault** dans [À propos des clés, des secrets et des certificats Azure Key Vault](../../key-vault/general/about-keys-secrets-certificates.md).

## <a name="enable-customer-managed-keys-for-your-speech-resource"></a>Activer des clés gérées par le client pour votre ressource de reconnaissance vocale

Pour activer des clés gérées par le client dans le portail Azure, procédez comme suit :

1. Accédez à votre ressource de reconnaissance vocale.
1. Dans le panneau **Paramètres** pour votre ressource de reconnaissance vocale, cliquez sur **Chiffrement**. Sélectionnez l’option **Clés gérées par le client**, comme illustré dans l’illustration suivante.

 ![Capture d’écran montrant comment sélectionner des clés gérées par le client](media/custom-commands/select-cmk.png)

## <a name="specify-a-key"></a>Spécifier une clé

Après avoir activé les clés gérées par le client, vous pourrez spécifier une clé à associer à la ressource Cognitive Services.

### <a name="specify-a-key-as-a-uri"></a>Spécifier une clé en tant qu’URI

Pour spécifier une clé en tant qu’URI, procédez comme suit :

1. Pour localiser l’URI de la clé dans le portail Azure, naviguez jusqu'à votre coffre de clés, puis sélectionnez le paramètre **Clés**. Sélectionnez la clé souhaitée, puis cliquez dessus pour afficher ses versions. Sélectionnez une version de clé pour afficher les paramètres de cette version.
1. Copiez la valeur du champ **Identificateur de clé**, qui fournit l’URI.

    ![Capture d’écran montrant l’URI de la clé du coffre de clés](../media/cognitive-services-encryption/key-uri-portal.png)

1. Dans les paramètres de **Chiffrement** de votre service de reconnaissance vocale, choisissez l’option **Entrer l’URI de la clé**.
1. Collez l’URI que vous avez copié dans le champ **URI de clé**.

1. Spécifiez l’abonnement qui contient le coffre de clés.
1. Enregistrez vos modifications.

### <a name="specify-a-key-from-a-key-vault"></a>Spécifiez une clé à partir d’un coffre de clés

Pour spécifier une clé à partir d’un coffre de clés, assurez-vous d’abord que vous disposez d’un coffre de clés contenant une clé. Pour spécifier une clé à partir d’un coffre de clés, procédez comme suit :

1. Choisissez l’option **Sélectionner dans le coffre de clés**.
1. Sélectionnez le coffre de clés contenant la clé que vous souhaitez utiliser.
1. Sélectionnez la clé dans le coffre de clés.

   ![Capture d’écran montrant une option de clé gérée par le client](media/custom-commands/configure-cmk-fromKV.png)

1. Enregistrez vos modifications.

## <a name="update-the-key-version"></a>Mettre à jour la version de la clé

Lorsque vous créez une nouvelle version d’une clé, mettez à jour la ressource de reconnaissance vocale afin qu’elle utilise la nouvelle version. Procédez comme suit :

1. Accédez à votre ressource de reconnaissance vocale et affichez les paramètres de **Chiffrement**.
1. Saisissez l’URI de la nouvelle version de clé. Vous pouvez également sélectionner à nouveau le coffre de clés et la clé pour mettre à jour la version.
1. Enregistrez vos modifications.

## <a name="use-a-different-key"></a>Utiliser une autre clé

Pour modifier la clé utilisée pour le chiffrement, procédez comme suit :

1. Accédez à votre ressource de reconnaissance vocale et affichez les paramètres de **Chiffrement**.
1. Entrez l’URI de la nouvelle clé. Vous pouvez également sélectionner le coffre de clés et choisir une nouvelle clé.
1. Enregistrez vos modifications.

## <a name="rotate-customer-managed-keys"></a>Permuter des clés gérées par le client

Vous pouvez permuter une clé gérée par le client dans Azure Key Vault en fonction de vos stratégies de conformité. Une fois la rotation de clé effectuée, vous devez mettre à jour la ressource de reconnaissance vocale pour utiliser le nouvel URI de clé. Pour savoir comment mettre à jour la ressource afin d’utiliser une nouvelle version de la clé dans le portail Azure, consultez [Mettre à jour la version de la clé](#update-the-key-version).

La rotation de la clé ne déclenche pas de rechiffrement de données dans la ressource. Aucune autre action n’est requise de la part de l’utilisateur.

## <a name="revoke-access-to-customer-managed-keys"></a>Révoquer l’accès aux clés gérées par le client

Pour révoquer l’accès aux clés gérées par le client, utilisez PowerShell ou Azure CLI. Pour plus d’informations, consultez [Azure Key Vault PowerShell](/powershell/module/az.keyvault//) ou [Interface de ligne de commande Azure Key Vault](/cli/azure/keyvault). La révocation de l’accès bloque l’accès à toutes les données de la ressource Cognitive Services, car Cognitive Services n’a pas accès à la clé de chiffrement.

## <a name="disable-customer-managed-keys"></a>Désactiver les clés gérées par le client

Lorsque vous désactivez des clés gérées par le client, votre ressource de reconnaissance vocale est chiffrée avec des clés gérées par Microsoft. Pour désactiver les clés gérées par le client, procédez comme suit :

1. Accédez à votre ressource de reconnaissance vocale et affichez les paramètres de **Chiffrement**.
1. Désactivez la case à cocher en regard du paramètre **Utiliser votre propre clé**.

## <a name="next-steps"></a>Étapes suivantes

* [Formulaire de demande de clé gérée par le client du service de reconnaissance vocale](https://aka.ms/cogsvc-cmk)
* [En savoir plus sur Azure Key Vault](../../key-vault/general/overview.md)
* [Que sont les identités managées ?](../../active-directory/managed-identities-azure-resources/overview.md)
