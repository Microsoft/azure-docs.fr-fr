---
title: Utiliser le Portail Microsoft Azure pour configurer des clés gérées par le client pour le service Import/Export
description: Découvrez comment utiliser le portail Azure afin de configurer des clés gérées par le client avec Azure Key Vault pour le service Azure Import/Export. Les clés gérées par le client vous permettent de créer, faire pivoter, désactiver et révoquer des contrôles d’accès.
services: storage
author: alkohli
ms.service: storage
ms.topic: how-to
ms.date: 05/06/2020
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: fb91a490083629101470565a630b659c090e071b
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843364"
---
# <a name="use-customer-managed-keys-in-azure-key-vault-for-importexport-service"></a>Utiliser des clés gérées par le client dans Azure Key Vault pour le service Import/Export

Azure Import/Export protège les clés BitLocker utilisées pour verrouiller les lecteurs à l’aide d’une clé de chiffrement. Par défaut, les clés BitLocker sont chiffrées avec des clés gérées par Microsoft. Pour un contrôle supplémentaire sur les clés de chiffrement, vous pouvez également fournir des clés gérées par le client.

Les clés gérées par le client doivent être créées et stockées dans Azure Key Vault. Pour plus d’informations sur le coffre de clés Azure, consultez la page [Présentation du coffre de clés Azure](../../key-vault/general/overview.md)

Cet article explique comment utiliser des clés gérées par le client avec le service Import/Export dans le [portail Azure](https://portal.azure.com/).

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez les points suivants :

1. Vous avez créé une tâche d’importation ou d’exportation conformément aux instructions de :

    - [Création d’une tâche d’importation pour les objets BLOB](storage-import-export-data-to-blobs.md).
    - [Création d’une tâche d’importation pour les fichiers](storage-import-export-data-to-files.md).
    - [Création d’une tâche d’exportation pour les objets BLOB](storage-import-export-data-from-blobs.md)

2. Vous disposez d’une instance Azure Key Vault existante avec une clé que vous pouvez utiliser pour protéger votre clé BitLocker. Pour savoir comment créer un coffre de clés à l’aide du portail Azure, consultez [Démarrage rapide : Créer un coffre de clés Azure Key Vault à l'aide du portail Azure](../../key-vault/general/quick-create-portal.md).

    - Les options **Suppression réversible** et **Ne pas vider** sont définies sur votre instance Key Vault existante. Ces propriétés ne sont pas activées par défaut. Pour activer ces propriétés, consultez les sections intitulées **Activation de la suppression réversible** et **Activation de la protection contre le vidage** dans l’un des articles suivants :

        - [Guide pratique pour utiliser la suppression réversible avec Azure Power​Shell](../../key-vault/general/key-vault-recovery.md).
        - [Guide pratique pour utiliser la suppression réversible avec Azure CLI](../../key-vault/general/key-vault-recovery.md).
    - Le coffre de clés existant doit avoir une clé RSA d’une taille de 2048 ou plus. Pour plus d’informations sur les clés, consultez [À propos des clés](../../key-vault/keys/about-keys.md).
    - Le coffre de clés doit se trouver dans la même région que le compte de stockage pour vos données.  
    - Si vous n’avez pas d’instances Azure Key Vault existantes, vous pouvez également en créer de façon intégrée comme décrit dans la section suivante.

## <a name="enable-keys"></a>Activer les clés

La configuration de la clé gérée par le client pour votre service Import/Export est facultative. Par défaut, le service Import/Export utilise une clé gérée par Microsoft pour protéger votre clé BitLocker. Pour activer des clés gérées par le client dans le portail Azure, procédez comme suit :

1. Accédez au panneau **Vue d’ensemble** de votre tâche d’importation.
2. Dans le volet droit, sélectionnez **Choisir la méthode de chiffrement de vos clés BitLocker**.

    ![Choisir l’option de chiffrement](./media/storage-import-export-encryption-key-portal/encryption-key-1.png)

3. Dans le panneau **Chiffrement**, vous pouvez afficher et copier la clé BitLocker de l’appareil. Sous **Type de chiffrement**, vous pouvez choisir la façon dont vous souhaitez protéger votre clé BitLocker. Par défaut, une clé gérée par Microsoft est utilisée.

    ![Afficher la clé BitLocker](./media/storage-import-export-encryption-key-portal/encryption-key-2.png)

4. Vous avez la possibilité de spécifier une clé gérée par le client. Après avoir sélectionné la clé gérée par le client, **Sélectionnez un coffre de clés et une clé**.

    ![Sélectionner la clé gérée par le client](./media/storage-import-export-encryption-key-portal/encryption-key-3.png)

5. Dans le panneau **Sélectionner une clé dans Azure Key Vault**, l’abonnement est automatiquement renseigné. Pour **Coffre de clés**, vous pouvez sélectionner un coffre de clés existant dans la liste déroulante.

    ![Sélectionner ou créer un Azure Key Vault](./media/storage-import-export-encryption-key-portal/encryption-key-4.png)

6. Vous pouvez également sélectionner **Créer** pour créer un coffre de clés. Dans le panneau **Créer un coffre de clés**, saisissez le groupe de ressources et le nom du coffre de clés. Acceptez toutes les autres valeurs par défaut. Sélectionnez **Vérifier + créer**.

    ![Créer un coffre Azure Key Vault](./media/storage-import-export-encryption-key-portal/encryption-key-5.png)

7. Passez en revue les informations associées à votre coffre de clés, puis sélectionnez **Créer**. Patientez quelques minutes avant la fin de la création du coffre de clés.

    ![Créer un Azure Key Vault](./media/storage-import-export-encryption-key-portal/encryption-key-6.png)

8. Dans **Sélectionner une clé dans Azure Key Vault**, vous pouvez sélectionner une clé dans le coffre de clés existant.

9. Si vous avez créé un coffre de clés, sélectionnez **Créer** pour créer une clé. La taille de la clé RSA peut être supérieure ou égale à 2048.

    ![Créer une nouvelle clé dans Azure Key Vault](./media/storage-import-export-encryption-key-portal/encryption-key-7.png)

    Si la suppression réversible et la protection contre le vidage ne sont pas activées lorsque vous créez le coffre de clés, le coffre de clés est mis à jour pour que la suppression réversible et la protection contre le vidage soient activées.

10. Indiquez le nom de votre clé, acceptez les autres valeurs par défaut, puis sélectionnez **Créer**.

    ![Créer une nouvelle clé](./media/storage-import-export-encryption-key-portal/encryption-key-8.png)

11. Sélectionnez la **Version**, puis choisissez **Sélectionner**. Vous êtes averti qu’une clé est créée dans votre coffre de clés.

    ![Nouvelle clé créée dans le coffre de clés](./media/storage-import-export-encryption-key-portal/encryption-key-9.png)

Dans le panneau **Chiffrement**, vous pouvez voir le coffre de clés et la clé sélectionnée pour votre clé gérée par le client.

> [!IMPORTANT]
> Vous pouvez uniquement désactiver les clés gérées par Microsoft et passer à des clés gérées par le client à n’importe quel moment de la tâche d’importation/exportation. Toutefois, vous ne pouvez pas désactiver la clé gérée par le client une fois que vous l’avez créée.

## <a name="troubleshoot-customer-managed-key-errors"></a>Résoudre les erreurs pour les clés gérées par le client

Si vous recevez des erreurs liées à votre clé gérée par le client, utilisez le tableau suivant pour résoudre les problèmes :

| Code d'erreur     |Détails     | Récupérable ?    |
|----------------|------------|-----------------|
| CmkErrorAccessRevoked | L’accès à la clé gérée par le client est révoqué.                                                       | Oui, vérifier si : <ol><li>Le coffre de clés contient toujours le fichier MSI dans la stratégie d’accès.</li><li>Les autorisations d’extraction, de renvoi à la ligne et de désencapsulage sont activées pour la stratégie d’accès.</li><li>Si le coffre de clés se trouve dans un réseau virtuel derrière le pare-feu, vérifiez si **Autoriser les services de confiance Microsoft** est activé.</li><li>Vérifiez si le MSI de la ressource du travail a été réinitialisé sur la valeur `None` à l’aide des API.<br>Si oui, redéfinissez la valeur sur `Identity = SystemAssigned`. Cela recrée l’identité pour la ressource du travail.<br>Lorsque la nouvelle identité a été créée, activez autorisations `Get`, `Wrap`, et `Unwrap` sur la nouvelle identité dans la stratégie d’accès du coffre de clés.</li></ol>                                                                                            |
| CmkErrorKeyDisabled      | La clé gérée par le client est désactivée.                                         | Oui, en activant la version de clé     |
| CmkErrorKeyNotFound      | Impossible de trouver la clé gérée par le client. | Oui, si la clé a été supprimée mais qu’elle est toujours dans la durée de vidage, en utilisant [Undo-AzKeyVaultKeyRemoval](/powershell/module/az.keyvault/undo-azkeyvaultkeyremoval).<br>ELSE <ol><li>Oui, si le client dispose de la clé sauvegardée et la restaure.</li><li>« NON » dans le cas contraire.</li></ol>
| CmkErrorVaultNotFound |Le coffre de clés de la clé gérée par le client est introuvable. |   Si le coffre de clés a été supprimé :<ol><li>Oui, s’il s’agit de la durée de la protection contre le vidage, à l’aide des étapes de [Récupérer un coffre de clés](../../key-vault/general/soft-delete-overview.md#key-vault-recovery).</li><li>Non au-delà de la durée de la protection contre le vidage.</li></ol><br>Sinon, si le coffre de clés a été migré sur un autre locataire, oui, il peut être récupéré à l’aide d’une des étapes ci-dessous :<ol><li>Restaurez le coffre de clés sur l’ancien locataire.</li><li>Définissez `Identity = None`, puis redéfinissez la valeur sur `Identity = SystemAssigned`. Cela supprime et recrée l’identité une fois que la nouvelle identité a été créée. Activez les autorisations `Get`, `Wrap` et `Unwrap` sur la nouvelle identité dans la stratégie d’accès du coffre de clés.</li></ol>|

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md)