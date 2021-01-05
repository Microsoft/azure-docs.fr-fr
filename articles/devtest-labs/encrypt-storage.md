---
title: Chiffrer un compte de stockage Azure utilisé par un labo dans Azure DevTest Labs
description: Découvrez comment configurer le chiffrement d’un compte de stockage Azure utilisé par un labo dans Azure DevTest Labs
ms.topic: how-to
ms.date: 07/29/2020
ms.openlocfilehash: dcede89fb23c532742e41121688bcb51a5a73833
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92149304"
---
# <a name="encrypt-azure-storage-used-by-a-lab-in-azure-devtest-labs"></a>Chiffrer un compte de stockage Azure utilisé par un labo dans Azure DevTest Labs
Chaque labo créé dans Azure DevTest Labs est créé avec un compte de stockage Azure associé. Ce compte de stockage est utilisé pour les raisons suivantes : 

- Stockage de documents de [formules](devtest-lab-manage-formulas.md) pouvant être utilisés pour créer des machines virtuelles.
- Stockage de résultats d’artefact qui incluent les journaux d’extension et de déploiement générés à partir de l’application des artefacts. 
- [Téléchargement de disques durs virtuels (VHD) pour créer des images personnalisées dans le labo.](devtest-lab-create-template.md)
- Mise en cache d’[artefacts](add-artifact-vm.md) et de [modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md) fréquemment utilisés pour accélérer la récupération lors de la création d’une machine virtuelle ou d’un environnement.

> [!NOTE]
> Les informations ci-dessus sont essentielles au fonctionnement du labo. Elles sont stockées pendant la durée de vie du labo (et des ressources de labo), sauf si elles sont supprimées explicitement. La suppression manuelle de ces ressources peut entraîner des erreurs de création de machines virtuelles de labo et/ou l’endommagement de formules pour une utilisation ultérieure. 

## <a name="locate-the-storage-account-and-view-its-contents"></a>Localiser le compte de stockage et examiner son contenu

1. Dans la page d’accueil du labo, sélectionnez le **groupe de ressources** dans la page **Vue d’ensemble**. Vous devez voir la page **Groupe de ressources** pour le groupe de ressources qui contient le labo. 

    :::image type="content" source="./media/encrypt-storage/overview-resource-group-link.png" alt-text="Sélection du groupe de ressources dans la page Vue d’ensemble":::
1. Sélectionnez le compte de stockage Azure du labo. La convention de nommage pour le compte de stockage du labo est la suivante : `a<labNameWithoutInvalidCharacters><4-digit number>`. Par exemple, si le nom du labo est `contosolab`, le nom du compte de stockage peut être `acontosolab7576`. 

    :::image type="content" source="./media/encrypt-storage/select-storage-account.png" alt-text="Sélection du groupe de ressources dans la page Vue d’ensemble":::
3. Dans la page **Compte de stockage**, sélectionnez **Explorateur Stockage (préversion)** dans le menu de gauche, puis sélectionnez **CONTENEURS D’OBJETS BLOB** pour rechercher le contenu pertinent associé au labo. 

   :::image type="content" source="./media/encrypt-storage/storage-explorer.png" alt-text="Sélection du groupe de ressources dans la page Vue d’ensemble" lightbox="./media/encrypt-storage/storage-explorer.png":::

## <a name="encrypt-the-lab-storage-account"></a>Chiffrer le compte de stockage du labo
Le Stockage Azure chiffre automatiquement vos données lors de leur conservation dans le cloud. Le chiffrement du Stockage Azure protège vos données et vous aide à répondre aux engagements de votre entreprise en matière de sécurité et de conformité. Pour plus d'informations, consultez [Fonctionnalité de chiffrement du service Stockage Azure pour les données au repos](../storage/common/storage-service-encryption.md).

Les données du compte de stockage du labo sont chiffrées à l’aide d’une **clé managée par Microsoft**. Vous pouvez vous reposer sur les clés managées par Microsoft pour le chiffrement des vos données, ou vous pouvez gérer le chiffrement avec vos propres clés. Si vous choisissez de gérer le chiffrement avec vos propres clés pour le compte de stockage du labo, vous pouvez spécifier une **clé gérée par le client** avec Azure Key Vault à utiliser pour le chiffrement/déchiffrement des données dans le stockage Blob et dans Azure Files. Pour plus d’informations sur les clés gérées par le client, consultez [Utiliser des clés gérées par le client avec Azure Key Vault pour gérer le chiffrement du stockage Azure](../storage/common/customer-managed-keys-overview.md).

Pour découvrir comment configurer les clés gérées par le client pour le chiffrement du stockage Azure, consultez les articles suivants : 

- [Azure portal](../storage/common/customer-managed-keys-configure-key-vault.md)
- [Azure PowerShell](../storage/common/customer-managed-keys-configure-key-vault.md)
- [Azure CLI](../storage/common/customer-managed-keys-configure-key-vault.md)


## <a name="manage-the-azure-blob-storage-life-cycle"></a>Gérer le cycle de vie du Stockage Blob Azure
Comme mentionné précédemment, les informations stockées dans le compte de stockage du labo sont essentielles au fonctionnement sans erreur du labo. À moins qu’elles ne soient explicitement supprimées, ces données restent dans le compte de stockage du labo pendant la durée de vie du labo ou pendant la durée de vie des machines virtuelles de labo spécifiques, en fonction du type de données.

### <a name="uploaded-vhds"></a>Disques durs virtuels téléchargés
Ces disques durs virtuels sont utilisés pour créer des images personnalisées. Leur suppression empêchera la création d’images personnalisées à partir de ces disques durs virtuels.

### <a name="artifacts-cache"></a>Cache d’artefacts
Ces caches sont recréés chaque fois que des artefacts sont appliqués. Ils sont actualisés avec le contenu le plus récent des référentiels référencés respectifs. Par conséquent, si vous supprimez ces informations pour enregistrer les dépenses liées au stockage, le relief est temporaire.

### <a name="azure-resource-manager-template-cache"></a>Cache de modèle Azure Resource Manager
Ces caches sont recréés chaque fois que des référentiels de modèles basés sur Azure Resource Manager sont connectés et lancés dans le labo. Ils sont actualisés avec le contenu le plus récent des référentiels référencés respectifs. Par conséquent, si vous supprimez ces informations pour enregistrer les dépenses liées au stockage, le relief est temporaire.

### <a name="formulas"></a>Formules
Ces documents sont utilisés pour prendre en charge l’option permettant de créer des formules à partir de machines virtuelles existantes et de créer des machines virtuelles à partir de formules. La suppression de ces documents de formules peut entraîner des erreurs lors des opérations suivantes :

- Création d’une formule à partir d’une machine virtuelle de labo existante
- Création ou mise à jour de formules 
- Création d’une machine virtuelle à partir d’une formule.

### <a name="artifact-results"></a>Résultats d’artefact
À mesure que les artefacts sont appliqués, la taille des résultats d’artefact respectifs peut augmenter au fil du temps en fonction du nombre et du type d’artefacts exécutés sur les machines virtuelles du labo. Ainsi, en tant que propriétaire du labo, vous pouvez contrôler le cycle de vie de ces documents. Pour plus d’informations, consultez [Gérer le cycle de vie du Stockage Blob Azure](../storage/blobs/storage-lifecycle-management-concepts.md).

> [!IMPORTANT]
> Nous vous recommandons d’effectuer cette étape pour réduire les dépenses associées au compte de stockage Azure. 

Par exemple, la règle suivante est utilisée pour définir une règle d’expiration de 90 jours spécifiquement pour les résultats d’artefact. Elle garantit que les résultats d’artefact plus anciens sont recyclés à partir du compte de stockage à une cadence régulière.

```json
{
  "rules": [
    {
      "name": "expirationRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "artifacts/results" ]
        },
        "actions": {
          "baseBlob": {
            "delete": { "daysAfterModificationGreaterThan": 90 }
          },
          "snapshot": {
            "delete": { "daysAfterCreationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes
Pour découvrir comment configurer les clés gérées par le client pour le chiffrement du stockage Azure, consultez les articles suivants : 

- [Azure portal](../storage/common/customer-managed-keys-configure-key-vault.md)
- [Azure PowerShell](../storage/common/customer-managed-keys-configure-key-vault.md)
- [Azure CLI](../storage/common/customer-managed-keys-configure-key-vault.md)