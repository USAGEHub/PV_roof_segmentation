
[[_TOC_]]

--------------------------------------------------------------------------------------------------------

# Release v0.9 (30-08-2024)

`identificazione_falde_v0.9.model3`


## Input data

* Area of Interest (AoI)
* Building footprints
* Digital Surface Model (DSM)


## Output attributes

* **orientamento_classe**: 1 = NE, 2 = SE, 3 = SW, 4 = NW
* **aspect_mean (esposizione media)**: orientamento medio della falda in gradi da Nord verso Est
* **slope_mean (pendenza media)**: pendenza media falda, in percentuale (0 = orizzontale, 100 = a 45°, crescendo fino a infinito = verticale) 
* **area**: area della proiezione orizzontale della falda


## Implementazione

* utilizzare il tool 'clip raster by extent' per clippare il layer raster "DSM" rispetto al layer vettoriale "AoI"
* utilizzare il tool 'estrai/ritaglia da estensione' per clippare il layer vettoriale "footprints" rispetto al layer vettoriale "AoI"
* utilizzare il tool "buffer" sul layer vettoriale "footprints" per applicare un margine di 5 metri ed essere sicuri di includere tutta l'estensione delle falde
* utilizzare il tool "dissolve" sul layer vettoriale "footprints" bufferizzato
* estrarre il layer raster "DSM" clippato sul layer vettoriale "footprints" bufferizzato e dissolto, utilizzando il tool "clip raster with mask" (aggiungere opzione "keep input resolution" e deselezionare "match the extent"). Questo per eliminare il "rumore" che genererebbero gli altri elementi nei passi successivi
* con questo DSM ridotto ai soli edifici bufferizzati
   * calcolare l'orientamento cardinale delle falde (in gradi) attraverso il tool "GDAL aspect" 
   * calcolare la pendenza delle falde (in percentuale) attraverso il tool "GDAL slope" 
* classificare i valori di aspect con lo strumento "raster calculator": 
   * orientamento_classe = (("aspect@1" = 360) OR ("aspect@1"  <= 90)) * 1 + 
   * (("aspect@1" >= 90) AND ("aspect@1" < 180)) * 2 +
   * (("aspect@1" >= 180) AND ("aspect@1" < 270)) * 3 +
   * (("aspect@1" >= 270) AND ("aspect@1" < 360)) * 4
* sul risultato precedente, utilizzare il tool "sieve" impostando la soglia a 5 per eliminare i cluster troppo piccoli
* sul risultato precedente, utilizzare il tool "poligonizza (da raster a vettore) con l'obiettivo di avere un poligono per falda in base al valore omogeneo di aspect per pixel contigui della stessa falda 
* utilizzare il tool 'clip' per clippare il layer vettoriale "falde" appena ottenuto rispetto al layer vettoriale "AoI" ritagliato
* determinare i seguenti attributi sintetici da associare alle singole falde:
   * aspect_mean: valore medio del layer raster "Aspect" sul layer vettoriale di output "Falde" ottenuto con il tool "statistiche zonali"  
   * slope_mean: valore medio del layer raster "Slope" sul layer vettoriale di output "Falde" ottenuto con il tool "statistiche zonali"
   * area falda: area della proiezione orizzontale della falda, ottenuto con il tool "aggiungi attributi della geometria" sul layer "Falde" 


## Possibili migliorie

* Per gestire il problema delle falde che cadono comunque in zone di confine di classe si può provare a fare due diverse e parallele classificazioni a 4 classi, la seconda leggermente ruotata rispetto alla prima, confrontando/fondendo poi i risultati edificio per edificio, tenendo "la soluzione migliore". Possibile spunto che indica due diverse classificazioni ruotate di 45° luna dall'altra: https://www.youtube.com/watch?v=W5ls8LjXeH8&t=777s


--------------------------------------------------------------------------------------------------------