---
ms.openlocfilehash: f031843243e64515549eaa4e850efcddca58d1f2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/29/2021
ms.locfileid: "98633473"
---
* Compte Azure avec un abonnement actif. Si vous n’en avez pas déjà un, [créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
  > [!NOTE]
  > Vous aurez besoin d’un abonnement Azure avec des autorisations pour créer des principaux de service (le **rôle propriétaire** permet d’effectuer cette opération). Si vous ne disposez pas des autorisations appropriées, contactez l’administrateur de votre compte pour qu’il vous les accorde. 
* [Visual Studio Code](https://code.visualstudio.com/), avec les extensions suivantes :
    * [Outils IoT Azure](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
        > [!TIP]
        > Lors de l’installation d’Azure IoT Tools, vous pouvez être invité à installer Docker. N’hésitez pas à ignorer cette invite.
    * [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
* [Python 3](https://www.python.org/downloads/) (3.6.9 ou version ultérieure), [Pip 3](https://pip.pypa.io/en/stable/installing/) et éventuellement [venv](https://docs.python.org/3/library/venv.html).
* Si vous n’avez pas suivi le guide de démarrage rapide [Détecter les événements de mouvement et d’émission](../../../detect-motion-emit-events-quickstart.md), effectuez les étapes suivantes :
     1. [Configurer des ressources Azure](../../../detect-motion-emit-events-quickstart.md#set-up-azure-resources)
     1. [Configurer votre environnement de développement](../../../detect-motion-emit-events-quickstart.md#set-up-your-development-environment)
     1. [Générer et déployer le manifeste de déploiement IoT Edge](../../../detect-motion-emit-events-quickstart.md#generate-and-deploy-the-deployment-manifest)
     1. [Préparer la supervision d’événements](../../../detect-motion-emit-events-quickstart.md#prepare-to-monitor-events)

> [!TIP]
> En cas de problèmes avec les ressources Azure créées, consultez notre **[guide de dépannage](../../../troubleshoot-how-to.md#common-error-resolutions)** qui couvre les problèmes couramment rencontrés.
