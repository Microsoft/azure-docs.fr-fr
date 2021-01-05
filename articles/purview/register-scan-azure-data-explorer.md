---
title: Guide pratique pour analyser Azure Data Explorer
description: Ce guide pratique explique en détail comment analyser Azure Data Explorer.
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 10/9/2020
ms.openlocfilehash: 01a1ded570d20d175b5e8eadb3e6cc8556155a85
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550308"
---
# <a name="register-and-scan-azure-data-explorer"></a>Inscription et analyse d’Azure Data Explorer

Cet article explique comment inscrire un compte Azure Data Explorer dans Azure Purview et configurer une analyse.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Azure Data Explorer prend en charge les analyses complètes et incrémentielles pour capturer les métadonnées et le schéma. Ces analyses permettent également de classer les données automatiquement selon des règles de classification système et personnalisées.

## <a name="prerequisites"></a>Prérequis

- Avant d’inscrire des sources de données, créez un compte Azure Purview. Pour plus d’informations sur la création d’un compte Purview, consultez [Démarrage rapide : Création d’un compte Azure Purview](create-catalog-portal.md).
- Vous devez être administrateur de la source de données Azure Purview.

## <a name="setting-up-authentication-for-a-scan"></a>Configuration de l’authentification pour une analyse

Il n’existe qu’une seule façon de configurer l’authentification pour Azure Data Explorer :

- Principal de service

### <a name="service-principal"></a>Principal du service

Si vous souhaitez appliquer l’authentification par principal de service pour les analyses, vous pouvez utiliser un principal de service existant ou en créer un. 

> [!Note]
> Si vous devez créer un principal de service, procédez comme suit :
> 1. Accédez au [portail Azure](https://portal.azure.com).
> 1. Sélectionnez **Azure Active Directory** dans le menu de gauche.
> 1. Sélectionnez **Inscriptions d’applications**.
> 1. Sélectionnez **+ Nouvelle inscription d’application**.
> 1. Entrez un nom pour **l’application** (nom du principal de service).
> 1. Sélectionnez **Comptes dans ce répertoire organisationnel uniquement**.
> 1. Pour l’URI de redirection, sélectionnez **Web** et entrez l’URL de votre choix. Il n’est pas nécessaire qu’elle soit réelle ni qu’elle fonctionne.
> 1. Sélectionnez ensuite **Inscription**.

Il est nécessaire de récupérer l’ID d’application et le secret du principal de service :

1. Accédez à votre principal de service sur le [Portail Azure](https://portal.azure.com).
1. Copiez les valeurs **ID d’application (client)** dans **Vue d’ensemble** et **Clé secrète client** dans **Certificats et secrets**.
1. Accédez à votre coffre de clés.
1. Sélectionnez **Paramètres > Secrets**.
1. Sélectionnez **+ Générer/importer**, puis entrez le **nom** de votre choix et la **Valeur** comme **Clé secrète client** de votre principal de service.
1. Sélectionnez **Créer** pour terminer.
1. Si votre coffre de clés n’est pas encore connecté à Purview, vous devrez [créer une connexion de coffre de clés](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Enfin, [créez des informations d’identification](manage-credentials.md#create-a-new-credential) à l’aide du principal de service pour configurer votre analyse.

#### <a name="granting-the-service-principal-access-to-your-azure-data-explorer-instance"></a>Octroi au principal du service de l’accès à l’instance Azure Data Explorer

1. Accédez au portail Azure. Ensuite, accédez à votre instance Azure Data Explorer.

1. Ajoutez le principal du service au rôle **AllDatabasesViewer** dans l’onglet **Autorisations** (cf. capture d’écran suivante).

    :::image type="content" source="./media/register-scan-azure-data-explorer/permissions-auth.png" alt-text="Capture d’écran montrant comment ajouter un principal de service dans les autorisations" border="true":::

## <a name="register-an-azure-data-explorer-account"></a>Inscription d’un compte Azure Data Explorer

Pour inscrire un nouveau compte Azure Data Explorer (Kusto) dans votre catalogue de données, procédez comme suit :

1. Accédez à votre compte Purview.
1. Sélectionnez **Sources** dans le volet de navigation de gauche.
1. Sélectionnez **Inscrire**.
1. Sous **Inscrire des sources**, sélectionnez **Azure Data Explorer**.
1. Sélectionnez **Continue** (Continuer)

:::image type="content" source="media/register-scan-azure-data-explorer/register-new-data-source.png" alt-text="Inscription d’une nouvelle source de données" border="true":::

Sur l’écran **Inscrire des sources (Azure Data Explorer (Kusto))** , procédez comme suit :

1. Entrez le **Nom** sous lequel la source de données apparaîtra dans le catalogue.
1. Choisissez la façon dont vous souhaitez pointer vers le compte de stockage de votre choix :
   1. Sélectionnez **À partir d’un abonnement Azure**, puis sélectionnez l’abonnement approprié dans la zone de liste déroulante **Abonnement Azure** et le cluster approprié dans la zone de liste déroulante **Cluster**.
   1. Vous pouvez également sélectionner **Entrer manuellement** et entrer un point de terminaison de service (URL).
1. Sélectionnez **Terminer** pour inscrire la source de données.

:::image type="content" source="media/register-scan-azure-data-explorer/register-sources.png" alt-text="Options d’inscription des sources" border="true":::

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Navigation dans le catalogue de données Azure Purview](how-to-browse-catalog.md)
- [Recherche dans le catalogue de données Azure Purview](how-to-search-catalog.md)
