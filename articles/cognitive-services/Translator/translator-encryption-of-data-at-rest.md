---
title: Chiffrement des données au repos par le traducteur
titleSuffix: Azure Cognitive Services
description: Microsoft vous permet de gérer vos abonnements Cognitive Services à l’aide de vos propres clés, appelées clés gérées par le client (CMK). Cet article traite du chiffrement des données au repos pour Traducteur et de l’activation et de la gestion de CMK.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: egeaney
ms.openlocfilehash: ff7b9f86cebc3c2479105d2a52aa92a265f8a1b3
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95016511"
---
# <a name="translator-encryption-of-data-at-rest"></a>Chiffrement des données au repos par le traducteur

Le Traducteur chiffre automatiquement vos données, que vous chargez pour créer des modèles de traduction personnalisés, lorsqu'elles sont conservées dans le cloud, ce qui vous permet de satisfaire aux exigences de votre organisation en matière de sécurité et de conformité.

## <a name="about-cognitive-services-encryption"></a>À propos du chiffrement Cognitive Services

Les données sont chiffrées et déchiffrées à l'aide du chiffrement [AES 256 bits](https://en.wikipedia.org/wiki/FIPS_140-2) certifié [FIPS 140-2](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). Le chiffrement et le déchiffrement sont transparents, ce qui signifie que le chiffrement et l’accès sont gérés automatiquement. Vos données étant sécurisées par défaut, vous n’avez pas besoin de modifier votre code ou vos applications pour tirer parti du chiffrement.

## <a name="about-encryption-key-management"></a>À propos de la gestion des clés de chiffrement

Par défaut, votre abonnement utilise des clés de chiffrement gérées par Microsoft. Si vous utilisez un niveau tarifaire qui prend en charge les clés gérées par le client, les paramètres de chiffrement de votre ressource sont disponibles dans la section **Chiffrement** du [portail Azure](https://portal.azure.com), comme illustré ci-dessous.

![Afficher les paramètres de chiffrement](../media/cognitive-services-encryption/encryptionblade.png)

Pour les abonnements qui prennent uniquement en charge les clés de chiffrement gérées par Microsoft, la section **Chiffrement** n'est pas disponible.

## <a name="customer-managed-keys-with-azure-key-vault"></a>Clés gérées par le client avec Azure Key Vault

Par défaut, votre abonnement utilise des clés de chiffrement gérées par Microsoft. Il existe aussi la possibilité de gérer votre abonnement avec vos propres clés appelées clés gérées par le client (CMK). Les CMK offrent plus de flexibilité pour créer, alterner, désactiver et révoquer des contrôles d’accès. Vous pouvez également effectuer un audit sur les clés de chiffrement utilisées pour protéger vos données. Si une CMK est configurée pour votre abonnement, un double chiffrement est fourni, ce qui offre une deuxième couche de protection, tout en vous permettant de contrôler la clé de chiffrement par le biais de votre coffre de clés Azure.

> [!IMPORTANT]
> Des clés gérées par le client sont disponibles pour tous les niveaux tarifaires du service Traducteur. Pour demander à utiliser des clés gérées par le client, remplissez et envoyez le [Formulaire de demande de clés gérées par le client du Traducteur](https://aka.ms/cogsvc-cmk). Comptez 3 à 5 jours ouvrés pour recevoir une réponse. Selon la demande, vous pouvez être mis en file d'attente. Dès l'approbation de votre demande d'utilisation de clés gérées par le client auprès du service Traducteur, vous devez créer une nouvelle ressource Traducteur. Une fois votre ressource Traducteur créée, vous pouvez utiliser Azure Key Vault pour configurer votre identité managée.

Afin d'activer les clés gérées par le client pour le Traducteur, procédez comme suit :

1. Créez votre nouvelle ressource Translator régionale ou Cognitive Services régionale. Cela ne fonctionnera pas avec une ressource globale.
2. Activez l'identité managée sur le portail Azure et ajoutez les informations relatives à vos clés gérées par le client.
3. Créez un nouvel espace de travail dans Custom Translator et associez ces informations d'abonnement.

### <a name="enable-customer-managed-keys"></a>Activer des clés gérées par le client

Vous devez utiliser Azure Key Vault pour stocker vos clés managées par le client. Vous pouvez créer vos propres clés et les stocker dans un coffre de clés, ou utiliser les API d’Azure Key Vault pour générer des clés. La ressource Cognitive Services et le coffre de clés doivent se trouver dans la même région et dans le même locataire Azure Active Directory, mais ils peuvent appartenir à des abonnements différents. Pour plus d’informations sur Azure Key Vault, consultez [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md).

Une nouvelle ressource Cognitive Services est toujours chiffrée à l'aide de clés gérées par Microsoft. Il est impossible d'activer des clés gérées par le client au moment de la création de la ressource. Les clés gérées par le client sont stockées dans Azure Key Vault, et le coffre de clés doit être configuré avec des stratégies d'accès qui accordent des autorisations de clé à l'identité managée associée à la ressource Cognitive Services. L'identité managée est disponible dès que la ressource est créée.

Pour en savoir plus sur l'utilisation des clés gérées par le client avec Azure Key Vault pour le chiffrement Cognitive Services, consultez :

- [Configurer les clés gérées par le client avec Key Vault pour le chiffrement Cognitive Services depuis le portail Azure](../Encryption/cognitive-services-encryption-keys-portal.md)

L'activation des clés gérées par le client active également une identité managée affectée par le système, une fonctionnalité d'Azure AD. Une fois activée, l'identité managée affectée par le système est inscrite auprès d'Azure Active Directory. Une fois inscrite, elle a accès au coffre de clés sélectionné lors de la configuration des clés gérées par le client. Découvrez-en plus sur les [identités managées](../../active-directory/managed-identities-azure-resources/overview.md).

> [!IMPORTANT]
> Si vous désactivez les identités managées affectées par le système, l'accès au coffre de clés est supprimé et toutes les données chiffrées avec les clés client deviennent inaccessibles. Toutes les fonctionnalités dépendant de ces données cessent de fonctionner. Tous les modèles que vous avez déployés sont également annulés. Toutes les données chargées sont supprimées de Custom Translator. Si les identités managées sont réactivées, le redéploiement du modèle n'est pas automatique.

> [!IMPORTANT]
> Les identités managées ne prennent actuellement pas en charge les scénarios entre annuaires. Lorsque vous configurez des clés managées par le client sur le portail Azure, une identité managée est automatiquement affectée. Si, par la suite, vous déplacez l'abonnement, le groupe de ressources ou la ressource d'un répertoire Azure AD vers un autre, l'identité managée associée à la ressource n'est pas transférée vers le nouveau locataire. Par conséquent, les clés gérées par le client peuvent ne plus fonctionner. Pour plus d’informations, consultez **Transfert d’un abonnement entre des répertoires Azure AD** dans [FAQ et problèmes connus en lien avec les identités managées pour ressources Azure](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).  

### <a name="store-customer-managed-keys-in-azure-key-vault"></a>Stocker les clés gérées par le client dans Azure Key Vault

Pour activer les clés gérées par le client, vous devez utiliser une instance d'Azure Key Vault afin de stocker vos clés. Vous devez activer les propriétés **Suppression réversible** et **Ne pas vider** sur le coffre de clés.

Seules les clés RSA de taille 2048 sont prises en charge par le chiffrement Cognitive Services. Pour plus d’informations sur les clés, consultez **Clés Key Vault** dans [À propos des clés, des secrets et des certificats Azure Key Vault](../../key-vault/general/about-keys-secrets-certificates.md).

> [!NOTE]
> Si l'ensemble du coffre de clés est supprimé, vos données n'apparaissent plus et aucun de vos modèles n'est déployé. Toutes les données chargées sont supprimées de Custom Translator. 

### <a name="revoke-access-to-customer-managed-keys"></a>Révoquer l’accès aux clés gérées par le client

Pour révoquer l’accès aux clés gérées par le client, utilisez PowerShell ou Azure CLI. Pour plus d’informations, consultez [Azure Key Vault PowerShell](/powershell/module/az.keyvault//) ou [Interface de ligne de commande Azure Key Vault](/cli/azure/keyvault). La révocation de l'accès bloque l'accès à toutes les données de la ressource Cognitive Services et vos modèles ne sont pas déployés, car Cognitive Services n'a pas accès à la clé de chiffrement. Toutes les données chargées sont également supprimées de Custom Translator.


## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur Azure Key Vault](../../key-vault/general/overview.md)
