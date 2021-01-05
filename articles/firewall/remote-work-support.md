---
title: Prise en charge du travail à distance par le pare-feu Azure
description: Cet article explique comment le pare-feu Azure peut prendre en charge vos exigences de travail à distance.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: victorh
ms.openlocfilehash: 3c0e2033ee559af38a6816bdfa611eea86b14dea
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658313"
---
# <a name="azure-firewall-remote-work-support"></a>Prise en charge du travail à distance par le pare-feu Azure

Le Pare-feu Azure est un service de sécurité réseau informatique managé qui protège vos ressources Réseau virtuel Azure. Il s’agit d’un service de pare-feu avec état intégral, doté d’une haute disponibilité intégrée et d’une scalabilité illimitée dans le cloud.

## <a name="virtual-desktop-infrastructure-vdi-deployment-support"></a>Prise en charge du déploiement VDI (Virtual Desktop Infrastructure)

Les politiques de télétravail obligent de nombreuses organisations informatiques à faire face à des changements fondamentaux en matière de capacité, de réseau, de sécurité et de gouvernance. Les employés ne sont pas protégés par les stratégies de sécurité en couche associées aux services locaux lorsqu’ils travaillent depuis leur domicile. Les déploiements VDI (Virtual Desktop Infrastructure) sur Azure peuvent aider les organisations à répondre rapidement à ce changement d’environnement. Toutefois, vous avez besoin d’un moyen de protéger l’accès Internet entrant/sortant vers et depuis ces déploiements VDI. Vous pouvez utiliser les [règles DNAT](rule-processing.md) du Pare-feu Azure avec ses capacités de filtrage basées sur [le renseignement sur les menaces](threat-intel.md) pour protéger vos déploiements VDI.

## <a name="azure-windows-virtual-desktop-support"></a>Prise en charge d’Azure Windows Virtual Desktop

Windows Virtual Desktop est un service complet de virtualisation de bureau et d’application s’exécutant dans Azure. Il s’agit de la seule infrastructure de bureau virtuel (VDI,Virtual Desktop Infrastructure) qui offre une gestion simplifiée, la fonctionnalité multisession de Windows 10, des optimisations des applications Microsoft 365 Entreprise et la prise en charge des environnements de Services Bureau à distance (RDS, Remote Desktop Services). Vous pouvez déployer et mettre à l’échelle vos applications et postes de travail Windows sur Azure en quelques minutes et bénéficier de fonctionnalités de sécurité et de conformité intégrées. Windows Virtual Desktop ne vous oblige pas à ouvrir un accès entrant à votre réseau virtuel. Toutefois, vous devez autoriser un ensemble de connexions réseau sortantes pour les machines virtuelles Windows Virtual Desktop qui s’exécutent dans votre réseau virtuel. Pour plus d’informations, consultez [Utiliser le pare-feu Azure pour protéger les déploiements de Windows Virtual Desktop](protect-windows-virtual-desktop.md).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur [Windows Virtual Desktop](../virtual-desktop/index.yml).