---
title: Disponibilité générale de Microsoft Threat Modeling Tool version du 09/04/2019
titleSuffix: Azure
description: Compilation des notes de publication pour l’outil de modélisation des menaces version 7.1.60408.1.
author: jegeib
ms.author: jegeib
ms.service: security
ms.subservice: security-develop
ms.topic: article
ms.date: 04/03/2019
ms.openlocfilehash: 69dd2c6cdba41779849b4eb6b889cde9b1d6e5c9
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913566"
---
# <a name="threat-modeling-tool-update-release-71604081---492019"></a>Mise à jour de Threat Modeling Tool version 7.1.60408.1 - 09/04/2019

La version 7.1.60408.1 de Microsoft Threat Modeling Tool (TMT) a été publiée le 9 avril 2019 et contient les modifications suivantes :

- Nouveaux gabarits pour Azure Key Vault et Azure Traffic Manager
- Le numéro de version TMT est maintenant affiché sur l’écran d’accueil
- Les liens de support ont été mis à jour
- Résolution des bogues

## <a name="feature-changes"></a>Évolution des fonctionnalités

### <a name="new-stencils-for-azure-key-vault-and-azure-traffic-manager"></a>Nouveaux gabarits pour Azure Key Vault et Azure Traffic Manager

![Capture d’écran des icônes Azure Key Vault et Azure Traffic Manager](./media/threat-modeling-tool-releases-71604081/tmt_keyvault_trafficmanager.PNG)

De nouveaux gabarits et de nouvelles menaces pour Azure Key Vault et Azure Traffic Manager ont été ajoutés à l’ensemble de gabarits Azure. Lors de l’ouverture de modèles basés sur l’ensemble de gabarits Azure, les utilisateurs seront invités à mettre à jour le gabarit associé au modèle. La mise à jour d’un modèle basé sur l’ensemble de gabarits Azure peut également être initiée manuellement à l’aide de la commande « Appliquer le modèle » dans le menu « Fichier » et en appliquant à nouveau le dernier fichier Services.tb7 d’Azure Cloud.

### <a name="tmt-version-number-is-now-shown-on-the-home-screen"></a>Le numéro de version TMT est maintenant affiché sur l’écran d’accueil

La version du client de Threat Modeling Tool est maintenant affichée sur l’écran d’accueil de l’application pour faciliter l’accès.

![Capture d’écran de Microsoft Threat Modeling Tool avec le numéro de version du client](./media/threat-modeling-tool-releases-71604081/tmt_version.PNG)

### <a name="support-links-have-been-updated"></a>Les liens de support ont été mis à jour

Tous les liens de support dans l’outil ont été mis à jour pour diriger les utilisateurs vers [tmtextsupport@microsoft.com](mailto:tmtextsupport@microsoft.com) au lieu d’un forum MSDN.

## <a name="system-requirements"></a>Configuration système requise

- Systèmes d’exploitation pris en charge
  - [Mise à jour anniversaire Windows 10](https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#HTkoK5Zdv0g2F2Zq.97) ou version ultérieure
- Version .NET obligatoire
  - [.Net 4.7.1](https://go.microsoft.com/fwlink/?LinkId=863262) ou version ultérieure
- Autres conditions requises
  - Une connexion Internet est nécessaire pour recevoir les mises à jour de l’outil ainsi que les modèles.

## <a name="documentation-and-feedback"></a>Documentation et commentaires

- La documentation de Threat Modeling Tool se trouve sur [docs.microsoft.com](threat-modeling-tool.md) et contient des informations [sur l’utilisation de l’outil](threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Étapes suivantes

Télécharger la version la plus récente de [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).
