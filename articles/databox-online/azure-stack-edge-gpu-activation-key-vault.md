---
title: Intégration d’Azure Key Vault avec la ressource Azure Stack Edge et l’activation de l’appareil
description: Décrit comment Azure Key Vault est associé à la gestion des secrets lors de l’activation d’un appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 10/10/2020
ms.author: alkohli
ms.openlocfilehash: ec7a32739940d53d976e73a7e170df96a0acc245
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96449514"
---
# <a name="azure-key-vault-integration-with-azure-stack-edge"></a>Intégration d’Azure Key Vault avec Azure Stack Edge 

Azure Key Vault est intégré à la ressource Azure Stack Edge pour la gestion des secrets. Cet article fournit des détails sur la création d’un Azure Key Vault pour la ressource Azure Stack Edge pendant l’activation de l’appareil et qui est ensuite utilisé pour la gestion des secrets. 


## <a name="about-key-vault-and-azure-stack-edge"></a>À propos de Key Vault et Azure Stack Edge

Le service cloud Azure Key Vault peut être utilisé pour stocker en toute sécurité les jetons, mots de passe, certificats, clés d’API et autres secrets, et en contrôler étroitement l’accès. Key Vault simplifie la création et le contrôle des clés de chiffrement utilisées pour chiffrer vos données. Pour plus d’informations sur les transactions autorisées et les frais correspondants, consultez [Tarification pour Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).

Pour le service Azure Stack Edge, l’un des secrets utilisés est la clé d’intégrité de canal (CIK). Cette clé vous permet de chiffrer vos secrets. Avec l’intégration du coffre de clés, le CIK est stocké en toute sécurité dans le coffre de clés. Pour plus d’informations, consultez la rubrique [Stocker en toute sécurité les secrets et clés](../key-vault/general/overview.md#securely-store-secrets-and-keys).


## <a name="key-vault-creation"></a>Création d’un coffre de clés

Un coffre de clés est créé pour la ressource Azure Stack Edge pendant le processus de génération de clé d’activation. 

- Lorsque vous créez une ressource Azure Stack Edge, vous devez inscrire le fournisseur de ressources *Microsoft.KeyVault*. Le fournisseur de ressources est automatiquement inscrit si vous disposez d’un accès propriétaire ou contributeur à l’abonnement. Le coffre de clés est créé dans le même abonnement et le même groupe de ressources que la ressource Azure Stack Edge. 

- Lorsque vous créez une ressource Azure Stack Edge, une identité du service géré Managed Service Identity (MSI) est également créé, qui persiste pendant la durée de vie de la ressource et communique avec le fournisseur de ressources sur le Cloud. 

    Lorsque la MSI est activée, Azure crée une identité approuvée pour la ressource Azure Stack Edge.

- Une fois que vous avez créé la ressource Azure Stack Edge et que vous générez une clé d’activation à partir du Portail Azure, un coffre de clés est créé. Ce coffre de clés est utilisé pour la gestion des secrets et persiste tant que la ressource Azure Stack Edge existe. 

    ![Key Vault créé pendant la génération de la clé d’activation](media/azure-stack-edge-gpu-deploy-prep/azure-stack-edge-resource-3.png)

- Vous pouvez choisir d’accepter le nom de clé par défaut ou spécifier un nom personnalisé pour le coffre de clés. Le nom du coffre de clés doit comprendre entre 3 et 24 caractères. Vous ne pouvez pas utiliser un coffre de clés qui est déjà en cours d’utilisation. <!--The MSI is then used to authenticate to key vault to retrieve secrets.--> 

    ![MSI créée lors de la création de la ressource de Azure Stack Edge](media/azure-stack-edge-gpu-deploy-prep/create-resource-8.png)

- Pour accéder au coffre de clés Azure, accédez aux **Propriétés** dans votre ressource Azure Stack Edge, puis sélectionnez le nom du coffre de clés. 

- Pour éviter toute suppression accidentelle, un verrou de ressource est activé sur le coffre de clés. Une suppression réversible est également activée sur le coffre de clés, ce qui permet la restauration du coffre de clés dans un délai de 90 jours en cas de suppression accidentelle. Pour plus d’informations, consulter [Vue d’ensemble de la suppression réversible d’Azure Key Vault](../key-vault/general/soft-delete-overview.md)

    Si le coffre de clés est supprimé par inadvertance et que la durée de la protection de purge de 90 jours n’est pas écoulée, procédez comme suit pour [récupérer votre de coffre de clés](../key-vault/general/key-vault-recovery.md#list-recover-or-purge-soft-deleted-secrets-keys-and-certificates). 

- Si vous disposiez d’une ressource Azure Stack Edge existante avant l’intégration d’Azure Key Vault à la ressource Azure Stack Edge, vous n’êtes pas affecté. Vous pouvez continuer à utiliser votre ressource Azure Stack Edge existante. 

- Lorsque votre ressource Azure Stack Edge est supprimée, Azure Key Vault est également supprimé avec la ressource. Vous êtes invité à confirmer l’opération. Si vous n’envisagez pas de supprimer ce coffre de clés, vous pouvez choisir de ne pas donner votre consentement. Seule la ressource Azure Stack Edge est supprimée, laissant le coffre de clés intact. 

- Si ce coffre de clés a été utilisé pour stocker d’autres clés, vous pouvez toujours le restaurer dans un délai de 90 jours après la suppression. Pendant cette période de protection de purge, le nom du coffre de clés ne peut pas être utilisé pour créer un coffre de clés.

Si vous rencontrez des problèmes liés au coffre de clés et à l’activation d’appareil, consultez [Résoudre les problèmes d’activation d’appareil](azure-stack-edge-gpu-troubleshoot-activation.md).

<!--## Key vault secret management

When you generate an activation key, the following events occur:

1. You request an activation key in the Azure portal. The request is then sent to Key Vault resource provider. 
1. A standard tier key vault with access policy is created and is locked by default. This key vault uses the default name or the custom name that you specified.
1. The key vault authenticates with MSI the request to generate activation key. The MSI is also added to the key vault access policy and a channel integrity key is generated and placed in the key vault.
1. The activation key is returned to the Azure portal. You can then copy this key and use it in the local UI to activate your device.-->



## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la façon de [Générer une clé d’activation](azure-stack-edge-gpu-deploy-prep.md#get-the-activation-key).