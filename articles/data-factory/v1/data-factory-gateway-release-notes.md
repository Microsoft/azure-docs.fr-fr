---
title: Notes de version pour la passerelle de gestion des données
description: Notes de version pour la passerelle de gestion des données
services: data-factory
author: nabhishek
manager: anandsub
ms.assetid: 14762e82-76d9-41c4-ba9f-14a54da29c36
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: 0538777cadf7935a6684932a17fadc10849518d8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001624"
---
# <a name="release-notes-for-data-management-gateway"></a>Notes de version pour la passerelle de gestion des données
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Runtime d’intégration auto-hébergé dans V2](../create-self-hosted-integration-runtime.md).

Un des défis de l’intégration de données modernes consiste à déplacer des données vers et depuis un site local et le cloud. Azure Data Factory effectue cette intégration avec la passerelle de gestion des données Microsoft, qui est un agent pouvant être installé en local pour activer le déplacement de données hybrides.

Consultez les articles suivants pour des informations détaillées sur la passerelle de gestion des données et son utilisation :

*  [Passerelle de gestion de données](data-factory-data-management-gateway.md)
*  [Déplacement de données entre des emplacements locaux et le cloud à l'aide d’Azure Data Factory](data-factory-move-data-between-onprem-and-cloud.md)


## <a name="current-version"></a>VERSION ACTUELLE 
Nous ne conservons plus les notes de publication ici. Pour obtenir les dernières notes de publication, cliquer [ici](https://go.microsoft.com/fwlink/?linkid=853077)




## <a name="earlier-versions"></a>Versions antérieures
## <a name="21063477"></a>2.10.6347.7
### <a name="enhancements-"></a>Améliorations
- Vous pouvez ajouter des entrées DNS pour autoriser Service Bus plutôt que d’autoriser toutes les adresses IP Azure à partir de votre pare-feu (si nécessaire). Vous pouvez trouver l’entrée DNS concernée sur le portail Azure (Data Factory -> Créer et déployer > Passerelles > ServiceUrls (dans JSON))
- Le connecteur HDFS prend désormais en charge le certificat public auto-signé en vous permettant d’ignorer la validation TLS.
- Résolu : problème lié à la passerelle en mode hors connexion pendant la mise à jour (en raison d’une différence d’heure)


## <a name="2963132"></a>2.9.6313.2
### <a name="enhancements-"></a>Améliorations
-   Vous pouvez ajouter des entrées DNS pour autoriser Service Bus plutôt que d’autoriser toutes les adresses IP Azure à partir de votre pare-feu (si nécessaire). Plus de détails ici.
-   Vous pouvez maintenant copier les données vers ou à partir d’un seul objet blob de blocs jusqu’à 4,75 To. Il s’agit de la taille maximale prise en charge pour ces objets (la limite antérieure était de 195 Go).
-   Résolu : problème de mémoire insuffisante lors de la décompression de plusieurs petits fichiers pendant l’activité de copie.
-   Résolu : problème d’index hors plage durant la copie de Document DB vers SQL Server avec la fonctionnalité d’idempotence.
-   Résolu : le script de nettoyage SQL ne fonctionne pas avec SQL Server à partir de l’Assistant de copie.
-   Résolu : le nom de colonne avec un espace à la fin ne fonctionne pas dans l’activité de copie.

## <a name="28662833"></a>2.8.66283.3
### <a name="enhancements-"></a>Améliorations
- Résolu : problème d’informations d’identification manquantes au redémarrage de l’ordinateur de la passerelle.
- Résolu : problème lié à l’inscription lors de la restauration de la passerelle à l’aide d’un fichier de sauvegarde.


## <a name="2762401"></a>2.7.6240.1
### <a name="enhancements-"></a>Améliorations
- Résolu : lecture incorrecte d’une valeur null décimale à partir d’Oracle comme source.

## <a name="2661922"></a>2.6.6192.2
### <a name="whats-new"></a>Nouveautés
- Les clients peuvent faire part de leurs commentaires sur l’expérience d’inscription à la passerelle.
- Prise en charge d’un nouveau format de compression : ZIP (Deflate)

### <a name="enhancements-"></a>Améliorations
- Amélioration des performances pour le récepteur d’Oracle, source HDFS.
- Correctif de bogue pour la mise à jour automatique de la passerelle, capacité de traitement parallèle de la passerelle.


## <a name="2561641"></a>2.5.6164.1
### <a name="enhancements"></a>Améliorations
- Expérience d’inscription de la passerelle améliorée et plus robuste. Vous pouvez maintenant suivre l’état d’avancement lors du processus d’inscription de la passerelle, ce qui rend l’expérience d’inscription plus réactive.
- Amélioration du processus de restauration de la passerelle. Vous pouvez toujours récupérer la passerelle même si vous n’avez pas le fichier de sauvegarde de passerelle avec cette mise à jour. Cela vous oblige à réinitialiser les informations d’identification du Service lié dans le portail.
- Résolution de bogue.

## <a name="2461511"></a>2.4.6151.1

### <a name="whats-new"></a>Nouveautés

- Vous pouvez désormais stocker localement les informations d’identification de la source de données. Les informations d’identification sont chiffrées. Les informations d’identification de la source de données peuvent être récupérées et restaurées à l’aide du fichier de sauvegarde qui peut être exporté à partir de la passerelle existante, le tout localement.

### <a name="enhancements-"></a>Améliorations

- Expérience d’inscription de la passerelle améliorée et plus robuste.
- Prise en charge la détection automatique de la configuration de la propriété QuoteChar pour le format Texte dans l’Assistant de copie et amélioration la précision globale de la détection de format.

## <a name="2361002"></a>2.3.6100.2

- Prise en charge de la détection automatique firstRowAsHeader et SkipLineCount dans l’Assistant de copie pour les fichiers texte dans le système de fichiers local et HDFS.
- Amélioration de la stabilité de la connexion réseau entre la passerelle et Service Bus
- Résolution de quelques bogues


## <a name="2260721"></a>2.2.6072.1

*  Permet de paramétrer le proxy HTTP pour la passerelle à l’aide du Gestionnaire de configuration de passerelle. Si configurés, Azure Blob, Azure Table, Azure Data Lake et Document DB sont accessibles via le proxy HTTP.
*  Prend en charge la gestion des en-têtes pour TextFormat lors de la copie des données depuis/vers un objet Blob Azure, Azure Data Lake Store, le système de fichiers local ou un système de fichiers HDFS local.
*  Prend en charge la copie des données d’objets blob d’ajouts et de pages, ainsi que des objets blob de blocs déjà pris en charge.
*  Introduit le nouvel état de passerelle **En ligne (limité)** , qui indique que la fonctionnalité principale de la passerelle fonctionne, à l’exception de la prise en charge de l’opération interactive pour l’assistant de copie.
*  Améliore la robustesse de l’inscription de la passerelle avec la clé d’inscription.

## <a name="216040"></a>2.1.6040.

*  Le pilote DB2 est désormais inclus dans le package d’installation de la passerelle. Il est inutile de l’installer séparément.
*  Le pilote DB2 prend désormais en charge z/OS et DB2 pour i (AS/400), ainsi que les plateformes déjà prises en charge (Windows, Unix et Linux).
*  Prend en charge l’utilisation d’Azure Cosmos DB comme source ou destination des banques de données locales
*  Prend en charge la copie de données depuis/vers un stockage d’objet blob à chaud ou à froid, ainsi que le compte de stockage à usage général déjà pris en charge.
*  Permet de vous connecter à SQL Server par le biais de la passerelle avec des droits de connexion à distance.  

## <a name="2060131"></a>2.0.6013.1

*  Vous pouvez sélectionner la langue/culture utilisée par une passerelle lors de l’installation manuelle.

*  Lorsqu’une passerelle ne fonctionne pas comme prévu, vous pouvez choisir d’envoyer les journaux d’activité des sept derniers jours de la passerelle à Microsoft pour faciliter la résolution du problème. Si la passerelle n’est pas connectée au service cloud, vous pouvez choisir d’enregistrer et d’archiver les journaux d’activité de la passerelle.  

*  Améliorations de l’interface utilisateur du gestionnaire de configuration de la passerelle :

    *  État de la passerelle plus visible dans l’onglet Accueil.

    *  Commandes réorganisées et simplifiées.

    *  Vous pouvez copier des données à partir d’un stockage à l’aide de [l’outil de copie sans code](data-factory-copy-data-wizard-tutorial.md). Pour plus d’informations sur cette fonctionnalité, consultez [Copie intermédiaire](data-factory-copy-activity-performance.md#staged-copy) .
*  Vous pouvez utiliser la passerelle de gestion des données pour entrer des données directement à partir d’une base de données SQL Server dans Azure Machine Learning.

*  Optimisation des performances

    * Amélioration des performances d’affichage du schéma/de l’aperçu par rapport à SQL Server dans l’outil de copie sans code.

## <a name="11259531"></a>1.12.5953.1

*  Résolution des bogues

## <a name="11159181"></a>1.11.5918.1

*  La taille maximale du journal des événements de la passerelle a été augmentée de 1 Mo à 40 Mo.

*  Une boîte de dialogue d’avertissement s’affiche si un redémarrage est nécessaire pendant la mise à jour automatique de la passerelle. Vous pouvez choisir de redémarrer immédiatement ou plus tard.

*  En cas d’échec de la mise à jour automatique, le programme d’installation de la passerelle retente la mise à jour automatique trois fois au maximum.

*  Optimisation des performances

    * Améliorez les performances lors du chargement de tables de grande taille à partir du serveur local dans le scénario de copie sans code.

*  Résolution des bogues

## <a name="11058921"></a>1.10.5892.1

*  Optimisation des performances

*  Résolution des bogues

## <a name="1958652"></a>1.9.5865.2

*  Capacité de mise à jour automatique Zero Touch
*  Nouvelle icône de barre d’état système avec des indicateurs d’état de la passerelle
*  Possibilité de mise à jour immédiate depuis le client
*  Possibilité de définir l’heure de planification de la mise à jour
*  Script PowerShell disponible pour activer/désactiver la mise à jour automatique
*  Prise en charge du format JSON  
*  Optimisation des performances
*  Résolution des bogues

## <a name="1858221"></a>1.8.5822.1

*  Amélioration de l’expérience de résolution des problèmes
*  Optimisation des performances
*  Résolution des bogues

### <a name="1757951"></a>1.7.5795.1

*  Optimisation des performances
*  Résolution des bogues

### <a name="1757641"></a>1.7.5764.1

*  Optimisation des performances
*  Résolution des bogues

### <a name="1657351"></a>1.6.5735.1

*  Prise en charge de la source/du récepteur HDFS en local
*  Optimisation des performances
*  Résolution des bogues

### <a name="1656961"></a>1.6.5696.1

*  Optimisation des performances
*  Résolution des bogues

### <a name="1656761"></a>1.6.5676.1

*  Prise en charge des outils de diagnostic dans le Gestionnaire de configuration
*  Prise en charge des colonnes de table pour les sources de données tabulaires pour Microsoft Azure Data Factory
*  Prise en charge d’Azure Synapse Analytics pour Azure Data Factory
*  Prise en charge de Reclusive dans BlobSource et FileSource pour Microsoft Azure Data Factory
*  Prise en charge de CopyBehavior (MergeFiles, PreserveHierarchy et FlattenHierarchy) dans BlobSink et FileSink avec copie binaire pour Microsoft Azure Data Factory
*  Prise en charge du signalement de progression de l’activité de copie pour Microsoft Azure Data Factory
*  Prise en charge de la validation de connectivité des sources de données pour Microsoft Azure Data Factory
*  Résolution des bogues

### <a name="1656721"></a>1.6.5672.1

*  Prise en charge des noms de tables pour la source de données ODBC pour Microsoft Azure Data Factory
*  Optimisation des performances
*  Résolution des bogues

### <a name="1656581"></a>1.6.5658.1

*  Prise en charge du récepteur de fichier pour Microsoft Azure Data Factory
*  Prise en charge de la conservation de la hiérarchie dans une copie binaire pour Microsoft Azure Data Factory
*  Prise en charge de l’idempotence de l’activité de copie pour Microsoft Azure Data Factory
*  Résolution des bogues

### <a name="1656401"></a>1.6.5640.1

*  Prise en charge de 3 sources de données supplémentaires pour Microsoft Azure Data Factory (ODBC, OData, HDFS)
*  Prise en charge du caractère guillemet dans l’analyseur CSV pour Microsoft Azure Data Factory
*  Prise en charge de la compression (BZip2)
*  Résolution des bogues

### <a name="1556121"></a>1.5.5612.1

*  Prise en charge de cinq bases de données relationnelles pour Microsoft Azure Data Factory (MySQL, PostgreSQL, DB2, Teradata et Sybase)
*  Prise en charge de la compression (Gzip et Deflate)
*  Optimisation des performances
*  Résolution des bogues

### <a name="1455491"></a>1.4.5549.1

*  Ajout de la prise en charge de source de données Oracle pour Microsoft Azure Data Factory
*  Optimisation des performances
*  Résolution des bogues

### <a name="1454921"></a>1.4.5492.1

*  Binaire unifié qui prend en charge à la fois des services Microsoft Azure Data Factory et Office 365 Power BI
*  Affinage du processus d’inscription et de l’interface utilisateur de configuration
*  Microsoft Azure Data Factory : prise en charge des entrées et sorties Azure pour la source de données SQL Server

### <a name="1253031"></a>1.2.5303.1

*  Correction du problème de délai d’expiration pour la prise en charge de connexions de source de données chronophages supplémentaires.

### <a name="1155268"></a>1.1.5526.8

*  .NET Framework 4.5.1 est requis pour l’installation.

### <a name="1051442"></a>1.0.5144.2

*  Aucune modification affectant les scénarios Microsoft Azure Data Factory.
