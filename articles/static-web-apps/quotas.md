---
title: Quotas dans Azure Static Web Apps (préversion)
description: En savoir plus sur les quotas associés à Azure Static Web Apps (préversion)
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: overview
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: 915f8675ffda0d70347905d11c7d93975b9d4526
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132735"
---
# <a name="quotas-in-azure-static-web-apps-preview"></a>Quotas dans Azure Static Web Apps (préversion)

Les quotas suivants sont disponibles pour Azure Static Web Apps (préversion).

> [!IMPORTANT]
> Azure Static Web Apps est en préversion publique et n’est pas destiné à une utilisation en environnement de production.

| Fonctionnalité                     | Plan Gratuit        |
|-----------------------------|------------------|
| Bande passante incluse          | 100 Go par mois |
| Dépassement de bande passante           | Non disponible      |
| Applications par abonnement Azure | 10               |
| Taille de l’application                    | 250 Mo           |
| Environnements de pré-production | 1                |
| Domaines personnalisés              | 1                |
| Autorisation<br><br>Avec des rôles personnalisés et des règles d’acheminement | Jusqu’à 25 utilisateurs finaux invités et rôles affectés |
| Azure Functions             | Disponible        |
| Contrat SLA                         | None             |

## <a name="github-storage"></a>Stockage GitHub

Les fonctionnalités GitHub Actions et Packages utilisent le stockage GitHub, qui possède son propre ensemble de quotas. Lorsque vous créez un site Azure Static Web Apps, GitHub stocke les artefacts de votre site, même après le déploiement.

Pour plus d’informations, consultez les ressources suivantes :

- [Gestion de l’espace de stockage de GitHub Actions](https://github.community/t5/GitHub-Actions/Managing-Actions-storage-space/td-p/38944)
- [À propos de la facturation de GitHub Actions](https://help.github.com/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions#about-billing-for-github-actions)
- [Gestion de votre limite de dépense pour GitHub Actions](https://help.github.com/github/setting-up-and-managing-billing-and-payments-on-github/managing-your-spending-limit-for-github-actions)

## <a name="next-steps"></a>Étapes suivantes

- [Vue d'ensemble](overview.md)
