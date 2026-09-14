# Changelog

Todos os tipos de alterações notáveis neste projeto serão documentados neste arquivo.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/), e este projeto adere ao [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- `data/course-recommendations.md` como arquivo de estado para recomendações do Curator
- Opção de pular quiz ou carregar perfil existente na inicialização do Maestro
- Campo `Data da Busca` com timestamp em `job-search-results.md` e `course-recommendations.md`
- `Fetch` como ferramenta de fallback no Curator quando firecrawl falha
- `find_path` como ferramenta disponível no Curator

### Changed
- Renomeado `career-agent-pt-br` para `recoloca-ia` em todos os arquivos e diretórios
- Unificado schema de `job-search-results.md` com campos `skills_matched`, `skills_missing`, `match_count` (PLAN.md + maestro.md)
- Corrigido nome dos agentes em maestro.md: `Searcher,CourseFinder` → `Scout,Curator`
- Unificado nome da área no quiz: `Data Science` → `Ciência de Dados` em todas as referências
- Corrigida classificação de nível de cursos: `Avançado` movido de intermediate para advanced
- Corrigido campo `Area de interesse` → `Área de interesse` (com acento) em todos os arquivos
- Corrigido campo `Habilidades interpessoais` → `Soft skills` em todos os arquivos
- Corrigida palavra em inglês `across` → `em` no README.md
- Otimização de tokens no despacho sequencial do Coach: `persona_reference` apenas no Despacho 1
- Curator agora continua mesmo se busca falhar para uma habilidade específica (fallback `site:alura.com.br`)
