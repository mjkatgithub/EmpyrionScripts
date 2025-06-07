### Bilder + Anleitung:
![](Screenshots/img1.png)
- Es werden auf der Ausgabe alle Items im Input Container angezeigt, die noch nicht zugeordnet sind.
- IDs müssen dann in dem [[#DB LCD]] dem gewünschten Container zugeordnet werden.
- Wenn ein Item zugeordnet ist, folgt eine entsprechende Ausgabe unter dem zweiten Trennstrich:
        - Grün: Erfolgreich verschoben (Diese Anzeige verschwindet nach ein paar Sekunden.)
    - Gelb: Container hat Limit erreicht - Item wurde übersprungen oder in Überlauf-Container verschoben
    - Bleibt Rot: Container nicht gefunden, oder Container gerade gesperrt. auch wenn z.B. von einem anderen Spieler gerade auf den Zielcontainer zugegriffen wird, bleibt die Anzeige Rot.

![](Screenshots/img2.png)

#### Benötigte Bauteile:
- 1x Container (Input)
- 3x LCD (Script, Datenbank, Anzeige)
#### Container-Namen:
- Input (Nur hier werden Items entnommen!): `_AUTOSORT_IN`
    - Kann im Script angepasst werden (Zeile 2.)
#### LCD-Namen:
- Script-LCD: `Script:Autosort*`
- DB-LCD: `DB_AUTOSORT`
- Ausgabe-LCD: `AutosortOutput`
#### Ausgabe-LCD:
- Optimiert auf Projektor mit Breite = 3, Höhe egal.
#### DB LCD:
- Mindestens ein Element mit "ContainerName" & "ItemIds" muss vorhanden sein.
- Das Format muss JSON kompatibel sein. (Siehe Beispiel unten)
- Elemente:
    - ContainerName ist der Name des Containers
    - ItemIds ist eine Kommagetrennte Liste an IDs die in den Container einsortiert werden sollen.
    - Lists ist der Name der Liste, die zusätzlich in den Container sortiert werden soll.
        - Die Namen der Listen finden sich [hier](https://github.com/GitHub-TC/EmpyrionScripting#vordefinierte-id-listen) (Readme des Empyrion Scripting Mod)
    - Limit (optional) ist die maximale Anzahl von Items, die in den Container gelegt werden sollen.
        - Wenn das Limit erreicht ist und kein OverflowContainer definiert ist, bleiben die Items im Quellcontainer.
    - OverflowContainer (optional) ist der Name des Containers, in den Items verschoben werden, wenn das Limit erreicht ist.
        - Wird nur berücksichtigt, wenn auch ein Limit definiert ist.
    - Die ItemIds haben Vorrang vor den Listen, soll also etwas aus einer Liste in einen anderen Container einsortiert werden, kann die ID auch einfach in der ItemId Liste eingetragen werden.
- Bei Item-Listen über 2000 Zeichen muss das JSON komprimiert werden, damit es auf den LCD passt. hierfür gibt es tools wie z.B: https://jsonformatter.org/json-minify
    - Beispiel der Fehlermeldung bei einem zu langen Datenbankeintrag:
![](Screenshots/img3.png)
#### Inhalt DB-LCD:
```json
[
    {
        "ContainerName": "1. Materialien",
        "ItemIds": "2247,4316,4360,4361",
        "Lists": "Ingot,Components"
    },
    {
        "ContainerName": "Eisenkiste",
        "ItemIds": "2247",
        "Limit": 50000
    },
    {
        "ContainerName": "Werkzeugkiste",
        "ItemIds": "4150,4152,4262",
        "Limit": 10000,
        "OverflowContainer": "Lagerkiste"
    }
]
```

#### Script
1. das folgende Script ist komprimiert und kann direkt auf dem LCD eingefügt werden.
    - Falls sich jemand das Script anschauen möchte, ist dieses mit besser Lesbarer Formatierung in main.hbs zu finden.
```handlebars
<align=center>Autosorter - {{datetime}}</align>{{~set 'SOURCE' '_AUTOSORT_IN'}}{{~set 'DBLCDNAME' 'DB_AUTOSORT'}}{{~set 'SEPARATOR' '<align=center>================================================================================================</align>'}}<align=center>Autosort Inhalt (bei Permanent bleibenden Elementen die IDs in die DB eintragen)</align>{{@root.Data.SEPARATOR}}ID<pos=4em></pos><pos=5em>Anzahl</pos><pos=11em>Name</pos>{{#items @root.E.S @root.Data.SOURCE}}<color=red>{{Id}}</color><pos=4em></pos><pos=5em><color=green>{{Count}}</color></pos><pos=11em><color=white>{{i18n Id}}</color></pos>{{/items}}{{@root.Data.SEPARATOR}}{{devices @root.E.S @root.Data.DBLCDNAME}}{{gettext .0}}{{#fromjson .}}{{#each .}}{{#jsontodictionary .}}{{#items @root.E.S @root.Data.SOURCE}}{{#test Id in ../ItemIds}}<color=red>TODO: {{../Count}} x {{i18n ../Id}} -> {{../../ContainerName}}</color>{{#if ../../Limit}}{{#move ../. @root.E.S ../../ContainerName ../../Limit}}<color=green>DONE: {{../../Count}} x {{i18n ../../Id}} -> {{../../../ContainerName}} (Limit: {{../../../Limit}})</color>{{else}}{{#if ../../../OverflowContainer}}<color=yellow>OVERFLOW: Moving to {{../../../OverflowContainer}}</color>{{#move ../. @root.E.S ../../../OverflowContainer}}<color=green>DONE: {{../../Count}} x {{i18n ../../Id}} -> {{../../../../OverflowContainer}} (Overflow)</color>{{/move}}{{else}}<color=yellow>SKIPPED: Container {{../../../ContainerName}} reached limit of {{../../../Limit}}</color>{{/if}}{{/move}}{{else}}{{#move ../. @root.E.S ../../ContainerName}}<color=green>DONE: {{../../Count}} x {{i18n ../../Id}} -> {{../../../ContainerName}}</color>{{/move}}{{/if}}{{/test}}{{/items}}{{split Lists ','}}{{~#each .}}{{#items @root.E.S @root.Data.SOURCE}}{{#test Id in (lookup @root.ids ../.)}}{{#if ../../../../Limit}}{{#move ../. @root.E.S ../../../../ContainerName ../../../../Limit}}<color=green>[LISTS]DONE: {{../../Count}} x {{i18n ../../Id}} -> {{../../../../../ContainerName}} (Limit: {{../../../../../Limit}})</color>{{else}}{{#if ../../../../../OverflowContainer}}<color=yellow>[LISTS]OVERFLOW: Moving to {{../../../../../OverflowContainer}}</color>{{#move ../. @root.E.S ../../../../../OverflowContainer}}<color=green>[LISTS]DONE: {{../../Count}} x {{i18n ../../Id}} -> {{../../../../../../OverflowContainer}} (Overflow)</color>{{/move}}{{else}}<color=yellow>[LISTS]SKIPPED: Container {{../../../../../ContainerName}} reached limit of {{../../../../../Limit}}</color>{{/if}}{{/move}}{{else}}{{#move ../. @root.E.S ../../../../ContainerName}}<color=green>[LISTS]DONE: {{../../Count}} x {{i18n ../../Id}} -> {{../../../../../ContainerName}}</color>{{/move}}{{/if}}{{/test}}{{/items}}{{/each}}{{/split}}{{/jsontodictionary}}{{/each}}{{/fromjson}}{{/gettext}}{{/devices}}