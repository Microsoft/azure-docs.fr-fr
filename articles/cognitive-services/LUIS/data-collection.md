---
title: Collecte de données
description: Découvrez les exemples de données à collecter lors du développement de votre application
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 05/06/2020
ms.openlocfilehash: bb00595a5d5be130bf6c1177004bf3042ef8f9f0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91327185"
---
# <a name="data-collection-for-your-app"></a>Collecte des données pour votre application

Des données sont nécessaires dans le cadre du développement d’une application Language Understanding (LUIS).

## <a name="data-used-in-luis"></a>Données utilisées dans LUIS

LUIS utilise du texte en tant que données pour l’apprentissage et le test de votre application LUIS pour la classification des [intentions](luis-concept-intent.md) et pour l’extraction des [entités](luis-concept-entity-types.md). Votre jeu de données doit être suffisamment fourni pour vous permettre de créer des jeux de données distincts destinés à l’apprentissage et au test qui présentent la diversité et la répartition spécifiées ci-dessous.  Les données présentes dans chacun de ces jeux ne doivent pas se chevaucher.

## <a name="training-data-selection-for-example-utterances"></a>Sélection des données d’apprentissage pour les exemples d’énoncés

Sélectionnez des énoncés pour vote jeu d’apprentissage en fonction des critères suivants :

* **Privilégiez les données réelles** :
    * **Données réelles d’une application cliente** : Sélectionnez les énoncés constitués de données réelles issues de votre application cliente.  Si le client envoie un formulaire web avec sa recherche aujourd’hui et que vous créez un bot, vous pouvez commencer par utiliser les données de ce formulaire web.
    * **Données participatives** : Si vous n’avez pas de données existantes, vous pouvez faire appel à la production participative d’énoncés.  Demandez à vos utilisateurs réels de fournir des énoncés pour votre scénario afin d’obtenir des données les plus proches possibles de celles que votre application rencontrera. Les énoncés humains participatifs sont bien plus intéressants que ceux générés par ordinateur.  Un jeu de données d’énoncés synthétiques générés sur des modèles spécifiques ne présente pas la plupart des variations naturelles observées dans des énoncés créés par des personnes réelles, et la généralisation fonctionnera mal en production.
* **Diversité des données** :
    * **Diversité des régions** : Veillez à ce que les données associées à chaque intention soient aussi diversifiées que possible notamment en termes de _formulation_ (choix des mots) et de _grammaire_.  Si vous effectuez l’apprentissage d’une intention concernant des stratégies de RH pour les jours de congés, veillez à utiliser des énoncés qui illustrent les termes usités dans toutes les régions concernées.  À titre d’exemple, on utilise en Europe l’expression `taking a holiday`, tandis qu’aux États-Unis, on dit plutôt `taking vacation days`.
    * **Diversité des langues** : Si vous avez des utilisateurs de différentes langues maternelles qui communiquent dans une seconde langue, veillez à utiliser des énoncés qui représentent des locuteurs non natifs.
    * **Diversité des entrées** : Tenez compte du mode d’entrée de vos données. Si vous recueillez des données auprès d’une personne, d’un service ou d’un périphérique d’entrée (micro) unique, celles-ci ne reflèteront vraisemblablement pas la diversité nécessaire pour que votre application apprenne à partir de tous les modes d’entrée.
    * **Diversité de la ponctuation** : Tenez compte du fait que les niveaux de ponctuation utilisés dans les applications de messagerie varient en fonction des personnes, et veillez à représenter cette diversité d’utilisation. Si vous utilisez des données orales, elles ne comportent aucune ponctuation, par conséquent vos données ne doivent pas en comporter non plus.
* **Répartition des données** : Veillez à ce que les données réparties entre les intentions représentent la même répartition des données que celle que reçoit votre application cliente. Si votre application LUIS doit classifier les énoncés constituant des demandes de planification de congés (50 %) mais reçoit également des énoncés concernant les jours de congés restants (20 %), des approbations de congés (20 %) ainsi que des conversations hors sujet et du bavardage (10 %), votre jeu de données doit comporter les pourcentages d’exemples correspondant à chaque type d’énoncé.
* **Utilisez toutes les formes de données** : Si votre application LUIS prélève des données de différentes formes, veillez à inclure ces formes dans vos énoncés d’apprentissage. Par exemple, si votre application cliente accepte des entrées orales et du texte tapé, vous devez avoir des énoncés générés par reconnaissance vocale et des énoncés tapés.  Vous allez constater des variations dans la façon de parler et la façon de taper, ainsi que des erreurs dans la reconnaissance vocale et des fautes de frappe.  Toutes ces variations doivent être représentées dans vos données d’apprentissage.
* **Exemples positifs et négatifs** : Pour son apprentissage, une application LUIS doit apprendre ce qu’est l’intention (positif) et ce qu’elle n’est pas (négatif). Dans LUIS, les énoncés ne peuvent être que positifs pour une seule intention. Lorsqu’un énoncé est ajouté à une intention, LUIS transforme automatiquement ce même exemple d’énoncé en exemple négatif pour toutes les autres intentions.
* **Données en dehors de l’étendue de l’application** : Si votre application reçoit des énoncés qui se trouvent en dehors de vos intentions définies, veillez à les fournir. Les exemples qui ne sont pas attribués à une intention définie particulière sont étiquetés avec l’intention **Aucune**.  Il est important d’avoir des exemples réalistes pour l’intention **Aucune** afin de prédire correctement les énoncés qui se trouvent en dehors de l’étendue des intentions définies.

    Par exemple, si vous créez un bot RH consacré aux congés et que vous avez trois intentions :
    * planifier ou modifier un congé
    * demander les jours de congés disponibles
    * approuver/refuser un congé

    vous devez veiller à avoir des énoncés qui concernent toutes ces intentions, mais également d’éventuels énoncés en dehors de cette étendue, que l’application devra présenter, par exemple :
    * `What are my medical benefits?`
    * `Who is my HR rep?`
    * `tell me a joke`
* **Exemples rares** : Votre application doit comporter des exemples rares et des exemples courants.  Si votre application n’a jamais vu d’exemples rares, elle ne sera pas capable de les identifier en production. Si vous utilisez des données réelles, vous pourrez prédire plus précisément le fonctionnement de votre application LUIS en production.

### <a name="quality-instead-of-quantity"></a>Qualité plutôt que quantité

Tenez compte de la qualité de vos données existantes avant d’en ajouter davantage.  Avec LUIS, vous utilisez le Machine Teaching.  La combinaison de vos étiquettes et des fonctionnalités de Machine Learning que vous définissez correspond à ce que votre application LUIS utilise.  Elle ne s’appuie pas simplement sur la quantité d’étiquettes pour effectuer la meilleure prédiction.  La diversité des exemples et leur représentation de ce que votre application LUIS verra en production sont les éléments les plus importants.

### <a name="preprocessing-data"></a>Prétraitement des données

Les étapes de prétraitement suivantes vous aideront à améliorer votre application LUIS :

* **Supprimer les doublons** : Il n’est pas grave en soi d’avoir des énoncés en double, mais ils sont inutiles, donc le fait de les supprimer évite d’avoir à les étiqueter.
* **Appliquer un prétraitement propre à une même application cliente** : Si votre application cliente, qui appelle le point de terminaison de prédiction LUIS, applique un traitement des données au moment de l’exécution avant d’envoyer le texte à LUIS, vous devez former l’application LUIS sur les données qui sont traitées de la même manière. Par exemple, si votre application cliente utilise la [Vérification orthographique Bing](../bing-spell-check/overview.md) pour corriger l’orthographe des entrées envoyées à LUIS, corrigez vos énoncés d’apprentissage et de test avant de les ajouter à LUIS.
* **N’appliquez pas de nouveaux processus de nettoyage que l’application cliente n’utilise pas** : Si votre application cliente accepte du texte généré par reconnaissance vocale directement sans nettoyage tel que la vérification grammaticale ou de la ponctuation, vos énoncés doivent refléter la même chose, notamment la ponctuation manquante et tout autre défaut de reconnaissance que vous devez prendre en compte.
* **Ne nettoyez pas les données** : Ne supprimez pas les éventuelles entrées incorrectes issues d’une reconnaissance vocale confuse, les frappes involontaires, le texte mal orthographié ou les fautes de frappe. Si votre application est confrontée à de telles entrées, il est important qu’elle soit formée et testée avec. Ajoutez une intention _entrée incorrecte_ si votre application n’est pas censée les comprendre. Étiquetez ces données pour aider votre application LUIS à prédire la réponse correcte au moment de l’exécution. Votre application cliente peut choisir une réponse appropriée à des énoncés inintelligibles, par exemple `Please try again`.

### <a name="labeling-data"></a>Étiquetage des données

* **Étiquetez le texte comme s’il était correct** : Les exemples d’énoncés doivent avoir toutes les formes d’une entité étiquetées. Cela comprend le texte mal orthographié, les fautes de frappe et les erreurs de traduction.

### <a name="data-review-after-luis-app-is-in-production"></a>Vérification des données une fois l’application LUIS en production

[Examinez les énoncés de point de terminaison](luis-concept-review-endpoint-utterances.md) pour surveiller le trafic des énoncés réels une fois que vous avez déployé une application en production.  Cela vous permet de mettre à jour vos énoncés d’apprentissage avec des données réelles, ce qui aura pour effet d’améliorer votre application. Les applications créées avec des données participatives ou de scénarios non réels doivent être améliorées en fonction de leur utilisation réelle.

## <a name="test-data-selection-for-batch-testing"></a>Sélection des données de test pour le test par lot

L’ensemble des principes ci-dessus relatifs aux énoncés d’apprentissage s’appliquent aux énoncés que vous devez utiliser pour votre [jeu de test](luis-concept-batch-test.md). Assurez-vous que la répartition entre les intentions et les entités reflète la répartition réelle aussi fidèlement que possible.

Ne réutilisez pas les énoncés de votre jeu d’apprentissage dans votre jeu de test. Cela aurait pour conséquence de biaiser vos résultats et ne vous donnerait pas une indication correcte de ce que votre application LUIS ferait en production.

Une fois la première version de votre application publiée, vous devez mettre à jour votre jeu de test avec des énoncés provenant du trafic réel pour qu’il reflète la répartition de votre production et que vous puissiez surveiller les performances réalistes au fil du temps.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment LUIS modifie vos données avant la prédiction](luis-concept-data-alteration.md)
