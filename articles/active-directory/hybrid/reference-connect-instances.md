---
title: 'Azure AD Connect : Instances de service Sync | Microsoft Docs'
description: Cette page décrit des considérations spéciales relatives aux instances d’Azure AD.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: f340ea11-8ff5-4ae6-b09d-e939c76355a3
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 05/27/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0c4d8b0a33763a967550453d8a205258f7583084
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90015260"
---
# <a name="azure-ad-connect-special-considerations-for-instances"></a>Azure AD Connect : Considérations spécifiques concernant les instances
Azure AD Connect est couramment utilisé avec l’instance mondiale d’Azure AD et Microsoft 365. Mais il existe également d’autres instances, qui ont des exigences différentes en matière d’URL et autres considérations spéciales.

## <a name="microsoft-cloud-germany"></a>Microsoft Cloud Allemagne
[Microsoft Cloud Allemagne](https://www.microsoft.de/cloud-deutschland) est un cloud souverain géré par un client allemand approuvé dans le domaine des données.

| URL à ouvrir dans le serveur proxy |
| --- |
| \*.microsoftonline.de |
| \*.windows.net |
| \+ Listes de révocation de certificat |

Quand vous vous connectez à votre locataire Azure AD, vous devez utiliser un compte du domaine onmicrosoft.de.

Fonctionnalités actuellement absentes de Microsoft Cloud Allemagne :

* L’**écriture différée de mot de passe** est disponible en préversion avec Azure AD Connect version 1.1.570.0 et ultérieures.
* Les autres services Azure AD Premium ne sont pas disponibles.

## <a name="microsoft-azure-government"></a>Microsoft Azure Government
Le [cloud Microsoft Azure Government](https://azure.microsoft.com/features/gov/) est un cloud du gouvernement des États-Unis.

Ce cloud a été pris en charge par des versions antérieures de DirSync. À partir de la build 1.1.180 d’Azure AD Connect, la nouvelle génération du cloud est prise en charge. Cette génération utilise des points de terminaison États-Unis uniquement et a sa propre liste d’URL à ouvrir dans votre serveur proxy.

| URL à ouvrir dans le serveur proxy |
| --- |
| \*.microsoftonline.com |
| \*.microsoftonline.us |
| \*.windows.net (requis pour la détection automatique d'un locataire Azure Government) |
| \*.gov.us.microsoftonline.com |
| \+ Listes de révocation de certificat |

> [!NOTE]
> À compter d'Azure AD Connect version 1.1.647.0, la définition de la valeur AzureInstance dans le Registre n’est plus nécessaire, à condition que *.windows.net soit ouvert sur vos serveurs proxy. Toutefois, pour les clients n'autorisant pas la connectivité Internet à partir de leurs serveurs Azure AD Connect, la configuration manuelle suivante peut être utilisée.

### <a name="manual-configuration"></a>Configuration manuelle

La procédure de configuration manuelle suivante est utilisée pour veiller à ce qu'Azure AD Connect utilise des points de terminaison de synchronisation Azure Government.

1. Lancez l’installation d’Azure AD Connect.
2. Quand vous voyez apparaître la première page où vous êtes censé accepter le CLUF, ne poursuivez pas, mais laissez l’Assistant Installation en cours d’exécution.
3. Lancez regedit et modifiez la clé de registre de la valeur `HKLM\SOFTWARE\Microsoft\Azure AD Connect\AzureInstance` à la valeur `4`.
4. Revenez à l’Assistant Installation d’Azure AD Connect, acceptez le CLUF et continuez. Pendant l’installation, veillez à utiliser le chemin d’accès d’installation de la **configuration personnalisée** (et non l’installation Express), puis poursuivez l'installation normalement.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
