---
title: Conseils sur la reprise d’activité pour Azure Data Lake Analytics
description: Découvrez comment planifier la reprise d’activité après sinistre pour vos comptes Azure Data Lake Analytics.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 06/03/2019
ms.openlocfilehash: ce5e96bf464eebb9b91e3ffac9312e89df9cbabe
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220973"
---
# <a name="disaster-recovery-guidance-for-azure-data-lake-analytics"></a>Conseils sur la reprise d’activité pour Azure Data Lake Analytics

Azure Data Lake Analytics est un service de travaux d’analyse à la demande, qui simplifie les Big Data. Au lieu de déployer, de configurer et d’optimiser le matériel, vous écrivez des requêtes pour transformer vos données et en extraire des informations pertinentes. Le service d’analyse peut traiter les travaux instantanément, quelle qu’en soit l’échelle, en définissant le compteur sur la puissance requise. Vous payez les travaux uniquement lorsque ceux-ci sont exécutés, ce qui rend le service économique. Cet article vous donne des conseils sur la façon de protéger vos travaux contre les rares pannes ou suppressions accidentelles qui peuvent se produire à l’échelle d’une région.

## <a name="disaster-recovery-guidance"></a>Guide de récupération d’urgence

Si vous utilisez Azure Data Lake Analytics, il est primordial de préparer votre propre plan de reprise d’activité. Cet article vous conseille sur l’élaboration d’un plan de reprise d’activité. Ces ressources complémentaires peuvent aussi vous aider à créer votre propre plan :
- [Défaillances et reprise d’activité après sinistre pour les applications Azure](/azure/architecture/reliability/disaster-recovery)
- [Guide technique de la résilience Azure](/azure/architecture/checklist/resiliency-per-service)

## <a name="best-practices-and-scenario-guidance"></a>Bonnes pratiques et conseils

Vous pouvez exécuter régulièrement un travail U-SQL dans un compte ADLA dans une région qui lit et écrit des tables U-SQL ainsi que des données non structurées.  Préparez la reprise d’activité après un éventuel sinistre en effectuant ces étapes :

1. Créez des comptes ADLA et ADLS dans la région secondaire à utiliser pendant une panne.

   > [!NOTE]
   > Étant donné que les noms de compte sont globalement uniques, utilisez un schéma de nommage cohérent qui indique clairement quel compte est secondaire.

2. Pour des données non structurées, consultez [Conseils sur la reprise d’activité pour les données dans Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-disaster-recovery-guidance.md).

3. Pour les données structurées stockées dans des bases de données et tables ADLA, créez des copies des artefacts de métadonnées telles que des bases de données, des tables, des fonctions table et des assemblys. Vous devez resynchroniser régulièrement ces artefacts pour prendre en compte les changements opérés en production. Par exemple, vous devez répliquer les nouvelles données ajoutées sur la région secondaire en copiant ces données et en les insérant dans la table secondaire.

   > [!NOTE]
   > Ces noms d’objet sont limités au compte secondaire et ne sont pas globalement uniques. Ils peuvent donc être identiques aux noms dans le compte de production principal.

Pendant une panne, vous devez mettre à jour vos scripts afin que les chemins d’entrée pointent vers le point de terminaison secondaire. Les utilisateurs doivent ensuite envoyer leurs travaux vers le compte ADLA dans la région secondaire. La sortie des travaux est alors écrite dans les comptes ADLA et ADLS dans la région secondaire.

## <a name="next-steps"></a>Étapes suivantes

[Conseils sur la reprise d’activité pour les données dans Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-disaster-recovery-guidance.md)
