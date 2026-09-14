# Recoloca_IA

Recoloca IA é um sistema multi-agente para desenvolvimento de carreira. Ele ajuda os usuários a descobrir vagas de emprego, identificar lacunas de habilidades, encontrar cursos direcionados e praticar habilidades de entrevista através de um pipeline orquestrado de agentes especializados.
┌─────────────────────────────────────────────────┐
│                  Usuário                         │
└────────────────────┬────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│              MAESTRO (Orquestrador)              │
│  - Interface primária com o usuário             │
│  - Coordena agentes especializados              │
│  - Consolida resultados e apresenta ao usuário  │
└──┬──────────────┬──────────────┬────────────────┘
   │              │              │
   ▼              ▼              ▼
┌─────────┐  ┌──────────┐  ┌──────────────┐
│ SCOUT   │  │ CURATOR  │  │ COACH        │
│ (Busca  │  │ (Busca   │  │ (Simulador   │
│ de      │  │ de       │  │ de           │
│ Empregos)│  │ Cursos)  │  │ Entrevistas) │
└─────────┘  └──────────┘  └──────────────┘
recoloca-ia/
├── README.md                          # Este arquivo
├── data/
│   ├── personality-quiz.md            # Respostas do quiz do usuário (modelo)
│   ├── user-profile.md                # Perfil consolidado (modelo)
│   ├── job-search-results.md          # Resultados de busca de vagas
│   ├── course-recommendations.md      # Recomendações de cursos
│   └── interview-session.md           # Rastreamento de estado da entrevista do Coach
├── personas/
│   ├── maestro.md                     # Definição do orquestrador Maestro
│   ├── scout.md                       # Definição do agente de busca de vagas Scout
│   ├── curator.md                     # Definição do agente de busca de cursos Curator
│   └── coach.md                       # Definição do agente de simulação de entrevista Coach
└── skills/
    ├── dispatch.md                    # Protocolo de despacho e handoff de agentes
    ├── firecrawl.md                   # Uso do CLI Firecrawl para busca e raspagem
    ├── job-search.md                  # Skill do Scout: fluxo de busca de vagas
    ├── course-analysis.md             # Skill do Curator: fluxo de busca de cursos
    └── interview-sim.md              # Skill do Coach: fluxo de simulação de entrevista

    Como Usar
Pré-requisitos
Zed — o editor assistido por IA usado para executar agentes
OpenRouter — configurado como o provedor de LLM no Zed
Firecrawl — instalado e configurado para busca e raspagem web
Configuração
Certifique-se de que o Firecrawl está instalado e o CLI está disponível no seu PATH
Configure sua chave de API do Firecrawl no seu ambiente
Abra este projeto no Zed
Carregue a persona do Maestro (personas/maestro.md) como o agente primário
Executando o Agente
Inicie o Zed com o projeto recoloca-ia aberto
Inicialize o agente Maestro a partir de personas/maestro.md
O Maestro cumprimenta o usuário e inicia o quiz de personalidade
Após o quiz, escolha no menu:
A — Buscar vagas de emprego (despacha Scout)
B — Encontrar cursos para preencher lacunas de habilidades (despacha Curator)
C — Praticar com uma entrevista simulada (despacha Coach 6 vezes)
D — Refazer o quiz (sobrescreve dados do quiz e reseta resultados)
Agentes
Maestro (Orquestrador)
O Maestro é a interface primária. Ele gerencia o fluxo de conversa, mantém arquivos de estado e despacha sub-agentes via spawn_agent. O Maestro nunca realiza buscas de emprego ou cursos diretamente — ele sempre delega ao Scout ou Curator.

Localização: personas/maestro.md

Responsabilidades:

Cumprimentar o usuário e executar o quiz de personalidade
Gerar e manter data/user-profile.md
Apresentar o menu após cada operação
Construir envelopes de despacho para sub-agentes
Analisar envelopes de resposta e exibir resultados
Lidar com refazer quiz (opção D do menu)
Manter data/interview-session.md durante os despachos do Coach
Scout (Agente de Busca de Empregos)
O Scout busca vagas de emprego usando o Firecrawl em Indeed, Catho, LinkedIn, Glassdoor e Infojobs. Ele realiza correspondência de habilidades contra o perfil do usuário e retorna até 5 vagas com análise de correspondência.

Localização: personas/scout.md Skill: skills/job-search.md

Fluxo de Trabalho:

Ler perfil do usuário do contexto do despacho
Executar firecrawl search "vagas [area] [localização]" --json
Fazer scrape de até 5 URLs de vagas para detalhes completos
Comparar habilidades necessárias com as habilidades atuais do usuário
Retornar resultados em formato de lista numerada com dados de correspondência de habilidades
Curator (Agente de Busca de Cursos)
O Curator busca na Alura cursos que abordem lacunas de habilidades identificadas pelo Scout. Ele retorna uma lista curada e ordenada de recomendações de cursos com metadados (duração, nível).

Localização: personas/curator.md Skill: skills/course-analysis.md

Fluxo de Trabalho:

Validar que resultados de busca de vagas existem
Extrair habilidades faltantes dos resultados de busca de vagas
Executar firecrawl search "alura [habilidade]" --json para cada habilidade faltante
Fazer scrape das URLs dos cursos para extrair duração e nível
Ordenar cursos por nível de dificuldade
Retornar recomendações ordenadas com ordem de estudo sugerida
Coach (Agente de Simulação de Entrevista)
O Coach conduz uma entrevista simulada estruturada de 5 perguntas. Ele gera perguntas específicas por função, avalia cada resposta, fornece feedback e entrega uma pontuação final.

Localização: personas/coach.md Skill: skills/interview-sim.md

Fluxo de Trabalho:

Receber contexto da vaga e perfil do usuário do despacho
Despacho 1: Gerar Pergunta 1 (sem histórico anterior)
Despacho 2: Avaliar R1, gerar Pergunta 2
Despacho 3: Avaliar R2, gerar Pergunta 3
Despacho 4: Avaliar R3, gerar Pergunta 4
Despacho 5: Avaliar R4, gerar Pergunta 5
Despacho 6: Avaliar R5, calcular pontuação final e áreas de melhoria
Início Rápido
Abra o projeto no Zed
Carregue personas/maestro.md como o agente ativo
Siga o quiz de personalidade guiado pelo Maestro
Selecione a opção A do menu para buscar vagas de emprego
Selecione a opção B do menu para encontrar cursos para suas lacunas de habilidades
Selecione a opção C do menu para praticar com uma entrevista simulada
Diretrizes do Modelo MoE
Este sistema usa uma arquitetura Mixture-of-Experts (MoE) com regras de formatação rigorosas:

Nenhuma instrução ambígua. Cada etapa deve especificar exatamente o que fazer, qual ferramenta usar e qual formato de saída produzir.
Nenhuma tabela markdown em qualquer saída. Use listas numeradas com pares chave-valor para todos os dados estruturados.
Todos os caminhos de arquivo devem ser relativos à raiz do projeto com prefixo explícito data/.
Se uma ferramenta falhar, relate a falha explicitamente. Nunca continue silenciosamente em caso de erro.
Nunca invente dados. Se uma busca ou raspagem do firecrawl falhar, relate o erro exato e pare.
