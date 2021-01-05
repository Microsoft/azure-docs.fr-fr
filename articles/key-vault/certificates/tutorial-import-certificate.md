---
title: Tutorial - Importer un certificat dans Key Vault avec le portail Azure | Microsoft Docs
description: Tutoriel expliquant comment importer un certificat dans Azure Key Vault
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: tutorial
ms.custom: mvc, devx-track-azurecli
ms.date: 04/16/2020
ms.author: sebansal
ms.openlocfilehash: 8689b6851ae219ee6f41ebf58736692e557b2344
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289729"
---
# <a name="tutorial-import-a-certificate-in-azure-key-vault"></a>Tutoriel : Importer un certificat dans Azure Key Vault

Azure Key Vault est un service cloud qui fonctionne comme un magasin sécurisé contenant des secrets. Vous pouvez stocker des clés, des mots de passe, des certificats et d’autres secrets en toute sécurité. Vous pouvez créer et gérer des coffres de clés Azure grâce au portail Azure. Dans ce tutoriel, vous allez créer un coffre de clés, puis l’utiliser pour importer un certificat. Pour plus d’informations sur Key Vault, consultez la [présentation](../general/overview.md).

Ce didacticiel explique les procédures suivantes :

> [!div class="checklist"]
> * Création d’un coffre de clés
> * Importation d’un certificat dans Key Vault à l’aide du portail.
> * Importation d’un certificat dans Key Vault à l’aide de CLI.
> * Importation d’un certificat dans Key Vault à l’aide de PowerShell.


Avant de continuer, lisez les [concepts de base de Key Vault](../general/basic-concepts.md). 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-a-vault"></a>Création d'un coffre

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.
2. Dans la zone de recherche, entrez **Key Vault**.
3. Dans la liste des résultats, choisissez **Key Vault**.
4. Dans la section Key Vault, choisissez **Créer**.
5. Dans la section **Créer un coffre de clés**, renseignez les informations suivantes :
    - **Name** : un nom unique est obligatoire. Pour ce guide de démarrage rapide, nous utilisons **Example-Vault**. 
    - **Abonnement**: Choisissez un abonnement.
    - Sous **Groupe de ressources**, choisissez **Créer** et entrez le nom du groupe de ressources.
    - Dans le menu déroulant **Emplacement**, choisissez un emplacement.
    - Conservez les valeurs par défaut des autres options.
6. Après avoir renseigné les informations ci-dessus, sélectionnez **Créer**.

Notez les deux propriétés ci-dessous :

* **Nom du coffre** : dans l’exemple, il s’agit d’**Example-Vault**. Vous allez utiliser ce nom pour les autres étapes.
* **URI du coffre** : dans l’exemple, il s’agit de https://example-vault.vault.azure.net/. Les applications qui utilisent votre coffre via son API REST doivent utiliser cet URI.

À ce stade, votre compte Azure est le seul autorisé à effectuer des opérations sur ce nouveau coffre.

![Sortie après la création du coffre de clés](../media/certificates/tutorial-import-cert/vault-properties.png)

## <a name="import-a-certificate-to-key-vault"></a>Importer un certificat dans Key Vault

Pour importer un certificat dans le coffre, vous devez disposer d’un fichier de certificat PEM ou PFX sur le disque. Dans le cas présent, nous allons importer un certificat dont le nom de fichier est **ExampleCertificate**.

> [!IMPORTANT]
> Dans Azure Key Vault, les formats de certificat pris en charge sont PFX et PEM. 
> - Le format de fichier .pem contient un ou plusieurs fichiers de certificat x509.
> - Le format de fichier .pfx est un format de fichier d’archive pour le stockage de plusieurs objets de chiffrement dans un fichier unique, c’est-à-dire un certificat de serveur (émis pour votre domaine), une clé privée correspondante et éventuellement une autorité de certification intermédiaire.  

1. Dans les pages des propriétés Key Vault, sélectionnez **Certificats**.
2. Cliquez sur **Generate/Import (Générer/Importer)** .
3. Dans l’écran **Créer un certificat**, choisissez les valeurs suivantes :
    - **Méthode de création de certificat** : Importation.
    - **Nom du certificat** : ExampleCertificate.
    - **Charger le fichier de certificat** : sélectionnez le fichier de certificat à partir du disque
    - **Mot de passe** : Si vous chargez un fichier de certificat protégé par mot de passe, indiquez ce mot de passe ici. Autrement, laissez le champ vide. Une fois le fichier de certificat importé, le coffre de clés supprime ce mot de passe.
4. Cliquez sur **Créer**.

![Propriétés du certificat](../media/certificates/tutorial-import-cert/cert-import.png)

En ajoutant un certificat à l’aide de la méthode d’**importation**, Azure Key Vault renseigne automatiquement les paramètres de certificat (c’est-à-dire la période de validité, le nom de l’émetteur, la date d’activation, etc.).

Quand vous recevez le message confirmant que le certificat a été correctement importé, vous pouvez cliquer dessus dans la liste pour afficher ses propriétés. 

![Capture d’écran qui indique où afficher les propriétés du certificat.](../media/certificates/tutorial-import-cert/current-version-hidden.png)

## <a name="import-a-certificate-using-azure-cli"></a>Importer un certificat à l’aide d’Azure CLI

Importez un certificat dans un coffre de clés spécifié. Pour importer dans Azure Key Vault un certificat valide existant contenant une clé privée, le fichier à importer peut être au format PFX ou PEM. Si le certificat est au format PEM, il doit contenir la clé ainsi que des certificats X509. Cette opération exige l’autorisation de certificates/d’importation.

```azurecli
az keyvault certificate import --file
                               --name
                               --vault-name
                               [--disabled {false, true}]
                               [--only-show-errors]
                               [--password]
                               [--policy]
                               [--subscription]
                               [--tags]
```

Découvrez-en plus sur les [paramètres](/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-import).

Après l’importation du certificat, vous pouvez voir le certificat au moyen de [Certificate show](/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-show)


```azurecli
az keyvault certificate show [--id]
                             [--name]
                             [--only-show-errors]
                             [--subscription]
                             [--vault-name]
                             [--version]
```

Vous venez de créer un coffre de clés, d’importer un certificat et d’afficher les propriétés du certificat.

## <a name="import-a-certificate-using-azure-powershell"></a>Importer un certificat à l’aide d’Azure PowerShell

```
Import-AzureKeyVaultCertificate
      [-VaultName] <String>
      [-Name] <String>
      -FilePath <String>
      [-Password <SecureString>]
      [-Tag <Hashtable>]
      [-DefaultProfile <IAzureContextContainer>]
      [-WhatIf]
      [-Confirm]
      [<CommonParameters>]
```

Découvrez-en plus sur les [paramètres](/powershell/module/azurerm.keyvault/import-azurekeyvaultcertificate?view=azurermps-6.13.0).


## <a name="clean-up-resources"></a>Nettoyer les ressources

D’autres démarrages rapides et didacticiels sur les coffres de clés reposent sur ce démarrage rapide. Si vous prévoyez d’utiliser d’autres démarrages rapides et didacticiels, il peut être utile de conserver ces ressources.
Si vous n’en avez plus besoin, supprimez le groupe de ressources. Ce faisant, vous supprimez le coffre de clés et les ressources associées. Pour supprimer le groupe de ressources à l’aide du portail :

1. Entrez le nom de votre groupe de ressources dans la zone Recherche en haut du portail. Lorsque vous voyez le groupe de ressources utilisé dans ce démarrage rapide dans les résultats de recherche, sélectionnez-le.
2. Sélectionnez **Supprimer le groupe de ressources**.
3. Dans le champ **TYPE THE RESOURCE GROUP NAME: (TAPER LE NOM DU GROUPE DE RESSOURCES :)** , tapez le nom du groupe de ressources et sélectionnez **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un coffre de clés et vous y avez importé un certificat. Pour en savoir plus sur Key Vault et sur la manière de l’intégrer à vos applications, consultez les articles ci-dessous.

- En savoir plus sur la [Gestion de la création de certificats dans Azure Key Vault](./create-certificate-scenarios.md)
- Consulter des exemples d’[importation de certificats à l’aide d’API REST](/rest/api/keyvault/importcertificate/importcertificate)
- Passer en revue les [bonnes pratiques relatives à Azure Key Vault](../general/best-practices.md)