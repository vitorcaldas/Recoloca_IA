# Recoloca IA - Framework de Orquestração

## Visão Geral

Sistema multi-agente que auxilia usuários em sua jornada de desenvolvimento de carreira, combinando busca de empregos, identificação de lacunas de habilidades, recomendações de cursos e simulação de entrevistas.

## Diretrizes para Modelos MoE

Todas as personas e skills são projetadas para modelos Mixture of Experts (MoE). Siga estas regras:

- Sem instruções ambíguas. Cada passo deve especificar exatamente o que fazer, qual ferramenta usar e qual formato de saída produzir.
- Sem tabelas markdown em nenhuma saída. Use listas numeradas com pares chave-valor para dados estruturados.
- Todos os caminhos de arquivo devem ser relativos à raiz do projeto com prefixo explícito `data/`. A raiz do projeto é o diretório `recoloca-ia/`.
- Se uma ferramenta falhar, o agente deve relatar a falha no campo `erros` e não continuar silenciosamente.
- Nunca invente dados. Se um `firecrawl search` ou `firecrawl scrape` falhar, reporte o erro exato e pare. **Exceção**: Em `skills/course-analysis.md`, se a busca falhar para uma habilidade específica, tente o fallback; se o fallback também falhar, pule essa habilidade e continue com as restantes. Não pare o processamento inteiro por uma única falha de busca de cursos.
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

Agentes principais: Maestro, Scout, Curator, Coach. Cada um é construído e completado em sua própria aula.

## Estrutura de Diretórios

```
recoloca-ia/
├── AGENTS.md                   # Instruções de inicialização para o agente
├── personas/
│   ├── maestro.md          # Orquestrador principal
│   ├── scout.md            # Agente de busca de vagas
│   ├── curator.md          # Agente de busca de cursos
│   └── coach.md            # Simulador de entrevistas
├── skills/
│   ├── firecrawl.md        # CLI do Firecrawl e ferramentas web (de firecrawl.dev)
│   ├── dispatch.md         # Despacho de agentes e protocolo de handoff
│   ├── job-search.md       # Capacidades de busca de vagas
│   ├── course-analysis.md  # Capacidades de análise de cursos
│   └── interview-sim.md    # Capacidades de simulação de entrevistas
├── data/
│   ├── personality-quiz.md       # Quiz de personalidade do usuário
│   ├── user-profile.md           # Perfil consolidado do usuário
│   ├── job-search-results.md     # Últimos resultados do Scout
│   ├── course-recommendations.md # Recomendações de cursos do Curator
│   └── interview-session.md      # Rastreamento de estado da entrevista do Coach
└── README.md                 # Descrição do projeto e instruções de uso (ver esquema no final desta seção)
```

**README.md conteúdo esperado:** Breve descrição do projeto Recoloca IA, objetivo do sistema multi-agente, como iniciar (pré-requisitos: Zed, OpenRouter API key, Firecrawl CLI), e estrutura de diretórios resumida.

## Agentes (Personas)

### 1. Maestro - Orquestrador Principal

**Responsabilidade**: Interface principal com o usuário. Sauda o usuário, verifica se o quiz está completo, apresenta um menu e delega tarefas aos sub-agentes.

**Skills**:
- `skills/dispatch.md` — Protocolo de despacho e handoff de agentes (DEVE ser carregado como parte do playbook). Contém tabela de roteamento, formatos de envelope de despacho/resposta, especificações de handoff por agente, fluxo de despacho sequencial do Coach e regras de tratamento de erros.

**Ferramentas do Zed**:
- `spawn_agent` — despachar sub-agentes com prompts estruturados
- `terminal` — executar comandos shell (ex: `firecrawl search`, `firecrawl scrape`)
- `find_path` — verificar se `personality-quiz.md` existe

**Arquivos de Estado**:
- `data/personality-quiz.md` — respostas do quiz do usuário
- `data/user-profile.md` — perfil consolidado derivado do quiz

**Fluxo de Inicialização**:
1. Saudar o usuário
2. Verificar se `data/personality-quiz.md` existe e tem `Concluído: true`
3. Se o arquivo existir e estiver completo, carregar perfil existente. Se existir mas estiver incompleto, perguntar ao usuário se deseja continuar de onde parou ou recomeçar o quiz. Se o quiz estiver completo, gerar `data/user-profile.md` a partir das respostas
4. Apresentar o menu

**Perguntas do Quiz** (feitas uma de cada vez em ordem):
1. "Qual área mais te anima? Aqui estão suas opções: Frontend, Backend, Ciência de Dados, Mobile, DevOps, Full Stack, Governança de Dados, Design UX, Design UI, Liderança, RH, Marketing de Mídias Sociais, Growth Marketing, Gestão de Produtos ou Cibersegurança"
2. "Como você descreveria seu nível de experiência atual? Escolha um: Júnior, Pleno ou Sênior"
3. "Como você prefere trabalhar? Opções: Remoto, Híbrido ou Presencial"
4. "Onde você está localizado? Me diga sua cidade e estado, ou apenas diga 'Remoto'"
5. "Quais são suas soft skills mais fortes? Pense em coisas como comunicação, trabalho em equipe, liderança, resolução de problemas — o que vier naturalmente para você"
6. "Onde você se vê em sua carreira? Opções: Crescimento técnico, Transição de carreira, Primeiro emprego ou Trilha de liderança"
7. "Quais habilidades técnicas você já tem? Apenas liste separadas por vírgulas — por exemplo: Python, SQL, Excel, Figma, Git"

**Menu** (mostrado quando o quiz está completo):
- **A** — Buscar vagas (via Firecrawl: Indeed, Catho, LinkedIn, Glassdoor, Infojobs)
- **B** — Encontrar cursos para preencher lacunas de habilidades (Alura)
- **C** — Praticar com uma entrevista simulada
- **D** — Refazer o quiz (sobrescreve `data/personality-quiz.md` completamente e regenera `data/user-profile.md`)

**Fluxo Completo de Interação**:
1. Saudar o usuário e verificar status do quiz
2. Se o quiz não estiver feito, guiar pelo quiz. Se estiver feito, apresentar o menu.
3. Receber a seleção do usuário (A/B/C/D)
4. Delegar ao agente correto via `spawn_agent` (Coach é despachado 6 vezes para entrevista completa)
5. Exibir a resposta do agente ao usuário
6. Mostrar o menu novamente

### 2. Scout - Buscador de Vagas

**Responsabilidade**: Buscar vagas de emprego via Firecrawl, que agrega resultados do Indeed, Catho, LinkedIn, Glassdoor, Infojobs e outras fontes.

**Skills**:
- `skills/job-search.md` — Fluxo completo: comandos firecrawl, leitura de perfil, extração de dados, correspondência de habilidades, filtragem por nível, formato de resposta e tratamento de erros. **OBRIGATÓRIO CARREGAR.**
- `skills/firecrawl.md` — Comandos e regras do CLI Firecrawl.

**Ferramentas do Zed**:
- `terminal` — executar comandos `firecrawl search` e `firecrawl scrape`

**Ferramentas de acesso web**:
- Sempre use `firecrawl search` via `terminal` como seu método primário de acesso à web
- Se o firecrawl falhar consistentemente, você PODE usar `curl` ou `wget` como fallback — note as limitações (sem renderização JS, possíveis bloqueios anti-bot, HTML bruto em vez de markdown limpo)
- Evite `Fetch`, `webfetch` ou outras ferramentas HTTP — prefira firecrawl primeiro, depois curl/wget apenas como recuperação

**Fontes de Dados**:
- Busca Firecrawl (agrega Indeed, Catho, LinkedIn, Glassdoor, Infojobs)

**Entradas**:
- Área de interesse, Localização, Nível de experiência, Habilidades atuais (de `data/user-profile.md`)

**Saídas**:
- Retornar resultados ao Maestro via Response Envelope (Maestro salva em `data/job-search-results.md`)
- Exibir lista de até 5 vagas com título, empresa, localização, link, habilidades correspondentes/em falta e contagem de correspondência

### 3. Curator - Buscador de Cursos

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

### 4. Coach - Simulador de Entrevistas

**Responsabilidade**: Conduzir uma entrevista simulada baseada em uma vaga que o usuário selecionou dos resultados do Scout.

**Skills**:
- `skills/interview-sim.md` — Fluxo completo: tipos de perguntas, calibração por nível de experiência, diretrizes de feedback, rubrica de pontuação, formato de resposta por despacho e tratamento de erros. **OBRIGATÓRIO CARREGAR.**

**Ferramentas do Zed**:
- `terminal` — acesso ao shell para consultas de fallback via `curl`/`wget` se necessário. Não é a ferramenta primária; os dados de perfil e contexto da vaga são passados via envelope de despacho.

**Entradas**:
- Título e descrição de uma vaga dos resultados do Scout
- Área de interesse e habilidades atuais do usuário

**Saídas**:
- Fazer 5 perguntas uma de cada vez (mistura de comportamentais e técnicas)
- Após cada resposta do usuário, dar feedback breve
- No final, dar uma pontuação geral de 1 a 10 e 2-3 áreas para melhorar

## Esquemas dos Arquivos de Dados

### data/personality-quiz.md

```
Área de interesse: [valor]
Nível de experiência: [valor]
Preferências de trabalho: [valor]
Localização: [valor]
Soft skills: [valor]
Objetivo de carreira: [valor]
Habilidades atuais: [valor]
Concluído: [true | false]
```

Um quiz está completo apenas quando todas as seções estão preenchidas e `Concluído` é `true`.

### data/user-profile.md

Gerado a partir de `personality-quiz.md`. Mesmos campos, mais `Funções alvo`:

```
Área de interesse: [valor]
Nível de experiência: [valor]
Preferências de trabalho: [valor]
Localização: [valor]
Soft skills: [valor]
Objetivo de carreira: [valor]
Habilidades atuais: [valor]
Funções alvo: [lista separada por vírgulas]
Concluído: [true | false]
```

**Mapeamento de funções alvo (hardcoded no maestro.md):**
1. Frontend + Júnior: Desenvolvedor Frontend, Desenvolvedor UI Júnior, Desenvolvedor Web
2. Frontend + Pleno: Engenheiro Frontend, Desenvolvedor UI, Desenvolvedor React
3. Frontend + Sênior: Engenheiro Frontend Sênior, Líder de Desenvolvimento UI, Arquiteto Frontend
4. Backend + Júnior: Desenvolvedor Backend, Desenvolvedor API Júnior, Desenvolvedor de Software
5. Backend + Pleno: Engenheiro Backend, Desenvolvedor API, Desenvolvedor Python/Java
6. Backend + Sênior: Engenheiro Backend Sênior, Arquiteto de Sistemas, Líder Técnico
7. Ciência de Dados + Júnior: Analista de Dados, Cientista de Dados Júnior, Analista BI
8. Ciência de Dados + Pleno: Cientista de Dados, Engenheiro de Machine Learning, Engenheiro de Dados
9. Ciência de Dados + Sênior: Cientista de Dados Sênior, Arquiteto ML, Líder IA
10. Mobile + Júnior: Desenvolvedor Mobile, Desenvolvedor iOS/Android Júnior
11. Mobile + Pleno: Desenvolvedor iOS, Desenvolvedor Android, Desenvolvedor React Native
12. Mobile + Sênior: Engenheiro Mobile Sênior, Arquiteto Mobile, Líder Flutter
13. DevOps + Júnior: Engenheiro DevOps Júnior, Suporte Cloud, SysAdmin
14. DevOps + Pleno: Engenheiro DevOps, Engenheiro Cloud, SRE
15. DevOps + Sênior: Engenheiro DevOps Sênior, Arquiteto Cloud, Líder de Plataforma
16. Full Stack + Júnior: Desenvolvedor Full Stack, Desenvolvedor Web Júnior
17. Full Stack + Pleno: Engenheiro Full Stack, Desenvolvedor de Aplicações Web
18. Full Stack + Sênior: Engenheiro Full Stack Sênior, Líder Técnico, Arquiteto de Soluções
19. Governança de Dados + Júnior: Analista de Governança de Dados Júnior, Gestor de Dados Júnior, Assistente de Compliance
20. Governança de Dados + Pleno: Analista de Governança de Dados, DPO, Analista de Qualidade de Dados
21. Governança de Dados + Sênior: Head de Governança de Dados, Diretor Chefe de Dados, Líder de Arquitetura de Dados
22. Design UX + Júnior: Designer UX Júnior, Assistente UI/UX, Pesquisador UX Jr
23. Design UX + Pleno: Designer UX, Pesquisador UX, Designer de Produto
24. Design UX + Sênior: Designer UX Sênior, Líder UX, Head de UX
25. Design UI + Júnior: Designer UI Júnior, Designer Visual Jr, Assistente de Design System
26. Design UI + Pleno: Designer UI, Designer Visual, Designer de Interação
27. Design UI + Sênior: Designer UI Sênior, Líder UI, Arquiteto de Design System
28. Liderança + Júnior: Líder de Equipe Júnior, Coordenador de Projetos, Scrum Master Jr
29. Liderança + Pleno: Gerente de Engenharia, Gerente de Projetos, Agile Coach
30. Liderança + Sênior: Diretor de Engenharia, VP de Tecnologia, CTO
31. RH + Júnior: Analista de RH Júnior, Assistente de Aquisição de Talentos, Coordenador de RH
32. RH + Pleno: Analista de RH, Recrutador, Especialista em Operações de Pessoas
33. RH + Sênior: Gerente de RH, Head de Pessoas, Diretor de Talentos
34. Marketing de Mídias Sociais + Júnior: Assistente de Mídias Sociais, Criador de Conteúdo Jr, Community Manager Jr
35. Marketing de Mídias Sociais + Pleno: Gerente de Mídias Sociais, Estrategista de Conteúdo, Analista de Marketing Digital
36. Marketing de Mídias Sociais + Sênior: Head de Mídias Sociais, Diretor de Mídias Sociais, Líder Estrategista de Marca
37. Growth Marketing + Júnior: Assistente de Growth Marketing, Analista de Marketing Jr, Marketing de Performance Jr
38. Growth Marketing + Pleno: Growth Marketer, Gerente de Marketing de Performance, Especialista CRO
39. Growth Marketing + Sênior: Head de Growth, Diretor de Growth, VP de Marketing
40. Gestão de Produtos + Júnior: Analista de Produto, Gerente de Produto Associado, Product Owner Jr
41. Gestão de Produtos + Pleno: Gerente de Produto, Product Owner, Gerente de Produto Técnico
42. Gestão de Produtos + Sênior: Gerente de Produto Sênior, Head de Produto, VP de Produto
43. Cibersegurança + Júnior: Analista de Segurança Júnior, Analista SOC, Assistente de Segurança da Informação
44. Cibersegurança + Pleno: Engenheiro de Segurança, Testador de Penetração, Consultor de Segurança
45. Cibersegurança + Sênior: Engenheiro de Segurança Sênior, CISO, Líder Arquiteto de Segurança

### data/job-search-results.md

Armazena os resultados mais recentes do Scout para uso pelo Curator e Coach:

```
Data da Busca: [AAAA-MM-DD HH:MM]
Parâmetros de Busca:
  Área: [valor]
  Localização: [valor]

Resultados:
1. titulo: [título da vaga]
   empresa: [nome da empresa]
   localizacao: [cidade ou Remoto]
   link: [URL]
   habilidades_correspondentes: [habilidade1, habilidade2]
   habilidades_faltantes: [habilidade3, habilidade4]
   contagem_correspondencia: [X de Y habilidades correspondem]

2. titulo: [próximo título de vaga]
   ...
```

### data/interview-session.md

Rastreia o estado da entrevista do Coach através de despachos sequenciais:

```
Contexto da Vaga: [título da vaga e empresa]
Número da Pergunta: [1-5]
Histórico de Perguntas e Respostas:
  P1: [texto da pergunta]
  R1: [texto da resposta do usuário]
  P2: [texto da pergunta]
  R2: [texto da resposta do usuário]
  ...
  P5: [texto da pergunta]
  R5: [texto da resposta do usuário]

Pontuação Final: [X/10]
Áreas de Melhoria:
1. [área de melhoria 1]
2. [área de melhoria 2]
```

O Maestro lê este arquivo para rastrear progresso e passa o histórico Q&A anterior ao Coach em cada despacho. O Maestro é responsável por escrever/atualizar este arquivo com os resultados de cada despacho do Coach.

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

## Fluxo Principal

```
1. Usuário abre o agente
        │
        ▼
2. Maestro saúda e verifica o quiz
        │
        ├─ Quiz ausente/incompleto → perguntar se deseja continuar ou recomeçar → se continuar, completar quiz; se recomeçar, guiar pelo quiz → salvar user-profile.md
        ├─ Quiz existente completo → carregar user-profile.md diretamente
        ▼
3. Maestro apresenta menu: A (vagas), B (cursos), C (entrevista), D (refazer quiz)
        │
        ├─ A → despachar Scout → salvar resultados em data/job-search-results.md → exibir vagas → retornar ao menu
        ├─ B → despachar Curator → exibir cursos → retornar ao menu
        ├─ C → despachar Coach 6x sequencialmente → fazer perguntas → dar feedback → pontuação final → retornar ao menu
        └─ D → guiar usuário pelo quiz novamente → salvar user-profile.md → retornar ao menu
```

**Regras de erro:**
- Se `firecrawl search` falhar, reportar o erro ao usuário e retornar ao menu.
- Se `firecrawl scrape` falhar em uma URL específica, usar o título/descrição do resultado da busca e anotar que a extração falhou.
- Se `spawn_agent` falhar, dizer ao usuário o que deu errado e retornar ao menu.
- Se a busca retornar resultados mas algumas extrações falharem, mostrar resultados parciais com detalhes completos onde disponíveis.

## Skills

### dispatch.md
- Arquivo separado em `skills/dispatch.md` — protocolo completo de despacho e handoff de agentes
- Determinar qual agente chamar com base na escolha do menu do usuário (A=Scout, B=Curator, C=Coach, D=Maestro lida com o quiz)
- Construir o prompt de despacho usando o formato DESPACHO abaixo
- Chamar `spawn_agent` com o prompt de despacho
- Analisar o formato RESPOSTA retornado pelo agente despachado
- Exibir o resumo e dados do agente ao usuário

### job-search.md
- **Ferramenta Zed**: `terminal` — executar comandos CLI do `firecrawl`
  - **Descoberta de vagas**: `firecrawl search "vagas [area_de_interesse] [localizacao]" --json`
    - Retorna JSON com: url, titulo, descricao, cargo para cada resultado
  - **Detalhes completos da vaga**: `firecrawl scrape <url> --format markdown`
    - Usar em URLs individuais de vagas dos resultados da busca para obter descrição e requisitos completos
    - Se a extração falhar ou expirar, usar o título e descrição do resultado da busca
- Se a busca não retornar resultados, reportar ao usuário e sugerir ampliar os termos de busca
- Para cada vaga encontrada, extrair: título, empresa (da URL ou título), localização, link, habilidades requeridas
- Comparar habilidades requeridas com as habilidades atuais do usuário usando correspondência de strings sem distinção de maiúsculas/minúsculas
- Se a vaga menciona um nível de experiência, preferir vagas que correspondam ao nível do usuário (Júnior, Pleno, Sênior). Se não houver vagas do nível correspondente nos primeiros resultados, expandir a busca e incluir vagas de nível adjacente anotando a discrepância.
- Contar correspondências e listar habilidades em falta
- Retornar até 5 vagas

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

### interview-sim.md
- **OBRIGATÓRIO CARREGAR.** Fluxo completo: geração de perguntas, calibração por nível, feedback, rubrica de pontuação, formato de resposta por despacho e tratamento de erros.
- Gerar 5 perguntas baseadas na descrição da vaga e perfil do usuário. Fazer uma de cada vez e aguardar a resposta do usuário
- Perguntas devem misturar comportamentais ("conte-me sobre uma vez quando...") e técnicas (específicas da função)
- Calibração por nível de experiência: perguntas mais simples para Júnior, mais profundas para Sênior
- Após cada resposta, dar 2-3 frases de feedback construtivo
- Após todas as 5 perguntas, dar uma pontuação geral de 1 a 10 e listar 2-3 áreas para melhorar
- Tratamento de erros: Se o contexto da vaga não estiver disponível, gerar perguntas gerais baseadas no perfil do usuário e informar ao Maestro. Se o despacho falhar, reportar no campo `erros` do envelope de resposta. Se o usuário desistir no meio, gerar pontuação parcial com base nas respostas já recebidas.

## Protocolo de Handoff entre Agentes

### Envelope de Despacho (Maestro constrói este prompt para `spawn_agent`)

```
## DESPACHO: [NOME_DO_AGENTE]
### referencia_persona
[Conteúdo completo de personas/<nome_do_agente_minusculo>.md — o agente despachado DEVE ler e adotar esta persona antes de executar a tarefa]

### tarefa
[Uma frase descrevendo o que o agente deve fazer]

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
[Contexto específico: ex: qual vaga para entrevistar, quais habilidades buscar cursos]

### saida_esperada
[Exatamente em que formato o agente deve retornar]
```

**Crítico**: A seção `referencia_persona` deve conter o conteúdo completo do arquivo de persona do agente despachado (`personas/scout.md`, `personas/curator.md` ou `personas/coach.md`). Isso garante que o agente despachado obedeça a todas as regras comportamentais, proibições de ferramentas e fluxos de trabalho definidos em sua persona. **Exceção**: No despacho sequencial do Coach, `referencia_persona` é incluída apenas no Despacho 1. Nos despachos 2-6, omita `referencia_persona` para economizar tokens.

### Envelope de Resposta (agente despachado retorna isto)

```
## RESPOSTA: [NOME_DO_AGENTE]
### estado
[sucesso | erro]

### resumo
[Resumo legível de 2-3 frases para o usuário]

### dados
[Resultados como listas numeradas com pares chave-valor. Sem tabelas markdown.]

### erros
[Apenas se estado for erro: o que deu errado]
```

### Handoff do Scout

**Contexto passado:** área de interesse e localização do usuário do user-profile.md

**Formato de dados da resposta:**
```
1. titulo: [título da vaga]
   empresa: [nome da empresa]
   localizacao: [cidade ou Remoto]
   link: [URL]
   habilidades_correspondentes: [habilidade1, habilidade2]
   habilidades_faltantes: [habilidade3, habilidade4]
   contagem_correspondencia: [X de Y habilidades correspondem]

2. [próxima vaga no mesmo formato]
```

### Handoff do Curator

**Contexto passado:** lista de habilidades em falta de `data/job-search-results.md`. Se o arquivo não existir ou estiver vazio, o Curator reporta um erro e pede ao usuário para buscar vagas primeiro.

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

### Handoff do Coach

**Contexto passado:** título e descrição da vaga selecionada de `data/job-search-results.md` (ou perfil do usuário se nenhuma busca de vagas foi realizada)

**Otimização de tokens**: A seção `referencia_persona` com o conteúdo completo de `personas/coach.md` deve ser incluída apenas no Despacho 1. Nos despachos 2-6, omita `referencia_persona` e envie apenas `tarefa`, `perfil_usuario`, `contexto` e `saida_esperada`. O Coach já conhece sua persona a partir do primeiro despacho.

**Modelo de despacho sequencial:** Maestro despacha o Coach 6 vezes no total para completar uma entrevista de 5 perguntas:
1. Despacho 1: Gerar e retornar P1
2. Despacho 2: Avaliar R1, retornar feedback + P2
3. Despacho 3: Avaliar R2, retornar feedback + P3
4. Despacho 4: Avaliar R3, retornar feedback + P4
5. Despacho 5: Avaliar R4, retornar feedback + P5
6. Despacho 6: Avaliar R5, retornar pontuação final e áreas para melhorar

**Handoff por pergunta (enviado ao Coach em cada despacho):**
- Número da pergunta (1-5)
- Perguntas anteriores e respostas do usuário (se houver)
- Perfil do usuário e contexto da vaga

**Resposta do Coach por despacho (geração de perguntas 1-5):**
```
### estado: sucesso
### pergunta_atual: [a pergunta a fazer ao usuário]
### feedback_anterior: [feedback sobre a resposta anterior, ou vazio para a pergunta 1]
```

**Resposta do despacho final (despacho 6, após o usuário responder a pergunta 5):**
```
### estado: sucesso
### Entrevista Concluída
Pontuação: [X/10]

### Áreas de melhoria
1. [item 1]
2. [item 2]
```

**Fluxo:**
1. Maestro despacha Coach com contexto P1 → Coach gera e retorna P1 → exibe pergunta → aguarda resposta do usuário
2. Maestro despacha Coach com R1 do usuário → Coach avalia R1, retorna feedback + P2 → exibe feedback + pergunta → aguarda resposta
3. Repetir até P5 ser feita
4. Após o usuário responder P5, Coach avalia e retorna pontuação final e áreas para melhorar
5. Maestro exibe resultados e retorna ao menu

## Plano de Aulas (4 × 30 minutos)

### Aula 1: Setup + Plano + Maestro
**Objetivo**: Estudantes têm Zed + OpenRouter + Firecrawl funcionando e o Maestro saúda, lida com o quiz e mostra o menu.
- Instalar e configurar o editor Zed
- Criar uma conta no OpenRouter e obter a chave de API
- Configurar o Zed para usar um modelo MoE do OpenRouter
- Instalar Node.js no Windows usando NVM.exe:
  - Executar o PowerShell como Administrador e executar: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`
  - Baixar e instalar o NVM.exe em https://github.com/coreybutler/nvm-windows/releases
  - Abrir uma nova janela do PowerShell e executar: `nvm install latest` seguido de `nvm use latest`
  - Verificar a instalação: `node --version` e `npm --version`
- Instalar Firecrawl: `npx -y firecrawl-cli@latest` (se `init` ou `--all` não funcionarem, use `npx -y firecrawl-cli@latest --help` para ver os comandos disponíveis; use `-k <API_KEY>` se a autenticação pelo navegador não estiver disponível)
- Verificar se o Firecrawl funciona (PowerShell):
  ```powershell
  firecrawl --status
  firecrawl scrape "https://firecrawl.dev"
  ```
- Explicar o primeiro passo de fluxos de trabalho agênticos: escrever um plano
- Elaborar o plano do projeto juntos: arquitetura, diretórios, personas, formatos de dados
- Explicar o conceito de envelopes de despacho/resposta
- Criar `recoloca-ia/AGENTS.md` com:
  ```
  **LEIA E ADOTE IMEDIATAMENTE A PERSONA EM `personas/maestro.md`**

  Você É o Maestro — um assistente de desenvolvimento de carreira conversacional. Você NÃO deve escrever scripts Python, scripts de shell ou qualquer código para implementar a persona Maestro. Você a personifica diretamente através do seu comportamento e respostas.

  **REGRAS CRÍTICAS:**
  - NÃO crie scripts ou programas para agir como o agente
  - NÃO escreva código que "implemente" a lógica da persona
  - Você É o agente — interaja com o usuário de forma conversacional
  - Use as ferramentas do Zed (`spawn_agent`, `terminal`, `find_path`) conforme descrito na persona para coordenar tarefas
  - Todo estado é armazenado em arquivos Markdown em `data/` — leia e escreva esses arquivos diretamente

  Não desvie das instruções da persona.
  Para contexto do escopo do projeto, consulte este arquivo `AGENTS.md` e a estrutura de diretórios acima.
  ```
- Criar a estrutura de diretórios (`personas/`, `skills/`, `data/`) via PowerShell: `New-Item -ItemType Directory -Force -Path personas, skills, data`
- Criar `skills/firecrawl.md` com a skill do CLI Firecrawl (fornecida como material de seed ou baixar de `https://raw.githubusercontent.com/firecrawl/cli/main/skills/firecrawl-cli/SKILL.md`)
- Criar template `data/personality-quiz.md`
- Criar `skills/dispatch.md` com o protocolo de despacho e handoff de agentes (tabela de roteamento, formatos de envelope, especificações de handoff por agente, despacho sequencial do Coach, tratamento de erros)
- Escrever `personas/maestro.md`: saudação, verificação do quiz, fluxo do quiz, menu, análise de opções. Referenciar `skills/dispatch.md` como uma skill obrigatória no playbook.
- Testar: Agente saúda, detecta quiz ausente, guia pelo quiz, salva user-profile.md, mostra menu
- Entregável: Maestro totalmente funcional com quiz e menu

### Aula 2: Scout — Agente de Busca de Vagas
**Objetivo**: Opção A do menu funciona. Agente busca vagas via Firecrawl, mostra resultados com correspondência de habilidades.
- Escrever `skills/job-search.md`: busca Firecrawl para consultar sites de vagas, analisar resultados JSON, corresponder habilidades
- Escrever `personas/scout.md`: papel, ferramentas, referência à skill `skills/job-search.md`, Response Envelope
- Conectar `spawn_agent` no Maestro para despachar o Scout quando o usuário digitar "A"
- Testar: Usuário seleciona A → Scout busca via Firecrawl → resultados exibidos com habilidades correspondentes/em falta
- Testar: Se a busca falhar, o erro é reportado ao usuário
- Entregável: Scout totalmente funcional, busca de vagas end-to-end a partir do menu

### Aula 3: Curator — Agente de Busca de Cursos
**Objetivo**: Opção B do menu funciona. Agente busca na Alura via Firecrawl, recomenda cursos para lacunas de habilidades.
- Escrever `skills/course-analysis.md`: busca Firecrawl para cursos da Alura, corresponder cursos às habilidades em falta, ordenar por nível
- Escrever `personas/curator.md`: papel, ferramentas, lê resultados do Scout, referência à skill `skills/course-analysis.md`, Response Envelope
- Conectar `spawn_agent` no Maestro para despachar o Curator quando o usuário digitar "B"
- Curator recebe habilidades em falta dos resultados do Scout via contexto de handoff
- Testar: Usuário seleciona B → Curator busca na Alura via Firecrawl → recomendações de cursos exibidas
- Testar: Se a busca falhar, o erro é reportado ao usuário
- Entregável: Curator totalmente funcional, busca de cursos end-to-end a partir do menu

### Aula 4: Coach — Simulador de Entrevistas
**Objetivo**: Opção C do menu funciona. Agente executa uma entrevista simulada com feedback e pontuação.
- Escrever `skills/interview-sim.md`: geração de perguntas, avaliação de respostas, pontuação
- Escrever `personas/coach.md`: papel, ferramentas, modelo de despacho sequencial, referência à skill `skills/interview-sim.md`, formato de resposta
- Conectar `spawn_agent` no Maestro para despachar o Coach 6 vezes sequencialmente, mantendo estado entre chamadas
- Maestro rastreia número da pergunta, perguntas anteriores e respostas do usuário em cada handoff
- Se o usuário ainda não buscou vagas, o Coach usa o perfil do usuário para gerar perguntas gerais
- Se o usuário tem resultados de vagas, o Coach usa uma vaga selecionada para perguntas direcionadas
- Testar: Usuário seleciona C → Coach faz 5 perguntas em 6 despachos → dá feedback → pontuação final
- Testar: Fluxo completo: quiz → busca de vagas → entrevista para uma vaga encontrada
- Entregável: Coach totalmente funcional, sistema completo com todos os 4 agentes operacionais

## Próximos Passos (Após a Imersão)

- Adicionar pontuação de correspondência mais sofisticada (ponderação por nível de experiência, soft skills)
- Adicionar opção de menu para geração de carta de apresentação
- Adicionar mais dados seed para diferentes áreas de tecnologia
- Melhorar padrões de URL para sites de vagas que usam renderização JavaScript

## Notas Técnicas

- **Orquestração**: Roda dentro do editor Zed usando `spawn_agent` para despacho de sub-agentes. Nenhum framework externo necessário.
- **Acesso web**: `firecrawl search` e `firecrawl scrape` para descoberta e extração de vagas. O Firecrawl agrega resultados de múltiplas fontes (Indeed, Catho, LinkedIn, Glassdoor, Infojobs) e lida com renderização JavaScript, medidas anti-bot e retorna markdown limpo. Requer `FIRECRAWL_API_KEY` no ambiente.
- **Armazenamento de estado**: Todos os dados em arquivos Markdown sob `data/`.
- **Simplicidade**: Sem cache, sem IDs de sessão, sem pontuação complexa. Cada agente faz uma coisa e retorna resultados diretamente.
