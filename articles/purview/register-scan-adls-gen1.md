---
title: Inscrire et analyser Azure Data Lake Storage (ADLS) Gen1
description: Ce tutoriel explique comment analyser des données d’Azure Data Lake Storage Gen1 dans Azure Purview.
author: shsandeep123
ms.author: sandeepshah
ms.service: data-catalog
ms.subservice: data-catalog-gen2
ms.topic: how-to
ms.date: 11/30/2020
ms.openlocfilehash: a831681f892de9f6aae50fa9a2fcf71e883fe6ba
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97693713"
---
# <a name="register-and-scan-azure-data-lake-storage-gen1"></a>Inscrire et analyser Azure Data Lake Storage Gen1

Cet article explique comment inscrire Azure Data Lake Storage Gen1 comme source de données dans Azure Purview et comment configurer une analyse.

> [!Note]
> Azure Data Lake Storage Gen2 est maintenant en disponibilité générale. Nous vous recommandons de commencer à l’utiliser dès aujourd'hui. Pour plus d’informations, consultez la [page du produit](https://azure.microsoft.com/services/storage/data-lake-storage/).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

La source de données Azure Data Lake Storage Gen1 prend en charge les fonctionnalités suivantes :

- **Analyses complètes et incrémentielles** pour capturer les métadonnées et la classification dans Azure Data Lake Storage Gen1

- **Traçabilité** entre les ressources de données pour les activités de copie/flux de données ADF

## <a name="prerequisites"></a>Prérequis

- Avant d’inscrire des sources de données, créez un compte Azure Purview. Pour plus d’informations sur la création d’un compte Purview, consultez [Démarrage rapide : Créer un compte Azure Purview](create-catalog-portal.md).
- Vous devez être un administrateur de source de données Azure Purview

## <a name="setting-up-authentication-for-a-scan"></a>Configuration de l’authentification pour une analyse

Les méthodes d’authentification suivantes sont prises en charge pour Azure Data Lake Storage Gen1 :

- Identité managée
- Principal du service

### <a name="managed-identity-recommended"></a>Identité managée (recommandé)

À des fins de facilité et de sécurité, vous souhaiterez peut-être utiliser Purview MSI pour vous authentifier auprès de votre magasin de données.

Pour configurer une analyse le MSI de Data Catalog, vous devez d’abord accorder à votre compte Purview l’autorisation d’analyser la source de données. Vous devez effectuer cette étape *avant* de configurer votre analyse, sinon votre celle-ci échouera.

#### <a name="adding-the-data-catalog-msi-to-an-azure-data-lake-storage-gen1-account"></a>Ajout du MSI Data Catalog à un compte Azure Data Lake Storage Gen1

Vous pouvez ajouter le MSI de Data Catalog au niveau de l’abonnement, du groupe de ressources ou de la ressource, en fonction des autorisations d’analyse que vous souhaitez lui accorder.

> [!Note]
> Vous devez être propriétaire de l’abonnement pour pouvoir ajouter une identité managée sur une ressource Azure.

1. Dans le [portail Azure](https://portal.azure.com), recherchez l’abonnement, le groupe de ressources ou la ressource (par exemple, un compte de stockage Azure Data Lake Storage Gen1) que le catalogue doit être autorisé à analyser.

2. Cliquez sur **Vue d’ensemble** et sélectionnez **Explorateur de données**

   :::image type="content" source="./media/register-scan-adls-gen1/access-control.png" alt-text="Choisir le contrôle d’accès":::

3. Cliquez sur **Accès** dans la barre de navigation supérieure

   :::image type="content" source="./media/register-scan-adls-gen1/access.png" alt-text="Cliquez sur Accès":::

4. Cliquez sur **Ajouter**. Ajoutez **Purview Catalog** dans Sélectionner un utilisateur ou un groupe. Sélectionnez les autorisations **Lire** et **Exécuter**. Veillez à choisir **Ce dossier et tous les enfants** dans l’option Ajouter à, comme indiqué dans la capture d’écran ci-dessous, puis cliquez sur **OK**
   :::image type="content" source="./media/register-scan-adls-gen1/managed-instance-authentication.png" alt-text="Détails de l’authentification MSI":::

5. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une nouvelle connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).

6. Enfin, [créez des informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide du principal de service pour configurer votre analyse
> [!Note]
> Une fois que vous avez ajouté le MSI du catalogue sur la source de données, attendez jusqu’à 15 minutes que les autorisations soient propagées avant de configurer une analyse.

Au bout de 15 minutes environ, le catalogue doit disposer des autorisations appropriées pour pouvoir analyser la ou les ressources.

### <a name="service-principal"></a>Principal du service

Pour utiliser un principal de service, vous devez d’abord en créer un en procédant comme suit :

1. Accédez au [portail Azure](https://portal.azure.com).

2. Dans le menu de gauche, sélectionnez **Azure Active Directory**.

3. Sélectionnez **Inscriptions d’applications**.

4. Sélectionnez **+ Nouvelle inscription d’application**.

5. Entrez un nom pour **l’application** (nom du principal de service).

6. Sélectionnez **Comptes dans ce répertoire organisationnel uniquement**.

7. Pour **URI de redirection**, sélectionnez **Web** et entrez l’URL de votre choix. Il n’est pas nécessaire qu’elle soit réelle ou qu’elle fonctionne.

8. Sélectionnez ensuite **Inscription**.

9. Copiez les valeurs à partir du nom d’affichage et de l’ID d’application.

#### <a name="adding-the-data-catalog-service-principal-to-an-azure-data-lake-storage-gen1-account"></a>Ajout du principal de service Data Catalog à un compte Azure Data Lake Storage Gen1
1. Dans le [portail Azure](https://portal.azure.com), recherchez l’abonnement, le groupe de ressources ou la ressource (par exemple, un compte de stockage Azure Data Lake Storage Gen1) que le catalogue doit être autorisé à analyser.

2. Cliquez sur **Vue d’ensemble** et sélectionnez **Explorateur de données**

   :::image type="content" source="./media/register-scan-adls-gen1/access-control.png" alt-text="Choisir le contrôle d’accès":::

3. Cliquez sur **Accès** dans la barre de navigation supérieure

   :::image type="content" source="./media/register-scan-adls-gen1/access.png" alt-text="Cliquez sur Accès":::

4. Cliquez sur **Ajouter**. Ajoutez **Application du principal de service** dans Sélectionner un utilisateur ou un groupe. Sélectionnez les autorisations **Lire** et **Exécuter**. Veillez à choisir **Ce dossier et tous les enfants** dans l’option Ajouter à, comme indiqué dans la capture d’écran ci-dessous, puis cliquez sur **OK**
   :::image type="content" source="./media/register-scan-adls-gen1/service-principal-authentication.png" alt-text="Détails de l’authentification du principal de service":::

5. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une nouvelle connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).

6. Enfin, [créez des informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide du principal de service pour configurer votre analyse.

## <a name="register-azure-data-lake-storage-gen1-data-source"></a>Inscrire la source de données Azure Data Lake Storage Gen1

Pour inscrire un nouveau compte ADLS Gen1 dans votre catalogue de données, procédez comme suit :

1. Accédez à votre Purview Data Catalog.
2. Sélectionnez **Sources** dans la barre de navigation à gauche.
3. Sélectionnez **Inscrire**.
4. Sous **Inscrire des sources**, sélectionnez **Azure Data Lake Storage Gen1**. 
5. Sélectionnez **Continuer**.

Dans l’écran Inscrire des sources (Azure Data Lake Storage Gen1), procédez comme suit :

1. Entrez un **nom** avec lequel la source de données sera répertoriée dans le catalogue.
2. Choisissez votre abonnement pour filtrer les comptes de stockage
3. Sélectionner un compte de stockage
4. Sélectionnez une collection ou créez-en une (facultatif)
5. Terminez pour inscrire la source de données.

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Parcourir le catalogue de données Azure Purview](how-to-browse-catalog.md)
- [Effectuer une recherche dans le catalogue de données Azure Purview](how-to-search-catalog.md)
