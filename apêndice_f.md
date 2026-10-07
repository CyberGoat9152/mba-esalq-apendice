As observações de uma data fixa são agrupadas em zonas espaciais por k-means
bidimensional sobre as coordenadas, com inicialização k-means++ e semente igual à data,
de modo que a mesma data produza sempre as mesmas zonas. O número de zonas segue
a regra prática k=n2, limitada a oito zonas, em que n é o número de pontos; o preço
de cada zona é a média ponderada pelo número de amostras de cada ponto (linguagem Go).
// passo de atribuição: cada ponto para o centro mais próximo
for i, p := range pts {
    best, bestD := 0, math.MaxFloat64
    for c := range centers {
        d := sqDist(p.Lat, p.Lon, centers[c][0], centers[c][1])
        if d < bestD {
            best, bestD = c, d
        }
    }
    assign[i] = best
}
// passo de atualização: centro = média das coordenadas dos membros
for c := range centers {
    if cnt[c] > 0 {
        centers[c][0] = sumLat[c] / float64(cnt[c])
        centers[c][1] = sumLon[c] / float64(cnt[c])
        }
}
// preço médio da zona, ponderado pelo nº de amostras de cada ponto
for _, m := range members {
    w := float64(m.Samples)
    pSum += m.Price * w
    wSum += w
}
z.MeanPrice = pSum / wSum 
