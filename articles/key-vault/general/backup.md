---
title: Sauvegarder un secret, une clé ou un certificat stocké dans Azure Key Vault | Microsoft Docs
description: Utilisez ce document pour sauvegarder un secret, une clé ou un certificat stocké dans Azure Key Vault.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 10/22/2020
ms.author: sudbalas
ms.openlocfilehash: c55c87cd8ab1b2cd1dbaf2c877eb341744d822ef
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93287474"
---
# <a name="azure-key-vault-backup"></a>Sauvegarde Azure Key Vault

Ce document vous montre comment sauvegarder des secrets, des clés et des certificats stockés dans votre coffre de clés. Une sauvegarde est destinée à vous fournir une copie hors connexion de tous vos secrets dans le cas improbable où vous perdriez l’accès à votre coffre de clés.

## <a name="overview"></a>Vue d’ensemble

Azure Key Vault fournit automatiquement des fonctionnalités permettant de garantir la disponibilité et d’empêcher toute perte de données. Sauvegardez les secrets uniquement si vous avez une justification métier stratégique. La sauvegarde de secrets dans votre coffre de clés peut introduire des défis opérationnels, comme la gestion de plusieurs ensembles de journaux, d’autorisations et de sauvegardes quand des secrets expirent ou permutent.

Key Vault assure la disponibilité dans les scénarios de sinistre et bascule automatiquement les demandes vers une région associée sans aucune intervention de l’utilisateur. Pour plus d’informations, consultez [Disponibilité et redondance d’Azure Key Vault](./disaster-recovery-guidance.md).

Si vous voulez une protection contre la suppression accidentelle ou malveillante de vos secrets, configurez les fonctionnalités de suppression réversible et de protection contre le vidage sur votre coffre de clés. Pour plus d’informations, consultez [Vue d’ensemble de la suppression réversible d’Azure Key Vault](./soft-delete-overview.md).

## <a name="limitations"></a>Limites

> [!IMPORTANT]
> Key Vault ne prend pas en charge la possibilité de sauvegarder plus de 500 versions antérieures d’un objet de clé, de secret ou de certificat. Toute tentative de sauvegarde d’un objet de clé, de secret ou de certificat peut entraîner une erreur. Il est impossible de supprimer les versions précédentes d’une clé, d’un secret ou d’un certificat.

Key Vault ne propose pour l’instant aucun moyen de sauvegarder l’ensemble d’un coffre de clés en une seule opération. Toute tentative d’utiliser les commandes listées dans ce document pour effectuer une sauvegarde automatisée d’un coffre de clés peut entraîner des erreurs et n’est pas prise en charge par Microsoft ni l’équipe Azure Key Vault. 

Tenez également compte des conséquences suivantes :

* La sauvegarde de secrets ayant plusieurs versions peut entraîner des erreurs de délai d’attente.
* Une sauvegarde crée une capture instantanée à un point dans le temps. Les secrets peuvent être renouvelés pendant une sauvegarde, provoquant une incohérence entre les clés de chiffrement.
* Le dépassement des limites du service de coffre de clés pour les demandes par seconde entraîne la limitation de votre coffre de clés et l’échec de la sauvegarde.

## <a name="design-considerations"></a>Remarques relatives à la conception

Quand vous sauvegardez un objet d’un coffre de clés (secret, clé ou certificat), l’opération de sauvegarde télécharge l’objet sous la forme d’un objet blob chiffré. Cet objet blob ne peut pas être déchiffré en dehors d’Azure. Pour obtenir des données utilisables à partir de cet objet blob, vous devez restaurer l’objet blob dans un coffre de clés dans le même abonnement Azure et la même [zone géographique Azure](https://azure.microsoft.com/global-infrastructure/geographies/).

## <a name="prerequisites"></a>Prérequis

Pour sauvegarder un objet de coffre de clés, vous devez disposer des éléments suivants : 

* Autorisations de niveau contributeur ou supérieur sur un abonnement Azure.
* Coffre de clés principal contenant les secrets que vous voulez sauvegarder.
* Un coffre de clés secondaire où les secrets seront restaurés

## <a name="back-up-and-restore-from-the-azure-portal"></a>Sauvegarder et restaurer à partir du portail Azure

Suivez les étapes de cette section pour sauvegarder et restaurer des objets à l’aide du portail Azure.

### <a name="back-up"></a>Sauvegarder

1. Accédez au portail Azure.
2. Sélectionnez votre coffre de clés.
3. Accédez à l’objet (secret, clé ou certificat) que vous voulez sauvegarder.

    ![Capture d’écran montrant où sélectionner le paramètre Clés et un objet dans un coffre de clés.](../media/backup-1.png)

4. Sélectionnez l’objet.
5. Sélectionnez **Télécharger la sauvegarde**.

    ![Capture d’écran montrant où sélectionner le bouton Télécharger la sauvegarde dans un coffre de clés.](../media/backup-2.png)
    
6. Sélectionnez **Télécharger**.

    ![Capture d’écran montrant où sélectionner le bouton Télécharger dans un coffre de clés.](../media/backup-3.png)
    
7. Stockez l’objet blob chiffré dans un emplacement sécurisé.

### <a name="restore"></a>Restaurer

1. Accédez au portail Azure.
2. Sélectionnez votre coffre de clés.
3. Accédez au type d’objet (secret, clé ou certificat) que vous voulez restaurer.
4. Sélectionnez **Restaurer la sauvegarde**.

    ![Capture d’écran montrant où sélectionner le bouton Restaurer la sauvegarde dans un coffre de clés.](../media/backup-4.png)
    
5. Accédez à l’emplacement où vous avez stocké l’objet blob chiffré.
6. Sélectionnez **OK**.

## <a name="back-up-and-restore-from-the-azure-cli"></a>Sauvegarder et restaurer à partir de l’interface de ligne de commande Azure

```azurecli
## Log in to Azure
az login

## Set your subscription
az account set --subscription {AZURE SUBSCRIPTION ID}

## Register Key Vault as a provider
az provider register -n Microsoft.KeyVault

## Back up a certificate in Key Vault
az keyvault certificate backup --file {File Path} --name {Certificate Name} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Back up a key in Key Vault
az keyvault key backup --file {File Path} --name {Key Name} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Back up a secret in Key Vault
az keyvault secret backup --file {File Path} --name {Secret Name} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Restore a certificate in Key Vault
az keyvault certificate restore --file {File Path} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Restore a key in Key Vault
az keyvault key restore --file {File Path} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Restore a secret in Key Vault
az keyvault secret restore --file {File Path} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

```

## <a name="next-steps"></a>Étapes suivantes

Activez [la journalisation et la supervision](./logging.md) pour Key Vault.