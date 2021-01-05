---
title: Modes de déploiement
description: Explique comment spécifier s’il faut utiliser un mode de déploiement complet ou incrémentiel avec Azure Resource Manager.
ms.topic: conceptual
ms.date: 07/22/2020
ms.openlocfilehash: bc499be4185905af7eaf71b3515895de9bee46d3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184040"
---
# <a name="azure-resource-manager-deployment-modes"></a>Modes de déploiement Azure Resource Manager

Lorsque vous déployez vos ressources, vous spécifiez que le déploiement est soit une mise à jour incrémentielle, soit une mise à jour complète. La différence entre ces deux modes réside dans la manière dont le Gestionnaire des ressources gère les ressources existantes dans le groupe de ressources qui ne se trouvent pas dans le modèle.

Pour les deux modes, Resource Manager essaie de créer toutes les ressources spécifiées dans le modèle. Si la ressource existe déjà dans le groupe de ressources et que ses paramètres sont conservés, aucune opération n’est effectuée pour cette ressource. Si vous modifiez les valeurs de propriété d’une ressource, la ressource est mise à jour avec ces nouvelles valeurs. Si vous essayez de mettre à jour l’emplacement ou le type d’une ressource existante, le déploiement échoue avec une erreur. Vous devez dans ce cas déployer une nouvelle ressource avec l’emplacement ou le type dont vous avez besoin.

Le mode par défaut est incrémentiel.

## <a name="complete-mode"></a>Mode Complet

En mode complet, le Gestionnaire des ressources **supprime** les ressources qui existent dans le groupe de ressources, mais qui ne sont pas spécifiées dans le modèle.

> [!NOTE]
> Effectuez toujours l’[opération de simulation](template-deploy-what-if.md) avant de déployer un modèle en mode complet. La simulation vous indique les ressources qui seront créées, supprimées ou modifiées. Effectuez une simulation pour éviter la suppression accidentelle de ressources.

Si votre modèle inclut une ressource qui n’est pas déployée parce que la [condition](conditional-resource-deployment.md) donne false, le résultat dépend de la version de l’API REST utilisée. Si vous utilisez une version antérieure à 2019-05-10, la ressource **n’est pas supprimée**. Avec 2019-05-10 ou ultérieur, elle **est supprimée**. Les dernières versions d’Azure PowerShell et d’Azure CLI suppriment la ressource.

Soyez prudent lorsque vous utilisez le mode Complet avec les [boucles de copie](copy-resources.md). Toutes les ressources qui ne sont pas spécifiées dans le modèle après la résolution de la boucle de copie sont supprimées.

Si vous déployez sur [plusieurs groupes de ressources dans un modèle](./deploy-to-resource-group.md), les ressources du groupe de ressources spécifié dans le déploiement peuvent être supprimées. Les ressources dans les groupes de ressources secondaires ne sont pas supprimées.

Il existe des différences dans la manière dont les types de ressources traitent les suppressions en mode complet. Les ressources parentes sont automatiquement supprimées lorsqu'elles ne figurent pas dans un modèle déployé en mode complet. Certaines ressources enfants ne sont pas automatiquement supprimées lorsqu'elles ne figurent pas dans le modèle. Toutefois, ces ressources enfants sont supprimées si la ressource parent est supprimée.

Par exemple, si votre groupe de ressources contient une zone DNS (type de ressource Microsoft.Network/dnsZones) et un enregistrement CNAME (type de ressource Microsoft.Network/dnsZones/CNAME), la zone DNS est la ressource parente de l'enregistrement CNAME. Si vous procédez à un déploiement en mode complet et que vous n'incluez pas la zone DNS dans votre modèle, la zone DNS et l'enregistrement CNAME sont tous deux supprimés. Si vous incluez la zone DNS dans votre modèle mais pas l’enregistrement CNAME, ce dernier n’est pas supprimé.

Pour plus d'informations sur la façon dont les types de ressources gèrent la suppression, consultez [Suppression de ressources Azure pour les déploiements en mode complet](complete-mode-deletion.md).

Si le groupe de ressources est [verrouillé](../management/lock-resources.md), le mode complet ne supprime pas les ressources.

> [!NOTE]
> Seuls les modèles de niveau racine prennent en charge le mode de déploiement complet. Pour les [modèles liés ou imbriqués](linked-templates.md), vous pouvez utiliser le mode incrémentiel.
>
> Les [déploiements de niveau abonnement](deploy-to-subscription.md) ne prennent pas en charge le mode complet.
>
> Actuellement, le portail ne prend en charge le mode complet.
>

## <a name="incremental-mode"></a>Mode incrémentiel

En mode incrémentiel, le Gestionnaire des ressources **conserve telles quelles** les ressources qui existent dans le groupe de ressources, mais qui ne sont pas spécifiées dans le modèle. Les ressources du modèle **sont ajoutées** au groupe de ressources.

> [!NOTE]
> Lors du redéploiement d’une ressource existante en mode incrémentiel, toutes les propriétés sont réappliquées. Les **propriétés ne sont pas ajoutées de manière incrémentielle**. On pense souvent à tort que les propriétés qui ne sont pas spécifiées dans le modèle restent inchangées. Si vous omettez de spécifier certaines propriétés, Resource Manager interprète le déploiement comme un remplacement de ces valeurs. Les propriétés qui ne sont pas incluses dans le modèle sont réinitialisées aux valeurs par défaut. Spécifiez toutes les valeurs non définies par défaut pour la ressource et pas seulement celles que vous mettez à jour. La définition de la ressource dans le modèle contient toujours l’état final de la ressource. Elle ne peut pas représenter une mise à jour partielle d’une ressource existante.
>
> Dans de rares cas, les propriétés que vous spécifiez pour une ressource sont en réalité implémentées en tant que ressource enfants. Par exemple, lorsque vous fournissez des valeurs de configuration de site pour une application web, ces valeurs sont implémentées dans le type de ressource enfant `Microsoft.Web/sites/config`. Si vous redéployez l’application web et spécifiez un objet vide pour les valeurs de configuration de site, la ressource enfant n’est pas mise à jour. Toutefois, si vous fournissez de nouvelles valeurs de configuration de site, le type de ressource enfant est mis à jour.

## <a name="example-result"></a>Résultat de l’exemple

Pour illustrer la différence entre les modes incrémentiel et complet, examinez le scénario suivant.

Le **groupe de ressources** contient :

* Ressource A
* Ressource B
* Ressource C

Le **modèle** contient :

* Ressource A
* Ressource B
* Ressource D

Lors du déploiement en mode **incrémentiel**, le groupe de ressources contient :

* Ressource A
* Ressource B
* Ressource C
* Ressource D

Lors du déploiement en mode **complet**, la ressource C est supprimée. Le groupe de ressources contient :

* Ressource A
* Ressource B
* Ressource D

## <a name="set-deployment-mode"></a>Définir un mode de déploiement

Pour définir le mode de déploiement lors du déploiement avec PowerShell, utilisez le paramètre `Mode`.

```azurepowershell-interactive
New-AzResourceGroupDeployment `
  -Mode Complete `
  -Name ExampleDeployment `
  -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\storage.json
```

Pour définir le mode de déploiement lors du déploiement avec Azure CLI, utilisez le paramètre `mode`.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --mode Complete \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS
```

L’exemple suivant montre un modèle lié dans le mode de déploiement incrémentiel :

```json
"resources": [
  {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "linkedTemplate",
      "properties": {
          "mode": "Incremental",
          <nested-template-or-external-template>
      }
  }
]
```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la création de modèles Resource Manager, consultez [Création de modèles Azure Resource Manager](template-syntax.md).
* Pour en savoir plus sur le déploiement de ressources, consultez [Déploiement d’une application avec un modèle Azure Resource Manager](deploy-powershell.md).
* Pour afficher les opérations pour un fournisseur de ressources, consultez [API REST Azure](/rest/api/).