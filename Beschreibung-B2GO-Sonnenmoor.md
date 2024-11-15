# Sonnenmoor

## Kommissionierung

### Beleg scannen und übernehmen:  
* **Ablauf:**
Der Beleg wird in der B2GO gescannt mittels Code-128 (Belegjahr-Belegnummer)
Der Beleg wird in den Verkaufsbelegen gesucht und die BelID abgegriffen.
Wurde eine BelID gefunden, wird der Endpunkt TransoformBeleg aufgerufen - welcher bei Erfolg, die BelID des Lieferscheines zurückgibt. 
Der Lieferschein wird automatisch in der B2GO geöffnet.
<br>
* **B2GO:**
Es gibt 2 verschiedene Modi die B2GO zu nutzen. Den Scanner-und den Standardmodus.
Welcher Modus verwendet wird, muss in der B2GOConfig Tabelle für jeden mobilen Benutzer eingetragen werden. 
Diese Tabelle wird bei der Einrichtung am Kundensystem in der Kundendatenbank angelegt.
Die folgende Beschreibung bezieht sich nur auf den für Sonnenmoor eingeführten Scanner-Modus.  <br>
In der B2GO wird das Modul Kommissionierung gestartet.
Es öffnet sich automatisch die Scanner-Maske mit dem fokusierten Ausgabefeld. Sobald der Barcode der Auftragsbestätigung gescannt wird, erscheint die Belegnummer im Format <Belegjahr-Belegnummer> im Ausgabefeld.
Dieses Feld schließt automatisch 3 Sekunden nach dem Scan. Im Hintergrund wird die BelID der Auftragsbestätigung ermittelt und die Beleg-Transformiergung gestartet.
Wenn kein neuer Beleg gescannt werden soll, kann die Scanner Maske mit dem Button "Abbrechen" geschlossen werden.
<br>
* **Metadata - MakroCall:**
Das Beleg Transform erfolgt mittels Makro via Kontextmenü Aufruf in der Metadata Lösung "BSB2GOSonnenmoorKommissionierung".
Das Makro ruft die entsprechende DLL BusinessSoftware.B2GO.Kommissionierung.Sonnenmoor.dll mit dem Parameter [BelID] auf.
<br>
* **Schnittstelle:**
Die Schnittstelle zwischen Applikationsserver und B2GO wurde mit einem PHP Endpunkt realisiert.
Der Endpunkt ist über die URL  "... api/ext/sage100/90/beleguebernahme/v1/" erreichbar und benötigt zusätzlich zu den ApplikationServer Zugriffsdaten die BelID als Parameter

``` json
{
  "token": "",
  "database": "OLDemoReweAbfA",
  "mandant": "88",
  "target": "beleguebernahme",
  "username": "sa",
  "password": "*******",
  "param": "{\"belid\":\"574\"}",
  "baseurl": "*********"
}
```

Result:
```
[{"results":[{"$key":"belid","Name":"belid","DataType":"Integer","Value":"582"}]}]
```

Im Endpunkt wird der ApplikationServer Request für einen "CallMacroDLL" gebaut.
Dadurch kann der ApplikationServer die BusinessSoftware.B2GO.Kommissionierung.Sonnenmoor.dll aufrufen.

DCM:
In der DCM TransformBeleg wird geprüft ob die Auftragsbestätigung bereits einen Referenzbeleg hat.
Wenn nicht, wird ein Lieferschein mit dieser BelID erstellt.
Die Positionsmengen des Lieferscheins werden in das Positionsfeld "VKZubehoreText" geschrieben.
Danach wird die Menge, MengeBasis, VorgangspositionsReferenzen auf 0 gesetzt,
gebuchte Chargen sowie Lagerplätze geleert.
Bei Erfolg, wird die neue BelID and den PHP Endpunkt zurückgeliefert und von dort weiter an die B2GO geschickt.

Chargenermittlung:
B2GO:
Der Kommissionier-Lieferschein wird nach dem Scan der Auftragsbestätigung automatisch erstellt und geöffnet.
Bei jedem Öffnen eines Kommissionier-Lieferscheins wird geprüft ob für diesen Beleg noch reservierte Chargen in vorhanden sind und gegebenenfalls, die Chargen freigegeben.
Es wird automatisch der Tab "Positionen" im Bearbeitungsmodus geöffnet.
Die Filterzeile "nach Belegpositionen filtern" wird automatisch fokussiert.
Somit kann sofort mit dem Scanvorgang der Belegpositionen begonnen werden.
Der gescannte Barcode muss die Artikelnummer beinhalten.
Wird eine Barcode gescannt, werden die Belegpositionen nach dieser Artikelnummer durchsucht.
Wir der Artikel im Beleg nicht gefunden erscheint ein Meldungsfenster "Hinweis- Artikel nicht gefunden". 
Die Artikelnummer bleibt zur Kontrolle in der Filterzeile angezeigt.
Diese kann dort manuell geändert und mit dem Lupensymbol erneut gesucht werden.
Alternativ kann die Artikelnummer mit dem X-Button in der Filterzeile gelöscht werden um danach erneut zu scannen.

Wird die Artikelnummer in den Belegpositionen gefunden, wird automatisch die Menge aus dem Feld "VKZubehoreText" ermittelt und die Preisfindung angestoßen.
Das neue Belegobjekt mit korrekter Menge sowie Preis wird zurückgegeben und es wird ermittelt ob es sich um einen Chargenartikel handelt.
Wenn nicht, wurde die Belegposition erfolgreich kommissioniert.

Handelt es sich um einen Chargenartikel, wird die Chargenermittlung über den Endpunkt "ProvideCharge" angestoßen.
Folgende Parameter werden an den Endpunkt übermittelt


 