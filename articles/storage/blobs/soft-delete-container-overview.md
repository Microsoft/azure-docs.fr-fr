---
title: Suppression réversible pour les conteneurs (préversion)
titleSuffix: Azure Storage
description: La fonctionnalité de suppression réversible pour les conteneurs (préversion) protège vos données, ce qui vous permet de récupérer plus facilement vos données en cas de modification ou de suppression malencontreuses de celles-ci par une application ou un autre utilisateur du compte de stockage.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 08/25/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 1ebfa137357e60a1f462ef90c6430cbf0cb5b968
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746589"
---
# <a name="soft-delete-for-containers-preview"></a>Suppression réversible pour les conteneurs (préversion)

La suppression réversible pour les conteneurs (préversion) protège vos données contre les modifications ou suppressions accidentelles ou erronées. Lorsque la suppression réversible de conteneur est activée pour un compte de stockage, tout conteneur supprimé et son contenu sont conservés dans Stockage Azure pour la période que vous spécifiez. Pendant la période de rétention, vous pouvez restaurer des conteneurs précédemment supprimés et tous les blobs qu’ils contiennent.

Pour la protection de bout en bout de vos données de blobs, Microsoft recommande d’activer les fonctionnalités de protection des données suivantes :

- Suppression réversible de conteneur, pour vous protéger de la suppression ou du remplacement accidentel d’un conteneur. Pour savoir comment activer la suppression réversible de conteneur, consultez [Activer et gérer la suppression réversible pour les conteneurs](soft-delete-container-enable.md).
- Suppression réversible de blob, pour vous protéger de la suppression ou du remplacement accidentel d’un blob. Pour savoir comment activer la suppression réversible de blob, consultez [Suppression réversible pour les blobs](soft-delete-blob-overview.md).
- Gestion des versions des objets blob, pour gérer automatiquement les versions précédentes d’un objet blob. Lorsque le contrôle de version est activé, vous pouvez restaurer une version antérieure d’un objet blob pour récupérer vos données si celles-ci ont été modifiées ou supprimées par erreur. Pour savoir comment activer le contrôle de version des blobs, consultez [Activer et gérer le contrôle de version des blobs](versioning-enable.md).

> [!WARNING]
> La suppression d’un compte de stockage est irréversible. La suppression réversible ne protège pas contre la suppression d’un compte de stockage. Pour empêcher toute suppression accidentelle d’un compte de stockage, configurez un verrou **CannotDelete** verrou sur la ressource du compte de stockage. Pour plus d’informations, consultez [Verrouiller les ressources pour empêcher les modifications inattendues](../../azure-resource-manager/management/lock-resources.md).

## <a name="how-container-soft-delete-works"></a>Fonctionnement de la suppression réversible de conteneur

Lorsque vous activez la suppression réversible de conteneur, vous pouvez spécifier une période de rétention pour les conteneurs supprimés qui est comprise entre 1 et 365 jours. La période de conservation par défaut est 7 jours. Pendant la période de rétention, vous pouvez récupérer un conteneur supprimé en appelant l’opération **Undelete Container** .

Lorsque vous restaurez un conteneur, vous pouvez le restaurer avec son nom d’origine si celui-ci n’a pas été réutilisé. Si le nom du conteneur d’origine a été utilisé, vous pouvez restaurer le conteneur avec un nouveau nom.

Une fois la période de rétention expirée, le conteneur est définitivement supprimé de Stockage Azure et ne peut pas être récupéré. La période de rétention commence au moment où le conteneur est supprimé. Vous pouvez modifier la période de rétention à tout moment, mais gardez à l’esprit qu’une période de rétention mise à jour s’applique uniquement aux conteneurs nouvellement supprimés. Les conteneurs supprimés avant la mise à jour seront définitivement supprimés conformément à la période de rétention qui était en vigueur au moment de leur suppression.

La désactivation de la suppression réversible de conteneur n’entraîne pas la suppression définitive des conteneurs qui ont été précédemment supprimés de manière réversible. Tous les conteneurs supprimés de manière réversible seront définitivement supprimés à l’expiration de la période de rétention qui était en vigueur au moment de leur suppression.

## <a name="about-the-preview"></a>À propos de la préversion

La suppression réversible de conteneur est disponible en préversion dans toutes les régions Azure.

> [!IMPORTANT]
> La préversion de la suppression réversible de conteneur est destinée uniquement à une utilisation hors production. Les contrats SLA (contrats de niveau de service) de production ne sont actuellement pas disponibles.

La version 2019-12-12 et les versions ultérieures de l’API REST de Stockage Azure prennent en charge la suppression réversible de conteneur.

### <a name="storage-account-support"></a>Prise en charge du compte de stockage

La suppression réversible de conteneur est disponible pour les types de comptes de stockage suivants :

- Comptes de stockage universel v2
- Comptes de stockage d’objets blob de blocs
- Comptes de stockage d’objets blob

Si votre compte de stockage est un compte v1 à usage général, utilisez le Portail Azure pour effectuer une mise à niveau vers un compte v2 à usage général. Pour plus d’informations sur les comptes de stockage, consultez [Vue d’ensemble des comptes de stockage Azure](../common/storage-account-overview.md).

Les comptes de stockage avec espace de noms hiérarchique activé pour une utilisation avec Azure Data Lake Storage Gen2 sont également pris en charge.

### <a name="register-for-the-preview"></a>S’inscrire pour la préversion

Pour vous inscrire à la préversion de la suppression réversible de conteneur, utilisez PowerShell ou Azure CLI pour envoyer une demande d’inscription de la fonctionnalité avec votre abonnement. Une fois votre demande approuvée, vous pouvez activer la suppression réversible de conteneur sur tout compte de stockage v2 universel, compte de stockage de blobs ou compte de stockage d’objets blob de blocs Premium.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vous inscrire à l’aide de PowerShell, appelez la commande [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature).

```powershell
# Register for container soft delete (preview)
Register-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName ContainerSoftDelete

# Refresh the Azure Storage provider namespace
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vous inscrire avec Azure CLI, appelez la commande [az feature register](/cli/azure/feature#az-feature-register).

```azurecli
az feature register --namespace Microsoft.Storage --name ContainerSoftDelete
az provider register --namespace 'Microsoft.Storage'
```

---

### <a name="check-the-status-of-your-registration"></a>Vérifier l’état de votre inscription

Pour vérifier l’état de votre inscription, utilisez PowerShell ou Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vérifier l’état de votre inscription à l’aide de PowerShell, appelez la commande [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature).

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName ContainerSoftDelete
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vérifier l’état de votre inscription à l’aide d’Azure CLI, appelez la commande [az feature](/cli/azure/feature#az-feature-show).

```azurecli
az feature show --namespace Microsoft.Storage --name ContainerSoftDelete
```

---

## <a name="pricing-and-billing"></a>Tarification et facturation

Aucuns frais supplémentaires ne sont facturés pour activer la suppression réversible de conteneur. Les données qui se trouvent dans des conteneurs supprimés de manière réversible sont facturées au même tarif que les données actives.

## <a name="next-steps"></a>Étapes suivantes

- [Configuration de la suppression réversible de conteneur](soft-delete-container-enable.md)
- [Suppression réversible pour les objets blob](soft-delete-blob-overview.md)
- [Contrôle de version des blobs](versioning-overview.md)
