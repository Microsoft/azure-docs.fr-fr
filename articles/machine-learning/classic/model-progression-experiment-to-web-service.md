---
title: 'ML Studio (classique) : Comment un modèle devient un service web - Azure'
description: Vue d’ensemble de la façon dont votre modèle Azure Machine Learning Studio (classique) peut passer du stade d’expérience de développement à celui d’un service web.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: likebupt
ms.author: keli19
ms.custom: previous-ms.author=yahajiza, previous-author=YasinMSFT
ms.date: 03/20/2017
ms.openlocfilehash: c92f8c74da76b2ac938892e27f3d6be9c70c3238
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95507264"
---
# <a name="how-a-machine-learning-studio-classic-model-progresses-from-an-experiment-to-a-web-service"></a>Comment faire évoluer un modèle Machine Learning Studio (classique) d’une simple expérience en un service web

**S’APPLIQUE À :**  ![Cette coche signifie que cet article s’applique à Machine Learning Studio (classique).](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Cette croix (X) signifie que cet article s’applique à Azure Machine Learning.](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Azure Machine Learning Studio (classique) fournit un canevas interactif qui permet de développer, d’exécuter, de tester et d’itérer une **_expérience_* _ représentant un modèle d’analyse prédictive. Il existe un large éventail de modules capables d’effectuer les opérations suivantes :

_ Entrer des données dans votre expérience
* Manipuler les données
* Former un modèle à l’aide d’algorithmes d’apprentissage automatique
* Noter le modèle
* Évaluer les résultats
* Sortir les valeurs finales

Une fois que vous êtes satisfait de votre expérience, vous pouvez la déployer en tant que ***service web Azure Machine Learning classique** _ ou _*_nouveau service web Azure Machine Learning_*_ afin de permettre aux utilisateurs d’envoyer de nouvelles données et de recevoir les résultats en retour.

Cet article donne une vue d’ensemble de la façon dont votre modèle d’apprentissage automatique peut passer du stade de l’expérience de développement à celui d’un service web opérationnel.

> [!NOTE]
> Il existe d’autres manières de développer et déployer des modèles d’apprentissage automatique, mais cet article se concentre uniquement sur l’utilisation de Machine Learning Studio (classique). Par exemple, pour lire une description sur la création d’un service web prédictif classique avec R, consultez le billet de blog relatif au [développement et au déploiement d’applications web prédictives à l’aide de RStudio et d’Azure Machine Learning Studio](/archive/blogs/machinelearning/build-deploy-predictive-web-apps-using-rstudio-and-azure-ml).
>
>

Bien qu’Azure Machine Learning Studio (classique) soit principalement conçu pour vous aider à développer et à déployer un _modèle d’analyse prédictive*, il est possible de l’utiliser pour développer une expérience qui n’inclut pas de modèle d’analyse prédictive. Une expérience peut, par exemple, simplement injecter des données et les manipuler avant de générer les résultats. À l’instar d’une expérience d’analyse prédictive, vous pouvez déployer cette expérience non prédictive en tant que service web. Ce processus sera toutefois plus simple car l’expérience n’est pas apparentée à l’apprentissage ou à la notation d’un modèle d’apprentissage automatique. Si cela ne représente pas l’utilisation habituelle de Studio (classique), nous allons l’inclure dans la discussion afin de pouvoir vous expliquer de manière exhaustive le mode de fonctionnement de Studio (classique).

## <a name="developing-and-deploying-a-predictive-web-service"></a>Développement et déploiement d’un service web prédictif
Voici les étapes effectuées pour une solution classique lorsque vous développez et déployez à l’aide de Machine Learning Studio (classique) :

![Flux de déploiement](./media/model-progression-experiment-to-web-service/model-stages-from-experiment-to-web-service.png)

*Figure 1 : Étapes d’un modèle d’analyse prédictive classique*

### <a name="the-training-experiment"></a>Expérience de formation
L’***expérience de formation** _ est la phase initiale de développement de votre service web dans Machine Learning Studio (classique). L’expérience de formation vous permet de développer, tester, itérer et former un modèle d’apprentissage automatique. Vous pouvez même former plusieurs modèles simultanément en recherchant la meilleure solution possible ; mais une fois que vous aurez terminé votre expérience, vous devrez sélectionner un seul modèle formé et supprimer le reste de l’expérience. Pour obtenir un exemple de développement d’une expérience d’analyse prédictive, consultez [Guide pas à pas : développer une solution d’analyse prédictive pour l’évaluation des risques de crédit dans Azure Machine Learning Studio (classique)](tutorial-part1-credit-risk.md).

### <a name="the-predictive-experiment"></a>Expérience prédictive
Après avoir effectué l’apprentissage d’un modèle dans votre expérience de formation, cliquez sur _*Configurer le service web** puis sélectionnez **Service web prédictif** dans Machine Learning Studio (classique) pour commencer le processus de conversion de votre expérience de formation en **_expérience prédictive_*_. L’objectif de l’expérience prédictive consiste à utiliser votre modèle formé afin d’évaluer les nouvelles données, ce dans le but de devenir opérationnel sous la forme d’un service web Azure.

Cette conversion s’effectue automatiquement de la manière suivante :

_ Conversion de l’ensemble des modules utilisés pour la formation en un seul module, enregistré comme modèle d’apprentissage
* Suppression de tous les modules superflus non liés à la notation
* Ajout des ports d’entrée et de sortie qui seront utilisés par le service web

Vous pouvez apporter des modifications supplémentaires pour accélérer le déploiement de votre expérience prédictive en tant que service web. Par exemple, si vous souhaitez que le service web génère uniquement un sous-ensemble de résultats, vous pouvez ajouter un module de filtrage avant le port de sortie.

Dans ce processus de conversion, l’expérience de formation n’est pas supprimée. Une fois le processus terminé, deux onglets s’affichent dans Studio (classique) : le premier pour l’expérience de formation, le second pour l’expérience prédictive. Ainsi, avant de déployer votre service web, vous pouvez apporter des modifications à l’expérience de formation et reconstruire l’expérience prédictive. Vous pouvez également enregistrer une copie de l’expérience de formation pour démarrer une autre ligne de l’expérimentation.

> [!NOTE]
> Lorsque vous cliquez sur **Service web prédictif**, vous démarrez un processus automatique permettant de convertir votre expérience de formation en expérience prédictive. Ce processus fonctionne bien dans la plupart des cas. Si vous avez défini une expérience de formation complexe (par exemple, si vous avez plusieurs chemins d’accès pour des formations que vous souhaitez relier entre elles), vous préférerez peut-être effectuer cette conversion manuellement. Pour plus d’informations, consultez [Guide pratique pour préparer votre modèle pour le déploiement dans Azure Machine Learning Studio (classique)](deploy-a-machine-learning-web-service.md).
>
>

### <a name="the-web-service"></a>Service web
Une fois que vous êtes satisfait et que votre expérience prédictive est prête, vous pouvez déployer votre service en tant que service web classique ou nouveau service web basé sur Azure Resource Manager. Pour rendre opérationnel votre modèle en le déployant en tant que *service web Machine Learning classique*, cliquez sur **Déployer le service web** puis sélectionnez **Déployer le service web [classique]** . Pour déployer en tant que *nouveau service web Machine Learning*, cliquez sur **Déployer le service web** puis sélectionnez **Déployer le service web [nouveau]** . Les utilisateurs peuvent maintenant envoyer des données à votre modèle à l’aide de l’API REST du service web, et recevoir les résultats en retour. Pour plus d’informations, consultez [Utilisation d’un service Web Microsoft Azure Machine Learning](consume-web-services.md).

## <a name="the-non-typical-case-creating-a-non-predictive-web-service"></a>Scénario non classique : création d’un service web non prédictif
Si votre expérience ne forme pas un modèle d’analyse prédictive, vous n’avez pas besoin de créer une expérience de formation et une expérience d’évaluation ; vous ne disposez que d’une seule expérience, que vous pouvez déployer en tant que service web. Machine Learning Studio (classique) détecte si votre expérience contient ou non un modèle prédictif en analysant les modules que vous avez utilisés.

Une fois que vous avez parcouru votre expérience et que vous la jugez satisfaisante :

1. Cliquez sur **Configurer le service web**, puis sélectionnez **Reformation du service web** : les nœuds d’entrée et de sortie sont automatiquement ajoutés.
2. Cliquez sur **Exécuter**.
3. Cliquez sur **Déployer le service web** puis sélectionnez **Déployer le service web [classique]** ou **Déployer le service web [nouveau]** selon l’environnement dans lequel vous souhaitez déployer le service.

Votre service web est désormais déployé, et vous pouvez y accéder et le gérer de la même manière qu’un service web prédictif.

## <a name="updating-your-web-service"></a>Mise à jour de votre service web
Maintenant que vous avez déployé votre expérience en tant que service web, que se passe-t-il si vous avez une mise à jour à effectuer ?

Tout dépend de ce que vous devez mettre à jour :

**Vous souhaitez modifier l’entrée ou la sortie, ou vous souhaitez modifier la façon dont le service web manipule les données**

Si vous ne modifiez pas le modèle, mais que vous changez simplement la façon dont le service web gère les données, vous pouvez modifier l’expérience prédictive puis cliquer de nouveau sur **Déployer le service web** et sélectionner **Déployer le service web [classique]** ou **Déployer le service web [nouveau]** . Le service web est arrêté, l’expérimentation prédictive mise à jour est déployée, puis le service web est redémarré.

Voici un exemple : Supposons que votre expérience prédictive retourne toute la ligne de données d’entrée avec le résultat prédit. Vous pouvez décider que le service web se contente simplement de retourner le résultat. Vous pouvez donc ajouter un module **Colonnes de projet** dans l’expérience prédictive, juste avant le port de sortie, pour exclure les colonnes qui ne correspondent pas au résultat. Lorsque vous cliquez sur **Déployer le service web** et sélectionnez **Déployer le service web [classique]** ou **Déployer le service web [nouveau]** , le service web est mis à jour.

**Vous souhaitez reformer le modèle avec de nouvelles données**

Si vous souhaitez conserver votre modèle d’apprentissage automatique, mais en le reformant avec de nouvelles données, vous avez deux possibilités :

1. **Reformer le modèle pendant l’exécution du service web** : si vous souhaitez reformer votre modèle pendant l’exécution du service web prédictif, vous pouvez procéder en apportant quelques modifications à l’expérience de formation pour en faire une **_expérience de reformation_ *_, puis la déployer en tant que _* service _web de reformation_**. Pour connaître la procédure à suivre, consultez la page [Reformation des modèles Machine Learning par programme](./retrain-machine-learning-model.md).
2. **Revenir à l’expérience de formation d’origine et utiliser d’autres données de formation pour développer votre modèle** : votre expérience prédictive est liée au service web, mais l’expérience de formation n’est pas directement liée de cette façon. Si vous modifiez l’expérience de formation d’origine, cliquez sur **Configurer le service web**. Vous obtiendrez alors une *nouvelle* expérience prédictive qui, une fois déployée, créera un *nouveau* service web. L’opération ne se résume pas à mettre simplement à jour le service web d’origine.

   Si vous devez modifier l’expérience de formation, ouvrez-la et cliquez sur **Enregistrer sous** pour effectuer une copie. Cette précaution garantira l’intégrité de l’expérience d’apprentissage d’origine, de l’expérience prédictive et du service web. Vous pouvez désormais créer un service web avec vos modifications. Après avoir déployé le nouveau service web, vous pouvez alors décider d’arrêter le service web précédent ou de le maintenir en exécution simultanée.

**Vous souhaitez former un modèle différent**

Si vous souhaitez apporter des modifications à votre expérience prédictive d’origine (par exemple, sélection d’un autre algorithme d’apprentissage automatique, test d’une méthode de formation différente, etc.), vous devez suivre la deuxième procédure décrite ci-dessus pour la reformation de votre modèle : ouvrez l’expérience de formation, cliquez sur **Enregistrer sous** pour effectuer une copie, puis commencez à développer votre modèle, à créer l’expérience prédictive et à déployer le service web. Cela créera un nouveau service web non lié à l’original. Vous pouvez alors décider du ou des services qui continueront à fonctionner.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur le processus de développement et d’expérimentation, voir les articles suivants :

* conversion de l’expérience : [Guide pratique pour préparer votre modèle pour le déploiement dans Azure Machine Learning Studio (classique)](deploy-a-machine-learning-web-service.md)
* déploiement du service web : [Déploiement d’un service web Azure Machine Learning](deploy-a-machine-learning-web-service.md)
* reformation du modèle : [Reformation des modèles Machine Learning par programme](./retrain-machine-learning-model.md)

Pour obtenir des exemples de l’ensemble du processus, consultez :

* [Didacticiel de Machine Learning : Créer votre première expérience dans Azure Machine Learning Studio (classique)](create-experiment.md)
* [Procédure pas à pas : Développer une solution d’analyse prédictive pour l’évaluation des risques de crédit dans Azure Machine Learning](tutorial-part1-credit-risk.md)