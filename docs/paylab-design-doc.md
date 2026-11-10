# PayLab — Documentação do Projeto

Gateway de pagamentos em sandbox, com API para integradores, mensageria e copiloto de IA

**Versão 1.0 (aprovada)** | 06/11/2026 | Autor: Guilherme E. Fonseca | Status: aprovada em 07/11/2026

## 1. Controle do documento

| Versão | Data | Descrição | Autor |
| --- | --- | --- | --- |
| 0.1 | 06/11/2026 | Primeira versão completa, para revisão e aprovação | Guilherme E. Fonseca |
| 1.0 | 07/11/2026 | Documento aprovado pelo responsável; make substituído por Task (ADR 0011); licença MIT definida | Guilherme E. Fonseca |

.

**Nome:** PayLab é um nome provisório, de caráter genérico, e pode ser trocado até a aprovação.

## 2. Visão do produto

### 2.1 Contexto e problema

Empresas que vendem online precisam cobrar de forma confiável. Na prática, integrar-se a meios de pagamento é difícil por quatro motivos: a confirmação do pagamento é assíncrona, as notificações podem chegar duplicadas ou fora de ordem, uma falha de rede pode gerar cobranças repetidas, e quando algo dá errado é difícil descobrir se o problema está no lado do lojista ou da plataforma. Desenvolvedores que integram perdem horas diagnosticando webhooks que não chegam.

### 2.2 Visão

O PayLab é uma plataforma de pagamentos em ambiente de teste (sandbox). Ela expõe uma API REST para criar cobranças Pix simuladas, registra cada movimento em um ledger de partidas dobradas, avisa o lojista por webhooks assinados e oferece um copiloto de IA que ajuda quem integra a entender e corrigir problemas.

### 2.3 Objetivos do produto

- **O1:** API consistente e idempotente: a mesma requisição repetida nunca gera duas cobranças.
- **O2:** ledger íntegro: todo lançamento é balanceado e imutável.
- **O3:** entrega confiável de eventos, com assinatura, retentativas e fila de mensagens mortas (DLQ).
- **O4:** copiloto de IA com respostas fundamentadas em fontes citadas e com aprovação humana para ações sensíveis.
- **O5:** operar em contêineres, em Kubernetes local e em uma janela curta de nuvem.
- **O6:** documentação, testes e métricas suficientes para apresentar o projeto numa entrevista técnica.

### 2.4 Objetivos de aprendizado

Os objetivos abaixo guiam as escolhas técnicas e justificam algumas decisões que, para um produto real, poderiam ser mais simples.

- Projetar APIs REST para terceiros: versionamento, idempotência, paginação, erros padronizados, OpenAPI.
- Arquitetura orientada a eventos: mensageria, outbox, retentativas, DLQ, consumidores idempotentes.
- Modelagem financeira: ledger de partidas dobradas, máquina de estados, liquidação.
- IA aplicada: RAG, LangChain, LangGraph, avaliação, guardrails e human-in-the-loop.
- Operação: Docker, Kubernetes, CI/CD, observabilidade, infraestrutura como código, AWS.

### 2.5 Personas

| Persona | Papel | Necessidade principal |
| --- | --- | --- |
| Marina | Desenvolvedora integradora (backend de um e-commerce) | Integrar rápido, entender erros e receber webhooks de forma confiável |
| Carlos | Operador financeiro do lojista | Ver cobranças, saldo, estornos e extrato sem depender de TI |
| Ana | Analista de risco | Revisar transações sinalizadas e entender o motivo |
| Admin | Administrador da plataforma | Gerir lojistas, taxas, chaves e auditoria |

### 2.6 Escopo do MVP

- Cadastro de lojistas (tenants) e chaves de API de sandbox.
- Cobranças Pix simuladas com código copia e cola e QR code, expiração e cancelamento.
- PSP simulado que confirma, recusa ou falha de forma configurável.
- Ledger de partidas dobradas, saldos, extrato e liquidação simulada.
- Estornos totais e parciais.
- Webhooks assinados com retentativas, DLQ, reenvio e histórico.
- Antifraude por regras e score, com fila de revisão manual.
- Copiloto de IA para integradores e explicação das decisões de antifraude.
- Painel web e uma loja de exemplo que integra pela API.

### 2.7 Fora do escopo

- Dinheiro real, Pix real, conexão com o Banco Central ou com instituições financeiras.
- Cartão de crédito e débito (envolve PCI DSS), boleto bancário real e split de pagamentos (possível evolução futura).
- KYC e onboarding regulatório, multimoeda, aplicativo móvel, emissão de nota fiscal.

### 2.8 Premissas e restrições

- O projeto é de estudo, feito por uma pessoa, com cerca de 10 horas por semana.
- Ferramentas gratuitas e de código aberto. Nuvem apenas em uma janela curta no final (condições do plano gratuito a conferir antes de criar a conta).
- Somente dados sintéticos. Nada de dados pessoais reais.
- LLM executado localmente (Ollama), com possibilidade de trocar por uma API depois.
- O sistema não é uma instituição de pagamento e não processa dinheiro real. Isso deve estar explícito no README e no painel.

### 2.9 Métricas de sucesso

As metas abaixo são iniciais e serão calibradas nas Fases 0 e 1.

| Métrica | Meta | Como medir |
| --- | --- | --- |
| Cobranças duplicadas por chave de idempotência | 0 em 1.000 requisições concorrentes repetidas | Teste automatizado de concorrência |
| Violações do invariante do ledger (débitos = créditos) | 0 | Job de verificação e testes de propriedade |
| Latência p95 de POST /v1/charges | abaixo de 300 ms a 50 req/s, ambiente local | Teste de carga com k6 |
| Webhooks entregues em até 60 s, com receptor saudável | 99% | Métrica da fila e testes ponta a ponta |
| Cobertura de testes no domínio | 80% ou mais | Relatório de cobertura no CI |
| Fidelidade das respostas do copiloto (faithfulness) | 0,80 ou mais no conjunto de referência | Avaliação com RAGAs no CI |
| Tempo para subir o ambiente do zero | menos de 5 minutos | task up em máquina limpa |

## 3. Glossário

| Termo | Definição |
| --- | --- |
| Tenant / lojista | Empresa cliente do PayLab, com dados isolados das demais |
| Cobrança (charge) | Pedido de pagamento criado pelo lojista |
| PSP | Provedor de serviços de pagamento. Aqui é simulado |
| Pix copia e cola / BR Code | Texto padronizado que representa uma cobrança Pix e pode virar QR code |
| Idempotência | Propriedade de repetir uma operação sem efeito adicional |
| Ledger | Livro-razão: registro imutável de lançamentos contábeis |
| Partida dobrada | Todo lançamento tem débito e crédito de mesmo valor |
| Webhook | Chamada HTTP que o PayLab faz ao lojista para avisar de um evento |
| HMAC | Assinatura criptográfica que prova a origem e a integridade de uma mensagem |
| Outbox | Padrão que grava o evento junto com o dado, na mesma transação, para publicar depois sem perdas |
| DLQ | Fila de mensagens que esgotaram as tentativas |
| At-least-once | Garantia de entrega de ao menos uma vez, o que exige consumidores idempotentes |
| RAG | Geração aumentada por recuperação: o modelo responde com base em documentos buscados |
| HITL | Human-in-the-loop: ações sensíveis só acontecem com aprovação humana |
| ADR | Registro curto de uma decisão arquitetural e de seus motivos |
| SLO | Meta de nível de serviço, medida por indicadores |

## 4. Requisitos funcionais

Prioridade (MoSCoW): **M** = deve ter no MVP, **S** = deveria ter, **C** = pode ter.

### 4.1 Acesso e chaves (AUT)

| ID | Requisito | Prior. | Critério de aceite |
| --- | --- | --- | --- |
| RF-01 | O administrador cadastra um lojista | M | Lojista criado com identificador único e dados isolados |
| RF-02 | O lojista cria, lista e revoga chaves de API de sandbox | M | Chave secreta exibida uma única vez; revogada deixa de funcionar de imediato |
| RF-03 | Usuários do painel entram com e-mail e senha (JWT) | M | Token expira; refresh funciona; senha com hash forte |
| RF-04 | Permissões por papel: admin, operador, analista de risco, leitura | M | Acesso negado com 403 fora do papel |

### 4.2 Cobranças (CHG)

| ID | Requisito | Prior. | Critério de aceite |
| --- | --- | --- | --- |
| RF-10 | Criar cobrança Pix com valor, descrição, cliente, expiração e metadata | M | Retorna 201 com id, status pending e código copia e cola |
| RF-11 | Consultar e listar cobranças, com filtros e paginação por cursor | M | Filtros por status, período e cliente; ordenação estável |
| RF-12 | Cancelar cobrança pendente | M | Somente em pending; evento charge.canceled emitido |
| RF-13 | Gerar o payload Pix e o QR code | M | Payload válido pelo formato do BR Code e com CRC correto |
| RF-14 | Expirar cobranças vencidas automaticamente | M | Status expired no prazo, com evento emitido |
| RF-15 | Idempotência por cabeçalho Idempotency-Key | M | Mesma chave e mesmo corpo devolvem a mesma resposta; corpo diferente devolve 422 |

### 4.3 PSP simulado (PSP)

| ID | Requisito | Prior. | Critério de aceite |
| --- | --- | --- | --- |
| RF-20 | Confirmar o pagamento após atraso configurável | M | Callback gera charge.paid no tempo configurado |
| RF-21 | Simular cenários de falha: recusa, timeout e notificação duplicada | M | Cada cenário selecionável por valor ou metadata da cobrança |
| RF-22 | O callback do PSP é autenticado | M | Callback sem assinatura válida é rejeitado |

### 4.4 Ledger e saldos (LED)

| ID | Requisito | Prior. | Critério de aceite |
| --- | --- | --- | --- |
| RF-30 | Cada evento financeiro gera lançamentos de partida dobrada | M | Soma de débitos igual à de créditos em cada transação |
| RF-31 | Consultar saldo por conta: a liquidar e disponível | M | Saldo bate com a soma dos lançamentos |
| RF-32 | Extrato com paginação | M | Lista cronológica com origem de cada lançamento |
| RF-33 | Liquidação simulada (D+1) | S | Job move valores de a liquidar para disponível |
| RF-34 | Lançamentos são imutáveis; correção por contra-lançamento | M | Nenhum UPDATE ou DELETE na tabela de lançamentos |

### 4.5 Estornos (REF)

| ID | Requisito | Prior. | Critério de aceite |
| --- | --- | --- | --- |
| RF-40 | Estorno total ou parcial de cobrança paga | M | Soma dos estornos nunca excede o valor pago |
| RF-41 | Estorno é idempotente | M | Repetição com mesma chave não duplica |
| RF-42 | Estorno gera contra-lançamentos e eventos | M | refund.created e refund.succeeded emitidos |

### 4.6 Webhooks e eventos (WHK)

| ID | Requisito | Prior. | Critério de aceite |
| --- | --- | --- | --- |
| RF-50 | Cadastrar endpoints de webhook e escolher eventos | M | URL validada; segredo de assinatura por endpoint |
| RF-51 | Entrega com assinatura HMAC e timestamp | M | Receptor consegue validar a assinatura com o exemplo da documentação |
| RF-52 | Retentativas com espera crescente e DLQ | M | Após esgotar as tentativas, o evento vai para a DLQ |
| RF-53 | Reenvio manual de eventos | S | Ação disponível na API e no painel |
| RF-54 | Histórico de tentativas de entrega | M | Status HTTP, latência e erro de cada tentativa |
| RF-60 | Catálogo de eventos versionados | M | Schema publicado para cada evento |
| RF-61 | API para listar e consultar eventos | M | Filtros por tipo e período |

### 4.7 Antifraude (FRD)

| ID | Requisito | Prior. | Critério de aceite |
| --- | --- | --- | --- |
| RF-70 | Regras de risco configuráveis | S | Regras ativáveis por lojista |
| RF-71 | Score de risco de 0 a 100 por cobrança | S | Score gravado e visível no detalhe |
| RF-72 | Ação conforme o score: aprovar, revisar ou bloquear | S | Limites configuráveis |
| RF-73 | Fila de revisão manual | S | Analista aprova ou rejeita, com registro em auditoria |
| RF-74 | Explicação em linguagem natural por IA | C | Explicação cita os sinais usados |

### 4.8 Copiloto de IA (CPL)

| ID | Requisito | Prior. | Critério de aceite |
| --- | --- | --- | --- |
| RF-80 | Chat sobre a documentação e o OpenAPI | S | Respostas citam as fontes usadas |
| RF-81 | Consulta eventos e tentativas de entrega do próprio lojista por ferramentas | S | Nunca acessa dados de outro lojista |
| RF-82 | Sugere a correção do problema encontrado | S | Sugestão traz o passo e a referência |
| RF-83 | Ações que alteram estado, como reenviar webhook, exigem aprovação humana | S | Sem aprovação, a ação não executa |
| RF-84 | Quando não há fundamento nas fontes, diz que não sabe | S | Conjunto de perguntas sem resposta é recusado corretamente |

### 4.9 Painel, loja exemplo e operação (WEB, DEMO, OPS)

| ID | Requisito | Prior. | Critério de aceite |
| --- | --- | --- | --- |
| RF-90 | Dashboard com volume, taxa de aprovação e saldo | S | Dados coerentes com o ledger |
| RF-91 | Lista de cobranças e detalhe com linha do tempo | M | Linha do tempo mostra estados e eventos |
| RF-92 | Gestão de chaves e webhooks no painel | M | Mesmas regras da API |
| RF-93 | Chat do copiloto no painel | S | Streaming de respostas |
| RF-94 | Fila de revisão de risco no painel | S | Ações de aprovar e rejeitar |
| RF-100 | Loja exemplo que integra pela API e recebe webhooks | M | Fluxo de compra ponta a ponta funcionando |
| RF-110 | Administração de lojistas e taxas | M | Alterações geram registro de auditoria |
| RF-111 | Registro de auditoria de ações sensíveis | M | Ator, ação, horário e correlation id |

## 5. Requisitos não funcionais

| ID | Categoria | Requisito | Verificação |
| --- | --- | --- | --- |
| RNF-01 | Consistência | Uma chave de idempotência nunca gera dois efeitos | Teste de concorrência |
| RNF-02 | Integridade financeira | Valores em inteiros (centavos), moeda BRL; ledger sempre balanceado | Testes de propriedade e job de verificação |
| RNF-03 | Desempenho | p95 de criação abaixo de 300 ms e de leitura abaixo de 150 ms na carga base | k6 |
| RNF-04 | Resiliência | A falha do broker não perde eventos; a lentidão de um receptor de webhook não bloqueia os demais | Teste de caos simples |
| RNF-05 | Escalabilidade | Serviços sem estado, escaláveis horizontalmente | Execução com várias réplicas no Kubernetes |
| RNF-06 | Segurança | TLS nos ambientes expostos, chaves de API guardadas como hash, segredos fora do repositório, HMAC, rate limit, checklist OWASP API Top 10 | Revisão e varredura automática |
| RNF-07 | Privacidade | Somente dados sintéticos; dados pessoais mascarados em logs e em prompts | Testes de mascaramento |
| RNF-08 | Observabilidade | Logs estruturados com correlation id, métricas RED e tracing | Painéis e traces de ponta a ponta |
| RNF-09 | Manutenibilidade | Camadas separadas, lint e tipagem estática, cobertura de 80% no domínio | CI |
| RNF-10 | Reprodutibilidade | Ambiente completo sobe com um comando | Teste em máquina limpa |
| RNF-11 | Portabilidade | Padrão 12-factor; mesma imagem em compose, Kubernetes e nuvem | Deploy nos três ambientes |
| RNF-12 | Documentação | OpenAPI gerado do código e sempre atualizado; ADRs para decisões relevantes | Checagem no CI |
| RNF-13 | Custo | Zero em desenvolvimento; teto de gasto e alerta de orçamento na nuvem | Configuração revisada antes do deploy |
| RNF-14 | IA | Respostas com citação; nenhum dado pessoal no prompt; latência p95 abaixo de 15 s com modelo local | Avaliação automática e traces |
| RNF-15 | Auditabilidade | Toda mutação financeira registra ator, horário e correlation id | Consulta ao log de auditoria |

## 6. Regras de negócio

Valores numéricos são propostas iniciais e ficam em configuração, não no código.

| ID | Regra |
| --- | --- |
| RN-01 | Valor mínimo de uma cobrança: R$ 1,00. Máximo: R$ 50.000,00. Todos os valores são inteiros em centavos e a moeda é BRL. |
| RN-02 | Estados da cobrança: pending, in\_review, paid, failed, expired, canceled, partially\_refunded e refunded. Só as transições do diagrama de estados (seção 7.5) são permitidas. |
| RN-03 | Expiração padrão de 30 minutos, configurável de 1 minuto a 7 dias. Cobrança expirada não pode ser paga. |
| RN-04 | Idempotência: a chave tem de 1 a 255 caracteres e é guardada por 24 horas. Mesma chave e mesmo corpo devolvem a resposta original. Mesma chave e corpo diferente devolvem 422. Requisições simultâneas com a mesma chave são serializadas. |
| RN-05 | Taxa: percentual configurável por lojista, com padrão ilustrativo de 0,99% sobre o valor, calculada quando o pagamento é confirmado. |
| RN-06 | Cada evento financeiro gera uma transação de ledger balanceada: soma dos débitos igual à soma dos créditos. Lançamentos nunca são alterados nem apagados. |
| RN-07 | Liquidação simulada: o valor pago fica como a liquidar e passa a disponível em D+1, por um job diário. |
| RN-08 | Estorno só de cobrança paga. A soma dos estornos não pode exceder o valor pago. O estorno exige saldo disponível suficiente; caso contrário falha com insufficient\_balance. A taxa não é devolvida (proposta, a validar). |
| RN-09 | Webhooks: sucesso é uma resposta 2xx em até 10 segundos. Tentativas com espera de 10 s, 1 min, 5 min, 30 min, 2 h, 6 h e 24 h (7 tentativas); depois disso o evento vai para a DLQ. |
| RN-10 | Entrega at-least-once: o mesmo evento pode chegar mais de uma vez. Cada evento tem id único e um número de sequência por cobrança, e o receptor deve deduplicar. |
| RN-11 | Assinatura: o cabeçalho PayLab-Signature carrega o timestamp e o HMAC-SHA256 de timestamp, ponto e corpo bruto, com o segredo do endpoint. A tolerância do timestamp é de 5 minutos. |
| RN-12 | Antifraude: score de 0 a 100. Abaixo de 50 aprova; de 50 a 79 vai para revisão; 80 ou mais bloqueia. Sinais iniciais: valor muito acima do histórico do lojista, muitas cobranças do mesmo cliente em poucos minutos e repetição de valores. |
| RN-13 | Copiloto: só acessa dados do próprio lojista, nunca revela chaves ou segredos, e qualquer ação que altere estado exige confirmação explícita de uma pessoa. |
| RN-14 | Dados pessoais (CPF, e-mail, telefone) são mascarados em logs e antes de qualquer chamada a um modelo de linguagem. |
| RN-15 | Chaves de API de sandbox têm prefixos pk\_test\_ e sk\_test\_. A chave secreta é exibida uma única vez e guardada apenas como hash. |

## 7. Arquitetura

### 7.1 Estilo e princípios

O PayLab usa uma arquitetura orientada a eventos, com um monólito modular em Django como núcleo e alguns serviços satélites (PSP simulado, antifraude e IA). A escolha evita o custo de microsserviços para tudo e mantém separado o que tem ciclo de vida, linguagem de uso ou escala diferentes.

Princípios:

- O ledger é a fonte da verdade financeira. Saldos são derivados dele.
- Entrega at-least-once com consumidores idempotentes.
- Contratos explícitos: OpenAPI para a API e JSON Schema para eventos.
- Padrão 12-factor: configuração por ambiente, serviços sem estado, logs em stdout.
- Segurança e observabilidade desde a primeira fase, não no final.
- A IA nunca está no caminho crítico do pagamento: ela explica e auxilia, não decide.

### 7.2 Visão de contexto (C4, nível 1)

Atores e sistemas ao redor do PayLab:

- **Sistema do lojista e loja exemplo:** chamam a API e recebem webhooks.
- **Operador, analista de risco e administrador:** usam o painel web.
- **PSP simulado:** confirma, recusa ou falha pagamentos Pix.
- **Provedor de LLM:** modelo de linguagem local (Ollama) que serve o copiloto.

### 7.3 Contêineres (C4, nível 2)

| Contêiner | Tecnologia | Responsabilidade |
| --- | --- | --- |
| core-api | Django, DRF | Módulos de contas, cobranças, ledger, estornos, webhooks, eventos, risco e auditoria. API pública e administrativa |
| workers | Celery | Publicação do outbox, expiração, liquidação, entrega de webhooks e retentativas |
| psp-simulator | FastAPI | Simula o Pix: confirma pagamentos com atraso e injeta falhas |
| risk-service | FastAPI | Regras e score de risco, consumindo eventos |
| ai-service | FastAPI, LangGraph | Copiloto (RAG, ferramentas, aprovação humana) e explicações de risco |
| web | Next.js, TypeScript | Painel do lojista, fila de revisão e chat do copiloto |
| demo-store | Next.js | Loja exemplo que integra pela API |
| broker | RabbitMQ | Mensageria entre os serviços |
| db | PostgreSQL 16 com pgvector | Dados, ledger, outbox e vetores do RAG |
| cache | Redis | Idempotência, rate limit e cache semântico |
| storage | MinIO | Documentos do RAG e exportações |
| llm | Ollama | Modelos locais de geração e de embeddings |
| observabilidade | OpenTelemetry, Prometheus, Grafana, Langfuse | Métricas, traces, painéis e rastreamento das chamadas de IA |

&#91;embedded content: arquitetura · contêineres e comunicação\]

O painel e a loja exemplo chamam a API por REST, e o painel também conversa com o ai-service no chat do copiloto. O PSP simulado responde por callback. Os eventos saem do núcleo pelo outbox e cada serviço consome a própria fila.

### 7.4 Comunicação

**Síncrona (HTTP/REST):** cliente para core-api, painel para core-api e para ai-service, PSP para core-api (callback).

**Assíncrona (AMQP):** o core-api grava o evento no outbox e um worker publica no exchange `paylab.events` (tipo topic). Chaves de roteamento: `charge.*`, `payment.*`, `refund.*` e `risk.*`. Cada consumidor tem sua fila, com DLQ própria: dispatcher de webhooks, risk-service e indexador de auditoria.

**Eventos internos e públicos:** os eventos internos (domínio) são mais granulares. Os eventos públicos, enviados por webhook, formam um contrato estável e versionado, derivado dos internos.

### 7.5 Padrões adotados

| Padrão | Para que serve no projeto |
| --- | --- |
| Outbox transacional | Gravar dado e evento na mesma transação e publicar depois, sem perder eventos |
| Consumidor idempotente (inbox) | Tolerar mensagens repetidas |
| Idempotency-Key | Evitar efeito duplicado em chamadas repetidas |
| Retentativa com backoff e jitter, mais DLQ | Absorver falhas temporárias de receptores |
| Circuit breaker por endpoint de webhook | Impedir que um receptor lento consuma toda a capacidade |
| Bloqueio pessimista e transações curtas | Proteger o ledger sob concorrência |
| Projeção de saldos (CQRS leve) | Ler saldos rápido sem recalcular todo o ledger |
| Máquina de estados explícita | Garantir que só transições válidas aconteçam |

**Estados da cobrança:** pending vai para paid, in\_review, failed, expired ou canceled. In\_review vai para paid ou failed. Paid vai para partially\_refunded ou refunded. Partially\_refunded vai para refunded. Estados failed, expired, canceled e refunded são finais.

&#91;embedded content: máquina de estados da cobrança · 8 estados\]

Pending vai para failed por recusa do PSP ou score alto, para expired quando o prazo vence e para canceled quando o lojista cancela. Um estorno parcial leva a paid para partially\_refunded; o estorno total leva para refunded.

### 7.6 Decisões arquiteturais (resumo dos ADRs)

| ADR | Decisão | Motivo | Alternativa considerada |
| --- | --- | --- | --- |
| 001 | Monólito modular em Django, com serviços satélites | Menos complexidade operacional, com separação onde há ganho real | Microsserviços completos |
| 002 | RabbitMQ, com adaptador para trocar por SQS e SNS na nuvem | Roteamento rico e fácil de rodar local; adaptador permite estudar a mensageria da AWS | Kafka |
| 003 | Ledger append-only, com saldos derivados e projetados | Auditabilidade e correção por contra-lançamento | Saldo mutável em coluna |
| 004 | Outbox em PostgreSQL | Consistência entre dado e evento sem transação distribuída | Publicar direto no broker |
| 005 | FastAPI para PSP, risco e IA | Async nativo e bom encaixe com bibliotecas de IA | Tudo em Django |
| 006 | pgvector no mesmo PostgreSQL | Um banco a menos para operar | Banco vetorial dedicado |
| 007 | LLM local com camada de abstração de provedor | Custo zero e troca fácil por uma API | Depender de uma API paga |
| 008 | Next.js com TypeScript no front | Mercado e contraste com o back | Django templates |
| 009 | Terraform para a infraestrutura | Reprodutível e destruível, importante com plano gratuito limitado | Console manual |
| 010 | Multi-tenancy com tenant\_id e Row Level Security | Isolamento reforçado no banco | Um banco por lojista |
| 011 | Task (Taskfile.yml) como executor de tarefas no lugar do make | Mesmo comando no Windows nativo, no Linux e no CI | make (via WSL ou pacotes de terceiros), scripts PowerShell, just |

## 8. Modelo de dados

### 8.1 Princípios

- Chaves primárias em UUID. Valores monetários em inteiros (centavos). Datas em timestamptz, UTC.
- Toda tabela de negócio tem tenant\_id e usa Row Level Security.
- Lançamentos do ledger, eventos e auditoria são append-only: sem UPDATE nem DELETE.
- Migrações versionadas e revisadas em pull request.

### 8.2 Entidades

| Tabela | Descrição | Campos principais | Restrições e índices |
| --- | --- | --- | --- |
| tenants | Lojistas | id, nome, status, fee\_policy\_id | Nome único |
| users | Usuários do painel | id, tenant\_id, e-mail, hash da senha, papel | E-mail único por tenant |
| api\_keys | Chaves de API | id, tenant\_id, prefixo, hash, escopo, revogada\_em | Índice por prefixo |
| customers | Clientes finais | id, tenant\_id, nome, documento mascarado | Índice por tenant |
| charges | Cobranças | id, tenant\_id, customer\_id, valor, status, payload\_pix, expira\_em, metadata | Índices em (tenant\_id, created\_at desc) e (tenant\_id, status) |
| charge\_status\_history | Histórico de estados | charge\_id, de, para, motivo, em | Append-only |
| refunds | Estornos | id, charge\_id, valor, status, motivo | Soma por cobrança validada na regra |
| fee\_policies | Políticas de taxa | id, percentual, fixo | Configuração |
| ledger\_accounts | Contas do ledger | id, tenant\_id (nulo para plataforma), tipo | Único por tenant e tipo |
| ledger\_transactions | Transações contábeis | id, charge\_id ou refund\_id, descrição, em | Append-only |
| ledger\_entries | Lançamentos | id, transaction\_id, account\_id, direção (D ou C), valor | Valor maior que zero; soma D igual à soma C por transação (trigger deferrable) |
| balance\_projection | Saldos derivados | account\_id, a\_liquidar, disponível, versão | Atualizada pelo mesmo commit do ledger |
| idempotency\_keys | Chaves de idempotência | tenant\_id, escopo, chave, hash do corpo, resposta, expira\_em | Único em (tenant\_id, escopo, chave) |
| outbox\_events | Eventos a publicar | id, tipo, payload, publicado\_em | Índice em não publicados |
| events | Log de eventos públicos | id, tenant\_id, tipo, sequência, payload | Índice em (tenant\_id, tipo, created\_at) |
| webhook\_endpoints | Endpoints do lojista | id, tenant\_id, url, segredo cifrado, eventos | URL validada |
| webhook\_deliveries | Tentativas de entrega | id, event\_id, endpoint\_id, tentativa, status\_http, latência, erro | Índice por evento |
| risk\_rules e risk\_assessments | Regras e avaliações | charge\_id, score, sinais, decisão | Um por cobrança |
| audit\_logs | Auditoria | ator, ação, alvo, correlation\_id, em | Append-only |
| kb\_documents e kb\_chunks | Base do copiloto | origem, versão, texto, embedding vector | Índice vetorial |
| copilot\_sessions, copilot\_messages e copilot\_actions | Conversas e ações pendentes de aprovação | tenant\_id, usuário, status | Ação só executa se aprovada |

### 8.3 Plano de contas do ledger

Contas do lojista (credit-normal): a receber e disponível. Contas da plataforma: compensação (clearing), receita de taxas e estornos a pagar.

Exemplo, cobrança paga de R$ 100,00 com taxa de 0,99%:

| Evento | Débito | Crédito |
| --- | --- | --- |
| Pagamento confirmado | Compensação R$ 100,00 | A receber (lojista) R$ 99,01 e Taxas R$ 0,99 |
| Liquidação D+1 | A receber R$ 99,01 | Disponível R$ 99,01 |
| Estorno de R$ 30,00 | Disponível R$ 30,00 | Compensação R$ 30,00 |

O saldo do lojista é a soma dos créditos menos a soma dos débitos nas contas dele. Em todos os casos, a soma dos débitos é igual à dos créditos.

## 9. Contrato da API e dos webhooks

### 9.1 Convenções

- Base `/v1`, JSON em UTF-8. Autenticação por `Authorization: Bearer sk_test_...`.
- Valores em centavos no campo `amount` e moeda no campo `currency`. Datas em ISO 8601, UTC.
- Cabeçalho `Idempotency-Key` em operações que criam recursos. Cabeçalho `X-Request-Id` devolvido em toda resposta e usado como correlation id.
- Paginação por cursor (`limit` e `starting_after`).
- Erros no formato problem+json, inspirado na RFC 9457, com tipo, título, detalhe e o id da requisição. Códigos usados: 400, 401, 403, 404, 409, 422, 429 e 5xx.
- Limite de requisições por chave, com cabeçalhos informando o restante. Versionamento por caminho (v1); mudanças incompatíveis geram v2.

### 9.2 Endpoints principais

| Método | Rota | Descrição | Idempotente |
| --- | --- | --- | --- |
| POST | /v1/charges | Cria cobrança Pix | Sim, com chave |
| GET | /v1/charges e /v1/charges/{id} | Lista e consulta | Sim |
| POST | /v1/charges/{id}/cancel | Cancela cobrança pendente | Sim |
| POST | /v1/charges/{id}/refunds | Cria estorno total ou parcial | Sim, com chave |
| GET | /v1/refunds/{id} | Consulta estorno | Sim |
| POST e GET | /v1/customers | Cria e lista clientes | POST com chave |
| GET | /v1/balance | Saldos a liquidar e disponível | Sim |
| GET | /v1/ledger/entries | Extrato | Sim |
| POST, GET e DELETE | /v1/webhook\_endpoints | Gerencia endpoints | POST com chave |
| GET | /v1/events e /v1/webhook\_deliveries | Eventos e tentativas | Sim |
| POST | /v1/events/{id}/resend | Reenvia evento | Sim |
| POST, GET e DELETE | /v1/api\_keys | Gerencia chaves | POST com chave |
| GET e POST | /v1/risk/reviews e /v1/risk/reviews/{id}/decision | Fila de revisão e decisão | Decisão idempotente |
| POST | /v1/copilot/chat | Conversa com o copiloto, em streaming | Não |
| POST | /v1/copilot/actions/{id}/approve | Aprova ação sugerida | Sim |
| POST | /psp/callbacks | Callback do PSP simulado (interno, autenticado) | Sim |

A API administrativa (`/admin/...`) cuida de lojistas e políticas de taxa e não faz parte do contrato público.

### 9.3 Exemplo de cobrança criada

```json
{
  "id": "chg_8f2a91",
  "object": "charge",
  "amount": 12990,
  "currency": "BRL",
  "status": "pending",
  "payment_method": {
    "type": "pix",
    "copy_paste": "000201...",
    "expires_at": "2026-11-06T15:30:00Z"
  },
  "metadata": { "order_id": "A-1042" },
  "created_at": "2026-11-06T15:00:00Z"
}
```

### 9.4 Webhooks

**Catálogo de eventos:** charge.created, charge.in\_review, charge.paid, charge.failed, charge.expired, charge.canceled, refund.created, refund.succeeded, refund.failed e risk.review\_required.

**Envelope:**

```json
{
  "id": "evt_31c0de",
  "type": "charge.paid",
  "api_version": "2026-11-01",
  "sequence": 3,
  "created_at": "2026-11-06T15:03:10Z",
  "data": { "object": { "id": "chg_8f2a91", "status": "paid" } }
}
```

**Assinatura (RN-11):** o receptor calcula o HMAC-SHA256 do texto `timestamp.corpo` com o segredo do endpoint, compara em tempo constante e rejeita timestamps com mais de 5 minutos.

**Boas práticas para quem recebe (também documentadas para os integradores):** responder 2xx rápido e processar de forma assíncrona; deduplicar pelo id do evento; usar o campo sequence para tratar eventos fora de ordem; validar sempre a assinatura.

## 10. Fluxos principais

### 10.1 Criação e pagamento de uma cobrança Pix

1. O sistema do lojista chama `POST /v1/charges` com `Idempotency-Key`.
2. A API autentica a chave, valida o corpo e verifica a idempotência no Redis e no banco.
3. Em uma única transação, grava a cobrança como pending, o histórico de estado e o evento `charge.created` no outbox.
4. A API devolve 201 com o código Pix copia e cola. A resposta é guardada para repetições da mesma chave.
5. Um worker publica o evento no RabbitMQ.
6. O PSP simulado, depois do atraso configurado, chama o callback autenticado informando o pagamento.
7. O core-api registra `payment.received`. O risk-service calcula o score.
8. Com score baixo, a cobrança vai para paid e o ledger grava a transação balanceada (pagamento, taxa). Com score médio, vai para in\_review. Com score alto, vai para failed.
9. O evento público correspondente (`charge.paid`, `charge.in_review` ou `charge.failed`) é entregue por webhook.

&#91;embedded content: sequência · criação e pagamento de uma cobrança Pix\]

O lojista recebe o código Pix na hora (201), e só depois de o PSP simulado avisar o pagamento e o risk-service calcular o score o core-api marca a cobrança como paid e avisa o lojista por webhook. O caminho de score médio ou alto é o da seção 10.4.

### 10.2 Estorno

1. O lojista chama `POST /v1/charges/{id}/refunds` com valor e chave de idempotência.
2. A API trava a cobrança, valida o estado, a soma dos estornos e o saldo disponível.
3. Em uma transação, grava o estorno, os contra-lançamentos do ledger e o evento no outbox.
4. O estorno muda para succeeded, a cobrança vai para partially\_refunded ou refunded, e os webhooks `refund.created` e `refund.succeeded` são entregues.

### 10.3 Entrega de webhook com retentativas

1. O dispatcher consome o evento público e, para cada endpoint inscrito, cria uma entrega.
2. Faz o POST com assinatura e timestamp, com limite de 10 segundos.
3. Resposta 2xx marca a entrega como entregue. Qualquer outra resposta, ou falha de rede, agenda a próxima tentativa conforme a RN-09.
4. Se o circuit breaker do endpoint estiver aberto, a tentativa espera em vez de consumir capacidade.
5. Esgotadas as tentativas, o evento vai para a DLQ e o lojista é avisado no painel. O reenvio manual reinicia o ciclo.

### 10.4 Revisão manual de risco

1. Com score entre 50 e 79, a cobrança fica em in\_review e o evento `risk.review_required` é emitido.
2. O analista abre a fila no painel, vê os sinais e a explicação gerada pela IA.
3. A decisão (aprovar ou rejeitar) é idempotente, registrada em auditoria e leva a cobrança para paid ou failed, com os eventos correspondentes.

### 10.5 Copiloto diagnosticando um webhook que falhou

1. A integradora pergunta no painel: por que meu webhook não chegou?
2. O ai-service mascara dados pessoais e classifica a intenção.
3. O agente busca na documentação (RAG) e consulta, por ferramentas somente leitura, os eventos e as tentativas de entrega do lojista.
4. Se encontra a causa (por exemplo, respostas 500 do receptor), explica e cita as fontes. Se sugere reenviar o evento, cria uma ação pendente.
5. A ação só executa depois de a pessoa aprovar no painel.

## 11. Inteligência artificial

### 11.1 Princípios

- A IA fica fora do caminho crítico do pagamento: ela explica e auxilia, nunca aprova ou nega uma transação sozinha.
- Toda resposta é fundamentada em fontes citadas. Sem fonte, a resposta é que não há informação suficiente.
- Privilégio mínimo: as ferramentas têm escopo por lojista, imposto no código e não no prompt.
- Uma pessoa decide qualquer ação que altere estado.

### 11.2 Casos de uso do MVP

| Caso | Descrição | Prioridade |
| --- | --- | --- |
| Copiloto do integrador | Perguntas sobre a API, diagnóstico de webhooks e de erros | S |
| Explicação de risco | Texto curto que explica ao analista por que uma cobrança foi sinalizada | C |

Disputas e chargebacks ficam fora do MVP.

### 11.3 Copiloto: grafo de execução (LangGraph)

| Nó | Função |
| --- | --- |
| classificar | Identifica a intenção: dúvida de documentação, diagnóstico ou fora de escopo |
| recuperar | Busca trechos relevantes na base (RAG) |
| escolher ferramenta | Decide se precisa consultar eventos ou tentativas do lojista |
| ferramentas de leitura | buscar\_eventos, listar\_tentativas, consultar\_cobranca |
| propor ação | Quando cabe, cria uma ação pendente (por exemplo, reenviar\_evento) |
| aguardar aprovação | Interrompe o fluxo até a pessoa aprovar ou recusar |
| responder | Gera a resposta com citações e valida o formato de saída |

O estado da conversa é persistido no PostgreSQL, o que permite retomar o fluxo após a aprovação.

### 11.4 RAG

- **Fontes:** documentação do projeto, OpenAPI, catálogo de eventos e um FAQ de erros comuns escrito para o estudo.
- **Chunking:** por seção, de 300 a 500 tokens com sobreposição pequena. O tamanho final será calibrado com o conjunto de avaliação.
- **Embeddings:** modelo multilíngue local servido pelo Ollama, a definir na Fase 8 com testes comparativos.
- **Recuperação:** busca híbrida (vetorial no pgvector mais busca textual do PostgreSQL) e reranking opcional.
- **Citação e recusa:** cada afirmação aponta a fonte. Abaixo de um limiar de similaridade, o copiloto responde que não sabe.

### 11.5 Antifraude

Regras configuráveis mais um modelo simples de scikit-learn (por exemplo, Isolation Forest ou gradient boosting), treinado em dados sintéticos. O score final combina as duas partes. O LLM só redige a explicação. Como os dados são sintéticos, o modelo serve para estudar o pipeline (treino, versionamento, avaliação) e não para afirmar desempenho real.

### 11.6 Guardrails

- Mascaramento de dados pessoais antes de qualquer chamada a modelo (RN-14).
- Conteúdo de logs, eventos e mensagens de terceiros é tratado como entrada não confiável, para mitigar prompt injection: delimitação clara no prompt, instruções de sistema fixas e ferramentas que não obedecem a comandos vindos do conteúdo.
- Saída validada por esquema (Pydantic). Limites de tamanho, de passos por turno e de requisições por usuário.
- Segredos e chaves nunca entram no contexto do modelo.

### 11.7 Avaliação

- Conjunto de referência de cerca de 50 perguntas, com respostas esperadas e fontes, incluindo perguntas que o copiloto deve recusar.
- Métricas com RAGAs: faithfulness, answer relevancy, context precision e context recall.
- Casos de segurança: tentativas de injeção em eventos e logs, e de acesso a dados de outro lojista.
- O CI roda a avaliação e falha se uma métrica cair abaixo do limiar combinado. Dois modelos locais serão comparados.

### 11.8 Observabilidade da IA

Todas as chamadas são rastreadas no Langfuse: prompts, trechos recuperados, ferramentas usadas, latência, tokens e feedback (polegar para cima ou para baixo) dado no painel.

### 11.9 Limitações assumidas

Modelos locais pequenos têm qualidade irregular em português. Esse risco é tratado pela avaliação e pela possibilidade de trocar o provedor pela camada de abstração (ADR 007).

## 12. Stack e ferramentas

As versões exatas serão fixadas na Fase 0.

| Camada | Escolha | Por quê | Alternativa |
| --- | --- | --- | --- |
| Linguagem | Python 3.12 | Base do projeto e do mercado-alvo | Go |
| Núcleo | Django e Django REST Framework | ORM, admin, migrações e experiência prévia | FastAPI para tudo |
| Documentação da API | drf-spectacular (OpenAPI) | Contrato gerado do código | Escrita manual |
| Serviços satélites | FastAPI, Pydantic v2 | Async e validação por tipos | Flask |
| Tarefas | Celery | Maduro, com agendamento e retentativas | Dramatiq, RQ |
| Mensageria | RabbitMQ | Roteamento por tópicos, DLQ nativa | Kafka, SQS |
| Banco | PostgreSQL 16 com pgvector | Transações fortes, RLS e vetores no mesmo banco | MySQL e banco vetorial separado |
| Cache | Redis | Idempotência, rate limit, cache semântico | Memcached |
| Armazenamento | MinIO (compatível com S3) | Mesmo código local e na AWS | Disco local |
| IA | LangChain, LangGraph, Ollama | Fluxos com estado e aprovação, modelos locais gratuitos | LlamaIndex, API paga |
| Avaliação e rastreamento de IA | RAGAs, Langfuse | Métricas de RAG e traces de código aberto | Soluções pagas |
| Machine learning | scikit-learn | Modelos simples e bem conhecidos | PyTorch |
| Front | Next.js, TypeScript | Mercado e contraste com o back | Vite com React |
| Testes | pytest, Hypothesis, Testcontainers, Schemathesis, k6, Playwright | Cobrem unidade, propriedade, contrato, carga e ponta a ponta | Outras ferramentas |
| Qualidade | ruff, mypy, pre-commit, Conventional Commits | Padronização e feedback rápido | flake8, black |
| Contêineres | Docker, Docker Compose | Ambiente reproduzível | Podman |
| Kubernetes | kind ou k3d, Kustomize ou Helm, Argo CD (opcional) | Cluster local gratuito | Minikube |
| CI/CD | GitHub Actions | Integrado ao repositório | GitLab CI |
| Segurança | Trivy, pip-audit, gitleaks, Dependabot | Varredura de imagens, dependências e segredos | Ferramentas pagas |
| IaC | Terraform | Reprodutível e destruível | CloudFormation |
| Nuvem (janela final) | AWS: ECS Fargate, RDS, SQS e SNS, S3, Secrets Manager, CloudWatch | Mercado e estudo de mensageria gerenciada | Outro provedor |
| Observabilidade | OpenTelemetry, Prometheus, Grafana, Loki | Padrão aberto e pilha gratuita | Datadog |

## 13. Segurança e conformidade

### 13.1 Modelo de ameaças resumido

| Ameaça | Exemplo | Mitigação |
| --- | --- | --- |
| Falsificação de identidade | Uso de chave de API vazada | Hash das chaves, rotação, escopos, rate limit, revogação imediata |
| Adulteração | Webhook alterado no caminho | HMAC com segredo por endpoint e timestamp |
| Repetição (replay) | Reenvio de um webhook válido | Tolerância de 5 minutos, id de evento e idempotência no receptor |
| Vazamento entre lojistas | Consulta devolve dados de outro tenant | Row Level Security, tenant\_id obrigatório e testes de isolamento |
| Negação de serviço | Receptor lento ou flood de requisições | Circuit breaker, timeouts, rate limit e filas |
| Elevação de privilégio | Copiloto executa ação indevida | Aprovação humana, escopo imposto no código, validação de saída |
| Prompt injection | Texto malicioso em um evento ou log | Conteúdo não confiável isolado, ferramentas sem obediência ao conteúdo, testes de segurança |
| Cadeia de suprimentos | Dependência comprometida | Versões fixadas, pip-audit, Trivy, Dependabot |
| Repúdio | Negar uma ação sensível | Auditoria append-only com ator, horário e correlation id |

### 13.2 Controles gerais

- Checklist do OWASP API Security Top 10 revisado a cada fase de API.
- Segredos em variáveis de ambiente e, na nuvem, no Secrets Manager. Nada de segredos no repositório, com gitleaks no CI.
- Senhas com Argon2, CORS restrito, cabeçalhos de segurança e TLS nos ambientes expostos.
- Menor privilégio no banco: usuários separados para aplicação, migração e leitura.

### 13.3 Conformidade e limites

- **LGPD:** somente dados sintéticos, minimização de campos, mascaramento em logs e prompts.
- **PCI DSS:** fora de escopo, porque não há dados de cartão.
- **Aviso de sandbox:** o produto não é instituição de pagamento e não movimenta dinheiro real, e isso aparece no README, no painel e na documentação da API.

## 14. Observabilidade

- **Logs:** JSON estruturado com correlation id (o mesmo `X-Request-Id`, propagado também nas mensagens do broker), sem dados pessoais.
- **Métricas:** RED (taxa, erros e duração) por endpoint, mais métricas de negócio: cobranças por status, tamanho e idade do outbox, tamanho das filas e da DLQ, taxa de sucesso das entregas de webhook e resultado da verificação do ledger.
- **Tracing:** OpenTelemetry acompanhando uma requisição da API até o consumidor, passando pelo outbox e pela fila, com o contexto do trace nos cabeçalhos das mensagens.
- **Painéis:** API, filas, ledger, webhooks e IA.

### 14.1 Indicadores e objetivos (SLOs)

Os objetivos valem para o ambiente de estudo e servem para treinar a prática. Serão calibrados depois da Fase 9.

| Indicador | Objetivo |
| --- | --- |
| Disponibilidade da API em testes de carga | 99% das requisições sem erro 5xx |
| Latência de criação de cobrança | p95 abaixo de 300 ms |
| Atraso do outbox | p95 abaixo de 5 s |
| Entrega de webhook (receptor saudável) | 99% em até 60 s |
| Violações do invariante do ledger | 0 |
| Latência do copiloto (modelo local) | p95 abaixo de 15 s |

Alertas de exemplo: DLQ com mensagens, outbox atrasado por mais de 1 minuto, falha do job de verificação do ledger e fila acumulando.

## 15. Estratégia de testes

| Tipo | O que cobre | Ferramenta | Quando roda |
| --- | --- | --- | --- |
| Unidade | Regras de domínio, máquina de estados, cálculo de taxa | pytest | Todo pull request |
| Propriedade | Ledger balanceado, saldo coerente, transições válidas | Hypothesis | Todo pull request |
| Integração | Banco, broker e cache reais | pytest com Testcontainers | Todo pull request |
| Contrato | API contra o OpenAPI e eventos contra o JSON Schema | Schemathesis e validação de schemas | Todo pull request |
| Concorrência | Idempotência e estornos simultâneos | pytest com threads ou asyncio | Todo pull request |
| Resiliência | Broker fora do ar, receptor lento, mensagem duplicada | Testes de caos simples | Diário |
| Carga | Latência e vazão | k6 | Sob demanda e diário |
| Ponta a ponta | Loja exemplo e painel | Playwright | Diário |
| Segurança | Isolamento entre lojistas, assinatura, injeção de prompt | pytest e casos de avaliação | Todo pull request |
| IA | Qualidade do RAG e recusas corretas | RAGAs | Diário e antes de releases |

**Portões de qualidade no CI:** lint e tipagem sem erros, cobertura mínima no domínio, testes verdes, varredura de segurança sem achados críticos e avaliação de IA acima dos limiares.

## 16. Infraestrutura, ambientes e CI/CD

### 16.1 Ambientes

| Ambiente | Como roda | Objetivo |
| --- | --- | --- |
| Local | Docker Compose, com `task up` | Desenvolvimento diário |
| CI | GitHub Actions | Validar cada mudança |
| Kubernetes local | kind ou k3d | Estudar manifests, escala e observabilidade |
| Nuvem (janela final) | AWS com Terraform | Estudar deploy real, por 1 a 2 semanas |

### 16.2 Pipeline de CI/CD

1. Lint e tipagem (ruff, mypy).
2. Testes de unidade, propriedade, integração e contrato.
3. Build das imagens.
4. Varreduras: Trivy (imagens), pip-audit (dependências) e gitleaks (segredos).
5. Avaliação de IA, nos casos em que a mudança afeta o copiloto.
6. Publicação das imagens no registro.
7. Deploy: no cluster local com Kustomize ou Helm; na janela de nuvem, com Terraform e ECS.

Fluxo de trabalho: trunk-based, com ramos curtos e pull requests pequenos, mesmo trabalhando sozinho, para treinar o processo.

### 16.3 Mapeamento para a AWS

| Componente | Local | AWS (janela final) |
| --- | --- | --- |
| core-api, workers, serviços | Contêineres no Compose e no kind | ECS Fargate |
| Mensageria | RabbitMQ | SQS e SNS, por meio do adaptador (ADR 002) |
| Banco | PostgreSQL com pgvector | RDS para PostgreSQL |
| Cache | Redis em contêiner | ElastiCache ou Redis em contêiner, conforme o custo |
| Armazenamento | MinIO | S3 |
| Segredos | Variáveis de ambiente | Secrets Manager |
| Logs e métricas | Prometheus, Grafana, Loki | CloudWatch |
| LLM | Ollama | Provavelmente desativado ou reduzido na janela, pelo custo de computação |

### 16.4 Custos

- Em desenvolvimento, o custo é zero.
- Antes de criar a conta na AWS, conferir as condições vigentes do plano gratuito, incluindo prazo e créditos. A conta só deve ser criada quando o projeto estiver pronto para o deploy.
- Definir um orçamento com alerta (AWS Budgets) antes de criar qualquer recurso, e destruir tudo com `terraform destroy` ao fim da janela.

## 17. Fases e cronograma

Estimativa para uma pessoa com cerca de 10 horas por semana. Os prazos são referências e serão ajustados ao fim de cada fase.

| Fase | Objetivo | Entregáveis principais | Critério de pronto | Semanas |
| --- | --- | --- | --- | --- |
| 0 | Fundação | Repositório, Compose, CI básico, ADRs iniciais, versões fixadas | `task up` sobe tudo; pipeline verde | 1 |
| 1 | Núcleo e multi-tenancy | Tenants, usuários, chaves de API, RLS, auditoria, esqueleto do OpenAPI | Teste prova isolamento entre lojistas | 2 |
| 2 | Cobranças e idempotência | Cobranças, máquina de estados, payload Pix, Idempotency-Key | Teste de concorrência sem duplicidade | 2 |
| 3 | Ledger e saldos | Ledger, projeção de saldos, liquidação, extrato | Testes de propriedade sem violar o invariante | 2 |
| 4 | PSP e eventos | PSP simulado, outbox, RabbitMQ, log de eventos | Pagamento ponta a ponta até o ledger | 2 |
| 5 | Webhooks | Endpoints, assinatura, retentativas, DLQ, histórico, reenvio, circuit breaker | Falhas simuladas se recuperam sozinhas | 2 |
| 6 | Estornos e antifraude | Estornos, risk-service, fila de revisão | Estorno concorrente consistente; revisão funcionando | 2 |
| 7 | Painel e loja exemplo | Next.js, loja que integra pela API | Compra ponta a ponta pela loja exemplo | 2 |
| 8 | Copiloto de IA | RAG, LangGraph, aprovação humana, avaliação, Langfuse | Métricas acima dos limiares; injeção bloqueada | 3 |
| 9 | Observabilidade e carga | OpenTelemetry, painéis, alertas, testes k6 | SLOs medidos e documentados | 1 |
| 10 | CI/CD e Kubernetes | Pipeline completo, manifests, cluster local | Deploy automatizado no kind | 2 |
| 11 | Nuvem e acabamento | Terraform, deploy na AWS, vídeo de demonstração, posts, teardown | Demonstração no ar e ambiente destruído depois | 2 |

Total aproximado: 23 semanas (cerca de 5 a 6 meses). Se for preciso reduzir, o menor recorte publicável é: Fases 0 a 5, uma versão simples da Fase 7 e a Fase 8.

**Marcos para divulgar:** fim da Fase 3 (ledger e idempotência), fim da Fase 5 (webhooks confiáveis), fim da Fase 8 (copiloto com avaliação) e fim da Fase 11 (demonstração completa).

&#91;embedded content: roadmap · 12 fases e 4 marcos\]

As fases seguem da esquerda para a direita e continuam na linha de baixo. Cada losango marca um ponto em que dá para publicar o que já existe. A Fase 6 inclui o antifraude, e a Fase 4 inclui o outbox e a mensageria.

## 18. Riscos

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| O escopo crescer além do MVP | Alta | Alto | Itens fora de escopo explícitos e mudanças só por nova versão do documento |
| Falta de tempo semanal | Média | Alto | Fases com entregas pequenas e recorte mínimo definido |
| Complexidade do ledger e da concorrência | Média | Alto | Testes de propriedade e de concorrência desde a Fase 2 |
| Qualidade do LLM local em português | Média | Médio | Avaliação, comparação de modelos e abstração de provedor |
| Computador sem memória suficiente para Ollama e o cluster | Média | Médio | Modelos menores, rodar o cluster só quando necessário |
| Custo ou expiração do plano gratuito na nuvem | Média | Médio | Conta criada só no fim, orçamento com alerta e destruição ao terminar |
| Dados reais entrarem por engano | Baixa | Alto | Gerador de dados sintéticos, revisão no CI e política no README |
| Perda de motivação no meio do caminho | Média | Alto | Marcos publicáveis a cada poucas fases |
| Versões de dependências quebrando o projeto | Média | Baixo | Versões fixadas e Dependabot |
| PSP simulado muito diferente da realidade | Média | Baixo | Documentar as diferenças e usar o formato real do BR Code |

## 19. Convenções do repositório e do processo

### 19.1 Estrutura

```
paylab/
  apps/
    core-api/        Django + DRF
    psp-simulator/   FastAPI
    risk-service/    FastAPI
    ai-service/      FastAPI + LangGraph
    web/             Next.js
    demo-store/      Next.js
  infra/
    compose/ k8s/ terraform/
  docs/
    adr/ api/ runbooks/
  .github/workflows/
  Taskfile.yml  README.md  .env.example
```

### 19.2 Processo

- Commits no padrão Conventional Commits. Pull requests pequenos, com modelo de PR (o que mudou, por quê, como testar).
- ADR para toda decisão relevante, usando um modelo curto (contexto, decisão, consequências).
- Versionamento semântico para a API e para os eventos.
- Código e identificadores em inglês; documentação em português.
- Licença do repositório: MIT.

### 19.3 Definição de pronto (por tarefa)

Testes escritos e verdes, tipagem e lint sem erros, OpenAPI e documentação atualizados, sem segredos no código, métricas e logs adequados, e revisão de segurança quando houver mudança de API ou de dados.

## 20. Pontos em aberto e aprovação

### 20.1 Decisões pendentes

| Item | Situação |
| --- | --- |
| Nome definitivo do projeto | Provisório: PayLab |
| Memória do computador e modelos do Ollama | A informar, para escolher os modelos de geração e de embeddings |
| Taxa devolvida ou não em um estorno | Proposta: não devolver (RN-08) |
| Biblioteca de interface do painel | A definir na Fase 7 |
| Licença do repositório | Definida: MIT |
| Escolha de Helm ou Kustomize | A definir na Fase 10 |
| Armazenamento S3 local (substituto do MinIO) | O MinIO deixou de publicar imagens gratuitas e foi arquivado em 2026. Escolher entre SeaweedFS, Garage ou RustFS na Fase 8 (ADR 0012). Onde o documento cita MinIO, leia como armazenamento compatível com S3. |

### 20.2 Checklist de aprovação

- [ ] A visão, o escopo e o que está fora de escopo estão corretos.
- [ ] Os requisitos funcionais e não funcionais representam o que será construído.
- [ ] As regras de negócio e a máquina de estados estão claras.
- [ ] A arquitetura, os padrões e as decisões (ADRs) fazem sentido.
- [ ] O modelo de dados e o plano de contas do ledger estão adequados.
- [ ] O contrato da API e dos webhooks está adequado.
- [ ] As fases e o cronograma são realistas para o tempo disponível.
- [ ] Os riscos e as pendências estão aceitos.

Após a aprovação, a versão 1.0 do documento é publicada e começa a Fase 0.

## 21. Referências

- Banco Central do Brasil: manual de padrões para iniciação do Pix (formato do BR Code).
- RFC 9457: problem details para APIs HTTP.
- OWASP API Security Top 10.
- Modelo C4 para documentação de arquitetura.
- The Twelve-Factor App.
- Documentação de idempotência e de webhooks de grandes gateways de pagamento, como inspiração.
- Documentação oficial de Django, DRF, FastAPI, LangGraph, RAGAs, Langfuse, RabbitMQ e PostgreSQL (pgvector e Row Level Security).
