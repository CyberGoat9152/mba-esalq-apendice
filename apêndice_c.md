A variação percentual compara a média das observações dos últimos 30 dias com a média dos 30 dias precedentes (Equação 2), aplicada de forma uniforme aos escopos global, categoria, subcategoria e produto.

```golang
now := time.Now().UTC().Truncate(24 * time.Hour)
periodStart := now.AddDate(0, 0, -30)        // janela atual: últimos 30 dias
prevStart := periodStart.AddDate(0, 0, -30)  // janela anterior: 30 dias prévios
// variação percentual entre as médias das duas janelas
func pct(curr, prev *float64) (float64, bool) {
  if curr == nil || prev == nil || *prev == 0 {
    return 0, false
  }
  return (*curr - *prev) / *prev * 100.0, true
}
```
