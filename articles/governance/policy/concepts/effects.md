---
title: Comprendre le fonctionnement des effets
description: Les définitions Azure Policy ont différents effets qui déterminent la manière dont la conformité est gérée et rapportée.
ms.date: 10/05/2020
ms.topic: conceptual
ms.openlocfilehash: 19811eca33be7dff4d9bee5b8bd89dd38f185a57
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91873946"
---
# <a name="understand-azure-policy-effects"></a>Comprendre les effets d’Azure Policy

Chaque définition de stratégie dans Azure Policy a un effet unique. Cet effet détermine ce qu’il se passe lorsque la règle de stratégie est évaluée pour une mise en correspondance. Les effets se comportent différemment selon qu’ils concernent une nouvelle ressource, une ressource mise à jour ou une ressource existante.

Une définition de stratégie prend en charge ces effets :

- [Append](#append)
- [Audit](#audit)
- [AuditIfNotExists](#auditifnotexists)
- [Deny](#deny)
- [DeployIfNotExists](#deployifnotexists)
- [Désactivé](#disabled)
- [Modify](#modify)

Les effets suivants sont _déconseillés_ :

- [EnforceOPAConstraint](#enforceopaconstraint)
- [EnforceRegoPolicy](#enforceregopolicy)

> [!IMPORTANT]
> À la place des effets **EnforceOPAConstraint** ou **EnforceRegoPolicy**, utilisez _audit_ et _deny_ avec le mode Fournisseur de ressources `Microsoft.Kubernetes.Data`. Les définitions de stratégie intégrées ont été mises à jour. Quand des attributions de stratégies existantes de ces définitions de stratégie intégrées sont modifiées, le paramètre _effect_ doit être remplacé par une valeur de la liste _allowedValues_ mise à jour.

## <a name="order-of-evaluation"></a>Ordre d’évaluation

Les demandes de création ou de mise à jour d’une ressource sont évaluées en premier par Azure Policy. Azure Policy répertorie toutes les affectations qui s’appliquent à la ressource, puis évalue la ressource en fonction de chaque définition. Pour un [mode Fournisseur de ressources](./definition-structure.md#resource-manager-modes), Azure Policy traite plusieurs des effets avant de transmettre la demande au fournisseur de ressources approprié. Cet ordre empêche un fournisseur de ressources d’effectuer un traitement inutile d’une ressource quand elle ne satisfait pas aux contrôles de gouvernance d’Azure Policy. Avec un [mode Fournisseur de ressources](./definition-structure.md#resource-provider-modes), le fournisseur de ressources gère l’évaluation et le résultat, et renvoie les résultats à Azure Policy.

- **Désactivé** est vérifié en premier pour déterminer si la règle de stratégie doit être évaluée.
- **Append** et **Modify** sont ensuite évalués. Puisque l’un comme l’autre peuvent modifier la requête, une modification peut empêcher le déclenchement d’un effet Audit ou Deny. Ces effets sont disponibles uniquement avec un mode Gestionnaire des ressources.
- **Deny** est ensuite évalué. L’évaluation de Deny avant Audit empêche la double journalisation d’une ressource indésirable.
- L’**audit** est évalué en dernier.

Une fois que le fournisseur de ressources a retourné un code de réussite sur une demande en mode Gestionnaire des ressources, **AuditIfNotExists** et **DeployIfNotExists** effectuent une évaluation pour déterminer si une journalisation ou une action de conformité supplémentaire est nécessaire.

## <a name="append"></a>Ajouter

Append permet d’ajouter des champs supplémentaires à la ressource demandée lors de la création ou de la mise à jour. Un exemple courant consiste à spécifier des adresses IP autorisées pour une ressource de stockage.

> [!IMPORTANT]
> Append est destiné à être utilisé avec des propriétés autres que des étiquettes. Même si Append peut ajouter des étiquettes à une ressource dans le cadre d’une requête de création ou de mise à jour, il est recommandé d’utiliser l’effet [Modify](#modify) pour les étiquettes.

### <a name="append-evaluation"></a>Évaluation Append

L’évaluation Append se produit avant que la requête ne soit traitée par un fournisseur de ressources lors de la création ou de la mise à jour d’une ressource. Append ajoute des champs à la ressource lorsque la condition **if** de la règle de stratégie est remplie. Si l’effet Append remplace une valeur dans la requête d’origine avec une valeur différente, il agit comme un effet Deny et rejette la demande. Pour ajouter une nouvelle valeur à un tableau existant, utilisez la version **[\[]\*\]** de l’alias.

Lorsqu’une définition de stratégie utilisant l’effet Append est exécutée dans le cadre d’un cycle d’évaluation, elle n’apporte pas de modifications aux ressources qui existent déjà. Au lieu de cela, elle marque comme non conforme toute ressource qui répond à la condition **if**.

### <a name="append-properties"></a>Propriétés d’Append

Un effet Append prend en charge la propriété obligatoire **details** uniquement, laquelle est un tableau de données. En tant que tableau, **details** peut contenir une ou plusieurs paires **champ/valeur**. Reportez-vous à la [structure de définition](definition-structure.md#fields) pour afficher la liste des champs autorisés.

### <a name="append-examples"></a>Exemples Append

Exemple 1 : paire **champ/valeur** unique utilisant un [alias](definition-structure.md#aliases) non **\[\*\]** avec un tableau **value** afin de définir des règles IP sur un compte de stockage. Lorsque l’alias non **\[\*\]** est un tableau, l’effet ajoute la **value** comme tableau entier. Si le tableau existe déjà, un événement de refus se produit à partir du conflit.

```json
"then": {
    "effect": "append",
    "details": [{
        "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules",
        "value": [{
            "action": "Allow",
            "value": "134.5.0.0/21"
        }]
    }]
}
```

Exemple 2 : paire **champ/valeur** unique utilisant un [alias](definition-structure.md#aliases) **\[\*\]** avec un tableau **value** afin de définir des règles IP sur un compte de stockage. En utilisant l’alias **\[\*\]** , l’effet ajoute la **value** à un tableau potentiellement préexistant. Si le tableau n’existe pas encore, il est créé.

```json
"then": {
    "effect": "append",
    "details": [{
        "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]",
        "value": {
            "value": "40.40.40.40",
            "action": "Allow"
        }
    }]
}
```

## <a name="audit"></a>Audit

Audit permet de créer un événement d’avertissement dans le journal d’activité lors de l’évaluation d’une ressource non conforme, mais il n’arrête pas la requête.

### <a name="audit-evaluation"></a>Évaluation Audit

Audit est le dernier effet vérifié par Azure Policy pendant la création ou la mise à jour d’une ressource. Pour un mode Gestionnaire des ressources, Azure Policy envoie ensuite la ressource au fournisseur de ressources. Audit fonctionne de la même façon pour une requête de ressource et un cycle d’évaluation. Pour les nouvelles ressources et celles mises à jour, Azure Policy ajoute une opération `Microsoft.Authorization/policies/audit/action` dans le journal d’activité et marque la ressource comme non conforme.

### <a name="audit-properties"></a>Propriétés d’Audit

Pour un mode Gestionnaire des ressources, l’effet audit n’a pas d’autres propriétés utilisables dans la condition **then** de la définition de stratégie.

Pour le mode Fournisseur de ressources `Microsoft.Kubernetes.Data`, l’effet audit contient les sous-propriétés de **details** supplémentaires suivantes.

- **constraintTemplate** (obligatoire)
  - Modèle de contrainte CustomResourceDefinition (CRD) qui définit de nouvelles contraintes. Le modèle définit la logique Rego, le schéma de contrainte et les paramètres de contrainte transmis via des objets **values** (valeurs) d’Azure Policy.
- **constraint** (obligatoire)
  - Implémentation CRD du modèle de contrainte. Utilise des paramètres transmis via des **valeurs** telles que `{{ .Values.<valuename> }}`. Dans l’exemple 2 ci-dessous, ces valeurs sont `{{ .Values.excludedNamespaces }}` et `{{ .Values.allowedContainerImagesRegex }}`.
- **values** (facultatif)
  - Définit des paramètres et valeurs à transmettre à la contrainte. Chaque valeur doit exister dans le modèle de contrainte CRD.

### <a name="audit-example"></a>Exemple Audit

Exemple 1 : Utilisation de l’effet audit pour les modes Gestionnaire des ressources.

```json
"then": {
    "effect": "audit"
}
```

Exemple 2 : Utilisation de l’effet audit pour le mode Fournisseur de ressources `Microsoft.Kubernetes.Data`. Les informations supplémentaires dans **details** définissent le modèle de contrainte et CRD à utiliser dans Kubernetes pour limiter les images de conteneur autorisées.

```json
"then": {
    "effect": "audit",
    "details": {
        "constraintTemplate": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/template.yaml",
        "constraint": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/constraint.yaml",
        "values": {
            "allowedContainerImagesRegex": "[parameters('allowedContainerImagesRegex')]",
            "excludedNamespaces": "[parameters('excludedNamespaces')]"
        }
    }
}
```

## <a name="auditifnotexists"></a>AuditIfNotExists

AuditIfNotExists active l’audit de ressources _liées_ à la ressource qui correspond à la condition **if**, mais dont les propriétés ne sont pas spécifiées dans les détails (**details**) de la condition **then**.

### <a name="auditifnotexists-evaluation"></a>Évaluation AuditIfNotExists

AuditIfNotExists s’exécute après qu’un fournisseur de ressources a traité une requête de création ou de mise à jour de ressource et a renvoyé un code d’état de réussite. L’effet Audit est déclenché s’il n’existe pas de ressources connexes ou si les ressources définies par **ExistenceCondition** ne retournent pas de valeur true. Pour les nouvelles ressources et celles mises à jour, Azure Policy ajoute une opération `Microsoft.Authorization/policies/audit/action` dans le journal d’activité et marque la ressource comme non conforme. Dans ce cas, la ressource qui a rempli la condition **if** est en fait la ressource qui est marquée comme non conforme.

### <a name="auditifnotexists-properties"></a>Propriétés d’AuditIfNotExists

La propriété **details** des effets AuditIfNotExists possède toutes les sous-propriétés qui définissent les ressources connexes testées.

- **Type** (obligatoire)
  - Spécifie le type de la ressource connexe à évaluer.
  - Si **details.type** est un type de ressource sous la ressource de condition **if**, la stratégie envoie des requêtes pour les ressources de ce **type** dans l’étendue de la ressource évaluée. Sinon, la stratégie envoie des requêtes dans le même groupe de ressources que la ressource évaluée.
- **Name** (facultatif)
  - Spécifie le nom exact de la ressource à tester et amène la stratégie à extraire une ressource spécifique au lieu de toutes les ressources du type spécifié.
  - Lorsque les valeurs de condition pour **if.field.type** et **then.details.type** correspondent, **Name** devient _required_ et doit être `[field('name')]`, ou `[field('fullName')]` pour une ressource enfant.
    Toutefois, un effet [audit](#audit) doit être considéré à la place.
- **ResourceGroupName** (facultatif)
  - Permet l’évaluation de la ressource connexe provenant d’un groupe de ressources différent.
  - Ne s’applique pas si **type** est une ressource qui serait située sous la ressource de condition **if**.
  - La valeur par défaut est le groupe de ressources de la ressource de condition **if**.
- **ExistenceScope** (facultatif)
  - Les valeurs autorisées sont _Subscription_ et _ResourceGroup_.
  - Définit la portée d’où la ressource connexe évaluée est extraite.
  - Ne s’applique pas si **type** est une ressource qui serait située sous la ressource de condition **if**.
  - Pour _ResourceGroup_, le traitement se limiterait au groupe de ressources de la ressource de condition **if** ou au groupe de ressources spécifié dans **ResourceGroupName**.
  - Pour _Subscription_, le traitement interroge l’abonnement entier pour la ressource associée.
  - La valeur par défaut est _ResourceGroup_.
- **ExistenceCondition** (facultatif)
  - Si elle n’est pas spécifiée, toute ressource connexe du **type** remplit les conditions de l’effet et ne déclenche pas l’audit.
  - Utilise la même langue que la règle de stratégie pour la condition **if** mais est évaluée individuellement sur chaque ressource associée.
  - Si une ressource connexe correspondante renvoie true, l’effet est satisfait et ne déclenche pas l’audit.
  - Peut utiliser [field()] pour vérifier l’équivalence des valeurs dans la condition **if**.
  - Par exemple, permet de vérifier que la ressource parent (dans la condition **if**) réside dans le même emplacement de la ressource en tant que ressource connexe correspondante.

### <a name="auditifnotexists-example"></a>Exemple AuditIfNotExists

Exemple : évalue les Machines virtuelles Microsoft Azure pour déterminer si l’extension Antimalware existe, puis effectue un audit si l’extension est absente.

```json
{
    "if": {
        "field": "type",
        "equals": "Microsoft.Compute/virtualMachines"
    },
    "then": {
        "effect": "auditIfNotExists",
        "details": {
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "existenceCondition": {
                "allOf": [{
                        "field": "Microsoft.Compute/virtualMachines/extensions/publisher",
                        "equals": "Microsoft.Azure.Security"
                    },
                    {
                        "field": "Microsoft.Compute/virtualMachines/extensions/type",
                        "equals": "IaaSAntimalware"
                    }
                ]
            }
        }
    }
}
```

## <a name="deny"></a>Deny

Deny empêche l’exécution d’une requête de ressource qui ne correspond pas aux normes définies via une définition de stratégie et qui fait échouer la requête.

### <a name="deny-evaluation"></a>Évaluation Deny

Lors de la création ou de la mise à jour d’une ressource correspondante dans un mode Gestionnaire des ressources, deny empêche l’envoi de la demande au fournisseur de ressources. La requête renvoie une erreur `403 (Forbidden)`. Dans le portail, l’erreur 403 (Interdit) peut être considérée comme l’état du déploiement qui a été empêché par l’affectation de stratégie. Pour un mode Fournisseur de ressources, le fournisseur de ressources gère l’évaluation de la ressource.

Lors de l’évaluation des ressources existantes, les ressources qui correspondent à une définition de stratégie Deny sont marquées comme non conformes.

### <a name="deny-properties"></a>Propriétés de Deny

Pour un mode Gestionnaire des ressources, l’effet deny n’a pas d’autres propriétés utilisables dans la condition **then** de la définition de stratégie.

Pour le mode Fournisseur de ressources `Microsoft.Kubernetes.Data`, l’effet deny contient les sous-propriétés de **details** supplémentaires suivantes.

- **constraintTemplate** (obligatoire)
  - Modèle de contrainte CustomResourceDefinition (CRD) qui définit de nouvelles contraintes. Le modèle définit la logique Rego, le schéma de contrainte et les paramètres de contrainte transmis via des objets **values** (valeurs) d’Azure Policy.
- **constraint** (obligatoire)
  - Implémentation CRD du modèle de contrainte. Utilise des paramètres transmis via des **valeurs** telles que `{{ .Values.<valuename> }}`. Dans l’exemple 2 ci-dessous, ces valeurs sont `{{ .Values.excludedNamespaces }}` et `{{ .Values.allowedContainerImagesRegex }}`.
- **values** (facultatif)
  - Définit des paramètres et valeurs à transmettre à la contrainte. Chaque valeur doit exister dans le modèle de contrainte CRD.

### <a name="deny-example"></a>Exemple Deny

Exemple 1 : Utilisation de l’effet deny pour les modes Gestionnaire des ressources.

```json
"then": {
    "effect": "deny"
}
```

Exemple 2 : Utilisation de l’effet deny pour le mode Fournisseur de ressources `Microsoft.Kubernetes.Data`. Les informations supplémentaires dans **details** définissent le modèle de contrainte et CRD à utiliser dans Kubernetes pour limiter les images de conteneur autorisées.

```json
"then": {
    "effect": "deny",
    "details": {
        "constraintTemplate": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/template.yaml",
        "constraint": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/constraint.yaml",
        "values": {
            "allowedContainerImagesRegex": "[parameters('allowedContainerImagesRegex')]",
            "excludedNamespaces": "[parameters('excludedNamespaces')]"
        }
    }
}
```

## <a name="deployifnotexists"></a>DeployIfNotExists

Comme pour AuditIfNotExists, une définition de stratégie DeployIfNotExists exécute un déploiement de modèle lorsque la condition est remplie.

> [!NOTE]
> Les [modèles imbriqués](../../../azure-resource-manager/templates/linked-templates.md#nested-template) sont pris en charge avec **deployIfNotExists**, mais les [modèles liés](../../../azure-resource-manager/templates/linked-templates.md#linked-template) ne sont actuellement pas pris en charge.

### <a name="deployifnotexists-evaluation"></a>Évaluation DeployIfNotExists

DeployIfNotExists s’exécute environ 15 minutes après qu’un fournisseur de ressources a traité une requête de création ou de mise à jour de ressource et a renvoyé un code d’état de réussite. Un déploiement de modèle est déclenché s’il n’existe pas de ressources connexes ou si les ressources définies par **ExistenceCondition** ne retournent pas de valeur true.
La durée du déploiement dépend de la complexité des ressources incluses dans le modèle.

Au cours d’un cycle d’évaluation, les définitions de stratégie ayant un effet DeployIfNotExists sur les ressources sont marquées comme non conformes, mais aucune action n’est effectuée sur ces ressources. Les ressources non conformes existantes peuvent être corrigées à l’aide d’une [tâche de correction](../how-to/remediate-resources.md).

### <a name="deployifnotexists-properties"></a>Propriétés de DeployIfNotExists

La propriété **details** de l’effet DeployIfNotExists comprend toutes les sous-propriétés qui définissent les ressources connexes testées, ainsi que le déploiement de modèle à exécuter.

- **Type** (obligatoire)
  - Spécifie le type de la ressource connexe à évaluer.
  - Commence par tenter d’extraire une ressource sous la ressource de condition **if**, puis effectue une requête dans le même groupe de ressources que la ressource de condition **if**.
- **Name** (facultatif)
  - Spécifie le nom exact de la ressource à tester et amène la stratégie à extraire une ressource spécifique au lieu de toutes les ressources du type spécifié.
  - Lorsque les valeurs de condition pour **if.field.type** et **then.details.type** correspondent, **Name** devient _required_ et doit être `[field('name')]`, ou `[field('fullName')]` pour une ressource enfant.
- **ResourceGroupName** (facultatif)
  - Permet l’évaluation de la ressource connexe provenant d’un groupe de ressources différent.
  - Ne s’applique pas si **type** est une ressource qui serait située sous la ressource de condition **if**.
  - La valeur par défaut est le groupe de ressources de la ressource de condition **if**.
  - L’exécution d’un déploiement de modèle s’effectue dans le groupe de ressources de cette valeur.
- **ExistenceScope** (facultatif)
  - Les valeurs autorisées sont _Subscription_ et _ResourceGroup_.
  - Définit la portée d’où la ressource connexe évaluée est extraite.
  - Ne s’applique pas si **type** est une ressource qui serait située sous la ressource de condition **if**.
  - Pour _ResourceGroup_, le traitement se limiterait au groupe de ressources de la ressource de condition **if** ou au groupe de ressources spécifié dans **ResourceGroupName**.
  - Pour _Subscription_, le traitement interroge l’abonnement entier pour la ressource associée.
  - La valeur par défaut est _ResourceGroup_.
- **ExistenceCondition** (facultatif)
  - Si elle n’est pas spécifiée, toute ressource connexe de **type** satisfait à l’effet en question et ne déclenche pas le déploiement.
  - Utilise la même langue que la règle de stratégie pour la condition **if** mais est évaluée individuellement sur chaque ressource associée.
  - Si une ressource connexe correspondante renvoie true, l’effet est satisfait et ne déclenche pas le déploiement.
  - Peut utiliser [field()] pour vérifier l’équivalence des valeurs dans la condition **if**.
  - Par exemple, permet de vérifier que la ressource parent (dans la condition **if**) réside dans le même emplacement de la ressource en tant que ressource connexe correspondante.
- **roleDefinitionIds** (obligatoire)
  - Cette propriété doit inclure un tableau de chaînes qui correspondent aux ID de rôle de contrôle de l’accès en fonction du rôle accessibles par l’abonnement. Pour plus d’informations, consultez [Correction - Configurer une définition de stratégie](../how-to/remediate-resources.md#configure-policy-definition).
- **DeploymentScope** (facultatif)
  - Les valeurs autorisées sont _Subscription_ et _ResourceGroup_.
  - Définit le type de déploiement à déclencher. _Subscription_ indique un [déploiement au niveau de l’abonnement](../../../azure-resource-manager/templates/deploy-to-subscription.md), _ResourceGroup_ indique un déploiement dans un groupe de ressources.
  - Une propriété _location_ doit être spécifiée dans _Deployment_ pour les déploiements au niveau de l’abonnement.
  - La valeur par défaut est _ResourceGroup_.
- **Deployment** (obligatoire)
  - Cette propriété doit inclure le déploiement de modèle complet, car elle est transmise à l’API PUT `Microsoft.Resources/deployments`. Pour plus d’informations, consultez [l’API REST Deployments](/rest/api/resources/deployments).

  > [!NOTE]
  > Toutes les fonctions à l’intérieur de la propriété **Deployment** sont évaluées en tant que composants du modèle, et non pas de la stratégie. La propriété **parameters** y fait exception car elle transmet les valeurs de la stratégie au modèle. Dans cette section, la propriété **value** sous le nom de paramètre du modèle permet d’effectuer la transmission de valeurs (voir _fullDbName_ dans l’exemple DeployIfNotExists).

### <a name="deployifnotexists-example"></a>Exemple DeployIfNotExists

Exemple : évalue les bases de données SQL Server pour déterminer si transparentDataEncryption est activée. Sinon, un déploiement destiné à l’activer est exécuté.

```json
"if": {
    "field": "type",
    "equals": "Microsoft.Sql/servers/databases"
},
"then": {
    "effect": "DeployIfNotExists",
    "details": {
        "type": "Microsoft.Sql/servers/databases/transparentDataEncryption",
        "name": "current",
        "roleDefinitionIds": [
            "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleGUID}",
            "/providers/Microsoft.Authorization/roleDefinitions/{builtinroleGUID}"
        ],
        "existenceCondition": {
            "field": "Microsoft.Sql/transparentDataEncryption.status",
            "equals": "Enabled"
        },
        "deployment": {
            "properties": {
                "mode": "incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {
                        "fullDbName": {
                            "type": "string"
                        }
                    },
                    "resources": [{
                        "name": "[concat(parameters('fullDbName'), '/current')]",
                        "type": "Microsoft.Sql/servers/databases/transparentDataEncryption",
                        "apiVersion": "2014-04-01",
                        "properties": {
                            "status": "Enabled"
                        }
                    }]
                },
                "parameters": {
                    "fullDbName": {
                        "value": "[field('fullName')]"
                    }
                }
            }
        }
    }
}
```

## <a name="disabled"></a>Désactivé

Cet effet peut s’avérer utile pour tester certaines situations ou lorsque la définition de stratégie a paramétré l’effet. Cette flexibilité permet de désactiver une seule affectation plutôt que de désactiver toutes les affectations de cette stratégie.

Une alternative à l’effet Désactivé est **enforcementMode**, qui est défini sur l’attribution de stratégie.
Lorsque **enforcementMode** est _Désactivé_, les ressources sont toujours évaluées. La journalisation, notamment les journaux d’activité, et l’effet de stratégie ne se produisent pas. Pour plus d’informations, consultez [Attribution de stratégie - Mode de mise en conformité](./assignment-structure.md#enforcement-mode).

## <a name="enforceopaconstraint"></a>EnforceOPAConstraint

Cet effet est utilisé avec une définition de stratégie _mode_ de `Microsoft.Kubernetes.Data`. Il est utilisé pour transmettre les règles de contrôle d’admission de Gatekeeper v3 définies avec le [Framework de contraintes d’OPA](https://github.com/open-policy-agent/frameworks/tree/master/constraint#opa-constraint-framework) à [Open Policy Agent](https://www.openpolicyagent.org/) (OPA) aux clusters Kubernetes sur Azure.

> [!IMPORTANT]
> Les définitions de stratégies en préversion limitée avec l’effet **EnforceOPAConstraint** et la catégorie **Service Kubernetes** associée sont _déconseillées_. À la place, utilisez les effets _audit_ et _deny_ avec le mode Fournisseur de ressources `Microsoft.Kubernetes.Data`.

### <a name="enforceopaconstraint-evaluation"></a>Évaluation d’EnforceOPAConstraint

Le contrôleur d’admission Open Policy Agent évalue les nouvelles requêtes du cluster en temps réel.
Toutes les 15 minutes, une analyse complète du cluster est réalisée, et les résultats sont renvoyés sur Azure Policy.

### <a name="enforceopaconstraint-properties"></a>Propriétés d’EnforceOPAConstraint

La propriété **details** de l’effet EnforceOPAConstraint contient les sous-propriétés qui décrivent la règle de contrôle d’admission de Gatekeeper v3.

- **constraintTemplate** (obligatoire)
  - Modèle de contrainte CustomResourceDefinition (CRD) qui définit de nouvelles contraintes. Le modèle définit la logique Rego, le schéma de contrainte et les paramètres de contrainte transmis via des objets **values** (valeurs) d’Azure Policy.
- **constraint** (obligatoire)
  - Implémentation CRD du modèle de contrainte. Utilise des paramètres transmis via des **valeurs** telles que `{{ .Values.<valuename> }}`. Dans l’exemple ci-dessous, ces valeurs sont `{{ .Values.cpuLimit }}` et `{{ .Values.memoryLimit }}`.
- **values** (facultatif)
  - Définit des paramètres et valeurs à transmettre à la contrainte. Chaque valeur doit exister dans le modèle de contrainte CRD.

### <a name="enforceopaconstraint-example"></a>Exemple EnforceOPAConstraint

Exemple : Règle de contrôle d’admission de Gatekeeper v3 pour définir les limites de ressources de mémoire et de processeur du conteneur dans Kubernetes.

```json
"if": {
    "allOf": [
        {
            "field": "type",
            "in": [
                "Microsoft.ContainerService/managedClusters",
                "AKS Engine"
            ]
        },
        {
            "field": "location",
            "equals": "westus2"
        }
    ]
},
"then": {
    "effect": "enforceOPAConstraint",
    "details": {
        "constraintTemplate": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-resource-limits/template.yaml",
        "constraint": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-resource-limits/constraint.yaml",
        "values": {
            "cpuLimit": "[parameters('cpuLimit')]",
            "memoryLimit": "[parameters('memoryLimit')]"
        }
    }
}
```

## <a name="enforceregopolicy"></a>EnforceRegoPolicy

Cet effet est utilisé avec une définition de stratégie _mode_ de `Microsoft.ContainerService.Data`. Il sert à transmettre des règles de contrôle d’admission de Gatekeeper v2 définies avec [Rego](https://www.openpolicyagent.org/docs/latest/policy-language/#what-is-rego) à [Open Policy Agent](https://www.openpolicyagent.org/) (OPA) sur [Azure Kubernetes Service](../../../aks/intro-kubernetes.md).

> [!IMPORTANT]
> Les définitions de stratégie limitées en préversion avec l’effet **EnforceRegoPolicy** et la catégorie **Service Kubernetes** associée sont _déconseillées_. À la place, utilisez les effets _audit_ et _deny_ avec le mode Fournisseur de ressources `Microsoft.Kubernetes.Data`.

### <a name="enforceregopolicy-evaluation"></a>Évaluation d’EnforceRegoPolicy

Le contrôleur d’admission Open Policy Agent évalue les nouvelles requêtes du cluster en temps réel.
Toutes les 15 minutes, une analyse complète du cluster est réalisée, et les résultats sont renvoyés sur Azure Policy.

### <a name="enforceregopolicy-properties"></a>Propriétés d’EnforceRegoPolicy

La propriété **details** de l’effet EnforceRegoPolicy contient les sous-propriétés qui décrivent la règle de contrôle d’admission Gatekeeper v2.

- **policyId** (obligatoire)
  - Nom unique transmis en tant que paramètre à la règle de contrôle d’admission Rego.
- **policy** (obligatoire)
  - Spécifie l’URI de la règle de contrôle d’admission Rego.
- **policyParameters** (facultatif)
  - Définit des paramètres et des valeurs à transmettre à la stratégie Rego.

### <a name="enforceregopolicy-example"></a>Exemple EnforceRegoPolicy

Exemple : Règle de contrôle d’admission Gatekeeper v2 pour autoriser uniquement les images de conteneur spécifiées dans AKS.

```json
"if": {
    "allOf": [
        {
            "field": "type",
            "equals": "Microsoft.ContainerService/managedClusters"
        },
        {
            "field": "location",
            "equals": "westus2"
        }
    ]
},
"then": {
    "effect": "EnforceRegoPolicy",
    "details": {
        "policyId": "ContainerAllowedImages",
        "policy": "https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/KubernetesService/container-allowed-images/limited-preview/gatekeeperpolicy.rego",
        "policyParameters": {
            "allowedContainerImagesRegex": "[parameters('allowedContainerImagesRegex')]"
        }
    }
}
```

## <a name="modify"></a>Modifier

Modify est utilisé pour ajouter, mettre à jour ou supprimer les propriétés ou étiquettes d’une ressource lors d’une création ou d’une mise à jour.
Un exemple courant consiste à mettre à jour les étiquettes des ressources telles que costCenter. Les ressources non conformes existantes peuvent être corrigées à l’aide d’une [tâche de correction](../how-to/remediate-resources.md). Une même règle Modify peut avoir autant d’opérations que vous le souhaitez.

Les opérations suivantes sont prises en charge par Modify :

- Ajouter, remplacer ou supprimer des étiquettes. Pour les étiquettes, une stratégie Modify doit avoir `mode` défini sur _Indexé_, sauf si la ressource cible est un groupe de ressources.
- Ajouter ou remplacer la valeur du type d’identité managée (`identity.type`) des machines virtuelles et des groupes de machines virtuelles identiques.
- Ajouter ou remplacer les valeurs de certains alias (préversion).
  - Utilisez `Get-AzPolicyAlias | Select-Object -ExpandProperty 'Aliases' | Where-Object { $_.DefaultMetadata.Attributes -eq 'Modifiable' }`.
    dans Azure PowerShell **4.6.0** ou une version ultérieure pour obtenir la liste des alias pouvant être utilisés avec Modify.

> [!IMPORTANT]
> Si vous gérez des étiquettes, il est recommandé d’utiliser Modify plutôt qu’Append, car Modify fournit des types d’opérations supplémentaires, ainsi que la possibilité de corriger les ressources existantes. Toutefois, Append est recommandé si vous n’êtes pas en mesure de créer une identité managée ou si Modify ne prend pas encore en charge l’alias de la propriété de ressource.

### <a name="modify-evaluation"></a>Évaluation Modify

L’évaluation Modify a lieu avant que la requête ne soit traitée par un fournisseur de ressources lors de la création ou de la mise à jour d’une ressource. Les opérations Modify sont appliquées au contenu de la requête lorsque la condition **if** de la règle de stratégie est remplie. Chaque opération Modify peut spécifier une condition qui détermine le moment où elle est appliquée. Les opérations dont les conditions sont évaluées comme _false_ sont ignorées.

Lorsqu’un alias est spécifié, les vérifications supplémentaires suivantes sont effectuées pour s’assurer que l’opération Modify ne modifie pas le contenu de la requête de manière à ce que le fournisseur de ressources le rejette :

- La propriété à laquelle l’alias correspond est marquée comme « modifiable » dans la version d’API de la requête.
- Le type de jeton dans l’opération Modify correspond au type de jeton attendu pour la propriété dans la version d’API de la requête.

Si l’une de ces vérifications échoue, l’évaluation de la stratégie revient à la sous-propriété **conflictEffect** spécifiée.

> [!IMPORTANT]
> Il est recommandée que les définitions de Modify qui incluent des alias utilisent l’_audit_ **Effet de conflit** afin d’éviter les échecs de requêtes à l’aide de versions d’API où la propriété correspondantes n’est pas « modifiable ». Si le même alias se comporte différemment entre les versions d’API, des opérations Modify conditionnelles peuvent être utilisées pour déterminer l’opération Modify utilisée pour chaque version de l’API.

Lorsqu’une définition de stratégie utilisant l’effet Modify est exécutée dans le cadre d’un cycle d’évaluation, elle n’apporte pas de modifications aux ressources qui existent déjà. Au lieu de cela, elle marque comme non conforme toute ressource qui répond à la condition **if**.

### <a name="modify-properties"></a>Propriétés de Modify

La propriété **details** de l’effet Modify comporte toutes les sous-propriétés qui définissent les autorisations nécessaires à la correction, ainsi que les **opérations** utilisées pour ajouter, mettre à jour ou supprimer les valeurs des étiquettes.

- **roleDefinitionIds** (obligatoire)
  - Cette propriété doit inclure un tableau de chaînes qui correspondent aux ID de rôle de contrôle de l’accès en fonction du rôle accessibles par l’abonnement. Pour plus d’informations, consultez [Correction - Configurer une définition de stratégie](../how-to/remediate-resources.md#configure-policy-definition).
  - Le rôle défini doit inclure toutes les opérations accordées au rôle [Contributeur](../../../role-based-access-control/built-in-roles.md#contributor).
- **conflictEffect** (facultatif)
  - Détermine la définition de stratégie « wins » dans le cas où plusieurs définitions de stratégie modifient la même propriété ou lorsque l’opération Modify ne fonctionne pas sur l’alias spécifié.
    - Pour les ressources nouvelles ou mises à jour, la définition de stratégie avec _deny_ est prioritaire. Les définitions de stratégie avec _audit_ ignorent toutes les **opérations**. Si plusieurs définitions de stratégie ont _deny_, la demande est refusée pour raison de conflit. Si toutes les définitions de stratégie ont _audit_, aucune des **opérations** des définitions de stratégie en conflit n’est traitée.
    - Pour les ressources existantes, si plusieurs définitions de stratégie ont _deny_, l’état de conformité est _Conflit_. Si au plus une définition de stratégie a _deny_, chaque attribution retourne l’état de conformité _Non conforme_.
  - Valeurs disponibles : _audit_, _deny_, _disabled_.
  - La valeur par défaut est _deny_.
- **operations** (obligatoire)
  - Tableau de toutes les opérations d’étiquette à effectuer sur les ressources correspondantes.
  - Propriétés :
    - **operation** (obligatoire)
      - Définit l’action à effectuer sur une ressource correspondante. Les options sont les suivantes : _addOrReplace_, _Add_, _Remove_. _Add_ a le même comportement que l’effet [Append](#append).
    - **field** (obligatoire)
      - Étiquette à ajouter, remplacer ou supprimer. Les noms d’étiquette doivent respecter la même convention de nommage que les autres [champs](./definition-structure.md#fields).
    - **value** (facultatif)
      - Valeur à affecter à l’étiquette.
      - Cette propriété est obligatoire si l’**opération** est _addOrReplace_ ou _Add_.
    - **condition** (facultatif)
      - Chaîne contenant une expression de langage Azure Policy avec [fonctions de stratégie](./definition-structure.md#policy-functions) qui prend la valeur _true_ ou _false_.
      - Ne prend pas en charge les fonctions de stratégie suivantes : `field()`, `resourceGroup()` et `subscription()`.

### <a name="modify-operations"></a>Opérations Modify

Le tableau de propriétés **operations** permet de modifier plusieurs étiquettes de différentes façons à partir d’une même définition de stratégie. Chaque opération est constituée des propriétés **operation**, **field** et **value**. La propriété operation détermine ce que fait la tâche de correction aux étiquettes, la propriété field détermine l’étiquette qui est modifiée et la propriété value définit le nouveau paramètre de l’étiquette. Voici des exemples de modifications d’étiquette :

- Définit l’étiquette `environment` sur « Test », même si elle a déjà une valeur.
- Supprime l’étiquette `TempResource`.
- Définit l’étiquette `Dept` sur le paramètre de stratégie _DeptName_ configuré pour l’attribution de stratégie.

```json
"details": {
    ...
    "operations": [
        {
            "operation": "addOrReplace",
            "field": "tags['environment']",
            "value": "Test"
        },
        {
            "operation": "Remove",
            "field": "tags['TempResource']",
        },
        {
            "operation": "addOrReplace",
            "field": "tags['Dept']",
            "value": "[parameters('DeptName')]"
        }
    ]
}
```

La propriété **operation** comprend les options suivantes :

|Opération |Description |
|-|-|
|addOrReplace |Ajoute la propriété/l’étiquette et la valeur définies à la ressource, même si la propriété/l’étiquette a déjà une autre valeur. |
|Ajouter |Ajoute la propriété/l’étiquette et la valeur définies à la ressource. |
|Supprimer |Supprime la propriété/l’étiquette définie de la ressource. |

### <a name="modify-examples"></a>Exemples Modify

Exemple 1 : Ajoutez l’étiquette `environment` et remplacez les étiquettes `environment` existantes par « Test » :

```json
"then": {
    "effect": "modify",
    "details": {
        "roleDefinitionIds": [
            "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
        ],
        "operations": [
            {
                "operation": "addOrReplace",
                "field": "tags['environment']",
                "value": "Test"
            }
        ]
    }
}
```

Exemple 2 : Supprimez l’étiquette `env` et ajoutez l’étiquette `environment`, ou remplacez les étiquettes `environment` existantes par une valeur paramétrable :

```json
"then": {
    "effect": "modify",
    "details": {
        "roleDefinitionIds": [
            "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
        ],
        "conflictEffect": "deny",
        "operations": [
            {
                "operation": "Remove",
                "field": "tags['env']"
            },
            {
                "operation": "addOrReplace",
                "field": "tags['environment']",
                "value": "[parameters('tagValue')]"
            }
        ]
    }
}
```

Exemple 3 : Assurez-vous qu’un compte de stockage n’autorise pas l’accès public aux blobs ; l’opération Modify est appliquée uniquement lors de l’évaluation des requêtes dont la version de l’API est supérieure ou égale à « 2019-04-01 » :

```json
"then": {
    "effect": "modify",
    "details": {
        "roleDefinitionIds": [
            "/providers/microsoft.authorization/roleDefinitions/17d1049b-9a84-46fb-8f53-869881c3d3ab"
        ],
        "conflictEffect": "audit",
        "operations": [
            {
                "condition": "[greaterOrEquals(requestContext().apiVersion, '2019-04-01')]",
                "operation": "addOrReplace",
                "field": "Microsoft.Storage/storageAccounts/allowBlobPublicAccess",
                "value": false
            }
        ]
    }
}
```

## <a name="layering-policy-definitions"></a>Superposition de définitions de stratégie

Une ressource peut subir les effets de plusieurs affectations. Ces affectations peuvent se situer dans la même portée ou dans des portées différentes. Chaque affectation est également susceptible d’avoir un effet différent. La condition et l’effet de chaque stratégie sont évalués indépendamment. Par exemple :

- Stratégie 1
  - Restreint l’emplacement de la ressource à 'westus'
  - Affectée à l’abonnement A
  - Effet Deny
- Stratégie 2
  - Restreint l’emplacement de la ressource à 'eastus'
  - Affectée au groupe de ressources B de l’abonnement A
  - Effet Audit
  
Cette configuration génère le résultat suivant :

- Toute ressource figurant déjà dans le groupe de ressources B dans 'eastus' est marquée comme conforme à la stratégie 2 et comme non conforme à la stratégie 1
- Toute ressource figurant déjà dans le groupe de ressources B mais pas dans 'eastus' est marquée comme non conforme à la stratégie 2 et comme non conforme à la stratégie 1 si elle ne figure pas dans 'westus'
- Toute nouvelle ressource figurant dans l’abonnement A mais pas dans 'westus' est refusée par la stratégie 1
- Toute nouvelle ressource de l’abonnement A et du groupe de ressources B dans 'westus' est créée et non conforme à la stratégie 2

Si la stratégie 1 et la stratégie 2 avaient pour effet Deny, le scénario serait le suivant :

- Toute ressource figurant déjà dans le groupe de ressources B mais pas dans 'eastus' est non conforme à la stratégie 2
- Toute ressource figurant déjà dans le groupe de ressources B mais pas dans 'westus' est non conforme à la stratégie 1
- Toute nouvelle ressource figurant dans l’abonnement A mais pas dans 'westus' est refusée par la stratégie 1
- Toute nouvelle ressource du groupe de ressources B de l’abonnement A est refusée

Chaque affectation est évaluée individuellement. Par conséquent, une ressource ne peut pas passer en cas d’écart en raison de différences dans la portée. La superposition de définitions de stratégie est considérée comme **cumulative et la plus restrictive**. Par exemple, si les stratégies 1 et 2 ont un effet Deny, une ressource est bloquée par les définitions de stratégie qui se chevauchent et qui sont en conflit. Si vous avez toujours besoin que la ressource soit créée dans l’étendue cible, révisez les exclusions de chaque attribution pour vérifier que les attributions de stratégies appropriées agissent sur les bonnes étendues.

## <a name="next-steps"></a>Étapes suivantes

- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la [Structure de définition Azure Policy](definition-structure.md).
- Découvrez comment [créer des stratégies par programmation](../how-to/programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](../how-to/get-compliance-data.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).