O índice segue a estrutura do IPCA: dentro de cada subcategoria, o relativo é a médiageométrica dos
relativos de preço dos produtos observados nos dois meses (Equação 3); entre subcategorias, a média simples,
encadeada em base 100 (Equação 4). Relativos fora de [0,5; 2] são descartados como erro de dado (linguagem TypeScript).

```golang
const REL_MIN = 0.5;
const REL_MAX = 2;

export function chainedIndex(rows: ProductMonthPrice[], months: string[]): IndexPoint[] {
  const byProduct = new Map<number, { sub: string; price: Map<string, number> }>();
  for (const r of rows) {
    let p = byProduct.get(r.product_id);
    if (!p) byProduct.set(r.product_id, (p = { sub: r.subcategory ?? "—", price: new Map() }));
    p.price.set(r.month, r.price);
  }

  const out: IndexPoint[] = [];
  let level = 100; // base 100 no primeiro mês

  months.forEach((m, i) => {
    if (i === 0) {
      out.push({ month: m, index: level, pct: null, products: 0, subcats: 0 });
      return;
    }
    const prev = months[i - 1];

    // agregado elementar = subcategoria: log dos relativos dos produtos vistos nos dois meses
    const logs = new Map<string, number[]>();
    for (const p of byProduct.values()) {
      const a = p.price.get(prev);
      const b = p.price.get(m);
      if (!a || !b) continue;
      const rel = b / a;
      if (rel < REL_MIN || rel > REL_MAX) continue; // erro de digitação ou troca de embalagem
      let l = logs.get(p.sub);
      if (!l) logs.set(p.sub, (l = []));
      l.push(Math.log(rel));
    }

    // Jevons dentro da subcategoria (Equação 3); média simples entre subcategorias (Equação 4)
    let sum = 0;
    for (const l of logs.values()) sum += Math.exp(l.reduce((s, v) => s + v, 0) / l.length);
    const rel = logs.size ? sum / logs.size : 1;
    level *= rel;
    out.push({ month: m, index: level, pct: (rel - 1) * 100, products: 0, subcats: logs.size });
  });

  return out;
}
```
