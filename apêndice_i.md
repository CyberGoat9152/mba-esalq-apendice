O teste exercita o agrupamento de duas regiões geográficas bem separadas e com
níveis de preço distintos, validando a separação em duas zonas, a ordenação por preço
médio e o sinal da variação de preço frente à data anterior (linguagem Go). 
```golang
func TestKMeansSeparatesTwoGroups(t *testing.T) {
	// Dois grupos geográficos bem separados, com níveis de preço distintos.
	pts := []Point{
		{Lat: -23.55, Lon: -46.63, Price: 30, PrevPrice: 28, Samples: 5}, // SP
		{Lat: -23.56, Lon: -46.64, Price: 32, PrevPrice: 30, Samples: 5},
		{Lat: -23.54, Lon: -46.62, Price: 31, PrevPrice: 29, Samples: 5},
		{Lat: -8.05, Lon: -34.88, Price: 10, PrevPrice: 12, Samples: 5}, // Recife
		{Lat: -8.06, Lon: -34.89, Price: 11, PrevPrice: 13, Samples: 5},
		{Lat: -8.04, Lon: -34.87, Price: 9, PrevPrice: 11, Samples: 5},
	}

	zones := KMeans(pts, 2, 42)
	if len(zones) != 2 {
		t.Fatalf("esperava 2 zonas, obteve %d", len(zones))
	}

	// Zonas ordenadas por preço médio ascendente: a mais barata primeiro.
	if zones[0].MeanPrice >= zones[1].MeanPrice {
		t.Fatalf("zonas não ordenadas por preço médio")
	}

	// A zona cara subiu frente à data anterior; a barata caiu.
	if zones[1].PctChange == nil || *zones[1].PctChange <= 0 {
		t.Errorf("esperava variação positiva na zona cara")
	}
	if zones[0].PctChange == nil || *zones[0].PctChange >= 0 {
		t.Errorf("esperava variação negativa na zona barata")
	}

	// Cada zona de 3 pontos forma um triângulo com o polígono fechado.
	for i, z := range zones {
		if len(z.Polygon) < 3 {
			t.Errorf("zona %d: polígono com poucos pontos", i)
		}
		if z.Polygon[0] != z.Polygon[len(z.Polygon)-1] {
			t.Errorf("zona %d: polígono não fechado", i)
		}
	}
}
```
