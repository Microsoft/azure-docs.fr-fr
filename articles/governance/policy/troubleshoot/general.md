---
title: Résolution des erreurs courantes
description: Découvrez comment résoudre les problèmes liés à la création de définitions de stratégie, aux divers Kits de développement logiciel (SDK) et au module complémentaire pour Kubernetes.
ms.date: 12/01/2020
ms.topic: troubleshooting
ms.openlocfilehash: f3667988d527100507d308887338278e1200d454
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96510996"
---
# <a name="troubleshoot-errors-using-azure-policy"></a>Résoudre les erreurs à l’aide d’Azure Policy

Vous pouvez rencontrer des erreurs lors de la création de définitions de stratégie, de l’utilisation du Kit de développement logiciel (SDK) ou de la configuration du module complémentaire [Azure Policy pour Kubernetes](../concepts/policy-for-kubernetes.md). Cet article décrit différentes erreurs générales qui peuvent se produire, et explique comment les résoudre.

## <a name="finding-error-details"></a>Recherche des détails de l’erreur

L’emplacement des détails de l’erreur dépend de l’action à l’origine de l’erreur.

- Quand vous utilisez une stratégie personnalisée, essayez-la dans le portail Azure pour obtenir des commentaires linting sur le schéma ou passez en revue les [données de conformité](../how-to/get-compliance-data.md) qui en résultent pour voir comment les ressources ont été évaluées.
- Lorsque vous travaillez avec différents Kit de développement logiciel (SDK), le SDK fournit des détails sur la raison de l’échec de la fonction.
- Lorsque vous utilisez le module complémentaire pour Kubernetes, commencez par la [journalisation](../concepts/policy-for-kubernetes.md#logging) dans le cluster.

## <a name="general-errors"></a>Erreurs générales.

### <a name="scenario-alias-not-found"></a>Scénario : Alias introuvable

#### <a name="issue"></a>Problème

Azure Policy utilise des [alias](../concepts/definition-structure.md#aliases) à mapper aux propriétés Azure Resource Manager.

#### <a name="cause"></a>Cause

Un alias incorrect ou inexistant est utilisé dans une définition de stratégie.

#### <a name="resolution"></a>Résolution

Tout d’abord, vérifiez que la propriété Resource Manager est dotée d’un alias. Utilisez l’[extension Azure Policy pour Visual Studio Code](../how-to/extension-for-vscode.md), [Azure Resource Graph](../../resource-graph/samples/starter.md#distinct-alias-values) ou le Kit de développement logiciel (SDK) pour rechercher les alias disponibles. Si l’alias d’une propriété Resource Manager n’existe pas, créez un ticket de support.

### <a name="scenario-evaluation-details-not-up-to-date"></a>Scénario : Les détails de l’évaluation ne sont pas à jour

#### <a name="issue"></a>Problème

Une ressource est à l’état « Non démarré » ou les détails de compatibilité ne sont pas à jour.

#### <a name="cause"></a>Cause

L’application d’une nouvelle stratégie ou initiative prend environ 30 minutes. Les ressources nouvelles ou mises à jour dans l’étendue d’une attribution existante sont disponibles environ 15 minutes plus tard. Une analyse de conformité standard se produit toutes les 24 heures. Pour plus d’informations, consultez les [déclencheurs d’évaluation](../how-to/get-compliance-data.md#evaluation-triggers).

#### <a name="resolution"></a>Résolution

Tout d’abord, attendez la durée appropriée pour que l’évaluation se termine et que les résultats de conformité soient disponibles dans le portail Azure ou le Kit de développement logiciel (SDK). Pour démarrer une nouvelle analyse d’évaluation avec Azure PowerShell ou l’API REST, consultez [Analyse d’évaluation à la demande](../how-to/get-compliance-data.md#on-demand-evaluation-scan).

### <a name="scenario-compliance-not-as-expected"></a>Scénario : État de conformité non attendu

#### <a name="issue"></a>Problème

Une ressource n’a pas l’état d’évaluation, _conforme_ ou _non conforme_, qui est attendu pour cette ressource.

#### <a name="cause"></a>Cause

La ressource n’est pas dans l’étendue appropriée pour l’attribution de stratégie, ou la définition de stratégie ne fonctionne pas comme prévu.

#### <a name="resolution"></a>Résolution

Suivez ces étapes pour résoudre les problèmes liés à la définition de votre stratégie :

1. Tout d’abord, attendez la durée appropriée pour que l’évaluation se termine et que les résultats de conformité soient disponibles dans le portail Azure ou le Kit de développement logiciel (SDK). Pour démarrer une nouvelle analyse d’évaluation avec Azure PowerShell ou l’API REST, consultez [Analyse d’évaluation à la demande](../how-to/get-compliance-data.md#on-demand-evaluation-scan).
1. Vérifiez que les paramètres d’attribution et l’étendue d’attribution sont correctement définis.
1. Vérifiez le [mode de définition de la stratégie](../concepts/definition-structure.md#mode) :
   - Mode 'all' (Tous) pour tous les types de ressources.
   - Mode 'indexed' (indexé) si la définition de la stratégie recherche des balises ou un emplacement.
1. Vérifiez que l’étendue de la ressource n’est pas [exclue](../concepts/assignment-structure.md#excluded-scopes) ou [exempte](../concepts/exemption-structure.md).
1. Si la conformité d’une attribution de stratégie indique `0/0` ressources, aucune ressource n’a été déterminée comme étant applicable dans l’étendue de l’attribution. Vérifiez à la fois la définition de la stratégie et l’étendue de l’attribution.
1. Pour une ressource non conforme supposée être conforme, découvrez comment [déterminer les raisons de la non-conformité](../how-to/determine-non-compliance.md). La comparaison de la définition avec la valeur de propriété évaluée indique la raison pour laquelle une ressource n’est pas conforme.
   - Si la **valeur cible** est incorrecte, révisez la définition de la stratégie.
   - Si la **valeur actuelle** est incorrecte, validez la charge utile de la ressource par le biais de `resources.azure.com`.
1. Consultez [Résoudre des problèmes : Application non conforme aux attentes](#scenario-enforcement-not-as-expected) pour découvrir d’autres problèmes et solutions courants.

Si vous rencontrez toujours un problème avec votre définition de stratégie intégrée dupliquée et personnalisée ou une définition personnalisée, créez un ticket de support sous **Création d’une stratégie** pour acheminer le problème correctement.

### <a name="scenario-enforcement-not-as-expected"></a>Scénario : Application non conforme aux attentes

#### <a name="issue"></a>Problème

Azure Policy n’agit pas sur une ressource sur laquelle il est supposé agir et il n’y a aucune entrée dans le [journal d’activité Azure](../../../azure-monitor/platform/platform-logs-overview.md).

#### <a name="cause"></a>Cause

L’attribution de stratégie a été configurée pour [enforcementMode](../concepts/assignment-structure.md#enforcement-mode) sur _Désactivé_. Si le mode d’application est désactivé, l’effet de stratégie n’est pas appliqué et il n’y a aucune entrée dans le journal d’activité.

#### <a name="resolution"></a>Résolution

Suivez ces étapes pour résoudre les problèmes liés à l’application de l’attribution de votre stratégie :

1. Tout d’abord, attendez la durée appropriée pour que l’évaluation se termine et que les résultats de conformité soient disponibles dans le portail Azure ou le Kit de développement logiciel (SDK). Pour démarrer une nouvelle analyse d’évaluation avec Azure PowerShell ou l’API REST, consultez [Analyse d’évaluation à la demande](../how-to/get-compliance-data.md#on-demand-evaluation-scan).
1. Vérifiez que les paramètres d’attribution et l’étendue d’attribution sont correctement définis et que **enforcementMode** est défini sur _Enabled_.
1. Vérifiez le [mode de définition de la stratégie](../concepts/definition-structure.md#mode) :
   - Mode 'all' (Tous) pour tous les types de ressources.
   - Mode 'indexed' (indexé) si la définition de la stratégie recherche des balises ou un emplacement.
1. Vérifiez que l’étendue de la ressource n’est pas [exclue](../concepts/assignment-structure.md#excluded-scopes) ou [exempte](../concepts/exemption-structure.md).
1. Vérifiez que la charge utile de la ressource correspond à la logique de la stratégie. Pour ce faire, vous pouvez [capturer une trace HAR](../../../azure-portal/capture-browser-trace.md) ou consulter les propriétés du modèle ARM.
1. Consultez [Résoudre des problèmes : État de conformité non attendu](#scenario-compliance-not-as-expected) pour découvrir d’autres problèmes et solutions courants.

Si vous rencontrez toujours un problème avec votre définition de stratégie intégrée dupliquée et personnalisée ou une définition personnalisée, créez un ticket de support sous **Création d’une stratégie** pour acheminer le problème correctement.

### <a name="scenario-denied-by-azure-policy"></a>Scénario : Refusé par Azure Policy

#### <a name="issue"></a>Problème

La création ou la mise à jour d’une ressource est refusée.

#### <a name="cause"></a>Cause

Une attribution de stratégie sur l’étendue de la ressource nouvelle ou mise à jour répond aux critères d’une définition de stratégie avec un effet [Refuser](../concepts/effects.md#deny). Les ressources répondant à ces définitions ne peuvent pas être créées ni mises à jour.

#### <a name="resolution"></a>Résolution

Le message d’erreur d’une attribution de stratégie de refus comprend la définition de stratégie et les ID d’attribution de la stratégie. Si vous avez besoin de retrouver les informations d’erreur indiquées dans le message, elles sont également disponibles dans le [journal d’activité](../../../azure-monitor/platform/activity-log.md#view-the-activity-log). Utilisez ces informations pour obtenir plus de détails afin de comprendre les restrictions relatives aux ressources et d’ajuster les propriétés de ressource dans votre demande pour qu’elles correspondent aux valeurs autorisées.

## <a name="template-errors"></a>Erreurs de modèle

### <a name="scenario-policy-supported-functions-processed-by-template"></a>Scénario : Fonctions prises en charge par Policy et traitées par le modèle

#### <a name="issue"></a>Problème

Azure Policy prend en charge un certain nombre de fonctions de modèle Azure Resource Manager (modèle ARM) et de fonctions qui sont uniquement disponibles dans une définition de stratégie. Resource Manager traite ces fonctions dans le cadre d’un déploiement plutôt que dans le cadre d’une définition de stratégie.

#### <a name="cause"></a>Cause

L’utilisation de fonctions prises en charge, telles que `parameter()` ou `resourceGroup()`, permet de traiter le résultat de la fonction au moment du déploiement au lieu de laisser la tâche à la définition de stratégie et au moteur Azure Policy.

#### <a name="resolution"></a>Résolution

Pour transmettre une fonction afin qu’elle fasse partie d’une définition de stratégie, placez dans une séquence d’échappement la chaîne entière à l’aide du caractère `[` de manière à ce que la propriété ressemble à `[[resourceGroup().tags.myTag]`. Le caractère d’échappement fait que Resource Manager traite la valeur comme une chaîne lors du traitement du modèle. Azure Policy place ensuite la fonction dans la définition de stratégie, ce qui lui permet d’être dynamique comme prévu. Pour plus d’informations, voir [Syntaxe et expressions dans les modèles Azure Resource Manager](../../../azure-resource-manager/templates/template-expressions.md).

## <a name="add-on-for-kubernetes-installation-errors"></a>Module complémentaire pour les erreurs d’installation de Kubernetes

### <a name="scenario-install-using-helm-chart-fails-on-password"></a>Scénario : L’installation à l’aide du chart Helm échoue sur le mot de passe

#### <a name="issue"></a>Problème

Échec de la commande `helm install azure-policy-addon` avec l’un des messages suivants :

- `!: event not found`
- `Error: failed parsing --set data: key "<key>" has no value (cannot end with ,)`

#### <a name="cause"></a>Cause

Le mot de passe généré comprend une virgule (`,`) sur laquelle le chart Helm est fractionné.

#### <a name="resolution"></a>Résolution

Placez dans une séquence d’échappement la virgule (`,`) dans la valeur de mot de passe lors de l’exécution de `helm install azure-policy-addon` à l’aide d’une barre oblique inverse (`\`).

### <a name="scenario-install-using-helm-chart-fails-as-name-already-exists"></a>Scénario : L’installation à l’aide du chart Helm échoue, car le nom existe déjà

#### <a name="issue"></a>Problème

Échec de la commande `helm install azure-policy-addon` avec le message suivant :

- `Error: cannot re-use a name that is still in use`

#### <a name="cause"></a>Cause

Le chart Helm portant le nom `azure-policy-addon` a déjà été installé ou partiellement installé.

#### <a name="resolution"></a>Résolution

Suivez les instructions pour [supprimer la stratégie Azure pour le module complémentaire Kubernetes](../concepts/policy-for-kubernetes.md#remove-the-add-on), puis réexécutez la commande `helm install azure-policy-addon`.

### <a name="scenario-azure-virtual-machine-user-assigned-identities-are-replaced-by-system-assigned-managed-identities"></a>Scénario : Les identités attribuées par l’utilisateur de la machine virtuelle Azure sont remplacées par des identités managées attribuées par le système

#### <a name="issue"></a>Problème

Après avoir attribué des initiatives de stratégie Guest Configuration pour auditer les paramètres des ordinateurs, les identités managées attribuées par l’utilisateur et affectées à l’ordinateur ne sont plus attribuées. Seule une identité managée attribuée par le système est attribuée.

#### <a name="cause"></a>Cause

Les définitions de stratégie précédemment utilisées dans les définitions DeployIfNotExists de Guest Configuration permettaient de s’assurer qu’une identité attribuée par le système est affectée à l’ordinateur, mais supprimaient aussi les attributions d’identité attribuées par l’utilisateur.

#### <a name="resolution"></a>Résolution

Les définitions qui auparavant provoquaient ce problème apparaissent comme \[Dépréciées\] et sont remplacées par des définitions de stratégie qui gèrent les prérequis sans supprimer l’identité managée attribuée par l’utilisateur. Une étape manuelle est nécessaire. Supprimez les attributions de stratégies existantes qui sont marquées comme \[Dépréciées\] et remplacez-les par les définitions de stratégie et d’initiative de stratégie prérequises mises à jour qui portent le même nom que l’original.

Pour obtenir une description détaillée, consultez le billet de blog suivant :

[Modification importante publiée pour les stratégies d’audit de Guest Configuration](https://techcommunity.microsoft.com/t5/azure-governance-and-management/important-change-released-for-guest-configuration-audit-policies/ba-p/1655316)

## <a name="add-on-for-kubernetes-general-errors"></a>Module complémentaire pour les erreurs générales de Kubernetes

### <a name="scenario-add-on-is-unable-to-reach-the-azure-policy-service-endpoint-due-to-egress-restrictions"></a>Scénario : Le module complémentaire ne peut pas joindre le point de terminaison du service Azure Policy en raison de restrictions de sortie

#### <a name="issue"></a>Problème

Le module complémentaire ne peut pas joindre le point de terminaison du service Azure Policy et retourne l’une des erreurs suivantes :

- `failed to fetch token, service not reachable`
- `Error getting file "Get https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/template.yaml: dial tcp 151.101.228.133.443: connect: connection refused`

#### <a name="cause"></a>Cause

Ce problème se produit lorsqu’une sortie de cluster est verrouillée.

#### <a name="resolution"></a>Résolution

Vérifiez que les domaines et les ports figurant dans les articles suivants sont ouverts :

- [Règles de réseau sortantes et noms FQDN requis pour les clusters AKS](../../../aks/limit-egress-traffic.md#required-outbound-network-rules-and-fqdns-for-aks-clusters)
- [Installer le module complémentaire Azure Policy pour Kubernetes avec Azure Arc (préversion)](../concepts/policy-for-kubernetes.md#install-azure-policy-add-on-for-azure-arc-enabled-kubernetes)

### <a name="scenario-add-on-is-unable-to-reach-the-azure-policy-service-endpoint-due-to-aad-pod-identity-configuration"></a>Scénario : Le module complémentaire ne peut pas joindre le point de terminaison du service Azure Policy en raison de la configuration aad-pod-identity

#### <a name="issue"></a>Problème

Le module complémentaire ne peut pas joindre le point de terminaison du service Azure Policy et retourne l’une des erreurs suivantes :

- `azure.BearerAuthorizer#WithAuthorization: Failed to refresh the Token for request to https://gov-prod-policy-data.trafficmanager.net/checkDataPolicyCompliance?api-version=2019-01-01-preview: StatusCode=404`
- `adal: Refresh request failed. Status Code = '404'. Response body: getting assigned identities for pod kube-system/azure-policy-8c785548f-r882p in CREATED state failed after 16 attempts, retry duration [5]s, error: <nil>`

#### <a name="cause"></a>Cause

Cette erreur se produit lorsque _add-pod-identity_ est installé sur le cluster et que les pods _Kube-system_ ne sont pas exclus dans _aad-pod-identity_.

Les pods NMI (Node Managed Identity) du composant _aad-pod-identity_ modifient les iptables des nœuds pour intercepter les appels vers le point de terminaison Azure Instance Metadata. Cette configuration signifie que toutes les requêtes adressées au point de terminaison Metadata sont interceptées par NMI, même si le pod n’utilise pas _aad-pod-identity_.
La CRD **AzurePodIdentityException** peut être configurée de manière à informer _aad-pod-identity_ que toutes les requêtes adressées au point de terminaison des métadonnées depuis un pod correspondant aux étiquettes définies dans la CRD doivent être envoyées par proxy sans aucun traitement dans NMI.

#### <a name="resolution"></a>Résolution

Excluez les pods système qui disposent de l’étiquette `kubernetes.azure.com/managedby: aks` dans l’espace de noms _kube-system_ dans _aad-pod-identity_ en configurant la CRD **AzurePodIdentityException**.

Pour plus d’informations, consultez [Désactiver AAD Pod Identity pour un pod ou une application spécifique](https://azure.github.io/aad-pod-identity/docs/configure/application_exception).

Pour configurer une exception, examinez l’exemple suivant :

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: mic-exception
  namespace: default
spec:
  podLabels:
    app: mic
    component: mic
---
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: aks-addon-exception
  namespace: kube-system
spec:
  podLabels:
    kubernetes.azure.com/managedby: aks
```

### <a name="scenario-the-resource-provider-isnt-registered"></a>Scénario : Le fournisseur de ressources n’est pas inscrit

#### <a name="issue"></a>Problème

Le module complémentaire peut joindre le point de terminaison du service Azure Policy, mais l’une des erreurs suivantes s’affiche dans les journaux du module complémentaire :

```
The resource provider 'Microsoft.PolicyInsights' is not registered in subscription '{subId}'. See
https://aka.ms/policy-register-subscription for how to register subscriptions.
```

ou

```
policyinsightsdataplane.BaseClient#CheckDataPolicyCompliance: Failure responding to request:
StatusCode=500 -- Original Error: autorest/azure: Service returned an error. Status=500
Code="InternalServerError" Message="Encountered an internal server error."
```

#### <a name="cause"></a>Cause

Le fournisseur de ressources `Microsoft.PolicyInsights` n’est pas inscrit et il doit l’être pour que le module complémentaire récupère les définitions de stratégie et retourne les données de conformité.

#### <a name="resolution"></a>Résolution

Inscrivez le fournisseur de ressources `Microsoft.PolicyInsights` dans l’abonnement du cluster. Pour obtenir des instructions, consultez [Inscrire un fournisseur de ressources](../../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider).

### <a name="scenario-the-subscription-is-disabled"></a>Scénario : L'abonnement est désactivé

#### <a name="issue"></a>Problème

Le module complémentaire peut joindre le point de terminaison du service Azure Policy, mais affiche l’erreur suivante :

```
The subscription '{subId}' has been disabled for azure data-plane policy. Please contact support.
```

#### <a name="cause"></a>Cause

Cette erreur signifie que l’abonnement a été identifié comme problématique ; l’indicateur de fonctionnalité `Microsoft.PolicyInsights/DataPlaneBlocked` a été ajouté pour bloquer l’abonnement.

#### <a name="resolution"></a>Résolution

Contactez l’équipe des fonctionnalités `azuredg@microsoft.com` pour examiner et résoudre ce problème.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

- Obtenez des réponses d’experts par le biais de [Microsoft Q&A](/answers/topics/azure-policy.html).
- Connectez-vous avec [@AzureSupport](https://twitter.com/azuresupport), qui est le compte Microsoft Azure officiel pour améliorer l’expérience client en connectant la communauté Azure aux ressources appropriées : réponses, support technique et experts.
- Si vous avez besoin de plus d’aide, vous pouvez signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.
