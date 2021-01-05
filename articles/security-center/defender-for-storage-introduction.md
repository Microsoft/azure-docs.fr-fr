---
title: Azure Defender pour le stockage - avantages et fonctionnalités
description: Découvrez les avantages et les fonctionnalités d’Azure Defender pour le stockage.
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 8979e315f188a5c21cce206c24f195f72096d438
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516511"
---
# <a name="introduction-to-azure-defender-for-storage"></a>Présentation d’Azure Defender pour le stockage


**Azure Defender pour Stockage** est une couche de sécurité intelligente native Azure qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos comptes de stockage. Il utilise les fonctionnalités avancées de l’IA de sécurité et de [Microsoft Threat Intelligence](https://go.microsoft.com/fwlink/?linkid=2128684) pour fournir des recommandations et des alertes de sécurité contextuelles.

Les alertes de sécurité sont déclenchées lorsque des anomalies se produisent dans l’activité. Ces alertes sont intégrées à Azure Security Center et envoyées par e-mail aux administrateurs d’abonnement avec les détails des activités suspectes et des recommandations sur la façon d’examiner et de corriger les menaces.


## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|Disponibilité générale (GA)|
|Prix :|**Azure Defender pour le stockage** est facturé comme indiqué sur [la page de tarification](security-center-pricing.md)|
|Types de stockage protégés :|[Stockage Blob](https://azure.microsoft.com/services/storage/blobs/)<br>[Azure Files](../storage/files/storage-files-introduction.md)<br>[Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Oui](./media/icons/yes-icon.png) Gouvernement des États-Unis<br>![Non](./media/icons/no-icon.png) Chine Gov, autres Gov|
|||


## <a name="what-kind-of-alerts-does-azure-defender-for-storage-provide"></a>Quels sont les types d’alertes fournis par Azure Defender pour le stockage ?

Les alertes de sécurité sont déclenchées dans les cas suivants :

- **Modèles d’accès suspects** – tels qu’un accès réussi à partir d’un nœud de sortie Tor ou d’une adresse IP considérée comme suspecte par Microsoft Threat Intelligence
- **Activités suspectes** – telles que l’extraction de données anormales ou un changement inhabituel d’autorisations d’accès
- **Téléchargement de contenu malveillant** – tel que des fichiers de logiciels malveillants potentiels (basés sur l’analyse de réputation du code de hachage) ou l’hébergement de contenu d’hameçonnage

Les alertes fournissent des détails sur l’incident qui les a déclenchées, ainsi que des suggestions pour enquêter et contrer les menaces. Les alertes peuvent être exportées vers Azure Sentinel ou toute autre solution SIEM tierce ou tout autre outil externe.

> [!TIP]
> Une bonne pratique consiste à [configurer Azure Defender pour Stockage](https://docs.microsoft.com/azure/storage/common/azure-defender-storage-configure?tabs=azure-security-center) au niveau de l’abonnement, mais vous pouvez également [le configurer sur des comptes de stockage individuels](https://docs.microsoft.com/azure/storage/common/azure-defender-storage-configure?tabs=azure-portal).


## <a name="what-is-hash-reputation-analysis-for-malware"></a>Qu’est-ce que l’analyse de réputation du code de hachage pour les programmes malveillants ?

Pour déterminer si un fichier téléchargé est suspect, Azure Defender pour le stockage utilise l’analyse de réputation de hachage prise en charge par [Microsoft Threat Intelligence](https://go.microsoft.com/fwlink/?linkid=2128684). Les outils de protection contre les menaces n’analysent pas les fichiers téléchargés, mais ils examinent les journaux de stockage et comparent le code de hachage des fichiers récemment téléchargés avec ceux des virus, chevaux de Troie, logiciels espions et ransomware connus. 

Lorsqu’un fichier est supposé contenir un logiciel malveillant, Security Center affiche une alerte et peut éventuellement envoyer un e-mail au propriétaire du stockage pour approbation afin de supprimer le fichier suspect. Pour configurer cette suppression automatique des fichiers signalée par l’analyse du code de hachage, déployez une [automatisation du flux de travail pour déclencher des alertes « Logiciel malveillant potentiel chargé sur un compte de stockage »](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-respond-to-potential-malware-uploaded-to-azure-storage/ba-p/1452005).

> [!NOTE]
> Pour activer les fonctionnalités de protection contre les menaces de Security Center, vous devez activer Azure Defender sur l’abonnement contenant les charges de travail applicables.
>
> Vous pouvez activer **Azure Defender pour le stockage** au niveau de l’abonnement ou de la ressource.



## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à propos d’Azure Defender pour le stockage.

Pour des informations connexes, consultez les articles suivants : 

- Qu’une alerte soit générée par Security Center ou reçue par Security Center à partir d’un autre produit de sécurité, vous pouvez l’exporter. Pour exporter vos alertes vers Azure Sentinel (ou un système SIEM tiers) ou tout autre outil externe, suivez les instructions indiquées dans [Exportation d’alertes vers SIEM](continuous-export.md).
- [Comment activer Advanced Defender pour le stockage](../storage/common/azure-defender-storage-configure.md)
- [Liste des alertes Azure Defender pour le stockage](alerts-reference.md#alerts-azurestorage)
- [Fonctionnalités de renseignement sur les menaces de Microsoft](https://go.microsoft.com/fwlink/?linkid=2128684)
