---
title: Inscrire et analyser Azure Data Lake Storage (ADLS) Gen2
description: Ce tutoriel explique comment analyser Azure Data Lake Storage Gen2.
author: shsandeep123
ms.author: sandeepshah
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/17/2020
ms.openlocfilehash: 0232682661387e635e6ce8bfb5fc6e03301d130c
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97694799"
---
# <a name="register-and-scan-azure-data-lake-storage-gen2"></a>Inscrire et analyser Azure Data Lake Storage Gen2

Cet article explique comment inscrire Azure Data Lake Storage Gen2 comme source de données dans Azure Purview et comment configurer une analyse.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

La source de données Azure Data Lake Storage Gen2 prend en charge les fonctionnalités suivantes :

- **Analyses complètes et incrémentielles** pour capturer les métadonnées et la classification dans Azure Data Lake Storage Gen2

- **Traçabilité** entre les ressources de données pour les activités de copie/flux de données ADF

## <a name="prerequisites"></a>Prérequis

Avant d’inscrire des sources de données, créez un compte Azure Purview. Pour plus d’informations sur la création d’un compte Purview, consultez [Démarrage rapide : Créer un compte Azure Purview](create-catalog-portal.md).

### <a name="setting-up-authentication-for-a-scan"></a>Configuration de l’authentification pour une analyse

Les méthodes d’authentification suivantes sont prises en charge pour Azure Data Lake Storage Gen2 :

- Identité managée
- Principal du service
- Clé du compte

#### <a name="managed-identity-recommended"></a>Identité managée (recommandé)

Lorsque vous choisissez **Identité managée**, pour configurer la connexion, vous devez d’abord accorder à votre compte Purview l’autorisation d’analyser la source de données :

1. Accédez à votre compte de stockage ADLS Gen2.
1. Dans le menu de navigation de gauche, sélectionnez **Contrôle d’accès (IAM)** . 
1. Sélectionnez **Ajouter**.
1. Définissez le **Rôle** sur **Lecteur de données blob de stockage**, puis entrez le nom de votre compte Azure Purview dans la zone d’entrée **Sélectionner**. Ensuite, sélectionnez **Enregistrer** pour attribuer cette attribution de rôle à votre compte Purview.

> [!Note]
> Pour plus d’informations, consultez les étapes sous [Autoriser l’accès aux objets blob et files d’attente avec Azure Active Directory](https://docs.microsoft.com/azure/storage/common/storage-auth-aad)

#### <a name="account-key"></a>Clé du compte

Lorsque la méthode d’authentification sélectionnée est **Clé de compte**, vous devez récupérer votre clé d’accès et la stocker dans le coffre de clés :

1. Accédez à votre compte de stockage ADLS Gen2
1. Sélectionnez **Paramètres > Clés d’accès**
1. Copiez votre *clé* et enregistrez-la quelque part pour les étapes suivantes
1. Accédez à votre coffre de clés.
1. Sélectionnez **Paramètres > Secrets**
1. Sélectionnez **+ Générer/importer** et entrez le **nom** et la **valeur** comme pour la *clé* de votre compte de stockage.
1. Sélectionnez **Créer** pour terminer
1. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une nouvelle connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account)
1. Enfin, [créez des informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide de la clé pour configurer votre analyse

#### <a name="service-principal"></a>Principal du service

Pour utiliser un principal de service, vous pouvez en utiliser un existant ou en créer un. 

> [!Note]
> Si vous devez créer un principal de service, procédez comme suit :
> 1. Accédez au [portail Azure](https://portal.azure.com).
> 1. Dans le menu de gauche, sélectionnez **Azure Active Directory**.
> 1. Sélectionnez **Inscriptions d’applications**.
> 1. Sélectionnez **+ Nouvelle inscription d’application**.
> 1. Entrez un nom pour **l’application** (nom du principal de service).
> 1. Sélectionnez **Comptes dans ce répertoire organisationnel uniquement**.
> 1. Pour URI de redirection, sélectionnez **Web** et entrez l’URL de votre choix. Il n’est pas nécessaire qu’elle soit réelle ou qu’elle fonctionne.
> 1. Sélectionnez ensuite **Inscription**.

Il est nécessaire de capturer l’ID d’application et le secret du principal de service :

1. Accédez à votre principal de service dans le [portail Azure](https://portal.azure.com)
1. Copiez les valeurs **ID d’application (client)** dans **Vue d’ensemble** et **Secret client** dans **Certificats & secrets**.
1. Accédez à votre coffre de clés.
1. Sélectionnez **Paramètres > Secrets**
1. Sélectionnez **+ Générer/importer** et entrez le **nom** de votre choix et la **valeur** comme le **Secret client** de votre principal de service
1. Sélectionnez **Créer** pour terminer
1. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une nouvelle connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account)
1. Enfin, [créez des informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide du principal de service pour configurer votre analyse

##### <a name="granting-the-service-principal-access-to-your-adls-gen2-account"></a>Accorder au principal de service l’accès à votre compte ADLS Gen2

1. Accédez à votre compte de stockage.
1. Dans le menu de navigation de gauche, sélectionnez **Contrôle d’accès (IAM)** . 
1. Sélectionnez **Ajouter**.
1. Définissez le **Rôle** sur **Lecteur de données blob de stockage**, puis entrez le nom ou ID d’objet de votre principal de service dans la zone d’entrée **Sélectionner**. Ensuite, sélectionnez **Enregistrer** pour fournir cette attribution de rôle à votre principal de service.
### <a name="firewall-settings"></a>Paramètres du pare-feu

> [!NOTE]
> Si un pare-feu est activé pour le compte de stockage, vous devez utiliser la méthode d’authentification **Identité managée** lors de la configuration d’une analyse.

1. Accédez à votre compte de stockage ASLS Gen2 dans le [portail Azure](https://portal.azure.com)
1. Accédez à **Paramètres > Mise en réseau** et
1. Sélectionnez **Réseaux sélectionnés** sous **Autoriser l’accès depuis**
1. Dans la section **Exceptions**, sélectionnez **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage** et appuyez sur **Enregistrer**

:::image type="content" source="./media/register-scan-adls-gen2/firewall-setting.png" alt-text="Capture d’écran montrant la configuration du pare-feu":::

## <a name="register-azure-data-lake-storage-gen2-data-source"></a>Inscrire la source de données Azure Data Lake Storage Gen2

Pour inscrire un nouveau compte ADLS Gen2 dans votre catalogue de données, procédez comme suit :

1. Accédez à votre compte Purview
2. Sélectionnez **Sources** dans la barre de navigation à gauche
3. Sélectionnez **Inscrire**.
4. Sous **Inscrire des sources**, sélectionnez **Azure Data Lake Storage Gen2**
5. Sélectionnez **Continue** (Continuer)

Dans l’écran **Inscrire des sources (Azure Data Lake Storage Gen2)** , procédez comme suit :

1. Entrez un **nom** avec lequel la source de données sera répertoriée dans le catalogue.
2. Choisissez votre abonnement pour filtrer les comptes de stockage
3. Sélectionner un compte de stockage
4. Sélectionnez une collection ou créez-en une (facultatif)
5. **Terminer** pour inscrire la source de données.

:::image type="content" source="media/register-scan-adls-gen2/register-sources.png" alt-text="options pour inscrire des sources" border="true":::

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Parcourir le catalogue de données Azure Purview](how-to-browse-catalog.md)
- [Effectuer une recherche dans le catalogue de données Azure Purview](how-to-search-catalog.md)
