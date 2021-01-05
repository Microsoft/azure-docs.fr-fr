---
title: Disponibilité générale de Microsoft Threat Modeling Tool version 16/10/2019 - Azure
description: Compilation des notes de publication pour l’outil de modélisation des menaces version 7.1.61015.1.
author: jegeib
ms.author: jegeib
ms.service: security
ms.topic: article
ms.date: 10/16/2019
ms.openlocfilehash: 761ad45496e799a555b60480ff575b9d8f30e984
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94516983"
---
# <a name="threat-modeling-tool-update-release-71610151---10162019"></a>Version mise à jour de Threat Modeling Tool 7.1.61015.1 - 16 octobre 2019

La version 7.1.61015.1 de Microsoft Threat Modeling Tool (TMT) a été mise en production le 16 octobre 2019 et contient les modifications suivantes :

- Améliorations de l’accessibilité
- Résolution des bogues
- De nouveaux gabarits pour Azure Logic Apps et Azure Data Explorer

## <a name="notable-bug-fixes"></a>Correctifs de bogues notables

### <a name="improved-backward-compatibility-with-files-created-in-threat-modeling-tool-2016"></a>Amélioration de la compatibilité descendante avec les fichiers créés dans « Threat Modeling Tool 2016 »

Plusieurs bogues liés à l’ouverture ou à l’affichage des fichiers de modèle de menace créés dans « Threat Modeling Tool 2016 » ont été résolus.

## <a name="feature-enhancements"></a>Améliorations de la fonctionnalité

### <a name="new-stencils-for-azure-logic-apps-and-azure-data-explorer"></a>De nouveaux gabarits pour Azure Logic Apps et Azure Data Explorer

De nouveaux gabarits pour Azure Logic Apps et Azure Data Explorer ont été ajoutés au gabarit Azure avec les menaces et les atténuations associées.

![Gabarits pour Azure Logic Apps et Azure Data Explorer](./media/threat-modeling-tool-releases-71610151/tmt-logic-apps.png)

## <a name="known-issues"></a>Problèmes connus

### <a name="errors-related-to-priority-values-outside-of-the-expected-ranges"></a>Erreurs liées aux valeurs de priorité en dehors des plages attendues

Certains clients ont signalé la réception du message d’erreur suivant lors de l’ouverture des fichiers créés dans le fichier « Threat Modeling Tool 2016 » ou des modèles personnalisés :

```output
System.InvalidOperationException: Invalid Priority value. Accepted values are [0..4] and 'High', 'Medium', 'Low' at ThreatModeling.Model.Threat.get_Priority()

System.ArgumentOutOfRangeException: Accepted values are 'High', 'Medium', and 'Low' Parameter name: value Actual value was 5.6. at ThreatModeling.Model.Threat.set_Priority(String value)
```

Ce problème est en cours de résolution

## <a name="system-requirements"></a>Configuration système requise

- Systèmes d’exploitation pris en charge
  - [Mise à jour anniversaire Windows 10](https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#HTkoK5Zdv0g2F2Zq.97) ou version ultérieure
- Version .NET obligatoire
  - [.Net 4.7.1](https://go.microsoft.com/fwlink/?LinkId=863262) ou version ultérieure
- Autres conditions requises
  - Une connexion Internet est nécessaire pour recevoir les mises à jour de l’outil ainsi que les modèles.

## <a name="documentation-and-feedback"></a>Documentation et commentaires

- La documentation de Threat Modeling Tool se trouve sur [docs.microsoft.com](./threat-modeling-tool.md) et contient des informations [sur l’utilisation de l’outil](./threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Étapes suivantes

Télécharger la version la plus récente de [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).