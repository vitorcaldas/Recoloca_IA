# Aula 4: Coach — Simulador de Entrevistas

## Visão Geral

Sistema multi-agente que auxilia usuários em sua jornada de desenvolvimento de carreira, combinando busca de empregos, identificação de lacunas de habilidades, recomendações de cursos e simulação de entrevistas.

**Objetivo da Aula**: Opção C do menu funciona. Agente executa uma entrevista simulada com feedback e pontuação.

## Diretrizes para Modelos MoE

Todas as personas e skills são projetadas para modelos Mixture of Experts (MoE). Siga estas regras:

- Sem instruções ambíguas. Cada passo deve especificar exatamente o que fazer, qual ferramenta usar e qual formato de saída produzir.
- Sem tabelas markdown em nenhuma saída. Use listas numeradas com pares chave-valor para dados estruturados.
- Todos os caminhos de arquivo devem ser relativos à raiz do projeto com prefixo explícito `data/`. A raiz do projeto é o diretório `recoloca-ia/`.
- Se uma ferramenta falhar, o agente deve relatar a falha no campo `erros` e não continuar silenciosamente.
- Nunca invente dados. Se um `firecrawl search` ou `firecrawl scrape` falhar, reporte o erro exato e pare. **Exceção**: Em `skills/course-analysis.md` (Aula 3), se a busca falhar para uma habilidade específica, tente o fallback; se o fallback também falhar, pule essa habilidade e continue com as restantes.
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

**Escopo da Aula 4**: Implementar o bloco Coach e conectá-lo ao Maestro com despacho sequencial de 6 chamadas.

## Estrutura de Diretórios (complemento da Aula 4)

```
recoloca-ia/
├── AGENTS.md                   # (já existe da Aula 1)
├── personas/
│   ├── maestro.md          # (já existe da Aula 1)
│   ├── scout.md            # (já existe da Aula 2)
│   ├── curator.md          # (já existe da Aula 3)
│   └── coach.md            # NOVO: Simulador de entrevistas
├── skills/
│   ├── firecrawl.md        # (já existe da Aula 1)
│   ├── dispatch.md         # (já existe da Aula 1)
│   ├── job-search.md       # (já existe da Aula 2)
│   ├── course-analysis.md  # (já existe da Aula 3)
│   └── interview-sim.md    # NOVO: Capacidades de simulação de entrevistas
└── data/
    ├── personality-quiz.md       # (já existe da Aula 1)
    ├── user-profile.md           # (já existe da Aula 1)
    ├── job-search-results.md     # (já existe da Aula 2)
    ├── course-recommendations.md # (já existe da Aula 3)
    └── interview-session.md      # NOVO: Rastreamento de estado da entrevista do Coach
```

## Coach - Simulador de Entrevistas

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

## Skills para Aula 4

### interview-sim.md

- **OBRIGATÓRIO CARREGAR.** Fluxo completo: geração de perguntas, calibração por nível, feedback, rubrica de pontuação, formato de resposta por despacho e tratamento de erros.
- Gerar 5 perguntas baseadas na descrição da vaga e perfil do usuário. Fazer uma de cada vez e aguardar a resposta do usuário
- Perguntas devem misturar comportamentais ("conte-me sobre uma vez quando...") e técnicas (específicas da função)
- Calibração por nível de experiência: perguntas mais simples para Júnior, mais profundas para Sênior
- Após cada resposta, dar 2-3 frases de feedback construtivo
- Após todas as 5 perguntas, dar uma pontuação geral de 1 a 10 e listar 2-3 áreas para melhorar
- Tratamento de erros: Se o contexto da vaga não estiver disponível, gerar perguntas gerais baseadas no perfil do usuário e informar ao Maestro. Se o despacho falhar, reportar no campo `erros` do envelope de resposta. Se o usuário desistir no meio, gerar pontuação parcial com base nas respostas já recebidas.

## Protocolo de Handoff do Coach

**Contexto passado:** título e descrição da vaga selecionada de `data/job-search-results.md` (ou perfil do usuário se nenhuma busca de vagas foi realizada)

**Otimização de tokens**: A seção `referencia_persona` com o conteúdo completo de `personas/coach.md` deve ser incluída apenas no Despacho 1. Nos despachos 2-6, omita `referencia_persona` e envie apenas `tarefa`, `perfil_usuario`, `contexto` e `saida_esperada`. O Coach já conhece sua persona a partir do primeiro despacho.

**Modelo de despacho sequencial:** Maestro despacha o Coach 6 vezes no total para completar uma entrevista de 5 perguntas:
1. Despacho 1: Gerar e retornar P1
2. Despacho 2: Avaliar R1, retornar feedback + P2
3. Despacho 3: Avaliar R2, retornar feedback + P3
4. Despacho 4: Avaliar R3, retornar feedback + P4
5. Despacho 5: Avaliar R4, retornar feedback + P5
6. Despacho 6: Avaliar R5, retornar pontuação final e áreas para melhorar

### Envelope de Despacho (Despacho 1)

```
## DESPACHO: COACH
### referencia_persona
[Conteúdo completo de personas/coach.md]

### tarefa
Conduzir entrevista simulada para a vaga de [titulo] na empresa [empresa]

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
Vaga: [título e descrição da vaga]
Número da pergunta: 1
Histórico: nenhum (primeira pergunta)

### saida_esperada
Estado, pergunta_atual e feedback_anterior (vazio para P1)
```

### Envelope de Despacho (Despachos 2-5)

```
## DESPACHO: COACH
### tarefa
Avaliar resposta anterior e gerar próxima pergunta

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
Vaga: [título e descrição da vaga]
Número da pergunta: [2-5]
Histórico:
  P1: [texto]
  R1: [texto]
  ...
  R[N-1]: [texto da resposta do usuário]

### saida_esperada
Estado, feedback_anterior (sobre R[N-1]) e pergunta_atual (P[N])
```

### Envelope de Despacho (Despacho 6 - final)

```
## DESPACHO: COACH
### tarefa
Avaliar resposta final e dar pontuação da entrevista

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
Vaga: [título e descrição da vaga]
Histórico completo:
  P1: [texto]
  R1: [texto]
  ...
  P5: [texto]
  R5: [texto da resposta do usuário]

### saida_esperada
Estado, pontuação final (X/10) e áreas de melhoria
```

### Resposta do Coach por despacho (geração de perguntas 1-5):

```
### estado: sucesso
### pergunta_atual: [a pergunta a fazer ao usuário]
### feedback_anterior: [feedback sobre a resposta anterior, ou vazio para a pergunta 1]
```

### Resposta do despacho final (despacho 6, após o usuário responder a pergunta 5):

```
### estado: sucesso
### Entrevista Concluída
Pontuação: [X/10]

### Áreas de melhoria
1. [item 1]
2. [item 2]
```

## Esquema do Arquivo de Dados

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

## Tasks da Aula 4

### 1. Escrever `skills/interview-sim.md`

Contendo:
- Como gerar perguntas comportamentais e técnicas baseadas na vaga e perfil
- Calibração por nível de experiência (Júnior/Pleno/Sênior)
- Diretrizes de feedback (2-3 frases por resposta)
- Rubrica de pontuação (1-10)
- Formato de resposta por despacho

### 2. Escrever `personas/coach.md`

Contendo:
- Papel e responsabilidade do Coach
- Ferramentas disponíveis (terminal como fallback)
- Modelo de despacho sequencial (6 despachos)
- Referência obrigatória à skill `skills/interview-sim.md`
- Formato de resposta por despacho
- Regras de tratamento de erros

### 3. Conectar `spawn_agent` no Maestro para despachar o Coach 6 vezes sequencialmente

- Quando o usuário digitar "C", o Maestro inicia o ciclo de 6 despachos
- Despacho 1: Coach gera P1 → Maestro exibe pergunta → aguarda resposta do usuário
- Despacho 2: Maestro envia R1 → Coach avalia e retorna feedback + P2 → exibe → aguarda resposta
- Repetir até P5
- Despacho 6: Maestro envia R5 → Coach avalia e retorna pontuação final + áreas para melhorar
- Maestro rastreia número da pergunta, perguntas anteriores e respostas do usuário em cada handoff
- Maestro salva estado em `data/interview-session.md`

### 4. Testar: entrevista simulada funcional

- Usuário seleciona C
- Coach faz 5 perguntas em 6 despachos
- Dá feedback após cada resposta
- Pontuação final exibida

### 5. Testar: fluxo completo end-to-end

- Quiz → busca de vagas → entrevista para uma vaga encontrada

## Fluxo da Aula 4

```
Usuário seleciona C no menu
        │
        ▼
Maestro verifica se há resultados de vagas em data/job-search-results.md
        │
        ├─ Tem resultados → pede ao usuário selecionar uma vaga
        └─ Sem resultados → usa perfil do usuário para perguntas gerais
        │
        ▼
─── DESPACHO 1 ───
Maestro envia despacho com referencia_persona completa
Coach gera P1 → retorna pergunta_atual
Maestro exibe P1 → aguarda resposta do usuário (R1)
        │
        ▼
─── DESPACHO 2 ───
Maestro envia despacho com R1 (sem referencia_persona)
Coach avalia R1 → retorna feedback + P2
Maestro exibe feedback + P2 → aguarda resposta (R2)
        │
        ▼
─── DESPACHOS 3-5 ───
Repetir padrão: avalia resposta anterior → feedback + próxima pergunta
        │
        ▼
─── DESPACHO 6 (FINAL) ───
Maestro envia despacho com R5
Coach avalia R5 → retorna pontuação + áreas de melhoria
        │
        ▼
Maestro exibe resultado final → salva em data/interview-session.md → retorna ao menu
```

## Regras de Erro

- Se o usuário ainda não buscou vagas, o Coach usa o perfil do usuário para gerar perguntas gerais.
- Se `spawn_agent` falhar, dizer ao usuário o que deu errado e retornar ao menu.
- Se o usuário desistir no meio da entrevista, salvar estado parcial em `data/interview-session.md` e retornar ao menu.

## Visão Geral do Sistema Completo

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

## Próximos Passos (Após a Imersão)

- Adicionar pontuação de correspondência mais sofisticada (ponderação por nível de experiência, soft skills)
- Adicionar opção de menu para geração de carta de apresentação
- Adicionar mais dados seed para diferentes áreas de tecnologia
- Melhorar padrões de URL para sites de vagas que usam renderização JavaScript

## Entregável da Aula 4

Coach totalmente funcional, sistema completo com todos os 4 agentes operacionais.
