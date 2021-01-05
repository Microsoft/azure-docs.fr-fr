---
title: Chiffrement des données au repos du service Language Understanding
titleSuffix: Azure Cognitive Services
description: Microsoft propose des clés de chiffrement gérées par Microsoft et vous permet également de gérer vos abonnements Cognitive Services à l’aide de vos propres clés, appelées clés gérées par le client (CMK). Cet article traite du chiffrement des données au repos pour Language Understanding (LUIS) et de l’activation et de la gestion de CMK.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: egeaney
ms.openlocfilehash: 2dcfff005eaaac034f5fed13b6d4d18e20d2afae
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018971"
---
# <a name="language-understanding-service-encryption-of-data-at-rest"></a>Chiffrement des données au repos du service Language Understanding

Le service Language Understanding chiffre automatiquement vos données lors de leur conservation dans le cloud. Le chiffrement du service Language Understanding protège vos données et vous aide à répondre aux engagements de votre entreprise en matière de sécurité et de conformité.

## <a name="about-cognitive-services-encryption"></a>À propos du chiffrement de Cognitive Services

Les données sont chiffrées et déchiffrées à l'aide du chiffrement [AES 256 bits](https://en.wikipedia.org/wiki/FIPS_140-2) certifié [FIPS 140-2](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). Le chiffrement et le déchiffrement sont transparents, ce qui signifie que le chiffrement et l’accès sont gérés automatiquement. Vos données étant sécurisées par défaut, vous n’avez pas besoin de modifier votre code ou vos applications pour tirer parti du chiffrement.

## <a name="about-encryption-key-management"></a>À propos de la gestion des clés de chiffrement

Par défaut, votre abonnement utilise des clés de chiffrement gérées par Microsoft. Il existe aussi la possibilité de gérer votre abonnement avec vos propres clés appelées clés gérées par le client (CMK). Les CMK offrent plus de flexibilité pour créer, alterner, désactiver et révoquer des contrôles d’accès. Vous pouvez également effectuer un audit sur les clés de chiffrement utilisées pour protéger vos données.

## <a name="customer-managed-keys-with-azure-key-vault"></a>Clés gérées par le client avec Azure Key Vault

Une option vous permet également de gérer votre abonnement avec vos propres clés. Les clés gérées par le client (CMK), également appelées BYOK (Bring Your Own Key), offrent plus de flexibilité pour créer, permuter, désactiver et révoquer des contrôles d'accès. Vous pouvez également effectuer un audit sur les clés de chiffrement utilisées pour protéger vos données.

Vous devez utiliser Azure Key Vault pour stocker vos clés managées par le client. Vous pouvez créer vos propres clés et les stocker dans un coffre de clés, ou utiliser les API d’Azure Key Vault pour générer des clés. La ressource Cognitive Services et le coffre de clés doivent se trouver dans la même région et dans le même locataire Azure Active Directory, mais ils peuvent appartenir à des abonnements différents. Pour plus d’informations sur Azure Key Vault, consultez [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md).

### <a name="customer-managed-keys-for-language-understanding"></a>Clés gérées par le client pour Language Understanding

Pour demander la possibilité d’utiliser des clés gérées par le client, complétez et envoyez le  [formulaire de demande de clé gérée par le client du service LUIS](https://aka.ms/cogsvc-cmk). Comptez environ 3 à 5 jours ouvrables pour obtenir des nouvelles sur le statut de votre demande. Selon la demande, vous pouvez être placé dans une file d’attente et approuvé lorsque de l’espace devient disponible. Une fois que vous avez approuvé l’utilisation de CMK avec LUIS, vous devez créer une ressource Language Understanding à partir du portail Azure et sélectionner E0 comme niveau tarifaire. La nouvelle référence (SKU) fonctionnera de la même façon que la référence F0 déjà disponible, à l’exception de CMK. Les utilisateurs n'auront pas la possibilité de procéder à la mise à niveau de la référence SKU F0 vers la nouvelle référence SKU E0.

![Image d’abonnement LUIS](../media/cognitive-services-encryption/luis-subscription.png)

### <a name="limitations"></a>Limites

Il existe certaines limitations lors de l’utilisation du niveau E0 avec les applications existantes/créées précédemment :

* La migration vers une ressource E0 sera bloquée. Les utilisateurs pourront uniquement migrer leurs applications vers les ressources F0. Une fois que vous avez migré une ressource existante vers F0, vous pouvez créer une ressource dans le niveau E0. Vous trouverez plus d’informations sur la [migration ici](./luis-migration-authoring.md).  
* Le déplacement d’applications vers ou depuis une ressource E0 est bloqué. Pour contourner cette limitation, vous pouvez exporter votre application existante et l’importer en tant que ressource E0.
* La fonctionnalité Vérification orthographique Bing n’est pas prise en charge.
* La journalisation du trafic de l’utilisateur final est désactivée si votre application est E0.
* La fonctionnalité de préparation vocale du service Azure Bot n’est pas prise en charge pour les applications du niveau E0. Cette fonctionnalité est disponible via le service Azure Bot, qui ne prend pas en charge CMK.
* La fonctionnalité de préparation vocale depuis le portail requiert Stockage Blob Azure. Pour plus d'informations, consultez [Bring Your Own Storage](../Speech-Service/speech-encryption-of-data-at-rest.md#bring-your-own-storage-byos-for-customization-and-logging).

### <a name="enable-customer-managed-keys"></a>Activer des clés gérées par le client

Une nouvelle ressource Cognitive Services est toujours chiffrée à l'aide de clés gérées par Microsoft. Il est impossible d'activer des clés gérées par le client au moment de la création de la ressource. Les clés gérées par le client sont stockées dans Azure Key Vault, et le coffre de clés doit être configuré avec des stratégies d’accès qui accordent des autorisations de clé à l’identité managée associée à la ressource Cognitive Services. L’identité managée est disponible uniquement une fois la ressource créée à l’aide du niveau tarifaire pour la CMK.

Pour en savoir plus sur l’utilisation des clés gérées par le client avec Azure Key Vault pour le chiffrement Cognitive Services, consultez :

- [Configurer les clés gérées par le client avec Key Vault pour le chiffrement Cognitive Services depuis le portail Azure](../Encryption/cognitive-services-encryption-keys-portal.md)

L'activation des clés gérées par le client active également une identité managée affectée par le système, une fonctionnalité d'Azure AD. Une fois activée, l'identité managée affectée par le système est inscrite auprès d'Azure Active Directory. Une fois inscrite, elle a accès au coffre de clés sélectionné lors de la configuration des clés gérées par le client. Découvrez-en plus sur les [identités managées](../../active-directory/managed-identities-azure-resources/overview.md).

> [!IMPORTANT]
> Si vous désactivez les identités managées affectées par le système, l'accès au coffre de clés est supprimé et toutes les données chiffrées avec les clés client deviennent inaccessibles. Toutes les fonctionnalités dépendant de ces données cessent de fonctionner.

> [!IMPORTANT]
> Les identités managées ne prennent actuellement pas en charge les scénarios entre répertoires. Lorsque vous configurez des clés managées par le client sur le portail Azure, une identité managée est automatiquement affectée. Si, par la suite, vous déplacez l'abonnement, le groupe de ressources ou la ressource d'un répertoire Azure AD vers un autre, l'identité managée associée à la ressource n'est pas transférée vers le nouveau locataire. Par conséquent, les clés gérées par le client peuvent ne plus fonctionner. Pour plus d’informations, consultez **Transfert d’un abonnement entre des répertoires Azure AD** dans [FAQ et problèmes connus en lien avec les identités managées pour ressources Azure](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).  

### <a name="store-customer-managed-keys-in-azure-key-vault"></a>Stocker les clés gérées par le client dans Azure Key Vault

Pour activer les clés gérées par le client, vous devez utiliser une instance d'Azure Key Vault afin de stocker vos clés. Vous devez activer les propriétés **Suppression réversible** et **Ne pas vider** sur le coffre de clés.

Seules les clés RSA de taille 2048 sont prises en charge par le chiffrement Cognitive Services. Pour plus d’informations sur les clés, consultez **Clés Key Vault** dans [À propos des clés, des secrets et des certificats Azure Key Vault](../../key-vault/general/about-keys-secrets-certificates.md).

### <a name="rotate-customer-managed-keys"></a>Permuter des clés gérées par le client

Vous pouvez permuter une clé gérée par le client dans Azure Key Vault en fonction de vos stratégies de conformité. Une fois la rotation de la clé effectuée, vous devez mettre à jour la ressource Cognitive Services pour utiliser le nouvel URI de clé. Pour savoir comment mettre à jour la ressource afin d’utiliser une nouvelle version de la clé dans le portail Azure, consultez la section intitulée **Mettre à jour la version de la clé** dans [Configurer les clés gérées par le client pour Cognitive Services à l’aide du portail Azure](../Encryption/cognitive-services-encryption-keys-portal.md).

La rotation de la clé ne déclenche pas de rechiffrement de données dans la ressource. Aucune autre action n’est requise de la part de l’utilisateur.

### <a name="revoke-access-to-customer-managed-keys"></a>Révoquer l’accès aux clés gérées par le client

Pour révoquer l’accès aux clés gérées par le client, utilisez PowerShell ou Azure CLI. Pour plus d’informations, consultez [Azure Key Vault PowerShell](/powershell/module/az.keyvault//) ou [Interface de ligne de commande Azure Key Vault](/cli/azure/keyvault). La révocation de l’accès bloque l’accès à toutes les données de la ressource Cognitive Services, car Cognitive Services n’a pas accès à la clé de chiffrement.

## <a name="next-steps"></a>Étapes suivantes

* [Formulaire de demande de clé gérée par le client du service LUIS](https://aka.ms/cogsvc-cmk)
* [En savoir plus sur Azure Key Vault](../../key-vault/general/overview.md)
