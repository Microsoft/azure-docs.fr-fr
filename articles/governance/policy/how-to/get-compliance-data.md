---
title: Obtenir les données de conformité de la stratégie
description: Les évaluations et les effets d’Azure Policy déterminent la conformité. Découvrez comment obtenir des détails sur la conformité de vos ressources Azure.
ms.date: 10/05/2020
ms.topic: how-to
ms.openlocfilehash: 112badce00ec56df0f80c7b51bb4789a414cdcbd
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94920234"
---
# <a name="get-compliance-data-of-azure-resources"></a>Obtenir les données de conformité des ressources Azure

Azure Policy offre, entre autres avantages, un insight et des contrôles sur les ressources d’un abonnement ou un [groupe d’administration](../../management-groups/overview.md) d’abonnements. Ce contrôle peut être effectué de différentes façons, notamment en empêchant la création de ressources au mauvais emplacement, en appliquant une utilisation commune et cohérente des balises ou en auditant les ressources existantes pour vérifier l’adéquation des configurations et paramètres. Dans tous les cas, les données générées par Azure Policy vous permettent de comprendre l’état de conformité de votre environnement.

Il existe plusieurs façons d’accéder aux informations de conformité générées par votre stratégie et vos affectations initiatives :

- À l’aide du [portail Azure](#portal)
- À l’aide de scripts de [ligne de commande](#command-line)

Avant d’examiner les méthodes de rapport sur la conformité, voyons à quel moment les informations de conformité sont mises à jour et passons en revue la fréquence et les événements qui déclenchent un cycle d’évaluation.

> [!WARNING]
> Si l’état de conformité indiqué est **Non inscrit**, vérifiez que le fournisseur de ressources **Microsoft.PolicyInsights** est inscrit et que l’utilisateur dispose d’autorisations de contrôle d’accès en fonction du rôle Azure (Azure RBAC) appropriées, comme décrit dans [Autorisations Azure RBAC dans Azure Policy](../overview.md#azure-rbac-permissions-in-azure-policy).

## <a name="evaluation-triggers"></a>Déclencheurs d’évaluation

Les résultats d’un cycle d’évaluation terminé sont disponibles dans le fournisseur de ressources `Microsoft.PolicyInsights` via les opérations `PolicyStates` et `PolicyEvents`. Pour plus d’informations sur les opérations de l’API REST Azure Policy Insights, consultez la page [Azure Policy Insights](/rest/api/policy-insights/).

Différents événements permettent d’évaluer les stratégies et initiatives assignées :

- Affectation d’une nouvelle stratégie ou initiative à une étendue. Il faut environ 30 minutes pour que l’affectation soit appliquée à l’étendue définie. Une fois celle-ci appliquée, le cycle d’évaluation commence pour les ressources de cette étendue compte tenu de la nouvelle stratégie ou initiative ; de plus, selon les effets utilisés par la stratégie ou l’initiative, les ressources sont marquées comme conformes, non conformes ou exemptées. Une stratégie ou une initiative volumineuse évaluée par rapport à une grande étendue de ressources peut prendre du temps. Par conséquent, il est impossible de déterminer à l’avance à quel moment s’achèvera le cycle d’évaluation. Une fois le cycle terminé, les résultats de conformité à jour sont disponibles dans le portail et dans les kits de développement logiciel.

- Mise à jour d’une stratégie ou initiative déjà assignée à une étendue. Dans ce scénario, le cycle et le temps d’évaluation sont les mêmes que pour le cas d’une nouvelle affectation à une étendue.

- Déploiement ou mise à jour d’une ressource dans une étendue avec une assignation via Azure Resource Manager, l’API REST ou un kit SDK pris en charge. Dans ce scénario, l’événement d’effet (ajout, audit, refus, déploiement) et l’état de conformité deviennent disponibles dans le portail et les Kits de développement logiciel (SDK) environ 15 minutes plus tard. Cet événement n’entraîne pas une évaluation des autres ressources.

- Une [exemption de stratégie](../concepts/exemption-structure.md) est créée, mise à jour ou supprimée. Dans ce scénario, l’affectation correspondante est évaluée pour l’étendue d’exemption définie.

- Cycle d’évaluation de conformité standard. Les affectations sont automatiquement réévaluées une fois par tranche de 24 heures. L’évaluation d’une stratégie ou d’une initiative volumineuse peut prendre un temps. Il est donc impossible de déterminer à l’avance à quel moment s’achèvera le cycle d’évaluation. Une fois le cycle terminé, les résultats de conformité à jour sont disponibles dans le portail et dans les kits de développement logiciel.

- Le fournisseur de ressources [Configuration d'invité](../concepts/guest-configuration.md) est mis à jour avec les détails de conformité par une ressource managée.

- Analyse de la demande

### <a name="on-demand-evaluation-scan"></a>Analyse d’évaluation à la demande

Une analyse d’évaluation d’un abonnement ou d’un groupe de ressources peut être démarrée en utilisant Azure CLI, Azure PowerShell, un appel à l’API REST ou l’[action GitHub pour l’analyse de conformité Azure Policy](https://github.com/marketplace/actions/azure-policy-compliance-scan).
Il s’agit d’un processus asynchrone.

#### <a name="on-demand-evaluation-scan---github-action"></a>Analyse d’évaluation à la demande – Action GitHub

Utilisez l’[action Analyse de conformité Azure Policy](https://github.com/marketplace/actions/azure-policy-compliance-scan) pour déclencher une analyse d’évaluation à la demande à partir de votre [workflow GitHub](https://docs.github.com/actions/configuring-and-managing-workflows/configuring-a-workflow#about-workflows) sur un nombre quelconque de ressources, de groupes de ressources ou d’abonnements et pour contrôler le workflow en fonction de l’état de conformité des ressources. Vous pouvez également configurer le workflow pour qu’il s’exécute à une heure planifiée afin d’obtenir l’état de conformité le plus récent au moment opportun. Si vous le souhaitez, cette action GitHub peut générer un rapport sur l’état de conformité des ressources analysées en vue d’une analyse plus poussée ou de l’archivage.

L’exemple suivant exécute une analyse de conformité pour un abonnement. 

```yaml
on:
  schedule:    
    - cron:  '0 8 * * *'  # runs every morning 8am
jobs:
  assess-policy-compliance:    
    runs-on: ubuntu-latest
    steps:         
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{secrets.AZURE_CREDENTIALS}} 

    
    - name: Check for resource compliance
      uses: azure/policy-compliance-scan@v0
      with:
        scopes: |
          /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Pour plus d’informations et pour obtenir des exemples de workflow, consultez le [référentiel Action GitHub pour l’analyse de conformité Azure Policy](https://github.com/Azure/policy-compliance-scan).

#### <a name="on-demand-evaluation-scan---azure-cli"></a>Analyse d’évaluation à la demande – Azure CLI

L’analyse de conformité est lancée avec la commande [az policy state trigger-scan](/cli/azure/policy/state#az_policy_state_trigger_scan).

Par défaut, `az policy state trigger-scan` démarre une évaluation pour toutes les ressources de l’abonnement actuel. Pour démarrer une évaluation sur un groupe de ressources spécifique, utilisez le paramètre **resource-group**. L’exemple suivant démarre une analyse de conformité dans l’abonnement actuel pour le groupe de ressources _MyRG_ :

```azurecli-interactive
az policy state trigger-scan --resource-group "MyRG"
```

Vous pouvez choisir de ne pas attendre que le processus asynchrone soit terminé avant de poursuivre avec le paramètre **no-wait**.

#### <a name="on-demand-evaluation-scan---azure-powershell"></a>Analyse d’évaluation à la demande – Azure PowerShell

L’analyse de conformité est lancée à l’aide de la cmdlet [Start-AzPolicyComplianceScan](/powershell/module/az.policyinsights/start-azpolicycompliancescan).

Par défaut, `Start-AzPolicyComplianceScan` démarre une évaluation pour toutes les ressources de l’abonnement actuel. Pour démarrer une évaluation sur un groupe de ressources spécifique, utilisez le paramètre **ResourceGroupName**. L’exemple suivant démarre une analyse de conformité dans l’abonnement actuel pour le groupe de ressources _MyRG_ :

```azurepowershell-interactive
Start-AzPolicyComplianceScan -ResourceGroupName 'MyRG'
```

Vous pouvez faire en sorte que PowerShell attende la fin de l’appel asynchrone avant de fournir la sortie des résultats ou exécuter l’analyse en arrière-plan sous la forme d’un [travail](/powershell/module/microsoft.powershell.core/about/about_jobs). Afin d’utiliser un travail PowerShell pour exécuter l’analyse de conformité en arrière-plan, utilisez le paramètre **AsJob** et définissez la valeur sur un objet, comme `$job` dans cet exemple :

```azurepowershell-interactive
$job = Start-AzPolicyComplianceScan -AsJob
```

Vous pouvez vérifier l’état du travail en vérifiant l’objet `$job`. Le travail est du type `Microsoft.Azure.Commands.Common.AzureLongRunningJob`. Utilisez `Get-Member` sur l’objet `$job` pour afficher les propriétés et les méthodes disponibles.

Pendant l’exécution de l’analyse de conformité, la vérification de l’objet `$job` génère des résultats tels que les suivants :

```azurepowershell-interactive
$job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
2      Long Running O… AzureLongRunni… Running       True            localhost            Start-AzPolicyCompliance…
```

Lorsque l’analyse de conformité est terminée, la propriété **État** prend la valeur _Terminé_.

#### <a name="on-demand-evaluation-scan---rest"></a>Analyse d’évaluation à la demande – REST

En tant que processus asynchrone, le point de terminaison REST pour démarrer l’analyse n’attend pas la fin de l’analyse pour répondre. Au lieu de cela, il fournit un URI pour interroger l’état de l’évaluation demandée.

Dans chaque URI d’API REST, vous devez remplacer les variables utilisées par vos propres valeurs :

- `{YourRG}` - À remplacer par le nom de votre groupe de ressources
- Remplacer `{subscriptionId}` par votre ID d’abonnement

L’analyse prend en charge l’évaluation des ressources dans un abonnement ou dans un groupe de ressources. Lancez une analyse pour une étendue avec une commande **POST** d’API REST en utilisant les structures d’URI suivantes :

- Abonnement

  ```http
  POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/triggerEvaluation?api-version=2019-10-01
  ```

- Resource group

  ```http
  POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{YourRG}/providers/Microsoft.PolicyInsights/policyStates/latest/triggerEvaluation?api-version=2019-10-01
  ```

L’appel retourne un état **202 Accepté**. Une propriété **Emplacement** est incluse dans l’en-tête de la réponse avec le format suivant :

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/asyncOperationResults/{ResourceContainerGUID}?api-version=2019-10-01
```

`{ResourceContainerGUID}` est généré de manière statique pour l’étendue demandée. Si une étendue exécute déjà une analyse à la demande, aucune nouvelle analyse n’est démarrée. Au lieu de cela, le même URI `{ResourceContainerGUID}` **d’emplacement** pour l’état est fourni à la nouvelle requête. Une commande **GET** d’API REST à l’URI **d’emplacement** retourne un **202 Accepté** tandis que l’évaluation est en cours. Une fois l’analyse de l’évaluation terminée, elle retourne un état **200 OK**. Le corps d’une analyse terminée est une réponse JSON avec l’état :

```json
{
    "status": "Succeeded"
}
```

#### <a name="on-demand-evaluation-scan---visual-studio-code"></a>Analyse d’évaluation à la demande-Visual Studio Code

L’extension Azure Policy pour Visual Studio Code peut exécuter une analyse d’évaluation pour une ressource spécifique. Cette analyse est un processus synchrone, contrairement aux méthodes Azure PowerShell et REST.
Pour plus d’informations et pour connaître les étapes à suivre, consultez [Évaluation à la demande avec l’extension VS Code](./extension-for-vscode.md#on-demand-evaluation-scan).

## <a name="how-compliance-works"></a>Principe de fonctionnement de la conformité

Dans une affectation, une ressource est dite **Non conforme** si elle ne respecte pas les règles de l’initiative ou de la stratégie et n’est pas _exemptée_. Le tableau suivant montre comment les différents effets des stratégies fonctionnent avec l’évaluation des conditions pour l’état de conformité résultant :

| État de la ressource | Résultat | Évaluation de a stratégie | État de conformité |
| --- | --- | --- | --- |
| Nouveauté ou mise à jour | Audit, Modify, AuditIfNotExist | True | Non conforme |
| Nouveauté ou mise à jour | Audit, Modify, AuditIfNotExist | False | Conforme |
| Exists | Deny, Audit, Append, Modify, DeployIfNotExist, AuditIfNotExist | True | Non conforme |
| Exists | Deny, Audit, Append, Modify, DeployIfNotExist, AuditIfNotExist | False | Conforme |

> [!NOTE]
> Les effets DeployIfNotExist et AuditIfNotExist nécessitent que l’instruction IF ait pour valeur TRUE et que la condition d’existence ait pour valeur FALSE pour être non conformes. Lorsque la valeur est TRUE, la condition IF déclenche l’évaluation de la condition d’existence pour les ressources associées.

Supposons, par exemple, que vous disposiez d’un groupe de ressources (ContosoRG), comprenant des comptes de stockage (en rouge) qui sont exposés sur des réseaux publics.

:::image type="complex" source="../media/getting-compliance-data/resource-group01.png" alt-text="Diagramme de comptes de stockage exposés sur des réseaux publics dans le groupe de ressources Contoso R G." border="false":::
   Diagramme montrant des images pour cinq comptes de stockage dans le groupe de ressources Contoso R G.  Les comptes de stockage un et trois sont en bleu, tandis que les comptes de stockage deux, quatre et cinq sont en rouge.
:::image-end:::

Dans cet exemple, vous devez faire attention aux risques de sécurité. Maintenant que vous avez créé une affectation de stratégie, elle est évaluée pour tous les comptes de stockage inclus et non exemptés du groupe de ressources ContosoRG. Elle effectue l’audit des trois comptes de stockage non conformes et en modifie l’état en conséquence pour afficher un état **Non conforme**.

:::image type="complex" source="../media/getting-compliance-data/resource-group03.png" alt-text="Diagramme de la conformité de compte de stockage dans le groupe de ressources Contoso R G." border="false":::
   Diagramme montrant des images pour cinq comptes de stockage dans le groupe de ressources Contoso R G. Des coches vertes apparaissent désormais en dessous des comptes de stockage un et trois, tandis que des signes d’avertissement rouges apparaissent désormais sous les comptes de stockage deux, quatre et cinq.
:::image-end:::

Outre les états **Conforme** et **Non conforme**, les stratégies et les ressources peuvent avoir quatre autres états :

- **Exempté** : La ressource se trouve dans l’étendue d’une affectation, mais a une [exemption définie](../concepts/exemption-structure.md).
- **En conflit** : Il existe deux définitions de stratégie ou plus avec des règles en conflit. Par exemple, deux définitions de stratégie ajoutent la même balise avec des valeurs différentes.
- **Non démarré** : Le cycle d’évaluation n’a pas démarré pour la stratégie ou la ressource.
- **Non inscrit** : Le fournisseur de ressources Azure Policy n’a pas été inscrit ou le compte connecté n’est pas autorisé à lire les données de conformité.

Azure Policy utilise les champs **type**, **nom** et **genre** de la définition pour déterminer si une ressource correspond. Lorsque la ressource correspond, elle est considérée comme applicable et présente l’état **Conforme**, **Non conforme** ou **Exempté**. Si le champ **type**, **nom** ou **genre** est la seule propriété dans la définition, toutes les ressources incluses et non exemptées sont considérées comme applicables et sont évaluées.

Le pourcentage de conformité est déterminé en divisant le nombre de ressources **conformes** et **exemptées** par le _nombre total de ressources_. Le _nombre total de ressources_ est défini comme étant la somme des ressources **conformes**, **non conformes**, **exemptées** et **en conflit**. La conformité globale est la somme des ressources distinctes **conformes** ou **exemptées** divisée par la somme de toutes les ressources distinctes. Dans l’image ci-dessous, il y a 20 ressources distinctes applicables et une seule **non conforme**.
La conformité globale des ressources est égale à 95 % (soit 19 sur 20).

:::image type="content" source="../media/getting-compliance-data/simple-compliance.png" alt-text="Capture d’écran des détails de conformité à la stratégie dans la page Conformité." border="false":::

> [!NOTE]
> La conformité réglementaire dans Azure Policy est une fonctionnalité en version préliminaire. Les propriétés de conformité du Kit de développement logiciel (SDK) et des pages dans le portail sont différentes pour les initiatives activées. Pour plus d’informations, voir [Conformité réglementaire](../concepts/regulatory-compliance.md)

## <a name="portal"></a>Portail

Le portail Azure permet de visualiser et comprendre l’état de conformité de votre environnement selon une représentation graphique. Sur la page **Stratégie**, l’option **Vue d’ensemble** fournit des détails sur les étendues disponibles pour la conformité des stratégies et des initiatives. En complément de l’état de conformité et du nombre par affectation, elle contient un graphique retraçant la conformité au cours des sept derniers jours. La page **Conformité** regroupe essentiellement les mêmes informations (à l’exception du graphique), mais avec également des options de tri et de filtrage supplémentaires.

:::image type="content" source="../media/getting-compliance-data/compliance-page.png" alt-text="Capture d’écran de la page Conformité, des options de filtrage et des détails." border="false":::

Comme une stratégie ou une initiative peut être affectée à différentes étendues, le tableau comprend l’étendue pour chaque affectation et le type de définition qui a été affecté. Le nombre de ressources et de stratégies non conformes est aussi indiqué pour chaque affectation. En sélectionnant une stratégie ou une initiative dans le tableau, vous obtenez davantage de détails sur la conformité de l’affectation concernée.

:::image type="content" source="../media/getting-compliance-data/compliance-details.png" alt-text="Capture d’écran de la page Détails de conformité, y compris les nombres et les détails de conformité des ressources." border="false":::

La liste des ressources dans l’onglet **Resource compliance (Conformité des ressources)** affiche l’état de l’évaluation des ressources existantes pour l’affectation actuelle. Par défaut, l’onglet est défini sur **Non conforme**, mais un filtre peut être appliqué.
Les événements (Append, Audit, Deny, Deploy, Modify) déclenchés par la requête pour créer une ressource sont affichés dans l’onglet **Événements**.

> [!NOTE]
> Pour une stratégie du moteur AKS, la ressource indiquée est le groupe de ressources.

:::image type="content" source="../media/getting-compliance-data/compliance-events.png" alt-text="Capture d’écran de l’onglet Événements sur la page Détails de conformité." border="false":::

<a name="component-compliance"></a> Pour les ressources du [mode Fournisseur de ressources](../concepts/definition-structure.md#resource-provider-modes), dans l’onglet **Conformité des ressources**, la sélection de la ressource ou un clic droit sur la ligne et la sélection de l’option **Afficher les détails de la conformité** ouvre les détails de conformité du composant. Cette page propose également des onglets pour afficher les stratégies attribuées à cette ressource, les événements, les événements de composant et l’historique des modifications.

:::image type="content" source="../media/getting-compliance-data/compliance-components.png" alt-text="Capture d’écran de l’onglet Conformité des composants et des détails de conformité pour une affectation de mode de fournisseur de ressources." border="false":::

Une fois de retour sur la page de conformité des ressources, cliquez avec le bouton droit sur la ligne de l’événement pour lequel vous souhaitez obtenir plus de détails et sélectionnez **Afficher les journaux d’activité**. La page Journal d’activité s’ouvre et les critères de recherche sont préfiltrés pour montrer les détails de l’affectation et des événements. Le journal d’activité fournit davantage de contexte ainsi que des informations supplémentaires sur ces événements.

:::image type="content" source="../media/getting-compliance-data/compliance-activitylog.png" alt-text="Capture d’écran du journal d’activité pour les activités et évaluations Azure Policy." border="false":::

### <a name="understand-non-compliance"></a>Comprendre la non-conformité

Lorsque le système détermine qu’une ressource est **non conforme**, plusieurs raisons justifient cela. Pour déterminer la raison d’une **non conformité** d’une ressource ou pour rechercher le ou la responsable de la modification, veuillez consulter [Déterminer une non-conformité](./determine-non-compliance.md).

## <a name="command-line"></a>Ligne de commande

Les informations disponibles dans le portail peuvent être récupérées à l’aide de l’API REST (y compris avec [ARMClient](https://github.com/projectkudu/ARMClient)), d’Azure PowerShell ou d’Azure CLI. Pour plus d’informations sur l’API REST, consultez la référence [Azure Policy Insights](/rest/api/policy-insights/). Les pages de référence de l’API REST contiennent, pour chaque opération, un bouton vert qui vous permet de la tester directement dans votre navigateur.

Utilisez ARMClient ou un outil similaire pour gérer l’authentification auprès d’Azure pour les exemples de l’API REST.

### <a name="summarize-results"></a>Synthétiser les résultats

Avec l’API REST, la synthèse peut être effectuée par conteneur, par définition ou par affectation. Voici un exemple de synthèse obtenu au niveau de l’abonnement à l’aide de la fonction de [résumé de l’abonnement](/rest/api/policy-insights/policystates/summarizeforsubscription) de Azure Policy Insights :

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/summarize?api-version=2019-10-01
```

La sortie offre une synthèse de l’abonnement. Dans l’exemple de sortie ci-dessous, la conformité est synthétisée sous **value.results.nonCompliantResources** et **value.results.nonCompliantPolicies**. Cette requête fournit des détails supplémentaires, notamment sur chaque affectation concernée par la non-conformité, fournit des informations sur la définition de chaque affectation. Chaque objet de stratégie dans la hiérarchie fournit un **queryResultsUri** qui peut être utilisé pour obtenir des détails supplémentaires à ce niveau.

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary",
    "@odata.count": 1,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary/$entity",
        "results": {
            "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant'",
            "nonCompliantResources": 15,
            "nonCompliantPolicies": 1
        },
        "policyAssignments": [{
            "policyAssignmentId": "/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77",
            "policySetDefinitionId": "",
            "results": {
                "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant' and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77'",
                "nonCompliantResources": 15,
                "nonCompliantPolicies": 1
            },
            "policyDefinitions": [{
                "policyDefinitionReferenceId": "",
                "policyDefinitionId": "/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "effect": "deny",
                "results": {
                    "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant' and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77' and PolicyDefinitionId eq '/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62'",
                    "nonCompliantResources": 15
                }
            }]
        }]
    }]
}
```

### <a name="query-for-resources"></a>Interroger des ressources

En reprenant l’exemple ci-dessus, **value.policyAssignments.policyDefinitions.results.queryResultsUri** nous donne un exemple d’URI pour toutes les ressources non conformes pour une définition de stratégie spécifique. Si l’on observe la valeur **$filter**, on remarque que ComplianceState est égal (eq) à « NonCompliant «, que PolicyAssignmentId est spécifié pour la définition de stratégie, puis pour PolicyDefinitionId proprement dit. Le PolicyAssignmentId est inclus dans le filtre car le PolicyDefinitionId peut figurer dans plusieurs stratégies ou affectations initiatives avec une diversité d’étendues. En spécifiant à la fois le PolicyAssignmentId et le PolicyDefinitionId, nous pouvons être explicites dans les résultats que nous recherchons. Auparavant, nous utilisions la valeur **latest (dernières)** pour PolicyStates : celle-ci définit automatiquement les paramètres **from (depuis)** et **to (jusqu’à)** d’une plage de temps dans les dernières 24 heures.

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant' and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77' and PolicyDefinitionId eq '/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62'
```

L’exemple de réponse ci-dessous a été ramené à une seule ressource non conforme par souci de concision. La réponse détaillée fournit plusieurs éléments de données sur la ressource, la stratégie ou initiative, et l’affectation. Notez que vous pouvez également voir quels paramètres d’affectation ont été transmis à la définition de la stratégie.

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 15,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
        "timestamp": "2018-05-19T04:41:09Z",
        "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/rg-tags/providers/Microsoft.Compute/virtualMachines/linux",
        "policyAssignmentId": "/subscriptions/{subscriptionId}/resourceGroups/rg-tags/providers/Microsoft.Authorization/policyAssignments/37ce239ae4304622914f0c77",
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
        "effectiveParameters": "",
        "ComplianceState": "NonCompliant",
        "subscriptionId": "{subscriptionId}",
        "resourceType": "/Microsoft.Compute/virtualMachines",
        "resourceLocation": "westus2",
        "resourceGroup": "RG-Tags",
        "resourceTags": "tbd",
        "policyAssignmentName": "37ce239ae4304622914f0c77",
        "policyAssignmentOwner": "tbd",
        "policyAssignmentParameters": "{\"tagName\":{\"value\":\"costCenter\"},\"tagValue\":{\"value\":\"Contoso-Test\"}}",
        "policyAssignmentScope": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags",
        "policyDefinitionName": "1e30110a-5ceb-460c-a204-c1c3969c6d62",
        "policyDefinitionAction": "deny",
        "policyDefinitionCategory": "tbd",
        "policySetDefinitionId": "",
        "policySetDefinitionName": "",
        "policySetDefinitionOwner": "",
        "policySetDefinitionCategory": "",
        "policySetDefinitionParameters": "",
        "managementGroupIds": "",
        "policyDefinitionReferenceId": ""
    }]
}
```

### <a name="view-events"></a>Afficher les événements

Lorsqu’une ressource est créée ou mise à jour, un résultat d’évaluation de stratégie est généré. Ces résultats sont appelés _événements de stratégie_. L’URI suivant permet d’afficher les événements de stratégie récents associés à l’abonnement.

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/default/queryResults?api-version=2019-10-01
```

Vos résultats doivent ressembler à l’exemple suivant :

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default",
    "@odata.count": 1,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default/$entity",
        "NumAuditEvents": 16
    }]
}
```

Pour plus d’informations sur l’interrogation des événements de stratégie, consultez l’article de référence intitulé [Événements Azure Policy](/rest/api/policy-insights/policyevents).

### <a name="azure-cli"></a>Azure CLI

Le groupe de commande [Azure CLI](/cli/azure/what-is-azure-cli) pour Azure Policy couvre la plupart des opérations disponibles dans REST ou dans Azure PowerShell. Pour obtenir la liste complète des commandes disponibles, consultez [Vue d’ensemble Azure CLI – Azure Policy ](/cli/azure/policy).

Exemple : Obtenir le résumé d’état de la stratégie affectée au premier plan qui comporte le plus grand nombre de ressources non conformes.

```azurecli-interactive
az policy state summarize --top 1
```

La partie supérieure de la réponse ressemble à l’exemple suivant :

```json
{
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary/$entity",
    "odataid": null,
    "policyAssignments": [{
            "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8",
            "policyDefinitions": [{
                "effect": "audit",
                "policyDefinitionGroupNames": [
                    ""
                ],
                "policyDefinitionId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
                "policyDefinitionReferenceId": "",
                "results": {
                    "nonCompliantPolicies": null,
                    "nonCompliantResources": 398,
                    "policyDetails": [{
                        "complianceState": "noncompliant",
                        "count": 1
                    }],
                    "policyGroupDetails": [{
                        "complianceState": "noncompliant",
                        "count": 1
                    }],
                    "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2020-07-14 14:01:22Z&$to=2020-07-15 14:01:22Z and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8' and PolicyDefinitionId eq '/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a'",
                    "resourceDetails": [{
                            "complianceState": "noncompliant",
                            "count": 398
                        },
                        {
                            "complianceState": "compliant",
                            "count": 4
                        }
                    ]
                }
            }],
    ...
```

Exemple : Obtenir l’enregistrement de l’état pour la ressource évaluée la plus récemment (par défaut, selon un horodatage dans l’ordre décroissant).

```azurecli-interactive
az policy state list --top 1
```

```json
[
  {
    "complianceReasonCode": "",
    "complianceState": "Compliant",
    "effectiveParameters": "",
    "isCompliant": true,
    "managementGroupIds": "{managementgroupId}",
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
    "odataid": null,
    "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/securitycenterbuiltin",
    "policyAssignmentName": "SecurityCenterBuiltIn",
    "policyAssignmentOwner": "tbd",
    "policyAssignmentParameters": "",
    "policyAssignmentScope": "/subscriptions/{subscriptionId}",
    "policyAssignmentVersion": "",
    "policyDefinitionAction": "auditifnotexists",
    "policyDefinitionCategory": "tbd",
    "policyDefinitionGroupNames": [
      ""
    ],
    "policyDefinitionId": "/providers/microsoft.authorization/policydefinitions/aa633080-8b72-40c4-a2d7-d00c03e80bed",
    "policyDefinitionName": "aa633080-8b72-40c4-a2d7-d00c03e80bed",
    "policyDefinitionReferenceId": "identityenablemfaforownerpermissionsmonitoring",
    "policyDefinitionVersion": "",
    "policyEvaluationDetails": null,
    "policySetDefinitionCategory": "security center",
    "policySetDefinitionId": "/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8",
    "policySetDefinitionName": "1f3afdf9-d0c9-4c3d-847f-89da613e70a8",
    "policySetDefinitionOwner": "",
    "policySetDefinitionParameters": "",
    "policySetDefinitionVersion": "",
    "resourceGroup": "",
    "resourceId": "/subscriptions/{subscriptionId}",
    "resourceLocation": "",
    "resourceTags": "tbd",
    "resourceType": "Microsoft.Resources/subscriptions",
    "subscriptionId": "{subscriptionId}",
    "timestamp": "2020-07-15T08:37:07.903433+00:00"
  }
]
```

Exemple : Obtenir des détails pour toutes les ressources non conformes du réseau virtuel.

```azurecli-interactive
az policy state list --filter "ResourceType eq 'Microsoft.Network/virtualNetworks'"
```

```json
[
  {
    "complianceReasonCode": "",
    "complianceState": "NonCompliant",
    "effectiveParameters": "",
    "isCompliant": false,
    "managementGroupIds": "{managementgroupId}",
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
    "odataid": null,
    "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8",
    "policyAssignmentName": "e0704696df5e4c3c81c873e8",
    "policyAssignmentOwner": "tbd",
    "policyAssignmentParameters": "",
    "policyAssignmentScope": "/subscriptions/{subscriptionId}",
    "policyAssignmentVersion": "",
    "policyDefinitionAction": "audit",
    "policyDefinitionCategory": "tbd",
    "policyDefinitionGroupNames": [
      ""
    ],
    "policyDefinitionId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionName": "2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionReferenceId": "",
    "policyDefinitionVersion": "",
    "policyEvaluationDetails": null,
    "policySetDefinitionCategory": "",
    "policySetDefinitionId": "",
    "policySetDefinitionName": "",
    "policySetDefinitionOwner": "",
    "policySetDefinitionParameters": "",
    "policySetDefinitionVersion": "",
    "resourceGroup": "RG-Tags",
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Microsoft.Network/virtualNetworks/RG-Tags-vnet",
    "resourceLocation": "westus2",
    "resourceTags": "tbd",
    "resourceType": "Microsoft.Network/virtualNetworks",
    "subscriptionId": "{subscriptionId}",
    "timestamp": "2020-07-15T08:37:07.901911+00:00"
  }
]
```

Exemple : Obtenir des événements liés aux ressources non conformes du réseau virtuel qui se sont produits après une date spécifique.

```azurecli-interactive
az policy state list --filter "ResourceType eq 'Microsoft.Network/virtualNetworks'" --from '2020-07-14T00:00:00Z'
```

```json
[
  {
    "complianceReasonCode": "",
    "complianceState": "NonCompliant",
    "effectiveParameters": "",
    "isCompliant": false,
    "managementGroupIds": "{managementgroupId}",
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
    "odataid": null,
    "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8",
    "policyAssignmentName": "e0704696df5e4c3c81c873e8",
    "policyAssignmentOwner": "tbd",
    "policyAssignmentParameters": "",
    "policyAssignmentScope": "/subscriptions/{subscriptionId}",
    "policyAssignmentVersion": "",
    "policyDefinitionAction": "audit",
    "policyDefinitionCategory": "tbd",
    "policyDefinitionGroupNames": [
      ""
    ],
    "policyDefinitionId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionName": "2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionReferenceId": "",
    "policyDefinitionVersion": "",
    "policyEvaluationDetails": null,
    "policySetDefinitionCategory": "",
    "policySetDefinitionId": "",
    "policySetDefinitionName": "",
    "policySetDefinitionOwner": "",
    "policySetDefinitionParameters": "",
    "policySetDefinitionVersion": "",
    "resourceGroup": "RG-Tags",
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Microsoft.Network/virtualNetworks/RG-Tags-vnet",
    "resourceLocation": "westus2",
    "resourceTags": "tbd",
    "resourceType": "Microsoft.Network/virtualNetworks",
    "subscriptionId": "{subscriptionId}",
    "timestamp": "2020-07-15T08:37:07.901911+00:00"
  }
]
```

### <a name="azure-powershell"></a>Azure PowerShell

Le module Azure PowerShell d’Azure Policy est disponible dans PowerShell Gallery sous le nom [Az.PolicyInsights](https://www.powershellgallery.com/packages/Az.PolicyInsights). Vous pouvez utiliser PowerShellGet pour installer le module à l’aide de `Install-Module -Name Az.PolicyInsights` (vérifiez que vous disposez de la dernière version [d’Azure PowerShell](/powershell/azure/install-az-ps)) :

```azurepowershell-interactive
# Install from PowerShell Gallery via PowerShellGet
Install-Module -Name Az.PolicyInsights

# Import the downloaded module
Import-Module Az.PolicyInsights

# Login with Connect-AzAccount if not using Cloud Shell
Connect-AzAccount
```

Le module comporte les cmdlets suivantes :

- `Get-AzPolicyStateSummary`
- `Get-AzPolicyState`
- `Get-AzPolicyEvent`
- `Get-AzPolicyRemediation`
- `Remove-AzPolicyRemediation`
- `Start-AzPolicyRemediation`
- `Stop-AzPolicyRemediation`

Exemple : Obtenir le résumé d’état de la stratégie affectée au premier plan qui comporte le plus grand nombre de ressources non conformes.

```azurepowershell-interactive
PS> Get-AzPolicyStateSummary -Top 1

NonCompliantResources : 15
NonCompliantPolicies  : 1
PolicyAssignments     : {/subscriptions/{subscriptionId}/resourcegroups/RG-Tags/providers/micros
                        oft.authorization/policyassignments/37ce239ae4304622914f0c77}
```

Exemple : Obtenir l’enregistrement de l’état pour la ressource évaluée la plus récemment (par défaut, selon un horodatage dans l’ordre décroissant).

```azurepowershell-interactive
PS> Get-AzPolicyState -Top 1

Timestamp                  : 5/22/2018 3:47:34 PM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/networkInterfaces/linux316
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
ComplianceState            : NonCompliant
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/networkInterfaces
ResourceLocation           : westus2
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
```

Exemple : Obtenir des détails pour toutes les ressources non conformes du réseau virtuel.

```azurepowershell-interactive
PS> Get-AzPolicyState -Filter "ResourceType eq '/Microsoft.Network/virtualNetworks'"

Timestamp                  : 5/22/2018 4:02:20 PM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/virtualNetworks/RG-Tags-vnet
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
ComplianceState            : NonCompliant
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/virtualNetworks
ResourceLocation           : westus2
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
```

Exemple : Obtenir des événements liés aux ressources non conformes du réseau virtuel qui se sont produits après une date spécifique, convertir vers un objet CSV et exporter dans un fichier.

```azurepowershell-interactive
$policyEvents = Get-AzPolicyEvent -Filter "ResourceType eq '/Microsoft.Network/virtualNetworks'" -From '2020-09-19'
$policyEvents | ConvertTo-Csv | Out-File 'C:\temp\policyEvents.csv'
```

La sortie de l’objet `$policyEvents` se présente comme suit :

```output
Timestamp                  : 9/19/2020 5:18:53 AM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/virtualNetworks/RG-Tags-vnet
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
ComplianceState            : NonCompliant
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/virtualNetworks
ResourceLocation           : eastus
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
TenantId                   : {tenantId}
PrincipalOid               : {principalOid}
```

Le champ **PrincipalOid** peut être utilisé pour obtenir un utilisateur spécifique avec l’applet de commande `Get-AzADUser` d’Azure PowerShell. Remplacez **{principalOid}** par la réponse que vous obtenez à partir de l’exemple précédent.

```azurepowershell-interactive
PS> (Get-AzADUser -ObjectId {principalOid}).DisplayName
Trent Baker
```

## <a name="azure-monitor-logs"></a>Journaux d’activité Azure Monitor

Si vous avez un [espace de travail Log Analytics](../../../azure-monitor/log-query/log-query-overview.md) dans lequel `AzureActivity` de la [solution Activity Log Analytics](../../../azure-monitor/platform/activity-log.md) est liée à votre abonnement, vous pouvez également afficher les résultats non conformes de l’évaluation des nouvelles ressources et de celles mises à jour en utilisant de simples requêtes Kusto et la table `AzureActivity`. Grâce aux informations des journaux d’activité Azure Monitor, des alertes peuvent être configurées de manière à signaler les problèmes de non-conformité.

:::image type="content" source="../media/getting-compliance-data/compliance-loganalytics.png" alt-text="Capture d’écran des journaux Azure Monitor montrant les actions Azure Policy dans la table AzureActivity." border="false":::

## <a name="next-steps"></a>Étapes suivantes

- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).
- Découvrez comment [créer des stratégies par programmation](programmatically-create.md).
- Découvrez comment [corriger des ressources non conformes](remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).
