---
title: Disponibilité générale de Microsoft Threat Modeling Tool version du 29/07/2020 – Azure
description: Compilation des notes de publication pour l’outil de modélisation des menaces version 7.3.00729.1.
author: jegeib
ms.author: jegeib
ms.service: security
ms.topic: article
ms.date: 07/29/2020
ms.openlocfilehash: 3e6fcd52ad9cb6c127c14bac2f33223fb921519e
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94516371"
---
# <a name="threat-modeling-tool-update-release-73007291---07292020"></a>Version mise à jour de Threat Modeling Tool 7.3.00729.1 – 29/07/2020

La version 7.3.00729.1 de Microsoft Threat Modeling Tool (TMT) a été publiée le 29 juillet 2020 et contient les modifications suivantes :

- Résolution des bogues
 
## <a name="known-issues"></a>Problèmes connus

### <a name="errors-related-to-tmt7application-file-deserialization"></a>Erreurs liées à la désérialisation du fichier TMT7. application

#### <a name="issue"></a>Problème

Certains clients ont signalé la réception du message d’erreur suivant lors du téléchargement de Threat Modeling Tool.

```
The threat model file '$PATH\TMT7.application' could not be deserialized. File is not an actual threat model or the threat model may be corrupted.
```

Cette erreur se produit parce que certains navigateurs ne prennent pas en charge nativement l’installation ClickOnce. Dans ces cas, le fichier d’application ClickOnce est téléchargé sur le disque dur de l’utilisateur.

#### <a name="workaround"></a>Solution de contournement

Cette erreur continue à s’afficher si le Threat Modeling Tool est lancé en double-cliquant sur le fichier TMT7. application. Toutefois, une fois l’erreur ignorée, l’outil fonctionnera normalement. Pour démarrer le Threat Modeling Tool, les utilisateurs doivent utiliser les raccourcis créés dans le menu Windows lors de l’installation au lieu de lancer le Threat Modeling Tool en double-cliquant sur le fichier TMT7.application.

## <a name="system-requirements"></a>Configuration système requise

- Systèmes d’exploitation pris en charge
  - [Mise à jour anniversaire Windows 10](https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#HTkoK5Zdv0g2F2Zq.97) ou version ultérieure
- Version .NET obligatoire
  - [.NET 4.7.1](https://go.microsoft.com/fwlink/?LinkId=863262) ou version ultérieure
- Autres conditions requises
  - Une connexion Internet est nécessaire pour recevoir les mises à jour de l’outil ainsi que les modèles.

## <a name="documentation-and-feedback"></a>Documentation et commentaires

- La documentation de Threat Modeling Tool se trouve sur [docs.microsoft.com](./threat-modeling-tool.md) et contient des informations [sur l’utilisation de l’outil](./threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Étapes suivantes

Télécharger la version la plus récente de [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).