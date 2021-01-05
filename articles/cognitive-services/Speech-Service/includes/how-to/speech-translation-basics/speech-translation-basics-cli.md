---
author: v-demjoh
ms.service: cognitive-services
ms.topic: include
ms.date: 04/13/2020
ms.author: v-demjoh
ms.openlocfilehash: a5796dfb066ea8ac7c59c02b29ef67b184c67edb
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94425037"
---
L’une des principales fonctionnalités du service Speech est la possibilité de reconnaître la parole humaine et de la traduire dans d’autres langues. Dans ce démarrage rapide, vous allez apprendre à utiliser le Kit de développement logiciel (SDK) Speech dans vos applications et produits afin d’effectuer une traduction vocale de qualité. Ce guide de démarrage rapide convertit la parole du microphone en texte dans une autre langue.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

[!INCLUDE [SPX Setup](../../spx-setup.md)]

## <a name="set-source-and-target-language"></a>Définir la langue source et la langue cible

Cette commande appelle l’interface CLI de reconnaissance vocale pour convertir la parole du microphone de l’italien au français.

```shell
 spx translate --microphone --source it-IT --target fr
```