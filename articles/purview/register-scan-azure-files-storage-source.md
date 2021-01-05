---
title: Guide pratique pour analyser des fichiers Azure
description: Ce guide pratique explique en détail comment analyser des fichiers Azure.
author: SunetraVirdi
ms.author: suvirdi
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 10/01/2020
ms.openlocfilehash: a0bd7a4cd8afafc16f05b4a37cd5723304ad931e
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550295"
---
# <a name="register-and-scan-azure-files"></a>Inscription et analyse de fichiers Azure Files

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Azure Files prend en charge les analyses complètes et incrémentielles pour capturer les métadonnées et y appliquer des classifications système et personnalisées.

## <a name="prerequisites"></a>Prérequis

- Avant d’inscrire des sources de données, créez un compte Azure Purview. Pour plus d’informations sur la création d’un compte Purview, consultez [Démarrage rapide : Création d’un compte Azure Purview](create-catalog-portal.md).
- Vous devez être administrateur de la source de données pour pouvoir configurer et planifier des analyses. Pour plus d’informations, consultez [Autorisations du catalogue](catalog-permissions.md).

## <a name="register-an-azure-files-storage-account"></a>Inscription d’un compte de stockage Azure Files

Pour inscrire un nouveau compte Azure Files dans votre catalogue de données, procédez comme suit :

1. Accédez à votre catalogue de données Purview.
1. Sélectionnez **Centre de gestion** dans le volet de navigation gauche.
1. Sélectionnez **Sources de données** sous **Sources et analyse**.
1. Sélectionnez **+Nouveau**.
1. Sous **Inscrire des sources**, sélectionnez **Azure Files**. Sélectionnez **Continuer**.

:::image type="content" source="media/register-scan-azure-files/register-new-data-source.png" alt-text="Inscription d’une nouvelle source de données" border="true":::

Sur l’écran **Inscrire des sources (Azure Files)** , procédez comme suit :

1. Entrez le **Nom** sous lequel la source de données apparaîtra dans le catalogue.
1. Choisissez la façon dont vous souhaitez pointer vers le compte de stockage de votre choix :
   1. Sélectionnez **À partir d’un abonnement Azure**, puis sélectionnez l’abonnement approprié dans la zone de liste déroulante **Abonnement Azure** et le compte de stockage approprié dans la zone de liste déroulante **Nom du compte de stockage**.
   1. Vous pouvez également sélectionner **Entrer manuellement** et entrer un point de terminaison de service (URL).
1. Sélectionnez **Terminer** pour inscrire la source de données.

:::image type="content" source="media/register-scan-azure-files/register-sources.png" alt-text="Options d’inscription des sources" border="true":::

## <a name="set-up-authentication-for-a-scan"></a>Configuration de l’authentification pour une analyse

Pour configurer l’authentification du stockage Azure Files à l’aide d’une clé de compte, procédez comme suit :

1. Sélectionner la méthode d’authentification **Clé de compte**.
2. Sélectionnez l’option **À partir d’un abonnement Azure**.
3. Sélectionnez votre abonnement Azure dans lequel se trouve le compte Azure Files.
4. Sélectionnez votre compte de stockage dans la liste.
5. Cliquez sur **Terminer**.

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Navigation dans le catalogue de données Azure Purview](how-to-browse-catalog.md)
- [Recherche dans le catalogue de données Azure Purview](how-to-search-catalog.md)