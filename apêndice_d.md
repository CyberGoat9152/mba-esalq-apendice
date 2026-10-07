A série por raio aplica um pré-filtro por caixa delimitadora em SQL — para que o índice descarte a maioria das linhas — e, em seguida, o teste exato do círculo pela distância de haversine, executado na aplicação. 

```golang
// pré-filtro por caixa delimitadora (aproveita o índice)
dLat := radiusM / 111320.0
cosLat := math.Cos(lat * math.Pi / 180.0)
dLon := radiusM / (111320.0 * cosLat)
//   ... WHERE d.lat BETWEEN lat-dLat AND lat+dLat
//         AND d.lon BETWEEN lon-dLon AND lon+dLon
if haversineM(lat, lon, plat, plon) > radiusM {
    continue // observação fora do círculo
}
// distância geodésica (haversine), em metros
func haversineM(lat1, lon1, lat2, lon2 float64) float64 {
     const R = 6371000.0
     rad := math.Pi / 180.0
     dLat := (lat2 - lat1) * rad
     dLon := (lon2 - lon1) * rad
     a := math.Sin(dLat/2)*math.Sin(dLat/2) +  math.Cos(lat1*rad)*math.Cos(lat2*rad)*math.Sin(dLon/2)*math.Sin(dLon/2)
     return R * 2 * math.Atan2(math.Sqrt(a), math.Sqrt(1-a))
} 
```
