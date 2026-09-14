# Skill Firecrawl

## Propósito

Esta skill fornece capacidades de busca e raspagem web para o sistema Recoloca IA. Ela usa o CLI do Firecrawl para descobrir empregos dos principais portais de emprego e encontrar cursos na Alura.

## Fontes Agregadas

O Firecrawl agrega conteúdo destas plataformas:
- Indeed
- Catho
- LinkedIn
- Glassdoor
- Infojobs

## Ferramentas

- **Ferramenta Zed**: `terminal`
- **CLI**: `firecrawl`

## Comandos

### Busca Web

Busque conteúdo nas fontes agregadas:

```
firecrawl search "[consulta]" --json
```

**Parâmetros**:
- `[consulta]` — a string de consulta de busca

**Retorna**: Array JSON onde cada resultado contém:
- `url` — a URL da página
- `title` — o título da página
- `description` — um breve trecho ou descrição

### Raspagem de Página

Faça scrape de uma única URL e extraia seu conteúdo como markdown:

```
firecrawl scrape <url> --format markdown
```

**Parâmetros**:
- `<url>` — a URL completa para fazer scrape

**Retorna**: Conteúdo da página formatado em markdown

**Fallback**: Se `firecrawl scrape` falhar ou expirar, use o `title` e `description` do resultado de busca como dados de fallback.

## Descoberta de Empregos

Busque vagas de emprego:

```
firecrawl search "vagas [area] [localização]" --json
```

**Parâmetros**:
- `[area]` — a área alvo do usuário (ex: Frontend, Backend, Ciência de Dados)
- `[localização]` — a localização do usuário (ex: São Paulo, Remoto)

**Exemplo**:
```
firecrawl search "vagas Frontend São Paulo" --json
```

## Busca de Cursos

Busque na Alura cursos correspondentes a uma habilidade:

```
firecrawl search "alura [habilidade]" --json
```

**Parâmetros**:
- `[habilidade]` — o nome da habilidade (ex: React, Python, Docker)

**Exemplo**:
```
firecrawl search "alura Docker" --json
```

## Tratamento de Erros

- **Código de saída diferente de zero**: Se qualquer comando firecrawl retornar um código de saída diferente de zero, relate a mensagem de erro exata. Não continue silenciosamente.
- **Timeout**: Se um comando expirar, relate-o como um erro de timeout. Não tente novamente mais de uma vez.
- **Resultados vazios**: Se uma busca não retornar resultados, relate que nenhum resultado foi encontrado. Não fabrique dados.
- **Falha de scrape**: Se `firecrawl scrape` falhar para uma URL específica, use dados de fallback do resultado de busca e note a URL com falha no campo de erros. Continue processando URLs restantes.
- **Campos ausentes**: Se o conteúdo raspado carecer de campos esperados (ex: duração, nível para cursos), marque-os como `não especificado` em vez de inventar valores.

## Regras Gerais

- Nunca invente ou adivinhe dados. Se o firecrawl falhar, relate o erro e pare.
- Sempre passe `--json` para `firecrawl search` para saída estruturada.
- Sempre passe `--format markdown` para `firecrawl scrape` para análise consistente.
- Processe URLs sequencialmente para evitar limitação de taxa.
