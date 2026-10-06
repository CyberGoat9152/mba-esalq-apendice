A rotina varre as observações brutas da série temporal e acumula, por produto, dia e
célula de geohash de precisão seis (≈ 1,2 km), a soma e a contagem de preços e a soma
das coordenadas, gravando ao final a média de preço e o centróide da célula. Reproduz-se
o núcleo do procedimento (linguagem Go). 

```go
err = src.ScanSince(ctx, since, func(productID int, ts time.Time,
lat, lon, price float64) error {
    if price <= 0 {
        return nil
    }
    day := time.Date(ts.Year(), ts.Month(), ts.Day(), 0, 0, 0, 0, time.UTC)
    gh := geohash.EncodeWithPrecision(lat, lon, geohashPrecision) // precisão 6
    k := cellKey{productID: productID, day: day, geohash: gh}
    c, ok := agg[k]
    if !ok {
        c = &cellAgg{}
        agg[k] = c
    }
    c.sum += price
    c.count++
    c.lat += lat
    c.lon += lon
    return nil
})
for k, c := range agg {
    avg := c.sum / float64(c.count)  // média de preço da célula
    lat := c.lat / float64(c.count)  // centroide das observações
    lon := c.lon / float64(c.count)
    writer.UpsertDailyPrice(ctx, k.productID, k.day, k.geohash, lat, lon, avg, c.count)
}
```
