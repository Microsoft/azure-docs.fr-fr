---
title: Historique de publication des versions
titleSuffix: Microsoft Genomics
description: Historique de publication des mises à jour vers le client Python Microsoft Genomics à l’aide de correctifs et de nouvelles fonctionnalités.
services: genomics
author: grhuynh
manager: cgronlun
ms.author: grhuynh
ms.service: genomics
ms.topic: conceptual
ms.date: 01/11/2019
ms.openlocfilehash: 20475e2cde1b42790740889cf341b3a0a0afccc0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76991080"
---
# <a name="version-release-history"></a>Historique de publication des versions
L’équipe Microsoft Genomics met régulièrement à jour le client Python Microsoft Genomics à l’aide de correctifs et de nouvelles fonctionnalités. 

## <a name="latest-release"></a>Dernière version
La version du client Python est la version 0.9.0. Cette version a été publiée le 6 février 2019 et prend en charge les workflows exécutés avec GATK 3.5 et GATK4. Elle prend en charge la sortie gVCF et peut accepter un argument facultatif pour la compression de la sortie.


## <a name="release-history"></a>Historique des mises en production 
De nouvelles versions du client Python Microsoft Genomics sont publiées une fois par an environ. Au fur et à mesure de leur publication, la liste des correctifs et des fonctionnalités est mise à jour ici. Quand de nouvelles versions sont publiées, la prise en charge des versions antérieures est maintenue pendant au moins 90 jours. Les versions antérieures qui ne sont plus prises en charge sont répertoriées sur cette page. 

### <a name="version-090"></a>Version 0.9.0
La version 0.9.0 comprend la prise en charge de la compression de la sortie. Cela équivaut à exécuter `-bgzip` puis `-tabix` dans la sortie vcf ou gvcf. Pour plus d’informations, consultez les [Questions fréquentes (FAQ)](frequently-asked-questions-genomics.md). 

### <a name="version-081"></a>Version 0.8.1
La version 0.8.1 inclut des correctifs mineurs.  

### <a name="version-080"></a>Version 0.8.0
La version 0.8.0 prend en charge GATK4 et les sorties de gVCF.  

### <a name="version-074"></a>Version 0.7.4
La version 0.7.4 inclut la prise en charge des jetons SAS au lieu des clés de compte dans l’entrée `config.txt`. Pour plus d’informations, consultez [Démarrage rapide sur les jetons SAS d’entrée](quickstart-input-sas.md). 

### <a name="version-073"></a>Version 0.7.3
La version 0.7.3 inclut des correctifs mineurs.

### <a name="version-072"></a>Version 0.7.2
La version 0.7.2 est la version initiale. Elle a été publiée le 1er novembre 2017.
