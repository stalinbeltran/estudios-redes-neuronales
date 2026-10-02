# `dim-nist` — reducir los dígitos de NIST de 8×8 a 4×4 contra la generalización

| | |
|---|---|
| **Inicio / fin (UTC)** | 2026-10-02, una unidad de systemd en el dev, ~20 min (un primer lanzamiento se negó al arrancar: 4000 pasos no son épocas enteras de 9; se fijó 3996 antes de entrenar) |
| **Instancias alquiladas** | **0** — droplet dev |
| **Coste real** | **0 $** |
| **Runs** | 30: `W ∈ {8,7,6,5,4}` × 5 semillas + control `w8-de4` × 5 |
| **Dataset** | `uci-optdigits-8px-r20261002` (dígitos 8×8 de UCI/NIST vía scikit-learn), train 180 / val 1617 |
| **Artefactos** | [`experimentos-cnn/2026-10-02-dimension-nist/`](https://github.com/stalinbeltran/experimentos-cnn/tree/tema-2/2026-10-02-dimension-nist) (rama `tema-2`); pesos en el almacén `foveal-vision-data/experimentos-cnn-resultados/dim-nist/` |

El gemelo de `dim-gen` (#24) sobre dígitos. **Resultado: lo contrario.** La mejor resolución es
la original (8 px, exactitud 0,851); 6 y 5 px son indistinguibles (0,842, 0,829): **W mínimo
suficiente 5 px**. **La resolución no estorba** y la brecha no crece. 4 px cae a 0,714 por **falta
de información**: la exactitud de train también cae (0,83) y el control —red de 8 px con la
información de 4— da 0,718, igual que `w4`. `w7` (0,765) es el zigzag por paridad que el criterio
anotó antes (kernel 4 con mapa final 1×1). Leídos juntos #24 y #26: la resolución estorba cuando
**sobra** (128 px para una caja), no cuando el dato ya está al límite (8 px para un dígito).

**No mueve `ESTADO.md`** (otra red, otro dato). Pendiente: fusionar `tema-2` en `main`. 13
escritores compartidos entre train y val: generaliza a dígitos nuevos, no a escritores nuevos.
