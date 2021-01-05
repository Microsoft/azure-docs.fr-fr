---
title: Guide pratique de réponse aux alertes Azure Defender pour Key Vault
description: Découvrez la procédure à suivre pour répondre aux alertes d’Azure Defender pour Key Vault.
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 67c556e44f07240b1ad1bcde61f40042da46def8
ms.sourcegitcommit: b8a175b6391cddd5a2c92575c311cc3e8c820018
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96122200"
---
# <a name="respond-to-azure-defender-for-key-vault-alerts"></a>Répondre aux alertes Azure Defender pour Key Vault
Lorsque vous recevez une alerte d’Azure Defender pour Key Vault, nous vous recommandons de l’examiner et d’y répondre de la façon suivante. Azure Defender pour Key Vault protège les applications et les informations d’identification. Ainsi, même si vous connaissez l’application ou l’utilisateur qui a déclenché l’alerte, il est important de vérifier la situation dans laquelle s’inscrit cette dernière.  

Chaque alerte d’Azure Defender pour Key Vault comprend les éléments suivants :

- ID de l'objet
- Nom d’utilisateur principal ou adresse IP de la ressource suspecte

> [!TIP]
> En fonction du *type* d’accès qui s’est produit, il se peut que certains champs ne soient pas disponibles. Par exemple, si c’est une application qui accède à votre coffre de clés, vous ne verrez pas de nom d’utilisateur principal associé. Si le trafic provient de l’extérieur d’Azure, il n’y aura aucun ID objet.

## <a name="step-1-contact"></a>Étape 1. Contact

1. Vérifiez si le trafic provient de votre locataire Azure. Si le pare-feu du coffre de clés est activé, il est probable que vous ayez donné accès à l’utilisateur ou à l’application qui a déclenché cette alerte.
1. Si vous ne pouvez pas vérifier la source du trafic, passez à [Étape 2. Atténuation immédiate](#step-2-immediate-mitigation).
1. Si vous pouvez identifier la source du trafic dans votre locataire, contactez l’utilisateur ou le propriétaire de l’application. 

> [!CAUTION]
> Azure Defender pour Key Vault est conçu pour aider à identifier les activités suspectes provoquées par le vol d’informations d’identification. **N’ignorez pas** l’alerte simplement parce que vous reconnaissez l’utilisateur ou l’application. Contactez le propriétaire de l’application ou l’utilisateur pour vérifier que l’activité est légitime. Vous pouvez créer une règle de suppression pour éliminer le bruit si nécessaire. Pour plus d’informations, consultez [Suppression d’alertes d’Azure Defender](alerts-suppression-rules.md).


## <a name="step-2-immediate-mitigation"></a>Étape 2. Atténuation immédiate 
Si vous ne reconnaissez pas l’utilisateur ni l’application, ou si vous pensez que l’accès n’aurait pas dû être autorisé :

- Si le trafic provient d’une adresse IP non reconnue :
    1. Activez le pare-feu Azure Key Vault conformément aux indications de [Configuration des pare-feu et réseaux virtuels Azure Key Vault](../key-vault/general/network-security.md).
    1. Configurez le pare-feu avec des ressources et des réseaux virtuels approuvés.

- Si la source de l’alerte était une application non autorisée ou un utilisateur suspect :
    1. Ouvrez les paramètres de stratégie d’accès du coffre de clés.
    1. Supprimez le principal de sécurité correspondant ou restreignez les opérations qu’il peut effectuer.  

- Si la source de l’alerte possède un rôle Azure Active Directory dans votre locataire :
    1. Contactez votre administrateur.
    1. Déterminez s’il est nécessaire de réduire ou de révoquer les autorisations Azure Active Directory.

## <a name="step-3-identify-impact"></a>Étape 3. Identification de l’impact 
Une fois l’impact atténué, examinez les secrets affectés dans votre coffre de clés :
1. Ouvrez la page « Sécurité » de votre coffre de clés Azure Key Vault et consultez l’alerte déclenchée.
1. Sélectionnez l’alerte précise qui a été déclenchée.
    Passez en revue la liste des secrets auxquels vous avez accédé et le timestamp.
1. Éventuellement, si vous avez activé les journaux de diagnostic du coffre de clés, examinez les opérations précédentes pour l’adresse IP de l’appelant, l’utilisateur principal ou l’ID objet correspondant.  

## <a name="step-4-take-action"></a>Étape 4. Effectuer une action 
Une fois que vous avez compilé la liste des secrets, des clés et des certificats qui ont été consultés par l’application ou l’utilisateur suspect, vous devez renouveler immédiatement ces objets.

1. Les secrets affectés doivent être désactivés ou supprimés de votre coffre de clés.
1. Si les informations d’identification ont été utilisées pour une application spécifique :
    1. Contactez l’administrateur de l’application et demandez-lui d’auditer son environnement pour identifier toute utilisation des informations d’identification depuis qu’elles ont été compromises.
    1. Si les informations d’identification compromises ont été utilisées, le propriétaire de l’application doit identifier les informations qui ont été consultées et atténuer l’impact.


## <a name="next-steps"></a>Étapes suivantes

Sur cette page a été expliqué le processus de réponse à une alerte d’Azure Defender pour Key Vault. Pour toute information connexe, consultez les pages suivantes :

- [Présentation d’Azure Defender pour Key Vault](defender-for-key-vault-introduction.md)
- [Supprimer les alertes d’Azure Defender](alerts-suppression-rules.md)
- [Exporter en continu les données Security Center](continuous-export.md)