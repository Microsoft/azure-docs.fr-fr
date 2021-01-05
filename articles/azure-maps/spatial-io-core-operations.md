---
title: Opérations d’E/S principales | Microsoft Azure Maps
description: Découvrez comment lire et écrire efficacement des données XML et délimitées à l’aide de bibliothèques principales à partir du module d’E/S spatiales.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/03/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-js
ms.openlocfilehash: 08966549d2c4d2bf701ee0dbfc5ed24f202396fc
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891374"
---
# <a name="core-io-operations"></a>Opérations d’E/S principales

En plus de fournir des outils pour lire des fichiers de données spatiales, le module d’E/S spatiales expose des bibliothèques sous-jacentes principales pour lire et écrire des données XML et délimitées rapidement et efficacement.

L’espace de noms `atlas.io.core` contient deux classes de bas niveau qui peuvent rapidement lire et écrire des données CSV et XML. Ces classes de base alimentent les lecteurs et les enregistreurs de données spatiales dans le module d’E/S spatiales. N’hésitez pas à les utiliser pour ajouter une prise en charge supplémentaire de la lecture et de l’écriture des fichiers CSV ou XML.
 
## <a name="read-delimited-files"></a>Lire des fichiers délimités

La classe `atlas.io.core.CsvReader` lit les chaînes qui contiennent des jeux de données délimités. Cette classe fournit deux méthodes pour lire les données :

- La fonction `read` lit le jeu de données complet et retourne un tableau de chaînes à deux dimensions représentant toutes les cellules du jeu de données délimité.
- La fonction `getNextRow` lit chaque ligne de texte dans un jeu de données délimité et retourne un tableau de chaînes représentant toutes les cellules de cette ligne du jeu de données. L’utilisateur peut traiter la ligne et en supprimer toute mémoire inutile avant de traiter la ligne suivante. Ainsi, la fonction utilise la mémoire de façon plus efficace.

Par défaut, le lecteur utilise la virgule comme délimiteur. Toutefois, le délimiteur peut être remplacé par un caractère unique ou défini sur `'auto'`. Quand la valeur est `'auto'`, le lecteur analyse la première ligne de texte de la chaîne. Ensuite, il sélectionne le caractère le plus courant dans le tableau ci-dessous pour l’utiliser comme délimiteur.

| Délimiteur | Caractère |
| :-- | :-- |
| Comma | `,` |
| Onglet | `\t` |
| Pipe | `|` |

Ce lecteur prend également en charge les qualificateurs de texte qui sont utilisés pour gérer les cellules qui contiennent le caractère délimiteur. Le caractère guillemet (`'"'`) est le qualificateur de texte par défaut, mais il peut être remplacé par n’importe quel caractère unique.

## <a name="write-delimited-files"></a>Écrire des fichiers délimités

La fonction `atlas.io.core.CsvWriter` écrit un tableau d’objets sous la forme d’une chaîne délimitée. Tout caractère unique peut être utilisé comme délimiteur ou qualificateur de texte. Le délimiteur par défaut est la virgule (`','`) et le qualificateur de texte par défaut est le caractère guillemet (`'"'`).

Pour utiliser cette classe, procédez comme suit :

- Créez une instance de la classe et définissez éventuellement un qualificateur de texte ou un délimiteur personnalisé.
- Écrivez des données dans la classe à l’aide de la fonction `write` ou de la fonction `writeRow`. Pour la fonction `write`, transmettez un tableau d’objets à deux dimensions représentant plusieurs lignes et cellules. Pour utiliser la fonction `writeRow`, transmettez un tableau d’objets représentant une ligne de données avec plusieurs colonnes.
- Appelez la fonction `toString` pour récupérer la chaîne délimitée. 
- Éventuellement, appelez la méthode `clear` pour rendre l’enregistreur réutilisable et réduire son allocation de ressources ou appelez la méthode `delete` pour supprimer l’instance d'enregistreur.

> [!Note]
> Le nombre de colonnes écrites sera restreint au nombre de cellules de la première ligne des données transmises à l’enregistreur.

## <a name="read-xml-files"></a>Lire des fichiers XML

La classe `atlas.io.core.SimpleXmlReader` est plus rapide lors de l’analyse des fichiers XML que `DOMParser`. Toutefois, la classe `atlas.io.core.SimpleXmlReader` requiert des fichiers XML correctement formatés. Les fichiers XML qui ne sont pas formatés correctement, par exemple des balises de fermeture manquantes, entraîneront probablement une erreur.

Le code suivant montre comment utiliser la classe `SimpleXmlReader` pour analyser une chaîne XML dans un objet JSON et la sérialiser dans le format souhaité.

```javascript
//Create an instance of the SimpleXmlReader and parse an XML string into a JSON object.
var xmlDoc = new atlas.io.core.SimpleXmlReader().parse(xmlStringToParse);

//Verify that the root XML tag name of the document is the file type your code is designed to parse.
if (xmlDoc && xmlDoc.root && xmlDoc.root.tagName && xmlDoc.root.tagName === '<Your desired root XML tag name>') {

    var node = xmlDoc.root;

    //Loop through the child node tree to navigate through the parsed XML object.
    for (var i = 0, len = node.childNodes.length; i < len; i++) {
        childNode = node.childNodes[i];

        switch (childNode.tagName) {
            //Look for tag names, parse and serialized as desired.
        }
    }
}
```

## <a name="write-xml-files"></a>Écrire des fichiers XML

La classe `atlas.io.core.SimpleXmlWriter` écrit des données XML correctement formatées de manière efficace pour la mémoire.

Le code suivant montre comment utiliser la classe `SimpleXmlWriter` pour générer une chaîne XML correctement formatée.

```javascript
//Create an instance of the SimpleXmlWriter class.
var writer = new atlas.io.core.SimpleXmlWriter();

//Start writing the document. All write functions return a reference to the writer, making it easy to chain the function calls to reduce the code size.
writer.writeStartDocument(true)
    //Specify the root XML tag name, in this case 'root'
    .writeStartElement('root', {
        //Attributes to add to the root XML tag.
        'version': '1.0',
        'xmlns': 'http://www.example.com',
         //Example of a namespace.
        'xmlns:abc': 'http://www.example.com/abc'
    });

//Start writing an element that has the namespace abc and add other XML elements as children.
writer.writeStartElement('abc:parent');

//Write a simple XML element like <title>Azure Maps is awesome!</title>
writer.writeElement('title', 'Azure Maps is awesome!');

//Close the element that we have been writing children to.
writer.writeEndElement();

//Finish writing the document by closing the root tag and the document.
writer.writeEndElement().writeEndDocument();

//Get the generated XML string from the writer.
var xmlString = writer.toString();
```

Le fichier XML généré à partir du code ci-dessus ressemble à ce qui suit.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root version="1.0" xmlns="http://www.example.com" xmlns:abc="http://www.example.com/abc">
    <abc:parent>
        <title>Azure Maps is awesome!</title>
    </abc:parent>
</root>
```

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

[CsvReader](/javascript/api/azure-maps-spatial-io/atlas.io.core.csvreader)

[CsvWriter](/javascript/api/azure-maps-spatial-io/atlas.io.core.csvwriter)

[SimpleXmlReader](/javascript/api/azure-maps-spatial-io/atlas.io.core.simplexmlreader)

[SimpleXmlWriter](/javascript/api/azure-maps-spatial-io/atlas.io.core.simplexmlwriter)

Pour obtenir plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

[Détails sur les formats de données pris en charge](spatial-io-supported-data-format-details.md)