# Nutzung externer Identifikatoren

## Was sind externe Identifikatoren?

Properties, deren zugeordnete Werte es erlauben, ihr Subjekt in einem externen System/Normdatei/Verzeichnis nachzuschlagen, werden in der Regel mit dem [typen](https://www.mediawiki.org/wiki/Wikibase/Indexing/RDF_Dump_Format#Properties) [`wikibase:ExternalId`](https://www.mediawiki.org/wiki/Wikibase/Indexing/RDF_Dump_Format#External_ID) versehen. Eine Abfrage dieses property types foerdert `3251` externe Identifier zutage (Oktober 2018):

```sparql
SELECT (COUNT(?property) as ?num) {
  ?property wikibase:propertyType wikibase:ExternalId .
}
```

Einige dieser Properties, beispielsweise *GND ID* (`P227`) sind zusaetzlich Instanzen der Klasse *Wikidata-Eigenschaft für Normdaten* (`PQ18614948`): von den besagten `3251` external ID Properties erfuellen `210` diese Bedingung. Allerdings ist nicht jede *Normdaten*-Property gleichzeitig ein externer Identifikator; unter den `19` *Normdaten*-Properties, die nicht als externe Identifikatoren ausgezeichnet sind, finden sich z.B. die Eigenschaften *Dewey-Dezimalklasifikation* (`P1036`) und *HURDAT-Kennung* (`P502`):

```sparql
SELECT ?property ?propertyLabel {
  ?property wdt:P31 wd:Q18614948 .
  MINUS {
    ?property wikibase:propertyType wikibase:ExternalId .
  }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

`wikibase:ExternalId` ist eine Eigenschaft einer Wikibase Property. Um ueber den SPARQL-Endpunkt von Wikidata auf das zugehoerige `wikibase:Property`-Objekt eines in einem Statement verwendeten Praedikats zugreifen zu koennen, stehen die Praedikate `wikibase:directClaim` und `wikibase:claim` zur Verfuegung, welche Property-Entitaeten auf
in den Namespaces `wdt:` bzw. `p:` addressierte Praedikate abbilden (Query aus [Wikidata Workshop auf der DHD 2018](https://github.com/clmb/wikidata_workshop/tree/master/SPARQL2_Identifikatoren#nutzung-von-identifikationen-auf-wikidata)):

```sparql
SELECT (COUNT(?propertyclaim) AS ?count)
WHERE 
{
  wd:Q762 ?propertyclaim ?value  .
  ?property wikibase:directClaim ?propertyclaim .
  ?property wikibase:propertyType wikibase:ExternalId .
}
```

Wenn `externalId`-Properties mit einem *URL-Formatierer* (`P1630`) ausgestattet sind (?), koennen die vergebenen Werte direkt als `URL`s bezogen werden, indem z.B. der normalisierte Wikidata Truthy Namespace (`wdtn:`) verwendet wird:

```sparql
SELECT ?item ?itemLabel ?value {
  ?item wdtn:P227 ?value ;
        wdt:P31 wd:Q5 ;
        wdt:P800 ?work .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

Bedingung dafuer scheint aber tatsaechlich zu sein, dasz die Property sowohl einen *URL-Formatierer* hat als auch `externalId` ist, denn eine Abfrage von *HURDAT*-Links ueber `wdtn:P502` liefert keine Ergebnisse, obwohl ein *URL-Formatierer* vorliegt.


## Auswahl externer Identifikatoren

...


## Property *exakte Übereinstimmung* (`P2888`)

Die Property *exakte Übereinstimmung* (`P2888`) ist kein externer Identifikator, sondern eine `wikibase:Url`. Trotzdem wird sie verwendet, um Wikidata-Items als Aequivalente zu externen Inhalten auszuzeichnen. Folgende Query liefert eine Liste von Personen (`Q5`), die anhand ihrer Eintragung auf der Webseite `nobelprize.org` identifiziert werden koennen:

```sparql
SELECT ?item ?itemLabel ?id {
  ?item wdt:P31 wd:Q5 .
  ?item wdt:P2888 ?id .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```


## Queries

Finde alle Instanzen von *Wikidata-Eigenschaft zur Identifizierung von Kunstwerken* ([`Q44847669`](https://www.wikidata.org/wiki/Q44847669)), finde heraus welche Institution diesen Identifier vergibt ([`P2378`](https://www.wikidata.org/wiki/Property:P2378)), von welchem Typ diese Institution ist, wie viele Items mit dem Identifier identifiziert werden, und wie viele von diesen Items auszerdem die Eigenschaft *Inventarnummer* ([`P217`](https://www.wikidata.org/wiki/Property:P217)) verwenden. 

```sparql
SELECT ?identifier 
       ?identifierLabel 
       ?inst 
       ?instLabel 
       ?instKat
       ?instKatLabel
       ?itemcount 
       ?inventarisiert 
       ?uninventarisiert {
  BIND (?itemcount - ?inventarisiert AS ?uninventarisiert) .
  { 
    SELECT ?identifier 
           ?inst 
           ?instKat
           (count(?item) AS ?itemcount) 
           (count(?inv) AS ?inventarisiert) WHERE {
             ?identifier wdt:P31 wd:Q44847669 ; 
                         wdt:P2378 ?inst ;
                         wikibase:directClaim ?identprop .
             ?inst wdt:P31 ?instKat .
             ?item ?identprop ?id . 
             OPTIONAL {
               ?item wdt:P217 ?inv . 
             }
           }
    GROUP BY ?identifier ?inst ?instKat
  }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
* [Run on WDQS](http://tinyurl.com/y8wlxamu)


