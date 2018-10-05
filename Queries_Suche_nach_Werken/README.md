# Zielsetzung

Welche SPARQL-Abfragen werden benötigt um nach bestimmten Kriterien Kunstwerke im Datenbestand von Wikidata zu finden?

## Suche nach Werken eines Künstlers

Es sind verschiedene Ansätze möglich. Beginnen wir mit einem Beispiel aus dem [Wikidata-Workshop von Claudia Müller-Birn](https://github.com/clmb/wikidata_workshop/tree/master/SPARQL1_Statements).

### Über die Eigenschaft Werke `P800`

Künstlern können über die Eigenschaft Werke (notable work) Kunstwerke zugewiesen werden, deren Urheber sie sind. Sucht man ausgehend vom Künster nach diesen aufgelisteten Werken, erhält man die Einträge in diese Auflistung:

```sparql
SELECT ?item ?itemLabel

WHERE
{
  wd:Q762 wdt: `P800`  ?item . # Leonardo da Vinci Werke

    SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en" . }
}
```
[auf WDQS ausprobieren ...](https://query.wikidata.org/#%23Suche%20alle%20Gem%C3%A4lde%20eines%20K%C3%BCnstlers%0A%0ASELECT%20%3Fitem%20%3FitemLabel%0AWHERE%20%0A%7B%0A%20%20wd%3AQ762%20wdt%3AP800%20%3Fitem%20.%20%23%20Leonardo%20da%20Vinci%20Werke%0A%20%20%0A%20%20%20%20SERVICE%20wikibase%3Alabel%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22%5BAUTO_LANGUAGE%5D%2Cen%22%20.%20%7D%0A%0A%7D)

Ergebnis (4.10.2018): 13 Werke

### Über die Eigenschaft Urheber `P170`

Umgekehrt ist in aller Regel bei einem Kunstwerk ein Urheber angegeben. Fragt man von dieser Seite nach Werken Leonardo da Vincis, also:

```sparql
SELECT ?item ?itemLabel
WHERE
 {
  ?item wdt:P170 wd:Q762 . # Eigenschaft Urheber: Leonardo da Vinci

  SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[auf WDQS ausprobieren ...](https://query.wikidata.org/#%23%20Suche%20nach%20Gem%C3%A4lden%20Leonardo%20da%20Vincis%2C%20per%20Urheber%0A%0ASELECT%20%3Fitem%20%3FitemLabel%20%0AWHERE%0A%20%7B%0A%20%20%3Fitem%20wdt%3AP170%20wd%3AQ762%20.%20%23%20Eigenschaft%20Urheber%3A%20Leonardo%20da%20Vinci%0A%20%20%20%0A%20%20SERVICE%20wikibase%3Alabel%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22%5BAUTO_LANGUAGE%5D%2Cen%22.%20%7D%0A%7D)

erhält man deutlich mehr Objekte.

Ergebnis (4.10.2018): 82 Werke

Offensichtlich wird die Eigenschaft `P800` Werke eher als optional verstanden und als Hiweis auf wichtige, nicht alle Werke (bezeichnend das englische Label: "**notable** work"), die Eigenschaft `P170` Urheber aber als notwendige Information zu einem Kunstwerk (soweit Urheber bekannt). Eine Stichprobe bestätigt dies:

### Stichprobe: Werke vs Urheber

Abfrageergebnisse vom 4.10.2018:

Künstler | Werke via `P800` | Werke via `P170`
-------- | -------- | --------
Leonardo da Vinci | 13 | 82
Cimabue | 4 | 31
Gerhard Richter | 3 | 136
Franz von Lenbach | 2 | 122
Poussin | 5 | 184
Jules Dupré | 3 | 79

### Definition von `P800` Werke

Dieser empirische Befund steht in Einklang mit der [Definition der Eigenschaft  `P800` ](https://www.wikidata.org/wiki/Property:P800 ):

Beschreibung DE: "relevante wissenschaftliche oder künstlerische Werke der Person"

Beschreibung EN: "notable scientific, artistic or literary work, or other work of significance among subject's works"
(Stand: 4.10.2018)

Auf der [Diskussionsseite zu  `P800` ](https://www.wikidata.org/wiki/Property_talk:P800) wird die beiläufig auch so diskutiert.

### Fazit

Die Suche nach Künstlern sollte immer über die Eigenschaft `P170` von Kunstwerken erfolgen, nicht über die Auswahlliste von Werken eines Künstlers (`P800`).

## Einschränken auf Kunstwerken

Sucht man gezielt nach Werken der Bildenden Kunst (also beispielsweise **nicht** nach literarischen Werken) kann man diese Einschränkung der Suche vorschalten:

```sparql
SELECT ?item ?itemLabel
WHERE
 {
  ?item wdt:P31 wd:Q838948 . # ist ein Kunstwerk
  ?item wdt:P170 wd:Q762 . # Eigenschaft Urheber: Leonardo da Vinci

  SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[auf WDQS ausprobieren ...](https://query.wikidata.org/#%23%20Suche%20nach%20Werken%20eines%20K%C3%BCnstlers%2C%20ausschlie%C3%9Flich%20Kunstwerke%0A%0ASELECT%20%3Fitem%20%3FitemLabel%0AWHERE%0A%20%7B%0A%20%20%3Fitem%20wdt%3AP31%20wd%3AQ838948%20.%20%23%20ist%20ein%20Kunstwerk%0A%20%20%3Fitem%20wdt%3AP170%20wd%3AQ762%20.%20%23%20Eigenschaft%20Urheber%3A%20Leonardo%20da%20Vinci%0A%0A%20%20SERVICE%20wikibase%3Alabel%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22%5BAUTO_LANGUAGE%5D%2Cen%22.%20%7D%0A%7D)

Was im Falle von Leonardo da Vinci ein einziges Ergebnis erbringt (Stand: 4.10.2018).

Der Grund für dieses enttäuschende Ergebnis: Die Werke Leonardos sind in der Regel spezifischer klassifiziert, etwa als Gemälde (`Q3305213`; 46 Ergebnisse), Zeichnung (`Q93184`, 25 Ergebnisse), Wandmalerei (`Q219423`1 Ergebnis) usw.

Wenn man die gesuchte Gattung kennt, kann man gezielt danach suchen:

```sparql
SELECT ?item ?itemLabel
WHERE
 {
  ?item wdt:P31 wd:Q93184 . # ist eine Zeichnung
  ?item wdt:P170 wd:Q762 . # Eigenschaft Urheber: Leonardo da Vinci

   SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[auf WDQS ausprobieren ...](https://query.wikidata.org/#%23%20Suche%20nach%20Werken%20eines%20K%C3%BCnstlers%2C%20ausschlie%C3%9Flich%20Zeichnungen%0A%0ASELECT%20%3Fitem%20%3FitemLabel%0AWHERE%0A%20%7B%0A%20%20%3Fitem%20wdt%3AP31%20wd%3AQ93184%20.%20%23%20ist%20eine%20Zeichnung%0A%20%20%3Fitem%20wdt%3AP170%20wd%3AQ762%20.%20%23%20Eigenschaft%20Urheber%3A%20Leonardo%20da%20Vinci%0A%0A%20%20%20SERVICE%20wikibase%3Alabel%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22%5BAUTO_LANGUAGE%5D%2Cen%22.%20%7D%0A%7D)

Wenn man aber alle Kunstwerke erfassen will, bietet SPARQL die Lösung an, nach der Klasse Kunstwerk und allen Unterklassen von Kunstwerk zu suchen:

```sparql
SELECT ?item ?itemLabel
WHERE
 {
   ?item wdt:P31/wdt:P279* wd:Q838948. # Ein Objekt der Klasse Kunstwerke oder eine ihrer Unterklassen
   ?item wdt:P170 wd:Q762 . # Eigenschaft Urheber: Leonardo da Vinci

  SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[auf WDQS ausprobieren ...](https://query.wikidata.org/#%23%20Suche%20nach%20Werken%20eines%20K%C3%BCnstlers%2C%20ausschlie%C3%9Flich%20Kunstwerke%0A%0ASELECT%20%3Fitem%20%3FitemLabel%0AWHERE%0A%20%7B%0A%20%20%20%3Fitem%20wdt%3AP31%2Fwdt%3AP279%2a%20wd%3AQ838948.%20%23%20Ein%20Objekt%20der%20Klasse%20Kunstwerke%20oder%20eine%20ihrer%20Unterklassen%0A%20%20%20%3Fitem%20wdt%3AP170%20wd%3AQ762%20.%20%23%20Eigenschaft%20Urheber%3A%20Leonardo%20da%20Vinci%0A%0A%20%20SERVICE%20wikibase%3Alabel%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22%5BAUTO_LANGUAGE%5D%2Cen%22.%20%7D%0A%7D%0A)

Iritierenderweise ergibt diese Abfrage **83** Ergebnisse (Stand: 5.10.2018), also eines mehr als die Abfrage ohne die Einschränkung auf Kunstwerke (s.o.). Wie kann das sein?

Der Vergleich der beiden Ergebnislisten ist in mehrfacher Hinsicht instruktiv. Vorab die Erklärung für die wundersame Werkvermehrung: Tatsächlich erbringt die Abfrage mit Einschränkung im Vergleich zur Abfrage ohne Einschränkung 3 Ergebnisse weniger und 4 Ergebnisse mehr (Saldo +1):

Es fallen heraus: Die Wikimedia-Liste mit den  Erfindungen des Leonardo da Vinci (`Q3289539`), die Wikimedia-Liste der Gemälde von Leonardo da Vinci (`Q1469799`) sowie der *Saal der Fünfhundert* (`Q2198705`) im Palazzo Vecchio, der als Halle (`Q240854`) klassifiziert ist, die als Teil eines Gebäudes gilt, nicht als Kunstwerk. Aus disziplinärer Perspektive würde man also sagen: Die ersten  beiden Ausschlüsse sind sachgerecht, der dritte ist hoch problematisch. Abgesehen von der Frage, ob Architektur Kunst ist, schon deshalb, weil Leonardo natürlich nicht die Halle (also den Bauteil) errichtet, sondern sie mit Fresken ausgestattet hat (korrekt erfasst im Qualifikator betroffener Teil `P518` zur Aussage über Leonardo als Urheber). Da die (nicht mehr existierende)  *Schlacht von Anghiari* (`Q2045726`), also das von Leonardo im Palazzo Vecchio ausgeführte Wandgemälde, selbst nocheinmal in der Liste auftaucht (und als Kunstwerk, nämlich Fresko `Q22669139` klassifiziert ist), wirkt sich die sachlich problematische Klassifizierung von *Saal der Fünfhundert* hier positiv aus, weil sie eine Dublette vermeidet.

Hinzu kommen: 4 Mehrfachnennungen, nämlich Werke, die eine doppelte Klassifizierung haben. Bei der *Schlacht von Anghiari* kommt zu Fresko `Q22669139` noch verschollenes Kunstwerk (`Q4140840`) hinzu, zwei Werke sind als Zeichnung (`Q93184`) und Gemälde (`Q3305213`) klassifiziert, eines als Gemälde und Skizze (`Q5078274`). Die Unterklassen Zeichnung (`Q93184`) und Gemälde (`Q3305213`) sind eigentlich jeweils über die Eigenschaft verschieden von (`P1889`) gegeneinander abgegrenzt, was ihre parallele Verwendung, abgesehen davon, ob sie sachlich im Hinblick auf die betreffenden Werke zutreffend ist, aus systematischen Gründen problematisch macht (problematisch ist allerdings auch die Abgrenzung über verschieden von `P1889`, aber das ist ein anderes Thema).

Abfragetechnisch lässt sich dieses verwirrende Bild sehr leicht durch die Verwendung des Keywords `DISTINCT` scharf stellen:

```sparql
SELECT DISTINCT ?item ?itemLabel
WHERE
 {
   ?item wdt:P31/wdt:P279* wd:Q838948. # Ein Objekt der Klasse Kunstwerke oder eine ihrer Unterklassen
   ?item wdt:P170 wd:Q762 . # Eigenschaft Urheber: Leonardo da Vinci

  SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[auf WDQS ausprobieren ...](https://query.wikidata.org/#%23%20Suche%20nach%20Werken%20eines%20K%C3%BCnstlers%2C%20ausschlie%C3%9Flich%20Kunstwerke%0A%0ASELECT%20DISTINCT%20%3Fitem%20%3FitemLabel%0AWHERE%0A%20%7B%0A%20%20%20%3Fitem%20wdt%3AP31%2Fwdt%3AP279%2a%20wd%3AQ838948.%20%23%20Ein%20Objekt%20der%20Klasse%20Kunstwerke%20oder%20eine%20ihrer%20Unterklassen%0A%20%20%20%3Fitem%20wdt%3AP170%20wd%3AQ762%20.%20%23%20Eigenschaft%20Urheber%3A%20Leonardo%20da%20Vinci%0A%0A%20%20SERVICE%20wikibase%3Alabel%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22%5BAUTO_LANGUAGE%5D%2Cen%22.%20%7D%0A%7D)

Ergebnis (5.10.2018): 79 Werke (Kunstwerke mit mehrfacher Klassifizkation werden nun nur einmal ausgegeben)

Dennoch sollte man sich bewusst sein, dass der Datenbestand in Wikidata aus fachwissenschaftlicher Perspektive "unscharf" ist, zum einen, weil Objekte fragwürdig klassifiziert wurden, zum anderen, weil die Klassendefinitionen und die Verknüpfung der Klassen nicht selten problematisch sind (Fresko `Q22669139` ist verschieden von - `P1889` - Fresko `Q134194` [eine Maltechnik] und Fresko `Q25631150` [ein Malwerkzeug]). Das ist bei einem Community-basierten, organisch wachsenden Projekt nicht zu kritisieren, man sollte sich aber der Problematik bewusst sein und nicht der Magie der exakten Zahlen erliegen.
