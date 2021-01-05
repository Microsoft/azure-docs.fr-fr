---
title: Entités nommées pour la santé
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 10/02/2020
ms.author: aahi
ms.openlocfilehash: 308b2a9cb00f44f0e330d4fef5592d8855ee3394
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94553181"
---
## <a name="text-analytics-for-health-categories-entities-and-attributes"></a>Analyse de texte pour la santé - Catégories, entités et attributs

[Analyse de texte pour la santé](../../how-tos/text-analytics-for-health.md) détecte les concepts médicaux dans les catégories suivantes.  (Notez que seul le texte en anglais est pris en charge dans cette préversion de conteneur et qu’une seule version de modèle est fournie dans chaque image conteneur.)


| Category  | Description  |
|---------|---------|
| ANATOMY | concepts qui capturent des informations sur le corps et les systèmes anatomiques, les sites, les emplacements ou les régions. |
 | DEMOGRAPHICS | concepts qui capturent des informations sur le sexe et l’âge. |
 | EXAMINATION | concepts qui capturent des informations sur les procédures de diagnostic et les tests. |
 | GENOMICS | concepts qui capturent des informations sur les gènes et les variantes. |
 | HEALTHCARE | concepts qui capturent des informations sur les événements administratifs, les environnements de soins et les professions de santé. |
 | MEDICAL CONDITION | concepts qui capturent des informations sur les diagnostics, les symptômes ou les signes. |
 | MEDICATION | concepts qui capturent des informations sur les médicaments, y compris les noms des médicaments, les classes, la posologie et la voie d’administration. |
 | SOCIAL | concepts qui capturent des informations sur les aspects sociaux pertinents d’un point de vue médical, comme la famille. |
 | TREATMENT | concepts qui capturent des informations sur les procédures thérapeutiques. |
  
Chaque catégorie peut inclure deux groupes de concepts :

* **Entités** : termes qui capturent des concepts médicaux tels que le diagnostic, le nom du médicament ou le nom du traitement.  Par exemple : *bronchitis* (bronchite) est un diagnostic et *aspirin* (aspirine) est un nom de médicament.  Les mentions dans ce groupe peuvent être liées à un ID de concept UMLS.
* **Attributs** : expressions qui fournissent plus d’informations sur l’entité détectée. Par exemple : *severe* (grave) est un qualificateur de condition pour *bronchitis* (bronchite) ou *81 mg* est une posologie pour *aspirin* (aspirine).  Les mentions dans cette catégorie ne seront PAS liées à un ID de concept UMLS.

En outre, le service reconnaît les relations entre les différents concepts, y compris les relations entre les attributs et les entités. Par exemple : *direction* et *body structure* (structure du corps) ou *dosage* (posologie) et *medication name* (nom de médicament) et les relations entre les entités, par exemple en cas de détection d’une abréviation.

## <a name="anatomy"></a>Anatomie

### <a name="entities"></a>Entités

**BODY_STRUCTURE** : systèmes corporels, emplacements ou régions anatomiques et sites du corps. Par exemple : arm, knee, abdomen, nose, liver, head, respiratory system, lymphocytes (bras, genou, abdomen, nez, foie, tête, système respiratoire, lymphocytes).

:::image type="content" source="../../media/ta-for-health/anatomy-entities-body-structure.png" alt-text="Exemple d’entité de structure du corps.":::


:::image type="content" source="../../media/ta-for-health/anatomy-entities-body-structure-2.png" alt-text="Exemple étendu d’entité de structure du corps.":::

### <a name="attributes"></a>Attributs

**DIRECTION** : termes directionnels qui caractérisent une structure du corps. Par exemple : left, lateral, upper, posterior (gauche, latéral, supérieur, postérieur).

:::image type="content" source="../../media/ta-for-health/anatomy-attributes.png" alt-text="Exemple d’attribut directionnel.":::

### <a name="supported-relations"></a>Relations prises en charge

* **DIRECTION_OF_BODY_STRUCTURE**

## <a name="demographics"></a>Démographie

### <a name="entities"></a>Entités

**AGE** : tous les termes et expressions relatifs à l’âge, y compris celui d’un patient, de membres de la famille et autres. Par exemple : 40-year-old, 51 yo, 3 months old, adult, infant, elderly, young, minor, middle-aged (40 ans, 51, 3 mois, adulte, nourrisson, personne âgée, jeune, mineur, âge moyen).

:::image type="content" source="../../media/ta-for-health/age-entity.png" alt-text="Exemple d’entité d’âge.":::

:::image type="content" source="../../media/ta-for-health/age-entity-2.png" alt-text="Autre exemple d’entité d’âge.":::


**GENDER** : termes qui révèlent le sexe du sujet. Par exemple : male, female, woman, gentleman, lady (homme, femme, féminin, masculin).

:::image type="content" source="../../media/ta-for-health/gender-entity.png" alt-text="Exemple d’une entité de sexe.":::

### <a name="attributes"></a>Attributs

**RELATIONAL_OPERATOR** : expressions qui expriment la relation entre une entité démographique et des informations supplémentaires.

:::image type="content" source="../../media/ta-for-health/relational-operator.png" alt-text="Exemple d’opérateur relationnel.":::

## <a name="examinations"></a>Examens

### <a name="entities"></a>Entités

**EXAMINATION_NAME** : procédures de diagnostic et tests. Par exemple : MRI, ECG, HIV test, hemoglobin, platelets count, scale systems (*Bristol stool scale*) (IRM, ECG, test du VIH, hémoglobine, nombre de plaquettes, systèmes d’échelle comme l’Échelle de Bristol).

:::image type="content" source="../../media/ta-for-health/exam-name-entities.png" alt-text="Exemple d’entité d’examen.":::

:::image type="content" source="../../media/ta-for-health/exam-name-entities-2.png" alt-text="Autre exemple d’entité de nom d’examen.":::

### <a name="attributes"></a>Attributs

**DIRECTION** : termes directionnels caractérisant un examen.

:::image type="content" source="../../media/ta-for-health/exam-direction-attribute.png" alt-text="Exemple d’attribut directionnel avec une entité de nom d’examen.":::

**MEASUREMENT_UNIT** : unité de l’examen. Par exemple, dans *hemoglobin > 9.5 g/dL*, le terme *g/dL* est l’unité du test *hemoglobin*.

:::image type="content" source="../../media/ta-for-health/exam-unit-attribute.png" alt-text="Exemple d’attribut d’unité de mesure avec une entité de nom d’examen.":::

**MEASUREMENT_VALUE** : valeur de l’examen. Par exemple, dans *hemoglobin > 9.5 g/dL*, le terme *9.5* est la valeur du test *hemoglobin*.

:::image type="content" source="../../media/ta-for-health/exam-value-attribute.png" alt-text="Exemple d’attribut de valeur de mesure avec une entité de nom d’examen.":::

**RELATIONAL_OPERATOR** : expressions qui expriment la relation entre un examen et des informations supplémentaires. Par exemple, la valeur de mesure requise pour un examen cible.

:::image type="content" source="../../media/ta-for-health/exam-relational-operator-attribute.png" alt-text="Exemple d’opérateur relationnel avec une entité de nom d’examen.":::

**TIME** : termes temporels relatifs au début et/ou à la durée d’un examen. Par exemple, quand le test a eu lieu.

:::image type="content" source="../../media/ta-for-health/exam-time-attribute.png" alt-text="Exemple d’attribut de temps avec une entité de nom d’examen.":::

### <a name="supported-relations"></a>Relations prises en charge

+ **DIRECTION_OF_EXAMINATION**
+   **RELATION_OF_EXAMINATION**
+   **TIME_OF_EXAMINATION**
+   **UNIT_OF_EXAMINATION**
+   **VALUE_OF_EXAMINATION**

## <a name="genomics"></a>Genomics

### <a name="entities"></a>Entités

**GENE** : toutes les mentions concernant les gènes. Par exemple : MTRR, F2.

:::image type="content" source="../../media/ta-for-health/genomics-entities.png" alt-text="Exemple d’une entité de gène.":::

**VARIANT** : toutes les mentions des variantes d’un gène. Par exemple : c.524C>T, (MTRR):r.1462_1557del96
  
## <a name="healthcare"></a>Santé

### <a name="entities"></a>Entités
  
**ADMINISTRATIVE_EVENT** : événements liés au système de santé, mais d’une nature administrative/semi-administrative. Par exemple : registration, admission, trial, study entry, transfer, discharge, hospitalization, hospital stay (enregistrement, admission, essai, entrée d’étude, transfert, décharge, hospitalisation, séjour hospitalier). 

:::image type="content" source="../../media/ta-for-health/healthcare-event-entity.png" alt-text="Exemple d’entité d’événement de santé.":::

**CARE_ENVIRONMENT** : environnement ou emplacement où les patients reçoivent des soins. Par exemple : emergency room, physician’s office, cardio unit, hospice, hospital (salle des urgences, bureau du médecin, unité de cardiologie, hospice, hôpital).

:::image type="content" source="../../media/ta-for-health/healthcare-environment-entity.png" alt-text="Cette capture d’écran montre un exemple d’entité d’environnement médical.":::

**HEALTHCARE_PROFESSION** : praticien de la santé avec ou sans licence. Par exemple : dentist, pathologist, neurologist, radiologist, pharmacist, nutritionist, physical therapist, chiropractor (dentiste, pathologiste, neurologue, radiologue, pharmacien, nutritionniste, physiothérapeute, chiropracteur).

:::image type="content" source="../../media/ta-for-health/healthcare-profession-entity.png" alt-text="Cette capture d’écran montre un autre exemple d’entité d’environnement médical.":::

:::image type="content" source="../../media/ta-for-health/healthcare-profession-entity-2.png" alt-text="Autre exemple d’entité d’environnement de santé.":::

## <a name="medical-condition"></a>État pathologique

### <a name="entities"></a>Entités

**DIAGNOSIS** : maladie, syndrome, empoisonnement. Par exemple : breast cancer, Alzheimer’s, HTN, CHF, spinal cord injury (cancer du sein, Alzheimer, HTN, CHF, lésion médullaire).

:::image type="content" source="../../media/ta-for-health/medical-condition-entity.png" alt-text="Exemple d’entité d’état pathologique.":::

:::image type="content" source="../../media/ta-for-health/medical-condition-entity-2.png" alt-text="Autre exemple d’entité d’état pathologique.":::

**SYMPTOM_OR_SIGN** : preuve subjective ou objective de la maladie ou d’autres diagnostics. Par exemple : chest pain, headache, dizziness, rash, SOB, abdomen was soft, good bowel sounds, well nourished (douleur de poitrine, mal de tête, vertiges, éruption, SOB, abdomen mou, sons des intestins appropriés, bien nourri).

:::image type="content" source="../../media/ta-for-health/medical-condition-symptom-entity.png" alt-text="Exemple d’entité de signe ou de symptôme d’état pathologique.":::

:::image type="content" source="../../media/ta-for-health/medical-condition-symptom-entity-2.png" alt-text="Autre exemple d’entité de signe ou de symptôme d’état pathologique.":::

### <a name="attributes"></a>Attributs

**CONDITION_QUALIFIER** : termes qualitatifs utilisés pour décrire un état pathologique. Toutes les sous-catégories suivantes sont considérées comme des qualificateurs :

1.  Expressions liées au temps : il s’agit de termes qui décrivent la dimension du temps de manière qualitative. Par exemple : sudden, acute, chronic, longstanding (soudain, aigu, chronique, longue durée). 
2.  Expressions de qualité :  Il s’agit de termes qui décrivent la « nature » de l’état pathologique. Par exemple : burning, sharp (brûlure, aigu).
3.  Expressions de gravité : severe, mild, a bit, uncontrolled (grave, léger, un peu, non contrôlé).
4.  Expressions d’étendue : local, focal, diffuse (local, concentré, diffus).
5.  Expressions de rayonnement : radiates, radiation (irradie, rayonnement).
6.  Échelle de condition Dans certains cas, une condition est caractérisée par une échelle, qui est une liste triée finie de valeurs. Par exemple : Patients with stage III pancreatic cancer (patients avec cancer pancréatique en la phase III).
7.  Cours de condition : Terme relatif au cours ou à la progression d’une condition. Par exemple : improvement, worsening, resolution, remission (amélioration, détérioration, guérison et rémission). 

:::image type="content" source="../../media/ta-for-health/condition-qualifier-diagnosis.png" alt-text="Exemple d’attribut de qualificateur de condition et d’entité de diagnostic.":::

:::image type="content" source="../../media/ta-for-health/condition-qualifier-diagnosis-2.png" alt-text="Autre exemple d’attribut de qualificateur de condition et d’entité de diagnostic.":::

:::image type="content" source="../../media/ta-for-health/conditional-qualifier-symptom-medication.png" alt-text="Exemple d’attribut de qualificateur de condition avec entités de symptôme et de médicament.":::

:::image type="content" source="../../media/ta-for-health/condition-qualifier-diagnosis-3.png" alt-text="Cette capture d’écran montre un autre exemple d’attribut de qualificateur de condition avec une entité de diagnostic.":::

:::image type="content" source="../../media/ta-for-health/condition-qualifier-symptom.png" alt-text="Cette capture d’écran montre un autre exemple d’attribut de qualificateur de condition avec une entité de diagnostic.":::

**DIRECTION** : termes directionnels caractérisant un état pathologique corporel.

:::image type="content" source="../../media/ta-for-health/medical-condition-direction-attribute.png" alt-text="Exemple d’attribut directionnel avec une entité d’état pathologique.":::

**FREQUENCY** : fréquence à laquelle un état pathologique s’est produite, se produit ou devrait se produire.

:::image type="content" source="../../media/ta-for-health/medical-condition-frequency-attribute.png" alt-text="Exemple d’attribut de fréquence avec une entité d’état pathologique.":::

:::image type="content" source="../../media/ta-for-health/medical-condition-frequency-attribute-2.png" alt-text="Autre exemple d’attribut directionnel avec une entité de symptôme ou de signe.":::

**MEASUREMENT_UNIT** : unité qui caractérise un état pathologique. Par exemple, dans *1.5x2x1 cm tumor*, le terme *cm* est l’unité de mesure pour *tumor* (tumeur). 

:::image type="content" source="../../media/ta-for-health/medical-condition-measure-unit-attribute.png" alt-text="Exemple d’attribut d’unité de mesure avec une entité d’état pathologique.":::

**MEASUREMENT_VALUE** : valeur qui caractérise un état pathologique. Par exemple, dans *1.5x2x1 cm tumor*, le terme *1.5x2x1* est la valeur de mesure pour *tumor* (tumeur). 

:::image type="content" source="../../media/ta-for-health/medical-condition-measure-value-attribute.png" alt-text="Cette capture d’écran montre un exemple d’attribut directionnel avec une entité de symptôme ou de signe.":::

**RELATIONAL_OPERATOR** : expressions qui expriment la relation entre un état pathologique et des informations supplémentaires. Par exemple, la valeur de temps ou de mesure. 

:::image type="content" source="../../media/ta-for-health/medical-condition-relational-operator.png" alt-text="Cette capture d’écran montre un autre exemple d’attribut directionnel avec une entité de symptôme ou de signe.":::

**TIME** : termes temporels relatifs au début et/ou à la durée d’un état pathologique. Par exemple, quand un symptôme a commencé (apparition) ou quand une maladie s’est déclarée.

:::image type="content" source="../../media/ta-for-health/medical-condition-time-attribute.png" alt-text="Cette capture d’écran montre un autre exemple d’attribut directionnel avec une entité de symptôme ou de signe.":::

### <a name="supported-relations"></a>Relations prises en charge

+ **DIRECTION_OF_CONDITION**
+   **QUALIFIER_OF_CONDITION**
+   **TIME_OF_CONDITION**
+   **UNIT_OF_CONDITION**
+   **VALUE_OF_CONDITION**

## <a name="medication"></a>Médicament

### <a name="entities"></a>Entités

**MEDICATION_CLASS** : ensemble de médicaments avec un mécanisme d’action similaire, un mode d’action associé, une structure chimique similaire et/ou qui sont utilisés pour traiter la même maladie. Par exemple : ACE inhibitor, opioid, antibiotics, pain relievers (inhibiteur ACE, opioïdes, antibiotiques, anti-douleur).

:::image type="content" source="../../media/ta-for-health/medication-entities-class.png" alt-text="Exemple d’entité de classe de médicament.":::

**MEDICATION_NAME** : mentions médicamenteuses, y compris les noms de marques et autres noms. Par exemple : Advil, Ibuprofen.

:::image type="content" source="../../media/ta-for-health/medication-entities-name.png" alt-text="Exemple d’entité de nom de médicament.":::

### <a name="attributes"></a>Attributs

**DOSAGE** : quantité de médicament demandée. Par exemple : Infuse Sodium Chloride solution *1000 mL* (Infuser solution de chlorure de sodium, 1 000 ml).

:::image type="content" source="../../media/ta-for-health/medication-dosage.png" alt-text="Exemple d’attribut de posologie de médicament.":::

**FREQUENCY** : fréquence à laquelle un médicament doit être pris.

:::image type="content" source="../../media/ta-for-health/medication-frequency.png" alt-text="Exemple d’attribut de fréquence de médicament.":::

:::image type="content" source="../../media/ta-for-health/medication-frequency-2.png" alt-text="Autre exemple d’attribut de fréquence de médicament.":::

**MEDICATION_FORM** : forme du médicament. Par exemple : solution, pill, capsule, tablet, patch, gel, paste, foam, spray, drops, cream, syrup (solution, pilule, capsule, tablette, patch, gel, pâte, mousse, spray, goutte, crème, sirop).

:::image type="content" source="../../media/ta-for-health/medication-form.png" alt-text="Exemple d’attribut de forme de médicament.":::

**MEDICATION_ROUTE** : méthode d’administration de médicament. Par exemple : oral, vaginal, IV, epidural, topical, inhaled (oral, vaginal, intraveineuse, épidurale, topique, inhalé).

:::image type="content" source="../../media/ta-for-health/medication-route.png" alt-text="Exemple d’attribut d’administration de médicament.":::

**RELATIONAL_OPERATOR** : expressions qui expriment la relation entre un médicament et des informations supplémentaires. Par exemple, la valeur de mesure requise.

:::image type="content" source="../../media/ta-for-health/medication-relational-operator.png" alt-text="Cette capture d’écran montre un exemple d’attribut d’opérateur relationnel avec une entité médicament.":::

:::image type="content" source="../../media/ta-for-health/medication-time.png" alt-text="Cette capture d’écran montre un autre exemple d’attribut d’opérateur relationnel avec une entité médicament.":::

### <a name="supported-relations"></a>Relations prises en charge

+ **DOSAGE_OF_MEDICATION**
+   **FORM_OF_MEDICATION**
+   **FREQUENCY_OF_MEDICATION**
+   **ROUTE_OF_MEDICATION**
+   **TIME_OF_MEDICATION**
  
## <a name="treatment"></a>Traitement

### <a name="entities"></a>Entités

**TREATMENT_NAME** : procédures thérapeutiques. Par exemple : knee replacement surgery, bone marrow transplant, TAVI, diet (chirurgie de remplacement du genou, transplantation de moelle osseuse, TAVI, régime).

:::image type="content" source="../../media/ta-for-health/treatment-entities-name.png" alt-text="Exemple d’entité de nom de traitement.":::

### <a name="attributes"></a>Attributs

**DIRECTION** : termes directionnels caractérisant un traitement.

:::image type="content" source="../../media/ta-for-health/treatment-direction.png" alt-text="Cette capture d’écran montre un exemple d’attribut de direction de traitement.":::

**FREQUENCY** : fréquence à laquelle un traitement se produit ou doit se produire.

:::image type="content" source="../../media/ta-for-health/treatment-frequency.png" alt-text="Cette capture d’écran montre un autre exemple d’attribut de direction de traitement.":::
 
**RELATIONAL_OPERATOR** : expressions qui expriment la relation entre un traitement et des informations supplémentaires.  Par exemple, le temps passé depuis la procédure précédente.

:::image type="content" source="../../media/ta-for-health/treatment-relational-operator.png" alt-text="Exemple d’attribut d’opérateur relationnel de traitement.":::

**TIME** : termes temporels relatifs au début et/ou à la durée d’un traitement. Par exemple, la date à laquelle le traitement a été donné.

:::image type="content" source="../../media/ta-for-health/treatment-time.png" alt-text="Cette capture d’écran montre un exemple d’attribut de temps de traitement.":::


### <a name="supported-relations"></a>Relations prises en charge

+ **DIRECTION_OF_TREATMENT**
+   **TIME_OF_TREATMENT**
+   **FREQUENCY_OF_TREATMENT**

## <a name="social"></a>Réseaux sociaux

### <a name="entities"></a>Entités

**FAMILY_RELATION** : mentions relatives à la famille du sujet. Par exemple : father, daughter, siblings, parents (père, fille, frères et sœurs, parents).

:::image type="content" source="../../media/ta-for-health/family-relation.png" alt-text="Cette capture d’écran montre un autre exemple d’attribut de temps de traitement.":::
