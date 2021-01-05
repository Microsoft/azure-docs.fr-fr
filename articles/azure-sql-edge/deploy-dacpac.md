---
title: Utilisation des packages SQL Database DACPAC et BACPAC – Azure SQL Edge
description: En savoir plus sur l’utilisation des packages dacpacs and bacpacs in Azure SQL Edge
keywords: SQL Edge, sqlpackage
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/03/2020
ms.openlocfilehash: 40bd0eda16f9f96dd356eef900369ab25854e9f9
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392246"
---
# <a name="sql-database-dacpac-and-bacpac-packages-in-sql-edge"></a>Packages SQL Database DACPAC et BACPAC dans SQL Edge

Azure SQL Edge est un moteur de base de données relationnelle optimisé conçu pour les déploiements IoT et de périphérie. Il repose sur les dernières versions du moteur de base de données Microsoft SQL, qui fournit des fonctionnalités de performances, de sécurité et de traitement des requêtes de pointe. Avec les fonctionnalités de gestion de base de données relationnelle de SQL Server les plus performantes du marché, Azure SQL Edge offre des fonctionnalités de diffusion en continu intégrées pour l’analyse en temps réel et le traitement des événements complexes.

Azure SQL Edge fournit un mécanisme natif vous permettant de déployer un package [SQL Database DACPAC et BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications) pendant ou après le déploiement de SQL Edge.

Vous pouvez déployer les packages SQL Database dacpac et bacpac sur SQL Edge à l’aide de la variable d’environnement `MSSQL_PACKAGE`. Vous pouvez configurer la variable d’environnement avec l’un des éléments suivants.  
- Emplacement du dossier local dans le conteneur SQL contenant les fichiers dacpac et bacpac. Ce dossier peut être mappé à un volume hôte à l’aide de points de montage ou de conteneurs de volume de données. 
- Chemin d’accès au fichier local dans le mappage du conteneur SQL au fichier dacpac ou bacpac. Ce dossier peut être mappé à un volume hôte à l’aide de points de montage ou de conteneurs de volume de données. 
- Chemin de fichier local dans le mappage du conteneur SQL à un fichier zip contenant les fichiers dacpac ou bacpac. Ce dossier peut être mappé à un volume hôte à l’aide de points de montage ou de conteneurs de volume de données. 
- URL SAP d’objet blob Azure pointant vers un fichier zip contenant les fichiers dacpac et bacpac.
- URL SAP d’objet blob Azure pointant vers un fichier dacpac ou bacpac. 

## <a name="use-a-sql-database-dac-package-with-sql-edge"></a>Utiliser un package DAC SQL Database avec SQL Edge

Pour déployer (ou importer) un package SQL Database DAC `(*.dacpac)` ou un fichier BACPAC `(*.bacpac)` à l’aide du stockage d’objets blob Azure et d’un fichier zip, procédez de la manière décrite ci-dessous. 

1. Créez/extrayez un package DAC, ou exportez un fichier Bacpac à l’aide du mécanisme mentionné ci-dessous. 
    - Créez ou extrayez un package DAC SQL Database. Pour savoir comment générer un package DAC pour une base de données SQL Server existante, consultez [Extraire une package DAC d’une base de données](/sql/relational-databases/data-tier-applications/extract-a-dac-from-a-database/).
    - Exportation d’un package DAC déployé ou d’une base de données. Pour plus d’informations sur la génération d’un fichier babac pour une base de données SQL Server existante, consultez [Exporter une application de la couche données](/sql/relational-databases/data-tier-applications/export-a-data-tier-application/).

2. Compressez le `*.bacpac` ou le fichier `*.dacpac`, et chargez-le dans un compte de stockage d’objets blob Azure. Pour plus d’informations sur le chargement de fichiers dans Stockage Blob Azure, consultez [Charger, télécharger et lister les objets blob avec le portail Azure](../storage/blobs/storage-quickstart-blobs-portal.md).

3. Générez une signature d’accès partagé pour le fichier zip à partir du portail Azure. Pour plus d’informations, consultez [Déléguer l’accès avec des signature d’accès partagé (SAP)](../storage/common/storage-sas-overview.md).

4. Mettez à jour la configuration du module SQL Edge pour inclure l’URI d’accès partagé du package DAC. Pour mettre à jour le module SQL Edge, procédez comme suit :

    1. Sur le portail Azure, accédez à votre déploiement IoT Hub.

    2. Dans le volet gauche, sélectionnez **IoT Edge**.

    3. Dans la page **IoT Edge** , recherchez et sélectionnez l’IoT Edge dans lequel le module SQL Edge est déployé.

    4. Dans la page **Appareil IoT Edge** , sélectionnez **Définir des modules**.

    5. Sur la page **Définir des modules** , cliquez sur le module Azure SQL Edge.

    6. Dans le volet **Mettre à jour le module IoT Edge** , sélectionnez **Variables d’environnement**. Ajoutez la variable d’environnement `MSSQL_PACKAGE` et spécifiez l’URL SAP générée à l’étape 3 ci-dessus en tant que valeur pour la variable d’environnement. 

    7. Sélectionnez **Update**.

    8. Sur la page **Définir des modules** , sélectionnez **Vérifier + créer**.

    9. Sur la page **Définir des modules** , sélectionnez **Créer**.

5. Après la mise à jour du module, les fichiers de package sont téléchargés, décompressés et déployés pour l’instance SQL Edge.

À chaque redémarrage du conteneur Azure SQL Edge, SQL Edge tente de télécharger le package de fichiers compressé et d’évaluer les modifications. Si une nouvelle version du fichier dacpac est rencontrée, les modifications sont déployées vers la base de données dans SQL Edge.

## <a name="known-issue"></a>Problème connu

Pendant certains déploiements DACPAC ou BACPAC, les utilisateurs peuvent rencontrer un dépassement du délai d’attente d’une commande, entraînant l’échec de l’opération de déploiement dacpac. Si vous rencontrez ce problème, utilisez SQLPackage.exe (ou les outils clients SQL) pour appliquer manuellement le déploiement DACPAC ou BACPAC. 

## <a name="next-steps"></a>Étapes suivantes

- [Déployer SQL Edge via le portail Azure](deploy-portal.md)
- [Transmettre des données en continu](stream-data.md)
- [Machine Learning et IA avec ONNX dans SQL Edge](onnx-overview.md)