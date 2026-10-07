O classificador usa saída estruturada (JSON Schema com enumerações) para
restringir a resposta do modelo aos valores válidos da taxonomia; o par retornado é
mapeado de volta ao identificador da subcategoria (Python).
```python
# saída estruturada: o modelo só pode responder valores da taxonomia
schema = {
    "type": "object",
    "properties": {
        "categoria":    {"type": "string", "enum": categories},
        "subcategoria": {"type": "string", "enum": subcategories},
    },
    "required": ["categoria", "subcategoria"],
}
resp = self._client.post(f"{self._base_url}/api/chat", json={
    "model": self._model,
    "messages": [
        {"role": "system", "content": _SYSTEM},
        {"role": "user", "content": prompt}
    ],
    "stream": False,
    "format": schema,
    "options": {"temperature": 0},  # determinístico 
    })
    answer = json.loads(resp.json()["message"]["content"])
    # mapeia o par (categoria, subcategoria) de volta ao identificador
    # mapeia o par (categoria, subcategoria) de volta ao identificador
    sub_id = by_pair.get((answer["categoria"], answer["subcategoria"]))
    if sub_id is None:     
    # par inconsistente: casa só pela subcategoria, se o nome for único     
    matches = [s.id for s in taxonomy if s.name == answer["subcategoria"]]     
    sub_id = matches[0] if len(matches) == 1 else None
```
