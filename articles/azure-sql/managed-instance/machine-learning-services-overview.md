---
title: Machine Learning Services dans Azure SQL Managed Instance (préversion)
description: Cet article fournit une vue d’ensemble ou Machine Learning Services dans Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: machine-learning
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: garyericson
ms.author: garye
ms.reviewer: sstein, davidph
manager: cgronlun
ms.date: 06/03/2020
ms.openlocfilehash: c805bacbd4a2219fb79168ad6426efd8b0a390df
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96324514"
---
# <a name="machine-learning-services-in-azure-sql-managed-instance-preview"></a>Machine Learning Services dans Azure SQL Managed Instance (préversion)

Machine Learning Services est une fonctionnalité d’Azure SQL Managed Instance (préversion) qui fournit un apprentissage automatique en base de données prenant en charge les scripts Python et R. La fonctionnalité inclut les packages Microsoft Python et R pour l’analyse prédictive haute performance et l’apprentissage automatique. Les données relationnelles peuvent être utilisées dans des scripts au moyen de procédures stockées, de scripts T-SQL contenant des instructions Python ou R ou de code R contenant T-SQL.

> [!IMPORTANT]
> Machine Learning Services est une fonctionnalité d’Azure SQL Managed Instance actuellement en préversion publique.
> Cette fonctionnalité en préversion est initialement disponible dans un nombre limité de régions aux États-Unis, en Asie, en Europe et en Australie avec des régions supplémentaires ajoutées plus tard.
>
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>
> [Inscrivez-vous à la préversion](#signup) ci-après.

## <a name="what-is-machine-learning-services"></a>Qu’est-ce que Machine Learning Services ?

Machine Learning Services dans Azure SQL Managed Instance vous permet d’exécuter des scripts Python et R en base de données. Vous pouvez vous en servir pour préparer et nettoyer des données, effectuer l’ingénierie des fonctionnalités et entraîner, évaluer et déployer des modèles Machine Learning dans une base de données. La fonctionnalité exécute vos scripts là où résident les données, ce qui vous évite d’avoir à transférer les données vers un autre serveur à travers le réseau.

Utilisez Machine Learning Services avec le support R/Python dans Azure SQL Managed Instance pour :

- **Exécuter des scripts R et Python pour la préparation des données et le traitement des données à usage général** : vous pouvez maintenant intégrer vos scripts R/Python à Azure SQL Managed Instance où résident vos données, au lieu de déplacer les données vers un autre serveur pour exécuter des scripts R et Python. Vous pouvez éliminer le besoin de déplacement des données et les problèmes associés à la latence, à la sécurité et à la conformité.

- **Effectuer l'apprentissage des modèles de Machine Learning dans la base de données** : vous pouvez effectuer l'apprentissage des modèles à l’aide de n’importe quel algorithme Open source. Vous pouvez facilement mettre à l'échelle votre formation sur l’ensemble du jeu de données plutôt qu’en vous fiant à des échantillons de jeux de données tirés hors de la base de données.

- **Déployez vos modèles et scripts en production dans des procédures stockées** : les scripts et les modèles formés peuvent être mis en œuvre simplement en les incorporant dans des procédures stockées T-SQL. Les applications se connectant à Azure SQL Managed Instance peuvent tirer parti des prédictions et de l’intelligence de ces modèles en appelant simplement une procédure stockée. Vous pouvez également utiliser la fonction PREDICT T-SQL native pour mettre en œuvre des modèles en vue d’un scoring rapide dans des scénarios de scoring en temps réel hautement simultanés.

Les distributions de base de Python et de R sont incluses dans Machine Learning Services. Vous pouvez installer et utiliser des frameworks et des packages open source comme PyTorch, TensorFlow et scikit-learn, en plus des packages Microsoft [revoscalepy](/sql/advanced-analytics/python/ref-py-revoscalepy) et [microsoftml](/sql/advanced-analytics/python/ref-py-microsoftml) pour Python, et [RevoScaleR](/sql/advanced-analytics/r/ref-r-revoscaler), [MicrosoftML](/sql/advanced-analytics/r/ref-r-microsoftml), [olapR](/sql/advanced-analytics/r/ref-r-olapr) et [sqlrutils](/sql/advanced-analytics/r/ref-r-sqlrutils) pour R.

<a name="signup"></a>

## <a name="sign-up-for-the-preview"></a>S’inscrire à la version préliminaire

Cette préversion publique limitée est soumise aux [conditions des préversions d'Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). 

Si vous souhaitez rejoindre le programme en préversion et accepter ces conditions, vous pouvez demander une inscription en créant un ticket de support Azure sur [ **https://azure.microsoft.com/support/create-ticket/**](https://azure.microsoft.com/support/create-ticket/). 

1. Dans la page **Créer un ticket de support**, cliquez sur **Créer un incident**.

1. Sur la page **Aide + support**, cliquez sur **Nouvelle demande de support** pour créer un ticket.

1. Sélectionnez les options suivantes :
   - Type de problème : **technique**
   - Abonnement : *sélectionnez votre abonnement*
   - Service – **SQL Managed Instance**
   - Resource – *sélectionnez votre instance gérée*
   - Récapitulatif : *fournissez une brève description de votre demande*
   - Type de problème : **Machine Learning Services pour SQL Managed Instance (préversion)**
   - Sous-type de problème : **Autre problème ou questions pratiques**

1. Cliquez sur **Suivant : Solutions**.

1. Lisez les informations sur la préversion, puis cliquez sur **Suivant : Détails**.

1. Sur cette page :
   - En réponse à la question **Essayez-vous de vous inscrire à la préversion ?** , sélectionnez **Oui**. 
   - Dans **Description**, entrez les détails de votre demande, à savoir le nom, la région et l’ID d’abonnement du serveur logique que vous souhaitez inscrire dans la préversion. Entrez d’autres détails, le cas échéant.
   - Sélectionnez votre méthode de contact préférée. 

1. Lorsque vous avez fini, cliquez sur **Suivant : Vérifier + créer**, puis sur **Créer**.

Une fois inscrit dans le programme, Microsoft vous intègre à la préversion publique et active Machine Learning Services sur votre base de données existante ou nouvelle.

L’utilisation de Machine Learning Services dans SQL Managed Instance n’est pas recommandée pour des charges de travail de production pendant la préversion publique.

## <a name="next-steps"></a>Étapes suivantes

- Prenez connaissance des [différences principales de Machine Learning SQL Services dans SQL Server](machine-learning-services-differences.md).
- Pour apprendre à utiliser Python dans Machine Learning Services, consultez [Exécuter des scripts Python](/sql/machine-learning/tutorials/quickstart-python-create-script?context=%2fazure%2fazure-sql%2fmanaged-instance%2fcontext%2fml-context&view=sql-server-ver15).
- Pour apprendre à utiliser R dans Machine Learning Services, consultez [Exécuter des scripts R](/sql/machine-learning/tutorials/quickstart-r-create-script?context=%2fazure%2fazure-sql%2fmanaged-instance%2fcontext%2fml-context&view=sql-server-ver15).
- Pour plus d’informations sur le machine learning sur d’autres plateformes SQL, consultez la [documentation sur le machine learning SQL](/sql/machine-learning/).