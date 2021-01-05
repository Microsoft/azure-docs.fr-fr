---
title: Déployer dans un réseau virtuel existant
description: Décrit comment permettre aux utilisateurs de votre application managée de sélectionner un réseau virtuel existant. Le réseau virtuel peut être en dehors de l’application managée.
author: tfitzmac
ms.topic: conceptual
ms.date: 05/11/2020
ms.author: tomfitz
ms.openlocfilehash: fa5e59b96aada06c2dd486094d9be6a52c79e43e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84260682"
---
# <a name="use-existing-virtual-network-with-azure-managed-applications"></a>Utiliser un réseau virtuel existant avec des applications managées Azure

Cet article vous montre comment définir une application managée Azure qui s’intègre à un réseau virtuel existant dans l’abonnement du consommateur. L’application managée donne au consommateur le choix entre créer un réseau virtuel ou utiliser un réseau virtuel existant. Le réseau virtuel existant peut se trouver en dehors du groupe de ressources managé.

## <a name="main-template"></a>Modèle principal

Examinons pour commencer le fichier **mainTemplate.json**. Le modèle complet de déploiement d’une machine virtuelle et des ressources associées est présenté ci-dessous. Plus loin, nous examinerons en détail les parties du modèle liées à l’utilisation d’un réseau virtuel existant.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/mainTemplate.json":::

Notez que le réseau virtuel est [déployé de manière conditionnelle](../templates/conditional-resource-deployment.md). Le consommateur passe une valeur de paramètre qui indique s’il faut créer un réseau virtuel ou utiliser un réseau virtuel existant. Si le consommateur sélectionne un nouveau réseau virtuel, la ressource est déployée. Sinon, la ressource est ignorée durant le déploiement.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/mainTemplate.json" range="111-132" highlight="2":::

La variable de l’ID de réseau virtuel a deux propriétés. Une propriété retourne l’ID de ressource quand un nouveau réseau virtuel est déployé. L’autre propriété retourne l’ID de ressource quand un réseau virtuel existant est utilisé. L’ID de ressource du réseau virtuel existant comprend le nom du groupe de ressources contenant le réseau virtuel.

L’ID de sous-réseau est construit à partir de la valeur de l’ID de réseau virtuel. La valeur correspondant à la sélection du consommateur est utilisée.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/mainTemplate.json" range="98-109" highlight="6-10":::

L’interface réseau est définie avec la variable de l’ID de sous-réseau.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/mainTemplate.json" range="142-163" highlight="16":::

## <a name="ui-definition"></a>Définition de l’interface utilisateur

Examinons à présent le fichier **createUiDefinition.json**. Voici le fichier complet :

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/createUiDefinition.json":::

Le fichier contient un élément de réseau virtuel.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/createUiDefinition.json" range="53-81":::

Cet élément permet au consommateur de sélectionner un réseau virtuel nouveau ou existant.

:::image type="content" source="./media/existing-vnet-integration/new-or-existing-vnet.png" alt-text="Réseau virtuel nouveau ou existant":::

Dans les sorties, incluez une valeur qui indique si le consommateur a sélectionné un réseau virtuel nouveau ou existant. Vous trouverez également une valeur d’identité managée.

> [!NOTE]
> La valeur de sortie de l’identité managée doit être nommée **managedIdentity**.

:::code language="json" source="~/resourcemanager-templates/managed-app-existing-vnet/createUiDefinition.json" range="136-148" highlight="6,12":::

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la création du fichier de définition d’interface utilisateur, consultez [CreateUiDefinition.json pour une expérience de création d’applications managées Azure](create-uidefinition-overview.md).
