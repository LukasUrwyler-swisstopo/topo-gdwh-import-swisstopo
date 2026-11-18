# GDWH & STAC Import Pipeline

> [!NOTE]
> Dieses Repository ist ein Wegweiser. Der Quellcode wird in der swisstopo-Organisation weiterentwickelt:
>
> **https://github.com/swisstopo/topo-gdwh-import**
>
> Das Repository ist privat. Zugriff haben Mitglieder der swisstopo GitHub-Organisation.

## Worum es geht

Desktop-Tool mit grafischer Oberfläche, das den Import von Orthophotos, Höhenmodellen und Punktwolken ins Geodatenwarehouse (GDWH) von swisstopo automatisiert. Es führt durch den ganzen Ablauf: Daten prüfen und bereinigen, XML-Metadaten erzeugen, NoData und Raumbezug korrekt setzen und das Datenpaket ins GDWH kopieren. Nach dem Import im GDWH folgt die Publikation via **STAC** automatisch.

Entwickelt bei swisstopo für die Produktion von Spezialbefliegungen, Rapid Mapping und Monitoring-Aufträgen.

## Unterstützte Produkte

| Produkt | Inhalt | Format |
|---|---|---|
| **SB_DOP** | Orthophoto | GeoTIFF, 8 bit, RGB bzw. RGBN |
| **SB_DOP_16** | Orthophoto-Einzellinien | GeoTIFF, 16 bit, NRGB |
| **SB_DSM** | Höhenmodell und Hillshade | GeoTIFF, Float32 bzw. 8 bit |
| **SB_DSM_PUNKTWOLKE** | photogrammetrische Punktwolke | LAZ, LAS 1.4 (PF6, mit Farbe PF7) |

Unterstützte Kamerasysteme: **Leica ADS80**, **ADS100** und **DMC-4**. Das Tool wählt das Kamerasystem anhand des Aufnahmejahrs vor und wendet die passenden Regeln an.

## Funktionen

- **Geführte Eingabe** aller Metadaten im GUI, mit Pflichtfeldern und Plausibilitätsprüfung. Kein manuelles Bearbeiten von Scripts.
- **Quellordner bereinigen** nach einer Whitelist, erst nach bestätigtem Sicherheitscheck.
- **XML-Metadaten** pro Datei erzeugen, inklusive Aufnahmezeiten aus den Befliegungslinien.
- **NoData-Tag und interne Maske** setzen. Wasserflächen mit echten Nullwerten im Nahen Infrarot bleiben dabei gültig.
- **Raumbezug prüfen und setzen:** LV95 (EPSG:2056), beim DSM mit Höhenbezug LN02 (EPSG:2056+5728). Ein falscher Höhenbezug wie LHN95 stoppt den Import, bevor eine Datei verändert wird.
- **Punktwolken ins Zielformat bringen:** LAS 1.4, CRS-Tags byte-exakt nach Referenzkachel, Kachelursprung aus dem Dateinamen.
- **Altdaten aufbereiten:** ADS80-Punktwolken aus ASCII-Dateien in 1-km-LAZ-Kacheln umrechnen und OrthoVista-Kacheln pixelgenau auf das 1-km-Grid schneiden.
- **Optionale Korrektur falscher NoData-Pixel** in Orthophotos, zum Beispiel bei überstrahlten Gletscherflächen oder tiefen Schatten.
- **STAC-Prüfung vor dem Import:** Existiert das Asset schon oder wartet ein gleiches Datenpaket, bleibt der Import gesperrt. So wird nichts versehentlich überschrieben.
- **Kopie mit Prüfsumme** (MD5 im selben Durchgang) und `files.csv` mit TileKey und Footprint.
- **Logdatei** pro Import und ein fortlaufendes Archiv-Log.

## Ablauf

```
1. Produkt und Kamerasystem wählen
2. Datenpaket im GDWH-Portal anlegen
3. Quell- und Zielpfad, Metadaten erfassen
4. Vorprüfungen: Dateinamen, Raumbezug, STAC
5. Sicherheitscheck bestätigen
6. Import: aufbereiten -> bereinigen -> CRS -> XML -> NoData -> Kopie
7. GDWH-Portal: CHECK und Import
8. STAC-Publikation automatisch
```

## Technik

- **Python** mit **tkinter**-GUI. Die Geodaten-Verarbeitung läuft getrennt davon in der OSGeo4W-Python-Umgebung.
- **GDAL/OGR** für Raster und Vektoren, **PDAL** für Punktwolken, **NumPy/SciPy** für die NoData-Analyse.
- Raumbezug: **EPSG:2056** (CH1903+ / LV95), Höhen **LN02** (EPSG:5728).
- Unit-Tests für die Verarbeitungsfunktionen, lauffähig ohne GDAL (Mock).

## Autor

Lukas Urwyler, swisstopo (Bundesamt für Landestopografie)
