---
title: Copier ou cloner une fabrique de données dans Azure Data Factory
description: Découvrir comment copier ou cloner une fabrique de données dans Azure Data Factory
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
author: chez-charlie
ms.author: chez
manager: jroth
ms.reviewer: maghan
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 304c39f4b6f7852068d4e72adfad2d41eeefc26c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85552963"
---
# <a name="copy-or-clone-a-data-factory-in-azure-data-factory"></a>Copier ou cloner une fabrique de données dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article montre comment copier ou cloner une fabrique de données dans Azure Data Factory.

## <a name="use-cases-for-cloning-a-data-factory"></a>Cas d’usage pour le clonage d’une fabrique de données

Voici quelques-uns des cas dans lesquels il peut être utile de copier ou cloner une fabrique de données :

- **Déplacement de Data Factory** vers une nouvelle région. Si vous souhaitez déplacer votre Data Factory vers une autre région, le meilleur moyen consiste à créer une copie dans la région ciblée et à supprimer celle qui existe déjà.

- **Attribution d’un nouveau nom à Data Factory**. Azure ne permet pas de renommer des ressources. Si vous souhaitez renommer une fabrique de données, vous pouvez cloner la fabrique de données avec un nom différent et supprimer la fabrique de données existante.

- **Débogage des modifications** lorsque les fonctionnalités de débogage ne sont pas suffisantes. Dans la plupart des scénarios, vous pouvez utiliser le [débogage](iterative-development-debugging.md). En d’autres termes, le test des modifications dans un environnement de bac à sable cloné est plus judicieux. Par exemple, le comportement de vos pipelines ETL paramétrés à l’activation d’un déclencheur lors de l’arrivée d’un fichier par rapport à la fenêtre de temps bascule, peut s’avérer difficile à tester via le débogage seul. Dans ce cas, vous souhaiterez peut-être cloner un environnement de bac à sable (sandbox) pour l’expérimentation. Comme Azure Data Factory facture principalement en fonction du nombre d’exécutions, une deuxième fabrique n’entraîne pas de frais supplémentaires.

## <a name="how-to-clone-a-data-factory"></a>Guide pratique de clonage d’une entité Data Factory

1. Comme condition préalable, vous devez commencer par créer votre fabrique de données cible à partir du portail Azure.

1. Si vous êtes en mode GIT :
    1. Chaque fois que vous publiez à partir du portail, le modèle Resource Manager de la fabrique est enregistré dans GIT, dans la branche adf\_publish.
    1. Connectez la nouvelle fabrique au _même_ dépôt et générez-la à partir de la branche adf\_publish. Les ressources, telles que les pipelines, les jeux de données et les déclencheurs, s’exécutent

1. Si vous êtes en mode réel :
    1. L’interface utilisateur Data Factory vous permet d’exporter la charge utile entière de votre fabrique de données dans un fichier de modèle Resource Manager et un fichier de paramètres. Ils sont accessibles à partir du bouton **Modèle ARM \ Exporter le modèle Resource Manager** dans le portail.
    1. Vous pouvez apporter les modifications appropriées au fichier de paramètres et échanger les nouvelles valeurs de la nouvelle fabrique.
    1. Ensuite, vous pouvez le déployer via les méthodes de déploiement de modèle Resource Manager standard.

1. Si vous avez un runtime d’intégration auto-hébergé dans votre fabrique source, vous devez le précréer avec le même nom dans la fabrique cible. Si vous souhaitez partager le runtime d’intégration auto-hébergé entre différentes fabriques, vous pouvez utiliser le modèle publié [ici](create-shared-self-hosted-integration-runtime-powershell.md) lors du partage du runtime d’intégration auto-hébergé.

1. Pour des raisons de sécurité, le modèle Resource Manager généré ne contient aucune information secrète, telle que les mots de passe des services liés. Par conséquent, vous devez fournir les informations d’identification en tant que paramètres de déploiement. Si l’entrée manuelle des informations d’identification n’est pas souhaitable pour vos paramètres, envisagez de récupérer les chaînes de connexion et les mots de passe à partir d’Azure Key Vault à la place. [En savoir plus](store-credentials-in-key-vault.md)

## <a name="next-steps"></a>Étapes suivantes

Passez en revue les conseils de création d’une fabrique de données dans le portail Azure à la rubrique [Créer une fabrique de données à l’aide de l’interface utilisateur d’Azure Data Factory](quickstart-create-data-factory-portal.md).
