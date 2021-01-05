---
title: Tutoriel - Mise à jour de la fréquence de rotation automatique d’un certificat dans Key Vault | Microsoft Docs
description: Tutoriel montrant comment mettre à jour la fréquence de rotation automatique d’un certificat dans Azure Key Vault à l’aide du portail Azure
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/16/2020
ms.author: sebansal
ms.openlocfilehash: 591d71e8cd6af2801540f5a1a41ad88b1f538e81
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844486"
---
# <a name="tutorial-configure-certificate-auto-rotation-in-key-vault"></a>Tutoriel : Configuration de la rotation automatique d’un certificat dans Key Vault

Azure Key Vault vous permet de provisionner, gérer et déployer facilement des certificats numériques. Les certificats peuvent être des certificats TLS/SSL signés par une autorité de certification (CA) ou un certificat auto-signé. Key Vault peut également demander et renouveler des certificats via des partenariats avec des autorités de certification, en fournissant une solution robuste pour la gestion du cycle de vie des certificats. Dans ce tutoriel, vous mettez à jour la période de validité, la fréquence de rotation automatique et l’autorité de certification d’un certificat.

Ce didacticiel explique les procédures suivantes :

> [!div class="checklist"]
> * Gérer un certificat à l’aide du portail Azure.
> * Ajouter un compte de fournisseur d’autorité de certification.
> * Mettre à jour la période de validité du certificat.
> * Mettre à jour la fréquence de rotation automatique du certificat.
> * Mettre à jour les attributs du certificat avec Azure PowerShell.

Avant de continuer, lisez les [concepts de base de Key Vault](../general/basic-concepts.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-a-vault"></a>Création d'un coffre

Créez un coffre de clés Azure à l’aide du [portail Azure](../general/quick-create-portal.md), d’[Azure CLI](../general/quick-create-cli.md) ou d’[Azure PowerShell](../general/quick-create-powershell.md). Dans l’exemple, le nom du coffre de clés est **Example-Vault**.

![Sortie après la création du coffre de clés](../media/certificates/tutorial-import-cert/vault-properties.png)

## <a name="create-a-certificate-in-key-vault"></a>Créer un certificat dans Key Vault

Créez un certificat ou importez un certificat dans le coffre de clés (voir les [Étapes de création d’un certificat dans Key Vault](../secrets/quick-create-portal.md)). Dans le cas présent, nous allons utiliser un certificat nommé **ExampleCertificate**.

## <a name="update-certificate-lifecycle-attributes"></a>Mettre à jour les attributs du cycle de vie des certificats

Dans Azure Key Vault, vous pouvez mettre à jour les attributs de cycle de vie d’un certificat à la fois avant et après la création du certificat.

Un certificat créé dans le coffre de clés peut être :

- Un certificat auto-signé.
- Un certificat créé avec une autorité de certification qui est associée à Key Vault.
- Un certificat auprès d’une autorité de certification qui n’est pas associée à Key Vault.

Les autorités de certification suivantes sont les fournisseurs actuellement associés à Key Vault :

- DigiCert : Key Vault propose des certificats TLS/SSL OV.
- GlobalSign : Key Vault propose des certificats TLS/SSL OV.

Key Vault fait pivoter automatiquement les certificats via des partenariats établis avec les autorités de certification. Étant donné que Key Vault demande et renouvelle automatiquement les certificats par le biais du partenariat, la fonctionnalité de rotation automatique n’est pas applicable pour les certificats créés avec des autorités de certification qui ne sont pas associées à Key Vault.

> [!NOTE]
> Un administrateur de compte d’un fournisseur d’autorité de certification crée des informations d’identification utilisables par Key Vault pour créer, renouveler et utiliser des certificats TSL/SSL.
![Autorité de certification](../media/certificates/tutorial-rotate-cert/cert-authority-create.png)
>

### <a name="update-certificate-lifecycle-attributes-at-the-time-of-creation"></a>Mettre à jour les attributs du cycle de vie du certificat au moment de la création

1. Dans les pages des propriétés Key Vault, sélectionnez **Certificats**.
1. Sélectionnez **Générer/Importer**.
1. Dans l’écran **Créer un certificat**, mettez à jour les valeurs suivantes :

   - **Période de validité** : entrez la valeur (en mois). La création de certificats à courte durée de vie est une pratique recommandée en matière de sécurité. Par défaut, la valeur de validité d’un certificat nouvellement créé est de 12 mois.
   - **Type d’action de la durée de vie** : Sélectionnez l’action de renouvellement automatique et d’alerte du certificat, puis mettez à jour le **Pourcentage de la durée de vie** ou le **Nombre de jours avant l’expiration**. Par défaut, le renouvellement automatique d’un certificat est défini à 80 % de sa durée de vie. Dans le menu déroulant, sélectionnez l'une des options suivantes.

        |  Automatically renew at a given time (Renouveler automatiquement à un moment donné)| Envoyer un e-mail à tous les contacts à un moment donné (Email all contacts at a given time) |
        |-----------|------|
        |La sélection de cette option *active* la rotation automatique. | La sélection de cette option *ne déclenche pas* la rotation automatique, elle alerte uniquement les contacts.|

1. Sélectionnez **Create** (Créer).

![Cycle de vie du certificat](../media/certificates/tutorial-rotate-cert/create-cert-lifecycle.png)

### <a name="update-lifecycle-attributes-of-a-stored-certificate"></a>Mettre à jour les attributs de cycle de vie d’un certificat stocké

1. Sélectionnez le coffre de clés.
1. Dans les pages des propriétés Key Vault, sélectionnez **Certificats**.
1. Sélectionnez le certificat à mettre à jour. Dans le cas présent, nous allons utiliser un certificat nommé **ExampleCertificate**.
1. Dans la barre de menus supérieure, sélectionnez **Stratégie d’émission**.

   ![Capture d’écran mettant en évidence le bouton Stratégie d’émission.](../media/certificates/tutorial-rotate-cert/cert-issuance-policy.png)

1. Dans l’écran **Stratégie d’émission**, mettez à jour les valeurs suivantes :

   - **Période de validité** : Mettez à jour la valeur (en mois).
   - **Type d’action de la durée de vie** : Sélectionnez l’action de renouvellement automatique et d’alerte du certificat, puis mettez à jour le **Pourcentage de la durée de vie** ou **Nombre de jours avant l’expiration**.

   ![Propriétés du certificat](../media/certificates/tutorial-rotate-cert/cert-policy-change.png)

1. Sélectionnez **Enregistrer**.

> [!IMPORTANT]
> La modification du type d’action de la durée de vie d’un certificat enregistre immédiatement les modifications pour les certificats existants.


### <a name="update-certificate-attributes-by-using-powershell"></a>Mettre à jour les attributs de certificat à l’aide de PowerShell

```azurepowershell


Set-AzureKeyVaultCertificatePolicy -VaultName $vaultName 
                                   -Name $certificateName 
                                   -RenewAtNumberOfDaysBeforeExpiry [276 or appropriate calculated value]
```

> [!TIP]
> Pour modifier la stratégie de renouvellement d’une liste de certificats, utilisez une entrée `File.csv` contenant `VaultName,CertName` comme dans l’exemple suivant :
> <br/>
 `vault1,Cert1` <br/>
>  `vault2,Cert2`
>
>  ```azurepowershell
>  $file = Import-CSV C:\Users\myfolder\ReadCSVUsingPowershell\File.csv 
> foreach($line in $file)
> {
> Set-AzureKeyVaultCertificatePolicy -VaultName $vaultName -Name $certificateName -RenewAtNumberOfDaysBeforeExpiry [276 or appropriate calculated value]
> }
>  ```
> 
Pour en savoir plus sur les paramètres, consultez [az keyvault certificate](/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-set-attributes).

## <a name="clean-up-resources"></a>Nettoyer les ressources

D’autres tutoriels de Key Vault s’appuient sur ce tutoriel. Si vous envisagez de travailler avec ces tutoriels, vous souhaiterez peut-être conserver ces ressources existantes.
Si vous n’en avez plus besoin, supprimez le groupe de ressources. Ce faisant, vous supprimez le coffre de clés et les ressources associées.

Pour supprimer le groupe de ressources à l’aide du portail :

1. Entrez le nom de votre groupe de ressources dans la zone **Recherche** en haut du portail. Lorsque vous voyez le groupe de ressources utilisé dans ce démarrage rapide dans les résultats de recherche, sélectionnez-le.
1. Sélectionnez **Supprimer le groupe de ressources**.
1. Dans le champ **TYPE THE RESOURCE GROUP NAME: (TAPER LE NOM DU GROUPE DE RESSOURCES :)** , tapez le nom du groupe de ressources et sélectionnez **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez mis à jour les attributs de cycle de vie d’un certificat. Pour en savoir plus sur Key Vault et sur la manière de l’intégrer à vos applications, consultez les articles suivants :

- En savoir plus sur la [Gestion de la création de certificats dans Azure Key Vault](./create-certificate-scenarios.md).
- Consultez la [Vue d’ensemble de Key Vault](../general/overview.md).
