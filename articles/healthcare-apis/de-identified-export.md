---
title: Exportation de données anonymisées (préversion) pour API Azure pour FHIR
description: Cet article explique comment configurer et utiliser une exportation anonymisée
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 9/28/2020
ms.author: matjazl
ms.openlocfilehash: bdbab0e032764d07119402686051d391376cb913
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91843593"
---
# <a name="exporting-de-identified-data-preview"></a>Exportation de données anonymisées (préversion)

> [!Note] 
> Les résultats obtenus lors de l’utilisation de l’exportation anonymisée varient en fonction de facteurs tels que les données entrées et les fonctions sélectionnées par le client. Microsoft est dans l’impossibilité d’évaluer les résultats d’une exportation anonymisée ou de déterminer l’acceptabilité pour les cas d’utilisation et les besoins de conformité du client. Aucune garantie n’est fournie quant au respect par l’exportation anonymisée d’exigences légales, réglementaires ou de conformité spécifiques.

La commande $export peut également être utilisée pour exporter des données anonymisées à partir du serveur FHIR. Elle utilise le moteur d’anonymisation des [outils FHIR pour l’anonymisation](https://github.com/microsoft/FHIR-Tools-for-Anonymization), et accepte les détails de configuration de l’anonymisation dans les paramètres de requête. Vous pouvez créer votre propre fichier de configuration de l’anonymisation ou utiliser l’[exemple de fichier de configuration](https://github.com/microsoft/FHIR-Tools-for-Anonymization#sample-configuration-file-for-hipaa-safe-harbor-method) pour la méthode HIPAA Safe Harbor comme point de départ. 

 `https://<<FHIR service base URL>>/$export?_container=<<container_name>>&_anonymizationConfig=<<config file name>>&_anonymizationConfigEtag=<<ETag on storage>>`

|Paramètre de requête.            | Exemple |Caractère facultatif| Description|
|---------------------------|---------|-----------|------------|
| _\_anonymizationConfig_   |DemoConfig.json|Requis pour l’exportation anonymisée |Nom du fichier de configuration. Consultez le format du fichier de configuration [ici](https://github.com/microsoft/FHIR-Tools-for-Anonymization#configuration-file-format). Ce fichier doit être conservé à l’intérieur d’un conteneur nommé **anonymisation** au sein du compte de stockage Azure configuré comme emplacement d’exportation. |
| _\_anonymizationConfigEtag_|"0x8D8494A069489EC"|Facultatif pour l’exportation anonymisée|Il s’agit de l’ETag du fichier de configuration. Vous pouvez récupérer l’ETag à l’aide de l’explorateur de stockage Azure à partir de la propriété blob|

> [!IMPORTANT]
> L’exportation brute et l’exportation anonymisée écrivent toutes deux dans le compte de stockage Azure spécifié dans le cadre de la configuration de l’exportation. Il est recommandé d’utiliser différents conteneurs correspondant à différentes configurations anonymisées, et de gérer l’accès utilisateur au niveau du conteneur.