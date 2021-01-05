---
title: Installer et déployer l’agent Linux C
description: Découvrez comment installer et déployer sur Linux l’agent de sécurité Defender pour IoT basé sur C.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/23/2019
ms.author: mlottner
ms.openlocfilehash: 8c03f6b882c8b1a64c9f256493c5d586b5fa0f89
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930652"
---
# <a name="deploy-defender-for-iot-c-based-security-agent-for-linux"></a>Déployer l’agent de sécurité Defender pour IoT basé sur C pour Linux

Ce guide explique comment installer et déployer sur Linux l’agent de sécurité Defender pour IoT basé sur C.

Dans ce guide, vous apprendrez comment :

> [!div class="checklist"]
> * Installer
> * Vérifier le déploiement
> * Désinstaller l’agent
> * Dépanner

## <a name="prerequisites"></a>Prérequis

Pour d’autres plateformes et versions de l’agent, consultez [Choisir l’agent de sécurité adéquat](how-to-deploy-agent.md).

1. Pour déployer l’agent de sécurité, les droits d’administrateur local sont requis sur l’ordinateur sur lequel vous souhaitez effectuer l’installation (sudo).

1. [Créez un module de sécurité](quickstart-create-security-twin.md) pour l’appareil.

## <a name="installation"></a>Installation

Pour installer et déployer l’agent de sécurité, procédez comme suit :

1. Téléchargez la version la plus récente sur votre machine, depuis [GitHub](https://aka.ms/iot-security-github-c).

1. Extrayez le contenu du package et accédez au dossier _/src/installation_.

1. Ajoutez des autorisations en cours d’exécution au **script InstallSecurityAgent** en exécutant la commande suivante :

   ```
   chmod +x InstallSecurityAgent.sh
   ```

1. Ensuite, exécutez la commande suivante :

   ```
   ./InstallSecurityAgent.sh -aui <authentication identity> -aum <authentication method> -f <file path> -hn <host name> -di <device id> -i
   ```

   Pour plus d’informations sur les paramètres d’authentification, consultez [Guide pratique pour configurer l’authentification](concept-security-agent-authentication-methods.md).

Le script effectue l’opération suivante :

1. Installation des composants requis.

1. Ajout d’un utilisateur de service (avec connexion interactive désactivée).

1. Installation de l’agent en tant que **démon**, ce qui suppose que l’appareil utilise **systemd** pour la gestion des services.

1. Configuration de l’agent avec les paramètres d’authentification fournis.

Pour obtenir de l’aide, exécutez le script avec le paramètre –help :

```./InstallSecurityAgent.sh --help```

### <a name="uninstall-the-agent"></a>Désinstaller l’agent

Pour désinstaller l’agent, exécutez le script avec le paramètre –-uninstall :

```./InstallSecurityAgent.sh -–uninstall```

## <a name="troubleshooting"></a>Dépannage

Vérifiez l’état du déploiement en exécutant la commande suivante :

```systemctl status ASCIoTAgent.service```

## <a name="next-steps"></a>Étapes suivantes

- Lire la [vue d’ensemble](overview.md) du service Defender pour IoT
- Découvrir plus en détail l’[architecture](architecture.md) de Defender pour IoT
- Activer le [service](quickstart-onboard-iot-hub.md)
- Consulter les [Questions fréquentes (FAQ)](resources-frequently-asked-questions.md)
- Comprendre les [alertes de sécurité](concept-security-alerts.md)
