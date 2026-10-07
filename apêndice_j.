Cada preço é convertido em preço relativo ao preço médio do mesmo produto
(Equação 5), descartando-se produtos vistos em um único mercado; a estatística H é
calculada sobre os postos, com correção para empates (Equações 6 e 7), e os pares de
mercados são comparados pelo teste de Dunn com correção de Bonferroni (linguagem TypeScript).
```golang
// preço relativo: 100 = preço habitual do produto no período (Equação 5)
export function relativePrices(obs: Observation[], marketOf: (o: Observation) => string): RelObs[] {
  const by = new Map<number, { sum: number; n: number; markets: Set<string> }>();
  for (const o of obs) {
    let a = by.get(o.product_id);
    if (!a) by.set(o.product_id, (a = { sum: 0, n: 0, markets: new Set() }));
    a.sum += o.price;
    a.n++;
    a.markets.add(marketOf(o));
  }
  return obs
    .filter((o) => by.get(o.product_id)!.markets.size >= 2) // sem informação regional
    .map((o) => {
      const a = by.get(o.product_id)!;
      return { ...o, value: (100 * o.price) / (a.sum / a.n) };
    });
}
// Kruskal-Wallis com correção para empates (Equações 6 e 7)
const { ranks, tieSum } = rank(all); // postos médios; tieSum = Σ(t³ − t)
let h = 0;
for (const key of keys) {
  const n = groups.get(key)!.length;
  const R = sum.get(key)!; // soma dos postos do mercado
  h += (R * R) / n;
  meanRanks.set(key, R / n);
}
h = (12 / (N * (N + 1))) * h - 3 * (N + 1);
const H = h / (1 - tieSum / (N * N * N - N));
const result = { H, df: k - 1, p: chi2Sf(H, k - 1), epsilon2: H / (N - 1) };

// Dunn par a par, com correção de Bonferroni
const s2 = (N * (N + 1)) / 12 - tieSum / (12 * (N - 1));
const m = (k * (k - 1)) / 2; // número de comparações
const z = (meanRanks.get(a)! - meanRanks.get(b)!) / Math.sqrt(s2 * (1 / na + 1 / nb));
const pAdj = Math.min(1, 2 * (1 - normCdf(Math.abs(z))) * m);
```
