---
title: Notes de publication du Kit de développement logiciel (SDK) Azure pour .NET 2.9
description: Notes de publication du Kit de développement logiciel (SDK) Azure pour .NET 2.9
services: app-service\web
documentationcenter: .net
author: chrissfanos
editor: ''
ms.assetid: c83d815b-fc19-4260-821e-7d2a7206dffc
ms.service: app-service
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 02/24/2017
ms.author: juliako
ms.openlocfilehash: 30013c0bd8ded31ccaf3cdc142bbf8dc6b375031
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/28/2018
---
# <a name="azure-sdk-for-net-29-release-notes"></a>Notes de publication du Kit de développement logiciel (SDK) Azure pour .NET 2.9

Cette rubrique comprend des notes de publication relatives aux versions 2.9 et 2.9.6 du Kit SDK Azure pour .NET.

## <a name="azure-sdk-for-net-296-release-summary"></a>Synthèse de publication du Kit SDK Azure pour .NET 2.9.6

Date de sortie : 16/11/2016
 
Dans cette version, aucune modification importante n’a été apportée au Kit SDK Azure 2.9. Par ailleurs, aucun processus de mise à niveau n’est requis pour tirer parti de ce SDK avec des projets existants de service cloud.

### <a name="visual-studio-2017-release-candidate"></a>Version finale de Visual Studio 2017

- Dans la version finale de Visual Studio 2017, cette version du Kit SDK pour .NET est intégrée dans la charge de travail Azure. Tous les outils requis pour le développement Azure équipent l’ensemble des versions de Visual Studio, à compter de la version finale 2017. For Visual Studio 2015 et Visual Studio 2013, le SDK est également disponible via WebPI. Nous arrêterons d’équiper les versions du Kit SDK Azure pour .NET pour Visual Studio 2013 au moment du lancement de la version finale de Visual Studio 2017. Suivez ce lien pour télécharger Visual Studio 2017 RC : https://www.visualstudio.com/vs/visual-studio-2017-rc/

### <a name="azure-diagnostics"></a>Azure Diagnostics

- Le comportement a été modifié pour stocker uniquement une chaîne de connexion partielle, la clé étant remplacée par un jeton de chaîne de connexion de stockage de diagnostics de services cloud. La véritable clé de stockage est désormais stockée dans le dossier de profil utilisateur, ce qui permet de contrôler son accès. Visual Studio lit la clé de stockage à partir du dossier de profil utilisateur, dans le cadre du processus local de débogage et de publication. 
- En réaction à la modification décrite ci-dessus, l’équipe Visual Studio Online a amélioré le modèle des tâches de déploiement des services cloud Azure, de manière à donner les moyens aux utilisateurs de spécifier la clé de stockage dédiée à la définition de l’extension de diagnostics lors de la publication sur Azure, durant l’intégration et le déploiement continus.
- Nous avons rendu possible le stockage d’une chaîne sécurisée de connexion et la création de jetons pour Azure Diagnostics, pour vous aider à résoudre les problèmes de configuration au sein des différents environnements.
 
### <a name="windows-server-2016-virtual-machines"></a>Machines virtuelles Windows Server 2016

- Visual Studio prend désormais en charge le déploiement de services Cloud sur les machines virtuelles de famille 5 du système d’exploitation (Windows Server 2016). Pour les services cloud existants, vous pouvez modifier vos paramètres de manière à cibler la nouvelle famille du système d’exploitation. Lors de la création de nouveaux services cloud, si vous décidez de créer le service à l’aide de .net 4.6 ou d’une version antérieure, le service utilisera par défaut la famille 5 du système d’exploitation.  Pour plus d’informations, passez en revue le [tableau de prise en charge de la famille de SE invité](https://azure.microsoft.com/documentation/articles/cloud-services-guestos-update-matrix/).

#### <a name="known-issues"></a>Problèmes connus

- Azure .NET SDK 2.9.6 a introduit une restriction qui bloque le déploiement des projets qui utilisent des infrastructures .NET non prises en charge (par exemple .NET 4.6) vers n’importe quel système d’exploitation < 5. Une solution de contournement est fournie [ici](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Azure%20Targets%20SDK%202.9).

 
### <a name="azure-in-role-cache"></a>In-Role Cache Azure 

- La prise en charge d’Azure In-Role Cache prend fin le 30 novembre 2016. Pour plus de détails, cliquez [ici](https://azure.microsoft.com/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/).

### <a name="azure-resource-manager-templates-for-azure-stack"></a>Modèles Azure Resource Manager pour Azure Stack

- Nous avons introduit Azure Stack en tant que cible de déploiement pour vos modèles Azure Resource Manager.


## <a name="azure-sdk-for-net-29-summary"></a>Synthèse du Kit SDK Azure pour .NET 2.9

## <a name="overview"></a>Vue d'ensemble
Ce document contient les notes de publication du Kit de développement logiciel (SDK) Azure pour .NET version 2.9. 

Pour plus d’informations sur les mises à jour de cette version, voir le [billet d’annonce du Kit de développement logiciel (SDK) Azure 2.9](https://azure.microsoft.com/blog/announcing-visual-studio-azure-tools-and-sdk-2-9/).

## <a name="azure-sdk-29-for-visual-studio-2015-update-2-and-visual-studio-15-preview"></a>Aperçu du Kit de développement logiciel (SDK) 2.9 pour Visual Studio 2015 Update 2 et Visual Studio "15"
Cette mise à jour inclut les correctifs suivants :

* Problème lié à REST API Client Generation dans lequel la chaîne "Unknown Type" s'affiche en tant que le nom du dossier de la génération du code et/ou le nom de l'espace de noms est déposé dans le code généré.
* Problème lié à Scheduled WebJobs dans lequel les informations d'authentification n'ont pas pu être transmises au processus d'approvisionnement de Scheduler.

Cette mise à jour inclut les nouvelles fonctionnalités suivantes :

* Prise en charge de services d'application secondaires dans l'onglet "Services" de la boîte de dialogue d'approvisionnement App Service. 

## <a name="azure-data-lake-tools-for-visual-studio-2015-update-2"></a>Outils Azure Data Lake pour Visual Studio 2015 Update 2
Les mises à jour incluent les éléments suivants :

* **Outils Azure Data Lake** pour Visual Studio sont maintenant fusionnés dans la version Azure SDK pour .NET. L'outil est automatiquement installé lorsque vous installez le Kit de développement logiciel (SDK) Azure. 
  
    L'outil étant fréquemment mis à jour, cliquez [ici](http://aka.ms/datalaketool) pour obtenir les mises à jour.
* **Explorateur de serveurs** vous permet désormais de tout afficher et de créer des entités de métadonnées U-SQL. Pour plus d’informations, consultez [ce](https://azure.microsoft.com/documentation/services/data-lake-analytics/) blog.

## <a name="hdinsight-tools"></a>Outils HDInsight
**Outils HDInsight** pour Visual Studio prennent maintenant en charge HDInsight version 3.3, y compris l'affichage des graphiques Tez et d'autres correctifs de langage.

## <a name="azure-resource-manager"></a>Azure Resource Manager
Cette version ajoute la prise en charge de [KeyVault](../azure-resource-manager/resource-manager-keyvault-parameter.md) pour les modèles Resource Manager.

## <a name="see-also"></a>Voir aussi
[Billet d’annonce du Kit de développement logiciel (SDK) Azure 2.9](https://azure.microsoft.com/blog/announcing-visual-studio-azure-tools-and-sdk-2-9/)

