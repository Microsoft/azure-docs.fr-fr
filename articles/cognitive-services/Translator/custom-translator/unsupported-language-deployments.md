---
title: Déploiements de langues non prises en charge – Custom Translator
titleSuffix: Azure Cognitive Services
description: Cet article explique comment déployer des paires de langues non prises en charge dans Azure Cognitive Services Custom Translator.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 04/24/2019
ms.author: swmachan
ms.openlocfilehash: 7498169c2c61b8337a90fbda082729f200f04635
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369033"
---
# <a name="unsupported-language-deployments"></a>Déploiements de langages non pris en charge

<!--Custom Translator provides the highest-quality translations possible using the latest techniques in neural machine learning. While Microsoft intends to make neural training available in all languages, there are some limitations that prevent us from being able to offer neural machine translation in all language pairs.-->  

Avec le retrait prochain du Microsoft Translator Hub, Microsoft annulera le déploiement de tous les modèles déployés via le hub. La plupart d’entre vous ont des modèles déployés dans le hub dont paires de langues ne sont pas prises en charge dans Custom Translator.  Nous ne souhaitez pas que les utilisateurs se trouvant dans cette situation n’aient aucune solution pour la traduction de leur contenu.

Nous avons élaboré un processus qui vous permet de déployer vos modèles non pris en charge via Custom Translator.  Ce processus vous permet de continuer à traduire du contenu à l’aide de la dernière API V3.  Ces modèles seront hébergés jusqu’à ce que vous décidiez d’annuler leur déploiement ou que la paire de langues devienne disponible dans Custom Translator.  Cet article explique le processus de déploiement des modèles avec des paires de langues non prises en charge.

## <a name="prerequisites"></a>Prérequis

Pour que vos modèles puissent être candidats au déploiement, ils doivent respecter les critères suivants :
* Le projet contenant le modèle doit avoir été migré du hub vers Custom Translator à l’aide de l’outil de migration.  Le processus de migration de projets et d’espaces de travail est décrit [ici](how-to-migrate.md).
* Lors de la migration, le modèle doit être dans l’état déployé.  
* La paire de langues du modèle ne doit pas être prise en charge dans Custom Translator.  Les paires de langues dont l’une des langues est prise en charge depuis ou vers l’anglais, mais qui n’incluent pas l’anglais, sont candidates pour des déploiements de langues non prises en charge.  Par exemple, un modèle de hub pour la paire de langues français-allemand est considéré comme une paire de langues non prise en charge, même si les paires de langues français-anglais et anglais-allemand sont prise en charge.

## <a name="process"></a>Process
Après avoir migré à partir du hub les modèles candidats au déploiement, vous pouvez les trouver en accédant à la page **Paramètres** de votre espace de travail, puis en faisant défiler l’écran jusqu’à la fin de la page où se trouve une section intitulée **Unsupported Translator Hub Trainings**.  Cette section apparaît uniquement si vous avez des projets remplissant les conditions préalables mentionnées ci-dessus.

![Capture d’écran mettant en évidence la section Formations non prises en charge par Translator Hub.](media/unsupported-language-deployments/unsupported-translator-hub-trainings.jpg)

Dans la page de sélection **Unsupported Translator Hub Trainings** , l’onglet **Unrequested trainings** contient des modèles éligibles pour un déploiement.  Sélectionnez les modèles que vous souhaitez déployer et envoyez une demande.   Avant l’échéance de déploiement du 30 avril, vous pouvez sélectionner autant de modèles que vous le souhaitez pour le déploiement.
 
![Capture d’écran montrant l’onglet Formations non demandées.](media/unsupported-language-deployments/unsupported-translator-hub-trainings-list.jpg)

Une fois envoyé, le modèle n’est plus disponible sous l’onglet **Unrequested trainings** , mais apparaît sous l’onglet **Requested trainings**.  Vous pouvez afficher vos formations demandées à tout moment.

![Migrer à partir du hub](media/unsupported-language-deployments/request-unsupported-trainings.jpg) 

## <a name="whats-next"></a>Quelle est l’étape suivante ?

Les modèles que vous avez sélectionnés pour déploiement sont enregistrés une fois le hub désaffecté, et le déploiement de tous les modèles est annulé.  Vous avez jusqu’au 24 mai pour soumettre des demandes de déploiement de modèles non pris en charge.  Nous allons déployer ces modèles le 15 juin, date à laquelle ils seront accessibles via l’API Translator V3.  Ils seront également disponibles via l’API V2 jusqu’au 1er juillet.  

Pour plus d’informations sur les dates importantes en lien avec l’abandon du hub, voir [ici](https://www.microsoft.com/translator/business/hub/).
Après déploiement, les frais d’hébergement normaux s’appliquent.  Pour en savoir plus, voir les [tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/).  

Contrairement aux modèles Custom Translator standard, les modèles du hub ne sont disponibles que dans une seule région, afin d’éviter les frais d’hébergement multirégion.  Après déploiement, vous pourrez annuler le déploiement de votre modèle de hub à tout moment via le projet Custom Translator migré.

## <a name="next-steps"></a>Étapes suivantes

- [Effectuer l’apprentissage d’un modèle](how-to-train-model.md).
- Commencez à utiliser votre modèle Custom Translator déployé via [Microsoft Translator Text API V3](../reference/v3-0-translate.md?tabs=curl).