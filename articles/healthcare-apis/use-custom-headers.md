---
title: Ajouter des données aux journaux d’audit en utilisant des en-têtes personnalisés – API Azure pour FHIR
description: Cet article explique comment ajouter des données à des journaux d’audit en utilisant des en-têtes HTTP personnalisés dans l’API Azure pour FHIR.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 10/13/2019
ms.openlocfilehash: 937be72bfec96119474e7effe9ba88a2cf253444
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86081841"
---
# <a name="add-data-to-audit-logs-by-using-custom-http-headers"></a>Ajouter des données aux journaux d’audit en utilisant des en-têtes HTTP personnalisés

Dans l’API Azure pour FHIR (Azure Fast Healthcare Interoperability Resources), un utilisateur peut souhaiter inclure des informations supplémentaires dans les journaux, qui proviennent du système appelant.

Par exemple, quand l’utilisateur de l’API est authentifié par un système externe, ce système transfère l’appel à l’API FHIR. Au niveau de la couche de l’API FHIR, les informations sur l’utilisateur d’origine ont été perdues, car l’appel a été transféré. Il peut s’avérer nécessaire de journaliser et conserver ces informations utilisateur à des fins d’audit ou d’administration. Le système appelant peut fournir l’identité de l’utilisateur, l’emplacement de l’appelant ou d’autres informations nécessaires dans les en-têtes HTTP, qui seront transmises avec l’appel transféré.

Vous pouvez voir ce flux de données dans le diagramme suivant :

:::image type="content" source="media/custom-headers/custom-headers-diagram.png" alt-text="Diagramme d’en-têtes personnalisés&quot;:::

Vous pouvez utiliser des en-têtes personnalisés pour capturer plusieurs types d’informations. Par exemple :

* Informations d’identité ou d’autorisation
* Origine de l’appelant
* Organisation d’origine
* Détails sur le système client (dossier médical électronique, portail des patients)

> [!IMPORTANT]
> Sachez que les informations envoyées dans les en-têtes personnalisés sont stockées dans un système de journalisation interne de Microsoft pendant 30 jours après leur mise à disposition dans Azure Log Monitoring. Nous vous recommandons de chiffrer les informations avant de les ajouter à des en-têtes personnalisés. Il est déconseillé de transmettre des informations PHI dans les en-têtes de clients.

Vous devez utiliser la convention de nommage suivante pour vos en-têtes HTTP : X-MS-AZUREFHIR-AUDIT-\<name>.

Ces en-têtes HTTP sont inclus dans un jeu de propriétés qui est ajouté au journal. Par exemple :

* X-MS-AZUREFHIR-AUDIT-USERID : 1234 
* X-MS-AZUREFHIR-AUDIT-USERLOCATION : XXXX
* X-MS-AZUREFHIR-AUDIT-XYZ : 1234

Ces informations sont ensuite sérialisées en JSON quand elles sont ajoutées à la colonne de propriétés du journal. Par exemple :

```json
{ &quot;X-MS-AZUREFHIR-AUDIT-USERID&quot; : &quot;1234&quot;,
&quot;X-MS-AZUREFHIR-AUDIT-USERLOCATION&quot; : &quot;XXXX&quot;,
&quot;X-MS-AZUREFHIR-AUDIT-XYZ&quot; : &quot;1234&quot; }
```
 
Comme pour tout en-tête HTTP, il est possible de répéter le même nom d’en-tête avec des valeurs différentes. Par exemple :

* X-MS-AZUREFHIR-AUDIT-USERLOCATION : HospitalA
* X-MS-AZUREFHIR-AUDIT-USERLOCATION : Urgence

Une fois ajoutées au journal, les valeurs sont combinées à l’aide d’une liste séparée par des virgules. Par exemple :

{ &quot;X-MS-AZUREFHIR-AUDIT-USERLOCATION&quot; : &quot;HospitalA, Emergency" }
 
Vous pouvez ajouter au maximum 10 en-têtes uniques (les répétitions d’un même en-tête avec des valeurs différentes comptent pour un). Au total, la longueur maximale de la valeur d’un en-tête est de 2 048 caractères.

Si vous utilisez la bibliothèque d’API du client C# Firefly, le code se présente comme suit :

```C#
FhirClient client;
client = new FhirClient(serverUrl);
client.OnBeforeRequest += (object sender, BeforeRequestEventArgs e) =>
{
    // Add custom headers to be added to the logs
    e.RawRequest.Headers.Add("X-MS-AZUREFHIR-AUDIT-UserLocation", "HospitalA");
};
client.Get("Patient");
```
## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez découvert comment ajouter des données à des journaux d’audit en utilisant des en-têtes personnalisés dans l’API Azure pour FHIR. À la prochaine étape, vous allez découvrir les autres paramètres que vous pouvez configurer dans l’API Azure pour FHIR.
 
>[!div class="nextstepaction"]
>[Paramètres supplémentaires](azure-api-for-fhir-additional-settings.md)
