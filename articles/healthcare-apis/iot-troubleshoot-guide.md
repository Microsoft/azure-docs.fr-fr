---
title: Connecteur Azure IoT pour FHIR (préversion) – Guide de dépannage et procédures
description: Comment résoudre des problèmes de messages d’erreur courants du Connecteur Azure IoT pour FHIR (préversion) et copier des fichiers de mappage
services: healthcare-apis
author: msjasteppe
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: troubleshooting
ms.date: 11/13/2020
ms.author: jasteppe
ms.openlocfilehash: 403b6656a47f56508682dcda2438a85d513fbfb1
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94630496"
---
# <a name="azure-iot-connector-for-fhir-preview-troubleshooting-guide"></a>Guide de dépannage du Connecteur Azure IoT pour FHIR (préversion)

Cet article décrit la procédure de résolution de messages et conditions d’erreur courants du Connecteur Azure IoT pour FHIR&#174; (Fast Healthcare Interoperability Resources)*.  

Vous apprendrez également à créer des copies du connecteur Azure IoT pour le fichier JSON de mappage de conversion FHIR (par exemple : Appareil et FHIR).  

Vous pouvez utiliser les copies JSON de mappage de conversion pour la modification et l’archivage en dehors du portail Azure.  

> [!TIP]
> Si vous comptez ouvrir un ticket de [support technique Azure](https://azure.microsoft.com/support/create-ticket/) pour le Connecteur Azure IoT pour FHIR, veillez à inclure des copies de votre fichiers JSON de mappage de conversion pour faciliter le processus de résolution des problèmes.

## <a name="device-and-fhir-conversion-mapping-json-template-validations-for-azure-iot-connector-for-fhir-preview"></a>Validations de modèle JSON de mappage de conversion de FHIR et d’appareil pour le Connecteur Azure IoT pour FHIR (préversion)
Cette section décrit le processus de validation que le Connecteur Azure IoT pour FHIR effectue pour valider les modèles JSON de mappage de conversion de FHIR et d’appareil avant d’autoriser leur enregistrement en vue d’une utilisation.  Ces éléments sont requis dans le JSON de mappage de conversion de FHIR et d’appareil.

**Mappage d’appareil**

|Élément|Obligatoire|
|:-------|:------|
|TypeName|True|
|TypeMatchExpression|True|
|DeviceIdExpression|True|
|TimestampExpression|True|
|Values[].ValueName|True|
|Values[].ValueExpression|True|

> [!NOTE]
> Values[].ValueName and Values[].ValueExpression
>
> Ces éléments sont requis uniquement si vous avez une entrée de valeur dans le tableau. L’absence de valeur mappée n’affecte en rien la validité. Cela est utilisé lorsque la télémétrie envoyée est un événement. Par exemple : Quand un appareil IoMT portable est mis en place ou supprimé. Le ou les éléments n’ont pas de valeur à l’exception d’un nom que le Connecteur Azure IoT pour FHIR met en correspondance et émet. Lors de la conversion de FHIR, le Connecteur Azure IoT pour FHIR le mappe à un concept codable basé sur le type sémantique. Aucune valeur réelle n’est renseignée.

**Mappage de FHIR**

|Élément|Obligatoire|
|:------|:-------|
|TypeName|True|

> [!NOTE]
> Il s’agit du seul élément de mappage de FHIR requis validé pour l’instant.

## <a name="error-messages-and-fixes-for-azure-iot-connector-for-fhir-preview"></a>Messages d'erreur et correctifs pour le Connecteur Azure IoT pour FHIR (préversion)

|Message|Affiché|Condition|Fix| 
|-------|---------|---------|---|
|Nom de mappage non valide, le nom de mappage doit être nu appareil ou FHIR.|API|Le type de mappage fourni n’est pas un appareil ou FHIR.|Utilisez l’un des deux types de mappages pris en charge (par exemple : appareil ou FHIR).|
|Échec de la validation. Les informations nécessaires sont manquantes ou non valides.|API et portail Azure|La tentative d’enregistrement d’un mappage de conversion ne dispose pas des informations ou éléments nécessaires.|Ajoutez les informations ou éléments de mappage de conversion manquantes, puis essayez d’enregistrer à nouveau le mappage de conversion.|
|Régénérez des paramètres de clé non définis.|API|Régénérez une demande de clé.|Incluez les paramètres dans la demande de régénération de clé.|
|Atteinte du nombre maximal d’instances de Connecteur IoT qui peuvent être approvisionnées dans cet abonnement.|API et portail Azure|Le quota d’abonnements du Connecteur Azure IoT pour FHIR est atteint (la valeur par défaut est (2) par abonnement).|Supprimez l’une des instances existantes du connecteur Azure IoT pour FHIR.  Utilisez un autre abonnement qui n’a pas atteint le quota d’abonnement.  Demandez une augmentation du quota d’abonnement.|
|Le déplacement de ressource n’est pas pris en charge pour l’API Azure compatible avec le Connecteur IoT pour la ressource FHIR.|API et portail Azure|Tentative d’effectuer une opération de déplacement sur une ressource d’API Azure pour FHIR qui a une ou plusieurs instances du Connecteur Azure IoT pour FHIR.|Supprimez des instances existantes du Connecteur Azure IoT pour FHIR pour effectuer l’opération de déplacement.|
|Connecteur IoT non approvisionné.|API|Tentative d’utilisation des services enfants (connexions & mappages) lorsque le parent (Connecteur Azure IoT pour FHIR) n’a pas été approvisionné.|Approvisionnez un Connecteur Azure IoT pour FHIR.|
|La demande n'est pas prise en charge.|API|Une requête d’API spécifique n’est pas prise en charge.|Utilisez la requête d’API correcte.|
|Le compte n’existe pas.|API|La tentative d’ajout d’un connecteur Azure IoT pour FHIR et de l’API Azure pour la ressource FHIR n’existe pas.|Créez l’API Azure pour la ressource FHIR, puis retentez l’opération.|
|La version de l’API Azure pour la ressource FHIR n’est pas prise en charge pour le Connecteur IoT.|API|Tentative d’utilisation d’un connecteur Azure IoT pour FHIR avec une version incompatible de la ressource API Azure pour FHIR.|Créez une nouvelle ressource API Azure pour FHIR (version R4) ou utilisez une ressource API Azure pour FHIR (version R4).

## <a name="why-is-my-azure-iot-connector-for-fhir-preview-data-not-showing-up-in-azure-api-for-fhir"></a>Pourquoi mes données dans le connecteur Azure IoT pour FHIR (préversion) n’apparaissent pas dans l’API Azure pour FHIR ?

|Problèmes potentiels|Correctifs|
|----------------|-----|
|Les données sont toujours en cours de traitement.|Les données sont transmises en lots à l’API Azure pour FHIR (toutes les 15 minutes environ).  Il est possible que les données soient toujours en cours de traitement et qu’un délai supplémentaire soit nécessaire pour que les données soient conservées dans l’API Azure pour FHIR.|
|Le fichier JSON de mappage de conversion d’appareil n’a pas été configuré.|Configurez et enregistrez le fichier JSON de mappage de conversion d’appareil conforme.|
|Le fichier JSON de mappage de conversion FHIR n’a pas été configuré.|Configurez et enregistrez le fichier JSON de mappage de conversion FHIR conforme.|
|Le message de l’appareil ne contient pas d’expression attendue définie dans le mappage de l’appareil.|Vérifiez les expressions JsonPath définies dans les jetons de correspondance de mappage d’appareil dans le message de l’appareil.|
|Aucune ressource d’appareil n’a été créée dans l’API Azure pour FHIR (type de résolution : recherche uniquement)*.|Créez une ressource d’appareil valide dans l’API Azure pour FHIR. Vérifiez que la ressource de l’appareil contient un identificateur qui correspond à l’identificateur de l’appareil fourni dans le message entrant.|
|Une ressource patient n’a pas été créée dans l’API Azure pour FHIR (type de résolution : recherche uniquement)*.|Créez une ressource patient valide dans l’API Azure pour FHIR.|
|La référence Device.patient n’est pas définie ou n’est pas valide (type de résolution : recherche uniquement)*.|Assurez-vous que la ressource d’appareil contient une référence [valide](https://www.hl7.org/fhir/device-definitions.html#Device.patient) à une ressource patient.| 

*Référence [Démarrage rapide : Déployer le connecteur Azure IoT (préversion) à l’aide du portail Azure](iot-fhir-portal-quickstart.md#create-new-azure-iot-connector-for-fhir-preview) pour obtenir une description fonctionnelle du connecteur Azure IoT pour les types de résolution FHIR (par exemple : Rechercher ou Créer).

## <a name="use-metrics-to-troubleshoot-issues-in-azure-iot-connector-for-fhir-preview"></a>Utiliser des métriques pour résoudre les problèmes dans le connecteur Azure IoT pour FHIR (préversion)

Le connecteur Azure IoT pour FHIR génère plusieurs métriques pour fournir des informations sur le processus de flux de données. Parmi des métriques prises en charge, *Nb total d’erreurs*, indique le nombre d’erreurs qui se produisent dans une instance du connecteur Azure IoT pour FHIR.

Chaque erreur est enregistrée avec un certain nombre de propriétés associées. Chaque propriété fournit un aspect différent de l’erreur, qui peut vous aider à identifier et à résoudre les problèmes. Cette section répertorie les différentes propriétés capturées pour chaque erreur dans la métrique *Nb total d’erreurs* et les valeurs possibles de ces propriétés.

> [!NOTE]
> Vous pouvez accéder à la métrique *Nb total d’erreurs* pour une instance du connecteur Azure IoT pour FHIR (préversion), comme décrit à la [page Métriques du connecteur Azure IoT pour FHIR (préversion)](iot-metrics-display.md).

Cliquez sur le graphique *Nb total d’erreurs*, puis cliquez sur le bouton *Ajouter un filtre* pour découper et couper la métrique d’erreur à l’aide de l’une des propriétés mentionnées ci-dessous.

### <a name="the-operation-performed-by-the-azure-iot-connector-for-fhir-preview"></a>Opération effectuée par le connecteur Azure IoT pour FHIR (préversion)

Cette propriété représente l’opération exécutée par le connecteur IoT lorsque l’erreur s’est produite. Une opération représente généralement l’étape de flux de données lors du traitement d’un message d’appareil. Voici la liste des valeurs possibles pour cette propriété.

> [!NOTE]
> Vous pouvez en savoir plus sur les différentes étapes du workflow dans le connecteur Azure IoT pour FHIR (préversion) [ici](iot-data-flow.md).

|Étape de flux de données|Description|
|---------------|-----------|
|Programme d’installation|Opération spécifique à la configuration de votre instance du connecteur IoT|
|Normalisation|Étape de flux de données où les données de l’appareil sont normalisées|
|Regroupement|Étape de flux de données dans laquelle les données normalisées sont regroupées|
|FHIRConversion|Étape de flux de données où les données normalisées regroupées sont transformées en ressource FHIR|
|Inconnu|Le type d’opération est inconnu quand une erreur s’est produite|

### <a name="the-severity-of-the-error"></a>Gravité de l’erreur

Cette propriété représente la gravité de l’erreur qui s’est produite. Voici la liste des valeurs possibles pour cette propriété.

|severity|Description|
|---------------|-----------|
|Avertissement|Un problème mineur existe dans le processus de flux de données, mais le traitement du message de l’appareil ne s’arrête pas|
|Error|Le traitement d’un message d’appareil spécifique a généré une erreur et d’autres messages peuvent continuer de s’exécuter comme prévu|
|Critique|Il existe un problème au niveau du système avec le connecteur IoT et aucun message ne doit être traité|

### <a name="the-type-of-the-error"></a>Type de l’erreur

Cette propriété signifie une catégorie pour une erreur donnée, qui représente fondamentalement un regroupement logique pour un type d’erreur similaire. Voici la liste des valeurs possibles pour cette propriété.

|Type d’erreur|Description|
|----------|-----------|
|DeviceTemplateError|Erreurs liées aux modèles de mappage d’appareils|
|DeviceMessageError|Des erreurs se sont produites lors du traitement d’un message d’appareil spécifique|
|FHIRTemplateError|Erreurs liées aux modèles de mappage FHIR|
|FHIRConversionError|Des erreurs se sont produites lors de la transformation d’un message en ressource FHIR|
|FHIRResourceError|Erreurs liées aux ressources existantes dans le serveur FHIR qui sont référencées par le connecteur IoT|
|FHIRServerError|Erreurs qui se produisent lors de la communication avec le serveur FHIR|
|GeneralError|Tous les autres types d’erreurs|

### <a name="the-name-of-the-error"></a>Nom de l’erreur

Cette propriété fournit le nom d’une erreur spécifique. Voici la liste de tous les noms d’erreur avec leur description et le ou les types d’erreurs associés, la gravité ainsi que la ou les étapes de flux de données.

|Nom de l’erreur|Description|Type(s) d’erreur|Gravité de l’erreur|Étape(s) de flux de données|
|----------|-----------|-------------|--------------|------------------|
|MultipleResourceFoundException|Une erreur s’est produite lors de la détection de plusieurs ressources de patient ou d’appareil dans le serveur FHIR pour les identificateurs respectifs présents dans le message de l’appareil|FHIRResourceError|Error|FHIRConversion|
|TemplateNotFoundException|Un modèle de mappage d’appareil ou de FHIR n’est pas configuré avec l’instance du connecteur IoT|DeviceTemplateError, FHIRTemplateError|Critique|Normalization, FHIRConversion|
|CorrelationIdNotDefinedException|L’ID de corrélation n’est pas spécifié dans le modèle de mappage d’appareil. CorrelationIdNotDefinedException est une erreur conditionnelle qui se produit uniquement lorsque l’observation FHIR doit regrouper les mesures de l’appareil à l’aide d’un ID de corrélation, mais qu’elle n’est pas configurée correctement|DeviceMessageError|Error|Normalisation|
|PatientDeviceMismatchException|Cette erreur se produit lorsque la ressource de l’appareil sur le serveur FHIR possède une référence à une ressource patient qui ne correspond pas à l’identificateur patient présent dans le message.|FHIRResourceError|Error|FHIRConversionError|
|PatientNotFoundException|Aucune ressource FHIR de patient n’est référencée par la ressource FHIR d’appareil associée à l’identificateur d’appareil présent dans le message de l’appareil. Notez que cette erreur se produit uniquement lorsque l’instance de connecteur IoT est configurée avec un type de résolution *Recherche*|FHIRConversionError|Error|FHIRConversion|
|DeviceNotFoundException|Aucune ressource d’appareil n’existe sur le serveur FHIR associé à l’identificateur de périphérique présent dans le message de l’appareil|DeviceMessageError|Error|Normalisation|
|PatientIdentityNotDefinedException|Cette erreur se produit lorsque l’expression d’analyse de l’identificateur du patient à partir du message de l’appareil n’est pas configurée sur le modèle de mappage d’appareil ou que l’identificateur du patient ne figure pas dans le message de l’appareil. Cette erreur se produit uniquement lorsque le type de résolution du connecteur IoT est défini sur *Créer*|DeviceTemplateError|Critique|Normalisation|
|DeviceIdentityNotDefinedException|Cette erreur se produit lorsque l’expression d’analyse de l’identificateur de l’appareil à partir du message de l’appareil n’est pas configurée sur le modèle de mappage d’appareil ou que l’identificateur de l’appareil ne figure pas dans le message de l’appareil.|DeviceTemplateError|Critique|Normalisation|
|NotSupportedException|Une erreur s’est produite lors de la réception d’un message d’appareil avec un format non pris en charge|DeviceMessageError|Error|Normalisation|

## <a name="creating-copies-of-the-azure-iot-connector-for-fhir-preview-conversion-mapping-json"></a>Création de copies du connecteur Azure IoT pour le fichier JSON de mappage de conversion FHIR (préversion)

La copie du connecteur Azure IoT pour les fichiers de mappage FHIR peut être utile pour la modification et l’archivage en dehors du site web du portail Azure.

Les copies du fichier de mappage doivent être fournies au support technique Azure lors de l’ouverture d’un ticket de support afin de faciliter le dépannage.

> [!NOTE]
> JSON est le seul format pris en charge pour les fichiers de mappage d’appareil et FHIR à ce stade.

> [!TIP]
> En savoir plus sur le connecteur Azure IoT pour le [fichier JSON de mappage de conversion d’appareil et FHIR](./iot-mapping-templates.md)

1. Sélectionnez **Connecteur IoT (préversion)** en bas à gauche du tableau de bord des ressources de l’API Azure pour FHIR dans la section **Compléments**.

   :::image type="content" source="media/iot-troubleshoot/map-files-main-with-box.png" alt-text="Connecteur1 IoT" lightbox="media/iot-troubleshoot/map-files-main-with-box.png":::

2. Sélectionnez le **« connecteur »** à partir duquel vous allez copier le fichier JSON de mappage de conversion.

   :::image type="content" source="media/iot-troubleshoot/map-files-select-connector-with-box.png" alt-text="Connecteur2 IoT" lightbox="media/iot-troubleshoot/map-files-select-connector-with-box.png":::

> [!NOTE]
> Ce processus peut également être utilisé pour copier et enregistrer le contenu du JSON **Configurer le mappage FHIR**.

3. Sélectionnez **Configurer le mappage d’appareil**.

    :::image type="content" source="media/iot-troubleshoot/map-files-select-device-with-box.png" alt-text="Connecteur3 IoT" lightbox="media/iot-troubleshoot/map-files-select-device-with-box.png":::

4. Sélectionnez le contenu du JSON et effectuez une opération de copie (par exemple, sélectionnez Ctrl + c). 

   :::image type="content" source="media/iot-troubleshoot/map-files-select-device-json-with-box.png" alt-text="Connecteur4 IoT" lightbox="media/iot-troubleshoot/map-files-select-device-json-with-box.png":::

5. Effectuez une opération de collage (par exemple, sélectionnez Ctrl + v) dans un nouveau fichier à l’intérieur d’un éditeur (par exemple, Visual Studio Code, Bloc-notes), et enregistrez le fichier avec une extension *.json.

> [!TIP]
> Si vous comptez ouvrir un ticket de [support technique Azure](https://azure.microsoft.com/support/create-ticket/) pour le Connecteur Azure IoT pour FHIR, veillez à inclure des copies de votre fichiers JSON de mappage de conversion pour faciliter le processus de résolution des problèmes.

## <a name="next-steps"></a>Étapes suivantes

Consultez les questions fréquemment posées sur le Connecteur Azure IoT pour FHIR.

>[!div class="nextstepaction"]
>[FAQ sur le Connecteur Azure IoT pour FHIR](fhir-faq.md)

*Sur le Portail Azure, le Connecteur Azure IoT pour FHIR est appelé Connecteur IoT (préversion). Le sigle FHIR est une marque déposée de HL7 et est utilisé avec l’autorisation de HL7.