# 🧠 GIN - Índice para Dados Complexos

Repositório criado para o estudo de índices GIN no PostgreSQL, aplicados em colunas do tipo `TEXT[]` para otimizar buscas por elementos internos (como *tags*).

---

## Criação da Tabela

```sql
CREATE TABLE artigos (
    id SERIAL PRIMARY KEY,
    titulo TEXT,
    tags TEXT[]
);
```

---

## Criação do Índice GIN

```sql
CREATE INDEX idx_artigos_tags_gin ON artigos USING GIN (tags);
```

---

## Exemplo de Consulta com `EXPLAIN ANALYZE`

```sql
EXPLAIN ANALYZE
SELECT * FROM artigos
WHERE tags @> ARRAY['saúde'];
```

---

## Geração de DataSet com 20 milhões de linhas

```sql
INSERT INTO artigos (titulo, tags)
SELECT 
    'Como ' || acao || ' ' || assunto || ' em ' || contexto AS titulo,
    ARRAY[
        tema_principal,
        'autor_' || (trunc(random() * 1000)::int),         
        'ano_' || (2020 + (random() * 5)::int)              
    ] || ARRAY[
        'topico_' || (trunc(random() * 3000)::int)          
    ] AS tags
FROM (
    SELECT 
        gs,
        acao,
        assunto,
        contexto,
        tema_principal
    FROM generate_series(1, 20000000) AS gs
    CROSS JOIN LATERAL (
        SELECT
            unnest(ARRAY[
                'entender', 'dominar', 'aplicar', 'evitar', 'melhorar'
            ]) AS acao,
            unnest(ARRAY[
                'machine learning', 'produtividade', 'investimentos',
                'ansiedade', 'viagens sustentáveis', 'linguagens de programação',
                'carreira em tech', 'marketing digital', 'nutrição', 'UX design'
            ]) AS assunto,
            unnest(ARRAY[
                '2024', 'pouco tempo', 'cenário atual',
                'trabalho remoto', 'ambientes hostis'
            ]) AS contexto,
            unnest(ARRAY[
                'tecnologia', 'negócios', 'psicologia',
                'viagens', 'saúde', 'design', 'carreira'
            ]) AS tema_principal
        ORDER BY random()
        LIMIT 1
    ) AS dados
) AS final;
```

---
