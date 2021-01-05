---
title: Détection intelligente des anomalies de type échec dans Application Insights | Microsoft Docs
description: Vous prévient en cas de modifications inhabituelles du taux d’échec des demandes adressées à votre application web et fournit une analyse du diagnostic. Aucune configuration n’est nécessaire.
ms.topic: conceptual
ms.date: 12/18/2018
ms.reviewer: yalavi
ms.openlocfilehash: 978b63d74e6be4104ff53eef66e9633c78b90eb8
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510632"
---
# <a name="smart-detection---failure-anomalies"></a>Détection intelligente des anomalies de type échec
[Application Insights](./app-insights-overview.md) vous alerte automatiquement en quasi temps réel si votre application web enregistre une hausse anormale du taux de requêtes ayant échoué. Il détecte une augmentation inhabituelle du nombre de demandes HTTP ou d’appels de dépendance signalés comme défaillants. Les requêtes ayant échoué ont généralement un code de réponse supérieur ou égal à 400. Pour vous aider à trier et diagnostiquer les causes du problème, les détails de l’alerte s’accompagnent d’une analyse des caractéristiques des échecs et des données d’application associées. Elle fournit également des liens vers le portail Application Insights pour un diagnostic plus poussé. La fonctionnalité ne requiert ni installation ni configuration, puisqu’elle utilise des algorithmes d’apprentissage automatique pour prédire le taux d’échec normal.

Cette fonctionnalité peut être utilisée pour toutes les applications web, hébergées dans le cloud ou sur vos propres serveurs, qui génèrent des données sur les requêtes ou les dépendances de l’application. C’est le cas, par exemple, si vous avez un rôle de worker qui appelle [TrackRequest()](./api-custom-events-metrics.md#trackrequest) ou [TrackDependency()](./api-custom-events-metrics.md#trackdependency).

Après que vous avez configuré [Application Insights pour votre projet](./app-insights-overview.md), si votre application génère une certaine quantité minimale de données, un délai de 24 heures est nécessaire à la détection intelligente des anomalies de type échec pour apprendre le comportement normal de votre application, être activée et pouvoir envoyer des alertes.

Voici un exemple d’alerte :

:::image type="content" source="./media/proactive-failure-diagnostics/013.png" alt-text="Exemple d’alerte de détection intelligente affichant l’analyse du cluster au moment de l’échec." lightbox="./media/proactive-failure-diagnostics/013.png":::

Les détails de l’alerte indiquent :

* le taux d’échec par rapport au comportement normal de l’application ;
* combien d’utilisateurs sont affectés : afin de savoir dans quelle mesure vous devez vous inquiéter ;
* un modèle caractéristique associé aux échecs. Cet exemple contient un code de réponse, un nom de requête (opération) et une version d’application spécifiques. Ces informations vous indiquent immédiatement où commencer la recherche dans votre code. Les autres possibilités peuvent être un type de navigateur ou un système d’exploitation client spécifique ;
* l’exception, le suivi des journaux et l’échec de dépendance (bases de données ou autres composants externes) qui semblent associés à des défaillances identifiées ;
* les liens directs aux recherches pertinentes sur les données dans Application Insights.

## <a name="benefits-of-smart-detection"></a>Avantages de la détection intelligente
Les [alertes de mesure](../platform/alerts-log.md) ordinaires vous indiquent un problème potentiel. Mais la détection intelligente démarre le travail de diagnostic automatiquement et effectue la majeure partie de l’analyse à votre place. Vous obtenez les résultats clairement empaquetés, ce qui vous permet d’accéder rapidement à l’origine du problème.

## <a name="how-it-works"></a>Fonctionnement
La détection intelligente analyse les données reçues de votre application, en particulier le taux d’échecs. Cette règle compte le nombre de demandes dont la propriété `Successful request` a la valeur false, et le nombre d’appels de dépendance dont la propriété `Successful call` a la valeur false. Pour les demandes, `Successful request == (resultCode < 400)` par défaut (sauf si vous avez ajouté du code personnalisé pour [filtrer](./api-filtering-sampling.md#filtering) ou générer vos propres appels [TrackRequest](./api-custom-events-metrics.md#trackrequest)). 

Les performances de votre application présentent un modèle de comportement typique. Certaines demandes ou certains appels de dépendance sont davantage sujets aux erreurs que d’autres ; et le taux d’échec global peut augmenter en même temps que la charge. La détection intelligente utilise Machine Learning pour rechercher ces anomalies.

À mesure qu’Application Insights reçoit les données de votre application web, la détection intelligente compare le comportement actuel avec les modèles observés au cours des derniers jours. Si une augmentation anormale du taux d’échec est observée par rapport aux performances précédentes, une analyse est déclenchée.

Quand une analyse est déclenchée, le service effectue une analyse des clusters pour chaque demande ayant échoué, dans le but d’identifier un modèle de valeurs caractérisant les échecs. 

Dans l’exemple ci-dessus, l’analyse a découvert que la plupart des échecs sont liés à un code de résultat, un nom de demande, un hôte de l’URL serveur et une instance de rôle spécifiques. 

Quand votre service est instrumenté avec ces appels, l’analyseur recherche une exception et un échec de dépendance en lien avec les requêtes dans le cluster qu’il a identifié, ainsi qu’un exemple de journaux de suivi associés à ces requêtes.

L’analyse obtenue vous est envoyée sous forme d’alerte, sauf si vous n’avez pas configuré cette option.

Comme pour les [alertes définies manuellement](../platform/alerts-log.md), vous pouvez inspecter l’état de l’alerte déclenchée, qui peut être résolue si le problème est corrigé. Configurez les règles d’alerte dans la page Alertes de votre ressource Application Insights. Cependant, contrairement aux autres alertes, vous n’avez pas besoin d’installer ni de configurer la détection intelligente. Si vous le souhaitez, vous pouvez la désactiver ou changer l’adresse de messagerie électronique cible.

### <a name="alert-logic-details"></a>Détails de la logique d’alerte

Les alertes sont déclenchées par notre algorithme d’apprentissage automatique propriétaire afin de ne pas pouvoir partager les détails exacts d’implémentation. Ceci dit, nous comprenons que vous deviez parfois en savoir plus sur le fonctionnement de la logique sous-jacente. Les principaux facteurs qui sont évalués pour déterminer si une alerte doit être déclenchée sont : 

* Analyse du pourcentage d’échec des demandes/dépendances dans une fenêtre de temps évolutive de 20 minutes.
* Comparaison du pourcentage d’échec dans les 20 dernières minutes avec le taux dans les 40 dernières minutes et au cours des sept derniers jours, et recherche des écarts significatifs qui dépassent X fois cet écart type.
* Utilisation d’une limite adaptative pour le pourcentage d’échec minimum, qui varie selon le volume des demandes/dépendances de l’application.
* Il existe une logique capable de résoudre automatiquement la condition d’analyse de l’alerte déclenchée si le problème n’est plus détecté pendant 8-24 heures.
  Remarque : Dans la conception actuelle, aucune notification ni action n’est envoyée quand une alerte de détection intelligente est résolue. Vous pouvez vérifier si une alerte de détection intelligente a été résolue dans le portail Azure.

## <a name="configure-alerts"></a>Configurer des alertes

Vous pouvez désactiver la règle d’alerte de détection intelligente à partir du portail ou à l’aide d’Azure Resource Manager ([voir l’exemple de modèle](./proactive-arm-config.md)).

Cette règle d’alerte est créée avec un [groupe d’actions](../platform/action-groups.md) associé nommé « Détection intelligente Application Insights » qui contient des actions de webhook et de messagerie, et peut être étendue pour déclencher des actions supplémentaires lorsque l’alerte se déclenche.

> [!NOTE]
> Les notifications par e-mail envoyées de cette règle d’alerte sont désormais envoyées par défaut aux utilisateurs associés aux rôles Lecteur d’analyse et Contributeur d’analyse de l’abonnement. Davantage d’informations à ce sujet sont disponibles [ici](./proactive-email-notification.md).
> Les notifications envoyées de cette règle d’alerte suivent le [schéma d’alerte courant](../platform/alerts-common-schema.md).
>

Ouvrez la page Alertes. Les règles d’alerte Anomalies des échecs sont affichées avec les alertes que vous avez définies manuellement, et vous pouvez savoir si elle se trouve actuellement à l’état d’alerte.

:::image type="content" source="./media/proactive-failure-diagnostics/021.png" alt-text="Dans la page de la ressource Application Insights, cliquez sur la vignette « Alertes », puis sur « Gérer les règles d’alerte »." lightbox="./media/proactive-failure-diagnostics/021.png":::

Cliquez sur l’alerte pour la configurer.

:::image type="content" source="./media/proactive-failure-diagnostics/032.png" alt-text="Écran de configuration des règles." lightbox="./media/proactive-failure-diagnostics/032.png":::

Notez que vous pouvez désactiver ou supprimer une règle Anomalies des échecs, mais que vous ne pouvez pas en créer une autre sur la même ressource Application Insights.

## <a name="example-of-failure-anomalies-alert-webhook-payload"></a>Exemple de charge utile webhook des alertes Anomalies des échecs

```json
{
    "properties": {
        "essentials": {
            "severity": "Sev3",
            "signalType": "Log",
            "alertState": "New",
            "monitorCondition": "Resolved",
            "monitorService": "Smart Detector",
            "targetResource": "/subscriptions/4f9b81be-fa32-4f96-aeb3-fc5c3f678df9/resourcegroups/test-group/providers/microsoft.insights/components/test-rule",
            "targetResourceName": "test-rule",
            "targetResourceGroup": "test-group",
            "targetResourceType": "microsoft.insights/components",
            "sourceCreatedId": "1a0a5b6436a9b2a13377f5c89a3477855276f8208982e0f167697a2b45fcbb3e",
            "alertRule": "/subscriptions/4f9b81be-fa32-4f96-aeb3-fc5c3f678df9/resourcegroups/test-group/providers/microsoft.alertsmanagement/smartdetectoralertrules/failure anomalies - test-rule",
            "startDateTime": "2019-10-30T17:52:32.5802978Z",
            "lastModifiedDateTime": "2019-10-30T18:25:23.1072443Z",
            "monitorConditionResolvedDateTime": "2019-10-30T18:25:26.4440603Z",
            "lastModifiedUserName": "System",
            "actionStatus": {
                "isSuppressed": false
            },
            "description": "Failure Anomalies notifies you of an unusual rise in the rate of failed HTTP requests or dependency calls."
        },
        "context": {
            "DetectionSummary": "An abnormal rise in failed request rate",
            "FormattedOccurenceTime": "2019-10-30T17:50:00Z",
            "DetectedFailureRate": "50.0% (200/400 requests)",
            "NormalFailureRate": "0.0% (over the last 30 minutes)",
            "FailureRateChart": [
                [
                    "2019-10-30T05:20:00Z",
                    0
                ],
                [
                    "2019-10-30T05:40:00Z",
                    100
                ],
                [
                    "2019-10-30T06:00:00Z",
                    0
                ],
                [
                    "2019-10-30T06:20:00Z",
                    0
                ],
                [
                    "2019-10-30T06:40:00Z",
                    100
                ],
                [
                    "2019-10-30T07:00:00Z",
                    0
                ],
                [
                    "2019-10-30T07:20:00Z",
                    0
                ],
                [
                    "2019-10-30T07:40:00Z",
                    100
                ],
                [
                    "2019-10-30T08:00:00Z",
                    0
                ],
                [
                    "2019-10-30T08:20:00Z",
                    0
                ],
                [
                    "2019-10-30T08:40:00Z",
                    100
                ],
                [
                    "2019-10-30T17:00:00Z",
                    0
                ],
                [
                    "2019-10-30T17:20:00Z",
                    0
                ],
                [
                    "2019-10-30T09:00:00Z",
                    0
                ],
                [
                    "2019-10-30T09:20:00Z",
                    0
                ],
                [
                    "2019-10-30T09:40:00Z",
                    100
                ],
                [
                    "2019-10-30T10:00:00Z",
                    0
                ],
                [
                    "2019-10-30T10:20:00Z",
                    0
                ],
                [
                    "2019-10-30T10:40:00Z",
                    100
                ],
                [
                    "2019-10-30T11:00:00Z",
                    0
                ],
                [
                    "2019-10-30T11:20:00Z",
                    0
                ],
                [
                    "2019-10-30T11:40:00Z",
                    100
                ],
                [
                    "2019-10-30T12:00:00Z",
                    0
                ],
                [
                    "2019-10-30T12:20:00Z",
                    0
                ],
                [
                    "2019-10-30T12:40:00Z",
                    100
                ],
                [
                    "2019-10-30T13:00:00Z",
                    0
                ],
                [
                    "2019-10-30T13:20:00Z",
                    0
                ],
                [
                    "2019-10-30T13:40:00Z",
                    100
                ],
                [
                    "2019-10-30T14:00:00Z",
                    0
                ],
                [
                    "2019-10-30T14:20:00Z",
                    0
                ],
                [
                    "2019-10-30T14:40:00Z",
                    100
                ],
                [
                    "2019-10-30T15:00:00Z",
                    0
                ],
                [
                    "2019-10-30T15:20:00Z",
                    0
                ],
                [
                    "2019-10-30T15:40:00Z",
                    100
                ],
                [
                    "2019-10-30T16:00:00Z",
                    0
                ],
                [
                    "2019-10-30T16:20:00Z",
                    0
                ],
                [
                    "2019-10-30T16:40:00Z",
                    100
                ],
                [
                    "2019-10-30T17:30:00Z",
                    50
                ]
            ],
            "ArmSystemEventsRequest": "/subscriptions/4f9b81be-fa32-4f96-aeb3-fc5c3f678df9/resourceGroups/test-group/providers/microsoft.insights/components/test-rule/query?query=%0d%0a++++++++++++++++systemEvents%0d%0a++++++++++++++++%7c+where+timestamp+%3e%3d+datetime(%272019-10-30T17%3a20%3a00.0000000Z%27)+%0d%0a++++++++++++++++%7c+where+itemType+%3d%3d+%27systemEvent%27+and+name+%3d%3d+%27ProactiveDetectionInsight%27+%0d%0a++++++++++++++++%7c+where+dimensions.InsightType+in+(%275%27%2c+%277%27)+%0d%0a++++++++++++++++%7c+where+dimensions.InsightDocumentId+%3d%3d+%27718fb0c3-425b-4185-be33-4311dfb4deeb%27+%0d%0a++++++++++++++++%7c+project+dimensions.InsightOneClassTable%2c+%0d%0a++++++++++++++++++++++++++dimensions.InsightExceptionCorrelationTable%2c+%0d%0a++++++++++++++++++++++++++dimensions.InsightDependencyCorrelationTable%2c+%0d%0a++++++++++++++++++++++++++dimensions.InsightRequestCorrelationTable%2c+%0d%0a++++++++++++++++++++++++++dimensions.InsightTraceCorrelationTable%0d%0a++++++++++++&api-version=2018-04-20",
            "LinksTable": [
                {
                    "Link": "<a href=\"https://portal.azure.com/#blade/AppInsightsExtension/ProactiveDetectionFeedBlade/ComponentId/{\"SubscriptionId\":\"4f9b81be-fa32-4f96-aeb3-fc5c3f678df9\",\"ResourceGroup\":\"test-group\",\"Name\":\"test-rule\"}/SelectedItemGroup/718fb0c3-425b-4185-be33-4311dfb4deeb/SelectedItemTime/2019-10-30T17:50:00Z/InsightType/5\" target=\"_blank\">View full details in Application Insights</a>"
                }
            ],
            "SmartDetectorId": "FailureAnomaliesDetector",
            "SmartDetectorName": "Failure Anomalies",
            "AnalysisTimestamp": "2019-10-30T17:52:32.5802978Z"
        },
        "egressConfig": {
            "displayConfig": [
                {
                    "rootJsonNode": null,
                    "sectionName": null,
                    "displayControls": [
                        {
                            "property": "DetectionSummary",
                            "displayName": "What was detected?",
                            "type": "Text",
                            "isOptional": false,
                            "isPropertySerialized": false
                        },
                        {
                            "property": "FormattedOccurenceTime",
                            "displayName": "When did this occur?",
                            "type": "Text",
                            "isOptional": false,
                            "isPropertySerialized": false
                        },
                        {
                            "property": "DetectedFailureRate",
                            "displayName": "Detected failure rate",
                            "type": "Text",
                            "isOptional": false,
                            "isPropertySerialized": false
                        },
                        {
                            "property": "NormalFailureRate",
                            "displayName": "Normal failure rate",
                            "type": "Text",
                            "isOptional": false,
                            "isPropertySerialized": false
                        },
                        {
                            "chartType": "Line",
                            "xAxisType": "Date",
                            "yAxisType": "Percentage",
                            "xAxisName": "",
                            "yAxisName": "",
                            "property": "FailureRateChart",
                            "displayName": "Failure rate over last 12 hours",
                            "type": "Chart",
                            "isOptional": false,
                            "isPropertySerialized": false
                        },
                        {
                            "defaultLoad": true,
                            "displayConfig": [
                                {
                                    "rootJsonNode": null,
                                    "sectionName": null,
                                    "displayControls": [
                                        {
                                            "showHeader": false,
                                            "columns": [
                                                {
                                                    "property": "Name",
                                                    "displayName": "Name"
                                                },
                                                {
                                                    "property": "Value",
                                                    "displayName": "Value"
                                                }
                                            ],
                                            "property": "tables[0].rows[0][0]",
                                            "displayName": "All of the failed requests had these characteristics:",
                                            "type": "Table",
                                            "isOptional": false,
                                            "isPropertySerialized": true
                                        }
                                    ]
                                }
                            ],
                            "property": "ArmSystemEventsRequest",
                            "displayName": "",
                            "type": "ARMRequest",
                            "isOptional": false,
                            "isPropertySerialized": false
                        },
                        {
                            "showHeader": false,
                            "columns": [
                                {
                                    "property": "Link",
                                    "displayName": "Link"
                                }
                            ],
                            "property": "LinksTable",
                            "displayName": "Links",
                            "type": "Table",
                            "isOptional": false,
                            "isPropertySerialized": false
                        }
                    ]
                }
            ]
        }
    },
    "id": "/subscriptions/4f9b81be-fa32-4f96-aeb3-fc5c3f678df9/resourcegroups/test-group/providers/microsoft.insights/components/test-rule/providers/Microsoft.AlertsManagement/alerts/7daf8739-ca8a-4562-b69a-ff28db4ba0a5",
    "type": "Microsoft.AlertsManagement/alerts",
    "name": "Failure Anomalies - test-rule"
}
```

## <a name="triage-and-diagnose-an-alert"></a>Triage et diagnostic d’une alerte

Une alerte indique qu’une augmentation anormale du taux de demandes ayant échoué a été détectée. Il est probable que votre application ou son environnement rencontre un problème.

Pour approfondir vos recherches, cliquez sur les liens « Afficher les détails complets dans Application Insights » dans cette page pour accéder directement à une [page de recherche](./diagnostic-search.md) filtrée sur les requêtes, l’exception, la dépendance ou les journaux de suivi pertinents. 

Vous pouvez également ouvrir le [portail Azure](https://portal.azure.com), accéder à la ressource Application Insights pour votre application et ouvrir la page Échecs.

Vous pouvez cliquer sur « Diagnostiquer les échecs » pour obtenir plus de détails et résoudre le problème plus aisément.

:::image type="content" source="./media/proactive-failure-diagnostics/051.png" alt-text="Recherche de diagnostic." lightbox="./media/proactive-failure-diagnostics/051.png#lightbox":::

D’après le pourcentage de requêtes et le nombre d’utilisateurs touchés, vous pouvez évaluer l’urgence du problème. Dans l’exemple ci-dessus, le taux d’échec de 78,5 % est comparé à un taux normal de 2,2 %, ce qui indique un problème. En revanche, seuls 46 utilisateurs ont été impactés. S’il s’agissait de votre application, vous pourriez évaluer le niveau de gravité.

Dans de nombreux cas, vous serez en mesure de diagnostiquer le problème rapidement à partir du nom de la requête, de l’exception, de l’échec de dépendance et du journal de suivi fournis.

Dans cet exemple, une exception est survenue dans SQL Database à cause du dépassement de la limite de requêtes.

:::image type="content" source="./media/proactive-failure-diagnostics/052.png" alt-text="Détails des requêtes ayant échoué." lightbox="./media/proactive-failure-diagnostics/052.png":::

## <a name="review-recent-alerts"></a>Consulter les alertes récentes

Cliquez sur **Alertes** dans la page de ressources Application Insights pour accéder aux alertes déclenchées les plus récentes :

:::image type="content" source="./media/proactive-failure-diagnostics/070.png" alt-text="Résumé des alertes." lightbox="./media/proactive-failure-diagnostics/070.png":::

## <a name="whats-the-difference-"></a>Quelle est la différence ?
La détection intelligente des anomalies de type échec vient compléter d’autres fonctionnalités d’Application Insights similaires, mais distinctes.

* Les [alertes de métrique](../platform/alerts-log.md), définies par vos soins, permettent de superviser un large éventail de métriques, comme l’occupation du processeur, les taux de demandes, le temps de chargement des pages, etc. Vous pouvez les utiliser pour savoir si vous devez ajouter des ressources, par exemple. En revanche, la détection intelligente des anomalies de type échec ne couvre qu’une petite gamme de métriques critiques (pour l’instant, le taux de requêtes en échec uniquement), conçues pour vous avertir en quasi temps réel si le taux de requêtes en échec de votre application web augmente par rapport à la normale. Contrairement aux alertes métriques, la détection intelligente définit et met à jour automatiquement les seuils en réponse aux changements dans le comportement. La détection intelligente démarre également le travail de diagnostic à votre place, ce qui vous aide à résoudre plus vite les problèmes.

* La [détection intelligente des anomalies de performance](proactive-performance-diagnostics.md) utilise également l’intelligence artificielle pour découvrir des modèles inhabituels dans vos mesures, sans qu’aucune configuration ne soit nécessaire. Toutefois, contrairement à la détection intelligente des anomalies de type échec, l’objectif de la détection intelligente des anomalies de performance est de repérer les segments de votre collecteur d’utilisation qui peuvent être mal pris en charge (par certaines pages sur un certain type de navigateur, par exemple). L’analyse est effectuée tous les jours, et si elle renvoie un résultat, il est souvent bien moins urgent qu’une alerte. En revanche, l’analyse des anomalies de type échec est exécutée en continu sur les données d’application entrantes. Si le taux d’échec du serveur est plus élevé que prévu, vous en êtes averti en quelques minutes.

## <a name="if-you-receive-a-smart-detection-alert"></a>Si vous recevez une alerte de la détection intelligente
*Pourquoi ai-je reçu cette alerte ?*

* Nous avons détecté une augmentation anormale du taux de demandes ayant échoué par rapport au taux de référence de la période précédente. Après analyse des échecs et des données d’application associées, nous pensons qu’il existe un problème que vous devez examiner.

*La notification signifie-t-elle obligatoirement que mon application rencontre un problème ?*

* Nous essayons de vous alerter sur l’interruption ou la dégradation de l’application, mais vous êtes la seule personne habilitée à comprendre pleinement la sémantique et l’impact sur l’application ou les utilisateurs.

*Donc, vous examinez les données de mon application ?*

* Non. Le service est entièrement automatique. Vous seul obtenez ces notifications. Vos données sont [privées](./data-retention-privacy.md).

*Dois-je m’abonner à cette alerte ?*

* Non. Chaque application qui envoie des données de requête inclut la règle d’alerte de la détection intelligente.

*Puis-je me désabonner ou obtenir des notifications envoyées à mes collègues ?*

* Oui. Dans les règles d’alerte, cliquez sur la règle de la détection intelligente pour la configurer. Vous pouvez désactiver cette alerte, ou modifier les destinataires de l’alerte.

*J’ai perdu le courrier électronique Où puis-je trouver les notifications dans le portail ?*

* Dans les journaux d’activité. Dans Azure, ouvrez la ressource Application Insights correspondant à votre application, puis sélectionnez Journaux d’activité.

*Certaines des alertes concernent des problèmes connus, et je ne souhaite pas les recevoir.*

* Vous pouvez utiliser la fonctionnalité de suppression des [règles d’action pour les alertes](../platform/alerts-action-rules.md).

## <a name="next-steps"></a>Étapes suivantes
Ces outils de diagnostic vous aident à inspecter les données reçues de votre application :

* [Metrics Explorer](../platform/metrics-charts.md)
* [Navigateur de recherche](./diagnostic-search.md)
* [Analytics : un puissant langage de requête](../log-query/log-analytics-tutorial.md)

Les détections intelligentes sont automatiques. Mais vous souhaitez peut-être configurer des alertes supplémentaires ?

* [Alertes de mesures configurées manuellement](../platform/alerts-log.md)
* [Tests web de disponibilité](./monitor-web-app-availability.md)