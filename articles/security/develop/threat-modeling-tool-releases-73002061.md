---
title: Disponibilité générale de Microsoft Threat Modeling Tool version du 02/11/2020 – Azure
description: Compilation des notes de publication pour l’outil de modélisation des menaces version 7.3.00206.1.
author: jegeib
ms.author: jegeib
ms.service: security
ms.topic: article
ms.date: 02/25/2020
ms.openlocfilehash: 9f9b162460cd2e7a624c1ad3f992011487d1b795
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94516932"
---
# <a name="threat-modeling-tool-update-release-73002061---02112020"></a>Version mise à jour de Threat Modeling Tool 7.3.00206.1 – 02/11/2020

La version 7.3.00206.1 de Microsoft Threat Modeling Tool (TMT) a été publiée le 11 février 2020 et contient les modifications suivantes :

- Résolution des bogues

## <a name="notable-bug-fixes"></a>Correctifs de bogues notables

### <a name="errors-related-to-priority-values-outside-of-the-expected-ranges"></a>Erreurs liées aux valeurs de priorité en dehors des plages attendues

Certains clients ont signalé la réception du message d’erreur suivant lors de l’ouverture des fichiers créés dans « Threat Modeling Tool 2016 » ou des modèles personnalisés :

```output
System.InvalidOperationException: Invalid Priority value. Accepted values are [0..4] and 'High', 'Medium', 'Low' at ThreatModeling.Model.Threat.get_Priority()

System.ArgumentOutOfRangeException: Accepted values are 'High', 'Medium', and 'Low' Parameter name: value Actual value was 5.6. at ThreatModeling.Model.Threat.set_Priority(String value)
```

Ce problème a été résolu dans cette version.

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