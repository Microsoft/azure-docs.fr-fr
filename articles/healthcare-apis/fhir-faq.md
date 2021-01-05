---
title: Questions fréquentes (FAQ) sur les services FHIR dans Azure - API Azure pour FHIR
description: Obtenez des réponses aux Questions fréquentes (FAQ) sur l’API Azure pour FHIR, par exemple l’emplacement de stockage des données utilisé par les API FHIR et la prise en charge des versions.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 08/03/2020
ms.author: matjazl
ms.openlocfilehash: 31ae5b780bf451e29a97f04202f804db27fc387a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452933"
---
# <a name="frequently-asked-questions-about-the-azure-api-for-fhir"></a>Forum aux questions sur l’API Azure pour FHIR

## <a name="azure-api-for-fhir-the-basics"></a>API Azure pour FHIR : Concepts de base

### <a name="what-is-fhir"></a>Qu’est-ce que FHIR ?
FHIR (Prononcez "fire") ou Fast Healthcare Interoperability Resources est une norme d’interopérabilité destinée à permettre l’échange de données médicales entre différents systèmes de santé. Cette norme a été développée par l’organisation HL7 et est en cours d’adoption par les organisations de la santé du monde entier. La version actuelle de FHIR est la version R4 (Release 4). L’API Azure pour FHIR prend en charge R4 et prend également en charge la version précédente STU3 (Standard for Trial Use 3). Pour plus d’informations sur FHIR, accédez à [HL7.org](http://hl7.org/fhir/summary.html).

### <a name="is-the-data-behind-the-fhir-apis-stored-in-azure"></a>Les données utilisées par les API FHIR sont-elles stockées dans Azure ?

Oui, les données sont stockées dans des bases de données managées dans Azure. L’API Azure pour FHIR ne fournit pas un accès direct à la banque de données sous-jacente.

### <a name="what-identity-provider-do-you-support"></a>Quel fournisseur d’identité prenez-vous en charge ?

Nous prenons actuellement en charge Microsoft Azure Active Directory en tant que fournisseur d’identité.

### <a name="what-is-the-recovery-point-objective-rpo-for-the-azure-api-for-fhir"></a>Qu’est-ce que l’objectif de point de récupération (RPO) pour l’API Azure pour FHIR ?
L’API Azure pour FHIR est adossée à Cosmos DB en tant que fournisseur de persistance. Pour cette raison, le RPO du service est le même que celui de [Cosmos DB (région unique)](../cosmos-db/consistency-levels.md) et est inférieur à 240 minutes.

### <a name="what-fhir-version-do-you-support"></a>Quelle version de FHIR prenez-vous en charge ?

Nous prenons en charge les versions 4.0.0 et 3.0.1 dans le cadre de l’API Azure pour FHIR (PaaS) et de FHIR Server pour Azure (open source).

Pour plus d’informations, consultez [Fonctionnalités prises en charge](fhir-features-supported.md). Découvrez ce qui a changé entre les versions de FHIR (à savoir lors du passage de la version STU3 à la version R4) dans l’[historique des versions de HL7 FHIR](https://hl7.org/fhir/R4/history.html).

Le connecteur Azure IoT pour FHIR (préversion) ne prend actuellement en charge que la version R4 de FHIR, et n’est visible que sur les instances R4 de l’API Azure pour FHIR.

### <a name="whats-the-difference-between-microsoft-fhir-server-for-azure-and-the-azure-api-for-fhir"></a>Quelle est la différence entre le serveur FHIR Microsoft pour Azure et l’API Azure pour FHIR ?

L’API Azure pour FHIR est une version hébergée et managée de la version open source de Microsoft FHIR Server pour Azure. Dans le service managé, Microsoft fournit toutes les opérations de maintenance et mises à jour. 

Lorsque vous exécutez le serveur FHIR pour Azure, vous avez un accès direct aux services sous-jacents, mais vous êtes responsable de la maintenance et de la mise à jour du serveur, ainsi que de tout travail de conformité requis en cas de stockage de données PHI.

Du point de vue du développement, chaque fonctionnalité qui ne s’applique pas uniquement au service géré est d’abord déployée sur le serveur Microsoft FHIR open source pour Azure. Une fois la version open source validée, elle est mise en production sur la solution PaaS API Azure pour FHIR. Le délai de mise en production entre la version open source et la solution PaaS dépend de la complexité des fonctionnalités et d’autres priorités relatives à la feuille de route. Le processus est identique pour tous nos services, tel le connecteur Azure IoT pour FHIR (préversion).

### <a name="where-can-i-see-what-is-releasing-into-the-azure-api-for-fhir"></a>Où puis-je voir les nouveautés de l’API Azure pour FHIR ?

Pour en savoir plus sur les nouveautés de l’API Azure pour FHIR, reportez-vous à la [version release](https://github.com/microsoft/fhir-server/releases) de FHIR Server open source. Depuis le mois de novembre 2020, nous balisons les éléments open source avec Azure-API-for-FHIR en cas de publication de ceux-ci sur le service géré. Ces fonctionnalités sont généralement disponibles deux semaines après leur publication sur la page de mise en production en open source. Nous avons également inclus [ici] (https://github.com/microsoft/fhir-server/blob/master/docs/Testing-Releases.md) ) des instructions sur la façon de tester la build dans votre propre environnement. Nous sommes en train d’étudier le meilleur moyen de partager d’autres mises à jour du service managé.

### <a name="in-which-regions-is-azure-api-for-fhir-available"></a>Dans quelles régions l’API Azure pour FHIR est-elle disponible ?

Elle est actuellement en disponibilité générale pour le grand public et le secteur public dans [plusieurs régions géographiques](https://azure.microsoft.com/global-infrastructure/services/?products=azure-api-for-fhir&regions=non-regional,us-east,us-east-2,us-central,us-north-central,us-south-central,us-west-central,us-west,us-west-2,canada-east,canada-central,usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-texas,usgov-virginia). Pour plus d’informations sur les services cloud de Microsoft pour le Secteur Public, consultez [Services Azure du FedRAMP](../azure-government/compliance/azure-services-in-fedramp-auditscope.md).

### <a name="where-can-i-see-what-is-releasing-into-the-azure-api-for-fhir"></a>Où puis-je voir les nouveautés de l’API Azure pour FHIR ?

Pour en savoir plus sur les nouveautés de l’API Azure pour FHIR, reportez-vous à la [version release](https://github.com/microsoft/fhir-server/releases) de FHIR Server open source. Nous avons ajouté l’étiquette Azure-API-for-FHIR aux éléments inclus dans la version du service managé, qui sont généralement disponibles deux semaines après leur affichage dans la page de mise en production en open source. Nous avons également inclus [ici](https://github.com/microsoft/fhir-server/blob/master/docs/Testing-Releases.md) des instructions sur la façon de tester la build dans votre propre environnement, si vous le souhaitez. Nous sommes en train d’étudier le meilleur moyen de partager d’autres mises à jour du service managé.

### <a name="what-is-smart-on-fhir"></a>Qu’est-ce que SMART sur FHIR ?

SMART (Substitutable Medical Applications and Reusable Technology) sur FHIR est un ensemble de spécifications ouvertes qui permettent d’intégrer les applications partenaires avec les serveurs FHIR et d’autres systèmes informatiques médicaux, par exemple les dossiers médicaux électroniques et les échanges d’informations médicales. En créant une application SMART sur FHIR, vous avez la garantie que votre application est accessible et exploitable par une multitude de systèmes distincts.
Authentification et API Azure pour FHIR. Pour plus d’informations sur SMART, accédez à [SMART Health IT](https://smarthealthit.org/).

### <a name="where-can-i-find-what-version-of-fhir-is-running-on-my-database"></a>Où puis-je trouver la version de FHIR en cours d’exécution sur ma base de données ? 

Vous pouvez trouver la version exacte de FHIR exposée dans l’énoncé de capacité sous la propriété « fhirVersion ».

## <a name="fhir-implementations-and-specifications"></a>Implémentations et spécifications de FHIR

### <a name="can-i-create-a-custom-fhir-resource"></a>Puis-je créer une ressource FHIR personnalisée ?

Nous n’autorisons pas les ressources FHIR personnalisées. Si vous avez besoin d’une ressource FHIR personnalisée, vous pouvez créer une ressource personnalisée en plus de la [Ressource de base](http://www.hl7.org/fhir/basic.html) avec des extensions. 

### <a name="are-extensions-supported-on-azure-api-for-fhir"></a>Les [extensions](https://www.hl7.org/fhir/extensibility.html) sont-elles prises en charge sur l’API Azure pour FHIR ?

Nous vous autorisons à charger toutes données JSON FHIR valides dans le serveur. Si vous souhaitez stocker la définition de structure qui définit l’extension, vous pouvez l’enregistrer en tant que ressource de définition de structure. Actuellement, vous ne pouvez pas effectuer de recherche sur les extensions.

### <a name="what-is-the-limit-on-_count"></a>Quelle est la limite de _count ?

La limite actuelle de _count est 100. Si vous définissez _count sur une valeur supérieure à 100, un avertissement apparaît dans l’offre groupée, indiquant que seuls 100 enregistrements seront présentés.

### <a name="are-there-any-limitations-on-the-group-export-functionality"></a>Existe-t-il des limitations sur la fonctionnalité d’exportation de groupe ?

Pour l’exportation de groupe, vous exportez uniquement les références incluses à partir du groupe, et non toutes les caractéristiques de la [ressource de groupe](https://www.hl7.org/fhir/group.html).

### <a name="can-i-post-a-bundle-to-the-azure-api-for-fhir"></a>Puis-je publier un bundle sur l’API Azure pour FHIR ?

Nous autorisons la publication de [bundles par lot](https://www.hl7.org/fhir/valueset-bundle-type.html), mais pas de bundles transactionnels dans l’API Azure pour FHIR. Vous pouvez utiliser l’implémentation open source de FHIR Server avec SQL pour publier des bundles transactionnels.

### <a name="how-can-i-get-all-resources-for-a-single-patient-in-the-azure-api-for-fhir"></a>Comment puis-je obtenir toutes les ressources associées à un patient spécifique dans l’API Azure pour FHIR ?

Nous prenons en charge la [recherche par compartiment](https://www.hl7.org/fhir/compartmentdefinition.html) dans l’API Azure pour FHIR. Cela vous permet d’obtenir toutes les ressources associées à un patient spécifique. Notez que le compartiment contient maintenant toutes les ressources associées au patient, mais pas le patient lui-même. Vous devrez également effectuer une recherche pour obtenir le patient si vous avez besoin d’inclure la ressource patient dans vos résultats.

Voici quelques exemples :

* GET Patient/<id>/*
* GET Patient/<id>/Observation
* GET Patient/<id>/Observation?code=8302-2

### <a name="what-is-the-default-sort-when-searching-for-resources-in-azure-api-for-fhir"></a>Quel est le tri par défaut lors de la recherche de ressources dans l’API Azure pour FHIR ?

Nous prenons en charge le tri sur la date de dernière mise à jour : _sort =_lastUpdated. Pour plus d’informations sur les autres paramètres de recherche pris en charge, consultez la [page des fonctionnalités prises en charge](./fhir-features-supported.md#search).

### <a name="how-does-export-work"></a>Comment la commande $export fonctionne-t-elle ?

La commande $export fait partie de la spécification FHIR : https://hl7.org/fhir/uv/bulkdata/export/index.html. Si le service FHIR est configuré avec une identité gérée et un compte de stockage, et si l’identité gérée a accès à ce compte de stockage, vous pouvez simplement appeler la commande $export sur l’API FHIR pour exporter toutes les ressources FHIR vers le compte de stockage. Pour plus d’informations, consultez notre [article sur la commande $export](./export-data.md).

## <a name="using-azure-api-for-fhir"></a>Utilisation de l’API Azure pour FHIR

### <a name="how-do-i-enable-log-analytics-for-azure-api-for-fhir"></a>Comment faire pour activer l’analytique des journaux d’activité pour l’API Azure pour FHIR ?

Nous activons la journalisation des diagnostics et autorisons la consultation des exemples de requêtes pour ces journaux. Pour plus d’informations sur l’activation des journaux d’audit et des exemples de requêtes, consultez [cette section](./enable-diagnostic-logging.md). Si vous souhaitez inclure des informations supplémentaires dans les journaux, découvrez comment [utiliser des en-têtes HTTP personnalisés](./use-custom-headers.md).

### <a name="where-can-i-see-some-examples-of-using-the-azure-api-for-fhir-within-a-workflow"></a>Où puis-je voir des exemples d’utilisation de l’API Azure pour FHIR dans un workflow ?

Nous avons une collection d’architectures de référence disponibles dans la [page Health Architecture sur GitHub](https://github.com/microsoft/health-architectures).

### <a name="where-can-i-see-an-example-of-connecting-a-web-application-to-azure-api-for-fhir"></a>Où puis-je voir un exemple de connexion d’une application web à l’API Azure pour FHIR ?

Nous avons une [page GitHub Health Architecture](https://aka.ms/health-architectures) contenant des exemples d’applications et de scénarios. Elle montre comment connecter une application web à l’API Azure pour FHIR.  

## <a name="azure-api-for-fhir-features-and-services"></a>Fonctionnalités et services de l’API Azure pour FHIR 

### <a name="is-there-a-way-to-encrypt-my-data-using-my-personal-key-not-a-default-key"></a>Existe-t-il un moyen de chiffrer mes données à l’aide de ma clé personnelle au lieu d’une clé par défaut ?

Oui, l’API Azure pour FHIR permet de configurer des clés gérées par le client, en tirant parti de la prise en charge de Cosmos DB. Pour plus d’informations sur le chiffrement de vos données avec une clé personnelle, consultez [cette section](./customer-managed-key.md).

## <a name="azure-api-for-fhir-preview-features"></a>API Azure pour FHIR : Fonctionnalités de préversion

### <a name="can-i-configure-scaling-capacity-for-azure-iot-connector-for-fhir-preview"></a>Puis-je configurer la capacité de mise à l’échelle pour un Connecteur Azure IoT pour FHIR (préversion) ?

Étant donné que le Connecteur Azure IoT pour FHIR est gratuit pendant la période de préversion publique, sa capacité de mise à l’échelle est fixe et limitée. La configuration de Connecteur Azure IoT pour FHIR disponible en préversion publique est supposée fournir un débit d’environ 200 messages par seconde. Une certaine forme de configuration de la capacité des ressources sera disponible en disponibilité générale.

### <a name="why-cant-i-install-azure-iot-connector-for-fhir-preview-when-private-link-is-enabled-on-azure-api-for-fhir"></a>Pourquoi ne puis-je pas installer un Connecteur Azure IoT pour FHIR quand Azure Private Link est activé sur l’API Azure pour FHIR ?

Un Connecteur Azure IoT pour FHIR ne prend pas en charge la capacité d’Azure Private Link pour l’instant. Par conséquent, si vous avez activé Azure Private Link sur l’API Azure pour FHIR, vous ne pouvez pas installer de Connecteur Azure IoT pour FHIR, et vice-versa. Cette limitation devrait disparaître lorsque Connecteur Azure IoT pour FHIR sera en disponibilité générale.
