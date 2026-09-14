# Aula 3: Curator — Agente de Busca de Cursos

## Visão Geral

Sistema multi-agente que auxilia usuários em sua jornada de desenvolvimento de carreira, combinando busca de empregos, identificação de lacunas de habilidades, recomendações de cursos e simulação de entrevistas.

**Objetivo da Aula**: Opção B do menu funciona. Agente busca na Alura via Firecrawl, recomenda cursos para lacunas de habilidades.

## Diretrizes para Modelos MoE

Todas as personas e skills são projetadas para modelos Mixture of Experts (MoE). Siga estas regras:

- Sem instruções ambíguas. Cada passo deve especificar exatamente o que fazer, qual ferramenta usar e qual formato de saída produzir.
- Sem tabelas markdown em nenhuma saída. Use listas numeradas com pares chave-valor para dados estruturados.
- Todos os caminhos de arquivo devem ser relativos à raiz do projeto com prefixo explícito `data/`. A raiz do projeto é o diretório `recoloca-ia/`.
- Se uma ferramenta falhar, o agente deve relatar a falha no campo `erros` e não continuar silenciosamente.
- Nunca invente dados. Se um `firecrawl search` ou `firecrawl scrape` falhar, reporte o erro exato e pare. **Exceção**: Nesta aula, se a busca falhar para uma habilidade específica, tente o fallback; se o fallback também falhar, pule essa habilidade e continue com as restantes. Não pare o processamento inteiro por uma única falha de busca de cursos.
- O agente NÃO deve escrever scripts Python, scripts de shell ou qualquer código para implementar personas. O agente personifica cada papel diretamente através do seu comportamento e respostas conversacionais. Personas são instruções comportamentais, não código a ser gerado.

## Arquitetura

```
┌─────────────────────────────────────────────────┐
│                  Usuário                          │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│              MAESTRO (Orquestrador)              │
│  - Interface principal com o usuário             │
│  - Coordena os agentes especializados            │
│  - Consolida resultados e apresenta ao usuário   │
└──┬──────────────┬──────────────┬────────────────┘
   │              │              │
   ▼              ▼              ▼
┌─────────┐  ┌──────────┐  ┌──────────────┐
│ SCOUT   │  │ CURATOR  │  │ COACH        │
│ (Busca  │  │ (Busca   │  │ (Simulação   │
│  de     │  │  de      │  │  de          │
│  Vagas) │  │  Cursos) │  │  Entrevistas)│
└─────────┘  └──────────┘  └──────────────┘
```

**Escopo da Aula 3**: Implementar o bloco Curator e conectá-lo ao Maestro.

## Estrutura de Diretórios (complemento da Aula 3)

```
recoloca-ia/
├── AGENTS.md                   # (já existe da Aula 1)
├── personas/
│   ├── maestro.md          # (já existe da Aula 1)
│   ├── scout.md            # (já existe da Aula 2)
│   └── curator.md          # NOVO: Agente de busca de cursos
├── skills/
│   ├── firecrawl.md        # (já existe da Aula 1)
│   ├── dispatch.md         # (já existe da Aula 1)
│   ├── job-search.md       # (já existe da Aula 2)
│   └── course-analysis.md  # NOVO: Capacidades de análise de cursos
└── data/
    ├── personality-quiz.md       # (já existe da Aula 1)
    ├── user-profile.md           # (já existe da Aula 1)
    ├── job-search-results.md     # (já existe da Aula 2)
    └── course-recommendations.md # NOVO: Recomendações de cursos do Curator
```

## Curator - Agente de Busca de Cursos

**Responsabilidade**: Buscar na Alura cursos que atendam às lacunas de habilidades identificadas pelo Scout.

**Skills**:
- `skills/course-analysis.md` — Fluxo completo: validação de pré-requisitos, extração de habilidades faltantes, comandos firecrawl, classificação de nível, ordenação, formato de resposta e tratamento de erros. **OBRIGATÓRIO CARREGAR.**
- `skills/firecrawl.md` — Comandos e regras do CLI Firecrawl.

**Ferramentas do Zed**:
- `terminal` — executar comandos `firecrawl search` e `firecrawl scrape`
- `find_path` — verificar se `data/job-search-results.md` existe (validação de pré-requisito: se ausente, reporta erro e pede busca de vagas primeiro)

**Ferramentas de acesso web**:
- Sempre use `firecrawl search` via `terminal` como seu método primário de acesso à web
- Se `firecrawl search` falhar para uma habilidade específica, tente o fallback: `firecrawl search "site:alura.com.br [habilidade]" --json`
- Se o firecrawl falhar consistentemente, você PODE usar `curl` ou `wget` como fallback para obter conteúdo bruto da página — note as limitações (sem renderização JS, possíveis bloqueios anti-bot, HTML bruto em vez de markdown limpo)
- Evite `Fetch`, `webfetch` ou outras ferramentas HTTP — prefira firecrawl primeiro, depois curl/wget apenas como recuperação

**Fonte de Dados**:
- Alura (alura.com.br/formacoes)

**Entradas**:
- Lista de habilidades em falta de `data/job-search-results.md`
- Área de interesse do usuário de `data/user-profile.md`

**Saídas**:
- Retornar resultados ao Maestro via Envelope de Resposta (Maestro salva em `data/course-recommendations.md`)
- Lista de até 5 cursos recomendados com nome, duração, nível e link
- Para cada curso, qual habilidade em falta ele aborda
- Uma ordem sugerida para realizá-los (iniciante primeiro, depois intermediário, depois avançado)

## Skills para Aula 3

### course-analysis.md

- **Ferramenta Zed**: `terminal` — executar comandos `firecrawl search`
  - Comando: `firecrawl search "alura [habilidade]" --json`
  - Fallback se busca falhar: `firecrawl search "site:alura.com.br [habilidade]" --json`
  - Priorize resultados do domínio `alura.com.br/formacoes`. Não navegue diretamente para a URL — use a busca para descobrir cursos relevantes.
- Se a busca falhar para uma habilidade específica, tente o fallback. Se o fallback também falhar, pule essa habilidade e continue com as restantes. Não pare o processamento inteiro por uma única falha.
- Para cada habilidade em falta dos resultados do Scout, buscar na Alura cursos correspondentes
- Extrair: nome do curso, duração, nível, link
- Classificação de nível: `iniciante` (título contém "Introdução", "Primeiros Passos", "Fundamentos", "Básico" ou "Para Iniciantes"), `intermediario` (título contém "Intermediário" ou implica conhecimento prévio), `avancado` (título contém "Avançado", "Profundo", "Expert", "Arquitetura" ou "Especialista")
- Ordenar resultados: cursos iniciantes primeiro, depois intermediário, depois avançado
- Retornar até 5 cursos

## Protocolo de Handoff do Curator

**Contexto passado:** lista de habilidades em falta de `data/job-search-results.md`. Se o arquivo não existir ou estiver vazio, o Curator reporta um erro e pede ao usuário para buscar vagas primeiro.

**Envelope de Despacho** (construído pelo Maestro):

```
## DESPACHO: CURATOR
### referencia_persona
[Conteúdo completo de personas/curator.md]

### tarefa
Buscar cursos na Alura para preencher lacunas de habilidades

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
Habilidades faltantes: [lista_de_habilidades_faltantes de job-search-results.md]
Area de interesse: [area]

### saida_esperada
Envelope de resposta com estado, resumo, dados (lista de cursos com ordem sugerida) e erros se houver
```

**Formato de dados da resposta:**
```
1. nome_curso: [título do curso]
   duracao: [ex: 20 horas]
   nivel: [iniciante | intermediario | avancado]
   aborda_habilidade: [nome da habilidade]
   link: [URL]

2. [próximo curso no mesmo formato]

Ordem sugerida:
1. [nome do curso]
2. [nome do curso]
3. [nome do curso]
```

## Esquema do Arquivo de Dados

### data/course-recommendations.md

Armazena as recomendações mais recentes do Curator:

```
Data da Busca: [AAAA-MM-DD HH:MM]

Recomendações:
1. nome_curso: [título do curso]
   duracao: [ex: 20 horas]
   nivel: [iniciante | intermediario | avancado]
   aborda_habilidade: [nome da habilidade]
   link: [URL]

2. [próximo curso no mesmo formato]

Ordem sugerida:
1. [nome do curso]
2. [nome do curso]
3. [nome do curso]
```

## Tasks da Aula 3

### 1. Escrever `skills/course-analysis.md`

Contendo:
- Comandos firecrawl para buscar cursos na Alura
- Como validar pré-requisitos e extrair habilidades faltantes
- Como classificar nível dos cursos (iniciante/intermediário/avançado)
- Regras de ordenação (iniciante primeiro)
- Formato de resposta esperado
- Tratamento de erros (pular habilidade que falha, continuar com as demais)

### 2. Escrever `personas/curator.md`

Contendo:
- Papel e responsabilidade do Curator
- Ferramentas disponíveis (terminal com firecrawl, find_path para validação de pré-requisito)
- Usa `find_path` para verificar se `data/job-search-results.md` existe; se ausente, reporta erro
- Referência obrigatória à skill `skills/course-analysis.md`
- Referência à skill `skills/firecrawl.md`
- Formato do Response Envelope
- Regras de tratamento de erros

### 3. Conectar `spawn_agent` no Maestro para despachar o Curator

- Quando o usuário digitar "B", o Maestro deve construir o envelope de despacho
- Chamar `spawn_agent` com o prompt de despacho
- Analisar a resposta do Curator
- Salvar recomendações em `data/course-recommendations.md`
- Exibir cursos ao usuário
- Retornar ao menu

### 4. Testar: busca de cursos funcional

- Usuário busca vagas primeiro (A) para gerar habilidades faltantes
- Usuário seleciona B
- Curator lê `data/job-search-results.md` para obter habilidades faltantes
- Curator busca na Alura via Firecrawl
- Recomendações de cursos exibidas com ordem sugerida

### 5. Testar: tratamento de erro

- Se a busca falhar, o erro é reportado ao usuário
- Agente retorna ao menu
- Se `data/job-search-results.md` não existir, Curator pede ao usuário para buscar vagas primeiro

## Fluxo da Aula 3

```
Usuário seleciona B no menu
        │
        ▼
Maestro verifica se data/job-search-results.md existe
        │
        ├─ Não existe → reporta erro e pede para buscar vagas primeiro
        │
        ▼
Maestro extrai habilidades faltantes dos resultados do Scout
        │
        ▼
Maestro constrói envelope de despacho para Curator
        │
        ▼
spawn_agent dispara Curator com persona + contexto
        │
        ▼
Curator executa firecrawl search "alura [habilidade]" para cada habilidade faltante
        │
        ▼
Curator classifica cursos por nível → ordena iniciante primeiro
        │
        ▼
Curator retorna envelope de resposta com até 5 cursos e ordem sugerida
        │
        ▼
Maestro salva em data/course-recommendations.md → exibe ao usuário → retorna ao menu
```

## Regras de Erro

- Se `firecrawl search` falhar para uma habilidade específica, tente o fallback. Se o fallback também falhar, pule essa habilidade e continue com as restantes. Não pare o processamento inteiro por uma única falha.
- Se `spawn_agent` falhar, dizer ao usuário o que deu errado e retornar ao menu.
- Se `data/job-search-results.md` não existir ou estiver vazio, o Curator reporta um erro e pede ao usuário para buscar vagas primeiro.

## Entregável da Aula 3

Curator totalmente funcional, busca de cursos end-to-end a partir do menu.
