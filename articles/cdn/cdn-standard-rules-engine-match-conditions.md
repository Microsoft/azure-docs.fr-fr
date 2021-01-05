---
title: Conditions de correspondance du moteur de règles Standard pour Azure CDN | Microsoft Docs
description: Documentation de référence sur les conditions de correspondance du moteur de règles Standard pour Azure Content Delivery Network (Azure CDN).
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: article
ms.date: 11/01/2019
ms.author: allensu
ms.openlocfilehash: b8050b973027ac91ede0ba98f4d1c76831da9828
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81259922"
---
# <a name="match-conditions-in-the-standard-rules-engine-for-azure-cdn"></a>Conditions de correspondance du moteur de règles Standard pour Azure CDN

Dans le [moteur de règles Standard](cdn-standard-rules-engine.md) pour Azure Content Delivery Network (Azure CDN), une règle se compose d’une ou de plusieurs conditions de correspondance et d’une action. Cet article fournit les descriptions détaillées des conditions de correspondance que vous pouvez utiliser dans le moteur de règles Standard pour Azure CDN.

La première partie d’une règle est une condition de correspondance ou un ensemble de conditions de correspondance. Dans le moteur de règles Standard pour Azure CDN, chaque règle peut avoir jusqu’à quatre conditions de correspondance. Une condition de correspondance identifie des types spécifiques de requêtes pour lesquelles des actions définies sont exécutées. Si vous utilisez plusieurs conditions de correspondance, elles seront regroupées à l’aide de la logique AND.

Par exemple, vous pouvez utiliser une condition de correspondance pour les tâches suivantes :

- Filtrer les requêtes en fonction d’une adresse IP, d’un pays ou d’une région spécifique.
- Filtrer les requêtes d’après les informations d’en-tête.
- Filtrer les requêtes provenant d’appareils mobiles ou de bureau.

## <a name="match-conditions"></a>Conditions de correspondance

Les conditions de correspondance suivantes peuvent être utilisées dans le moteur de règles Standard pour Azure CDN. 

### <a name="device-type"></a>Type d’appareil 

Identifie les requêtes effectuées à partir d’un appareil mobile ou de bureau.  

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Valeurs prises en charge
---------|----------------
Égal à, Non égal à | Mobile et Bureau

### <a name="http-version"></a>Version HTTP

Identifie les requêtes en fonction de la version HTTP de la requête.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Valeurs prises en charge
---------|----------------
Égal à, Non égal à | 2.0, 1.1, 1.0, 0.9, Tout

### <a name="request-cookies"></a>Cookies de requête

Identifie les requêtes en fonction des informations des cookies figurant dans la requête entrante.

#### <a name="required-fields"></a>Champs obligatoires

Nom du cookie | Opérateur | Valeur du cookie | Transformation de cas
------------|----------|--------------|---------------
String | [Liste des opérateurs standard](#standard-operator-list) | Chaîne, Int | Aucune transformation, en majuscules, en minuscules

#### <a name="key-information"></a>Informations essentielles

- Vous ne pouvez pas utiliser de valeurs de caractère générique (y compris des astérisques [\*]) quand vous spécifiez un nom de cookie. Vous devez utiliser un nom de cookie exact.
- Vous pouvez spécifier un seul nom de cookie par instance de cette condition de correspondance.
- Les comparaisons de noms de cookie respectent la casse.
- Pour spécifier plusieurs valeurs de cookie, utilisez un seul espace entre chaque valeur de cookie. 
- Les valeurs de cookie peuvent utiliser des valeurs de caractère générique.
- Si aucune valeur de caractère générique n’est spécifiée, seule une correspondance exacte satisfait cette condition de correspondance. Par exemple, « Valeur » correspond à « Valeur », mais pas à « Valeur1 ». 

### <a name="post-argument"></a>Argument Post

Identifie les requêtes en fonction des arguments définis pour la méthode de demande POST utilisée dans la requête. 

#### <a name="required-fields"></a>Champs obligatoires

Nom de l’argument | Opérateur | Valeur de l’argument | Transformation de cas
--------------|----------|----------------|---------------
String | [Liste des opérateurs standard](#standard-operator-list) | Chaîne, Int | Aucune transformation, en majuscules, en minuscules

### <a name="query-string"></a>Chaîne de requête

Identifie les requêtes qui contiennent un paramètre de chaîne de requête spécifique. Ce paramètre est défini sur une valeur qui correspond à un modèle spécifique. Les paramètres de chaîne de requête (par exemple, **paramètre = valeur**) dans l’URL de requête déterminent si cette condition est remplie. Cette condition de correspondance identifie un paramètre de chaîne de requête par son nom et accepte une ou plusieurs valeurs pour la valeur du paramètre.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Chaîne de requête | Transformation de cas
---------|--------------|---------------
[Liste des opérateurs standard](#standard-operator-list) | Chaîne, Int | Aucune transformation, en majuscules, en minuscules

### <a name="remote-address"></a>Adresse distante

Identifie les requêtes en fonction de l’emplacement ou de l’adresse IP du demandeur.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Valeurs prises en charge
---------|-----------------
Quelconque | N/A
Géocorrespondance | Code pays
Correspondance IP | Adresse IP (séparée par des espaces)
Aucun | N/A
Sans géocorrespondance | Code pays
Sans correspondance IP | Adresse IP (séparée par des espaces)

#### <a name="key-information"></a>Informations essentielles

- Utilisez la notation CIDR.
- Pour spécifier plusieurs adresses IP et blocs d’adresses IP, utilisez un espace unique entre les valeurs :
  - **Exemple IPv4** : *1.2.3.4 10.20.30.40* correspond aux requêtes qui proviennent de l’adresse 1.2.3.4 ou 10.20.30.40.
  - **Exemple IPv6** : *1:2:3:4:5:6:7:8 10:20:30:40:50:60:70:80* correspond aux requêtes qui proviennent de l’adresse 1:2:3:4:5:6:7:8 ou 10:20:30:40:50:60:70:80.
- La syntaxe d’un bloc d’adresses IP est l’adresse IP de base suivie d’une barre oblique et de la taille de préfixe. Par exemple :
  - **Exemple IPv4** : *5.5.5.64/26* correspond aux requêtes qui proviennent des adresses 5.5.5.64 à 5.5.5.127.
  - **Exemple IPv6** : *1:2:3:/48* correspond aux requêtes qui proviennent des adresses 1:2:3:0:0:0:0:0 à 1:2:3:ffff:ffff:ffff:ffff:ffff.

### <a name="request-body"></a>Corps de la demande

Identifie les requêtes en fonction d’un texte spécifique qui apparaît dans le corps de la requête.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Corps de la demande | Transformation de cas
---------|--------------|---------------
[Liste des opérateurs standard](#standard-operator-list) | Chaîne, Int | Aucune transformation, en majuscules, en minuscules

### <a name="request-header"></a>En-tête de requête

Identifie les requêtes qui utilisent un en-tête spécifique dans la requête.

#### <a name="required-fields"></a>Champs obligatoires

Nom de l’en-tête | Opérateur | Valeur de l’en-tête | Transformation de cas
------------|----------|--------------|---------------
String | [Liste des opérateurs standard](#standard-operator-list) | Chaîne, Int | Aucune transformation, en majuscules, en minuscules

### <a name="request-method"></a>Méthode de requête

Identifie les requêtes qui utilisent la méthode de demande spécifiée.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Valeurs prises en charge
---------|----------------
Égal à, Non égal à | GET, POST, PUT, DELETE, HEAD, OPTIONS, TRACE

#### <a name="key-information"></a>Informations essentielles

- Seule la méthode de demande GET peut générer le contenu mis en cache dans Azure CDN. Toutes les autres méthodes de requête sont traitées par proxy sur le réseau. 

### <a name="request-protocol"></a>Protocole de requête

Identifie les requêtes qui utilisent le protocole spécifié.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Valeurs prises en charge
---------|----------------
Égal à, Non égal à | HTTP, HTTPS

### <a name="request-url"></a>URL de la demande

Identifie les requêtes qui correspondent à l’URL spécifiée.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | URL de la demande | Transformation de cas
---------|-------------|---------------
[Liste des opérateurs standard](#standard-operator-list) | Chaîne, Int | Aucune transformation, en majuscules, en minuscules

#### <a name="key-information"></a>Informations essentielles

- Lorsque vous utilisez cette condition de règle, veillez à inclure les informations de protocole. Par exemple : *https://www.\<yourdomain\>.com*.

### <a name="url-file-extension"></a>Extension de fichier d’URL

Identifie les requêtes qui incluent l’extension de fichier spécifiée dans le nom de fichier dans l’URL de la requête.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Extension | Transformation de cas
---------|-----------|---------------
[Liste des opérateurs standard](#standard-operator-list) | Chaîne, Int | Aucune transformation, en majuscules, en minuscules

#### <a name="key-information"></a>Informations essentielles

- Pour l’extension, n’incluez pas de point de début ; par exemple, utilisez *html* au lieu de *.html*.

### <a name="url-file-name"></a>Nom de fichier d’URL

Identifie les requêtes qui incluent le nom de fichier spécifié dans l’URL de la requête.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Nom de fichier | Transformation de cas
---------|-----------|---------------
[Liste des opérateurs standard](#standard-operator-list) | Chaîne, Int | Aucune transformation, en majuscules, en minuscules

#### <a name="key-information"></a>Informations essentielles

- Si vous spécifiez plusieurs noms de fichier, séparez chaque nom par une espace. 

### <a name="url-path"></a>Chemin d'accès de l'URL

Identifie les requêtes qui incluent le chemin d’accès spécifié dans l’URL de la requête.

#### <a name="required-fields"></a>Champs obligatoires

Opérateur | Valeur | Transformation de cas
---------|-------|---------------
[Liste des opérateurs standard](#standard-operator-list) | Chaîne, Int | Aucune transformation, en majuscules, en minuscules

#### <a name="key-information"></a>Informations essentielles

- Une valeur de nom de fichier peut utiliser des valeurs de caractère générique. Par exemple, chaque modèle de nom de fichier peut contenir un ou plusieurs astérisques (*), où chaque astérisque correspond à une séquence d’un ou de plusieurs caractères.

## <a name="reference-for-rules-engine-match-conditions"></a>Informations de référence des conditions de correspondance du moteur de règles

### <a name="standard-operator-list"></a>Liste des opérateurs standard

Pour les règles qui acceptent les valeurs de la liste des opérateurs standard, les opérateurs suivants sont valides :

- Quelconque
- Égal à 
- Contient 
- Commence par 
- Se termine par 
- Inférieur à
- Inférieur ou égal à
- Supérieur à
- Supérieur ou égal à
- Aucun
- Ne contient pas
- Ne commence pas par 
- Ne se termine pas par 
- N’est pas inférieur à
- N’est pas inférieur ou égal à
- N’est pas supérieur à
- N’est pas supérieur ou égal à

Pour les opérateurs numériques tels que *Inférieur à* et *Supérieur ou égal à*, la comparaison utilisée est basée sur la longueur. Dans ce cas, la valeur de la condition de correspondance doit être un entier égal à la longueur que vous souhaitez comparer. 

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure CDN](cdn-overview.md)
- [Informations de référence sur le moteur de règles Standard](cdn-standard-rules-engine-reference.md)
- [Actions du moteur de règles Standard](cdn-standard-rules-engine-actions.md)
- [Appliquer HTTPS en utilisant le moteur de règles Standard](cdn-standard-rules-engine.md)
