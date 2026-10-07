O mecanismo calcula o viés de cada conta frente ao consenso das demais contas
confiáveis (Equação 9), retira a de maior viés acima do limiar e repete o processo até que
nenhuma conta o exceda (linguagem Python). 


```python
def reputacao(d):
    # mediana do preço de cada conta para cada produto
    pu = d.groupby(["product_id", "user"]).price.median().reset_index()
    grupos = [(g.user.to_numpy(), np.log(g.price.to_numpy()))
        for _, g in pu.groupby("product_id") if len(g) >= 3]
    confiaveis, sinalizadas = set(pu.user), set()
    while True:
        devs = {}
        for users, lv in grupos:
            ok = np.array([u in confiaveis for u in users])
            for i, u in enumerate(users):
                outros = ok.copy()
                outros[i] = False
                if u not in confiaveis or outros.sum() < 2:
                    continue  # consenso exige ao menos duas outras contas
                devs.setdefault(u, []).append(lv[i] - np.median(lv[outros]))
        # viés com sinal: honestos divergem para os dois lados, fraudes não
        pior, pior_s = None, np.log(1.10)
        for u, v in devs.items(): 
            if len(v) >= 5 and abs(np.median(v)) > pior_s:
                pior, pior_s = u, abs(np.median(v))
            if pior is None:
                return sinalizadas
        sinalizadas.add(pior)       # retira a pior conta e recalcula o consenso
        confiaveis.discard(pior)
```
