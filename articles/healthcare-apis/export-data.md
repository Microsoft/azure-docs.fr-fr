---
title: Exécuter l’exportation en appelant la commande $export sur API Azure pour FHIR
description: Cet article explique comment exporter des données FHIR en utilisant $export.
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 8/26/2020
ms.author: matjazl
ms.openlocfilehash: 74fe09895f49cc9f7c3cdf6b6c97c1624c3e9c0b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91839824"
---
# <a name="how-to-export-fhir-data"></a>Comment exporter des données FHIR


La fonctionnalité d’exportation en bloc autorise l’exportation de données à partir du serveur FHIR en fonction de la [spécification FHIR](https://hl7.org/fhir/uv/bulkdata/export/index.html). 

Avant d’utiliser $export, vous devez vous assurer que l’API Azure pour FHIR est configurée. Pour configurer les paramètres d’exportation et créer un compte de stockage Azure, consultez la [page Configurer les données d’exportation](configure-export-data.md).

## <a name="using-export-command"></a>Utilisation de la commande $export

Après avoir configuré l’API Azure pour FHIR pour l’exportation, vous pouvez utiliser la commande $export afin d’exporter les données à partir du service. Les données seront stockées dans le compte de stockage que vous avez spécifié lors de la configuration de l’exportation. Pour savoir comment appeler la commande $export dans le serveur FHIR, consultez la documentation sur la [spécification $export](https://hl7.org/Fhir/uv/bulkdata/export/index.html). 

La commande $export dans l’API Azure pour FHIR accepte un paramètre facultatif _\_container_ qui spécifie le conteneur à l’intérieur du compte de stockage configuré où les données doivent être exportées. Si un conteneur est spécifié, les données sont exportées vers celui-ci dans un nouveau dossier portant son nom. Si le conteneur n’est pas spécifié, elles sont exportées vers un nouveau conteneur dont le nom est généré de manière aléatoire. 

`https://<<FHIR service base URL>>/$export?_container=<<container_name>>`

## <a name="supported-scenarios"></a>Scénarios pris en charge

L’API Azure pour FHIR prend en charge $export au niveau du système, des patients et des groupes. Pour l’exportation de groupe, nous exportons toutes les ressources associées, mais n’exportons pas les caractéristiques du groupe.

> [!Note] 
> $export exporte les ressources dupliquées si la ressource se trouve dans un compartiment de plusieurs ressources ou dans plusieurs groupes.

En outre, la vérification de l’état d’exportation à l’aide de l’URL renvoyée par l’en-tête Location pendant la mise en file d’attente est prise en charge avec l’annulation du travail d’exportation réel.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à exporter des ressources FHIR à l’aide de la commande $export. Ensuite, vous pouvez passer en revue nos fonctionnalités prises en charge :
 
>[!div class="nextstepaction"]
>[Fonctionnalités prises en charge](fhir-features-supported.md)
