---
title: Connecter des données CEF à Azure Sentinel en préversion | Microsoft Docs
description: Connectez une solution externe qui envoie des messages CEF (Common Event Format) à Azure Sentinel en utilisant une machine Linux en tant que redirecteur de journaux.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/01/2020
ms.author: yelevin
ms.openlocfilehash: 4b34477a40530a0e6f26b59dd9707c019418b2a4
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655848"
---
# <a name="connect-your-external-solution-using-common-event-format"></a>Connectez votre solution externe en utilisant le format CEF

Lorsque vous connectez une solution externe qui envoie des messages CEF, il existe trois étapes pour la connexion à Azure Sentinel :

ÉTAPE 1 : [Connecter CEF en déployant un redirecteur Syslog/CEF](connect-cef-agent.md) ÉTAPE 2 : [Effectuez les étapes spécifiques à la solution](connect-cef-solution-config.md) ÉTAPE 3 : [Vérifiez la connectivité](connect-cef-verify.md)

Cet article décrit le fonctionnement de la connexion, liste les prérequis et indique les étapes de déploiement d’un mécanisme pour que des solutions de sécurité envoient des messages au format CEF (Common Event format) sur Syslog. 

> [!NOTE] 
> Les données sont stockées dans l’emplacement géographique de l’espace de travail sur lequel vous exécutez Azure Sentinel.

Pour établir cette connexion, vous devez déployer un serveur de redirecteur Syslog pour prendre en charge la communication entre l’appliance et Azure Sentinel.  Le serveur se compose d’une machine Linux dédiée (machine virtuelle ou locale) sur laquelle l’agent Log Analytics pour Linux est installé. 

Le diagramme suivant décrit la configuration dans le cas d’une machine virtuelle Linux dans Azure :

 ![CEF dans Azure](./media/connect-cef/cef-syslog-azure.png)

Cette configuration convient également si vous utilisez une machine virtuelle dans un autre cloud ou sur une machine locale : 

 ![CEF local](./media/connect-cef/cef-syslog-onprem.png)

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Veillez à configurer la sécurité de la machine en fonction de la stratégie de sécurité de votre organisation. Par exemple, vous pouvez configurer votre réseau de sorte qu’il s’accorde à la stratégie de sécurité de votre réseau d’entreprise, et modifier les ports et les protocoles dans le démon pour les adapter à vos besoins. Vous pouvez utiliser les instructions suivantes pour améliorer la configuration de la sécurité de votre machine :  [Sécurisation de la machine virtuelle dans Azure](../virtual-machines/security-policy.md), [Bonnes pratiques pour la sécurité réseau](../security/fundamentals/network-best-practices.md).

Pour utiliser la communication TLS entre la source Syslog et le redirecteur Syslog, vous devez configurer le démon Syslog (rsyslog or syslog-ng) pour qu’il communique dans TLS : [Chiffrement du trafic Syslog avec TLS –rsyslog](https://www.rsyslog.com/doc/v8-stable/tutorials/tls_cert_summary.html), [Chiffrement des messages du journal avec TLS – syslog-ng](https://support.oneidentity.com/technical-documents/syslog-ng-open-source-edition/3.22/administration-guide/60#TOPIC-1209298).
 
## <a name="prerequisites"></a>Prérequis

Assurez-vous que la machine Linux que vous utilisez en tant que redirecteur de journaux exécute l’un des systèmes d’exploitation suivants :

- 64 bits
  - CentOS 7 et 8, y compris les versions mineures (pas la version 6)
  - Amazon Linux 2017.09
  - Oracle Linux 7
  - Red Hat Enterprise Linux (RHEL) Server 7 et 8, y compris les versions mineures (pas la version 6)
  - Debian GNU/Linux 8 et 9
  - Ubuntu Linux 14.04 LTS, 16.04 LTS et 18.04 LTS
  - SUSE Linux Enterprise Server 12 et15

- 32 bits
  - CentOS 7 et 8, y compris les versions mineures (pas la version 6)
  - Oracle Linux 7
  - Red Hat Enterprise Linux (RHEL) Server 7 et 8, y compris les versions mineures (pas la version 6)
  - Debian GNU/Linux 8 et 9
  - Ubuntu Linux 14.04 LTS et 16.04 LTS
 
- Versions du démon
  - Syslog-ng : 2.1 - 3.22.1
  - Rsyslog : v8
  
- RFC Syslog pris en charge
  - RFC Syslog 3164
  - RFC Syslog 5424
 
Assurez-vous que votre machine répond également aux exigences suivantes : 

- Autorisations
  - Vous devez disposer d’autorisations élevées (sudo) sur votre machine. 

- Configuration logicielle requise
  - Vérifiez que Python 2.7 ou 3 est en cours d’exécution sur votre machine.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez découvert la façon dont Azure Sentinel collecte les journaux CEF des appliances et des solutions de sécurité. Pour savoir comment connecter votre solution à Azure Sentinel, consultez les articles suivants :

- ÉTAPE 1 : [Connecter CEF en déployant un redirecteur Syslog/CEF](connect-cef-agent.md)
- ÉTAPE 2 : [Effectuez les étapes spécifiques à la solution](connect-cef-solution-config.md)
- ÉTAPE 3 : [Vérifiez la connectivité](connect-cef-verify.md)

Pour en savoir plus sur la procédure à suivre avec les données que vous avez collectées dans Azure Sentinel, consultez les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).