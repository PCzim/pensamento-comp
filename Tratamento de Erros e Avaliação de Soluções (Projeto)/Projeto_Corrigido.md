# Projeto Corrigido – UrbanTrack

> Versão revisada do projeto com as correções aplicadas a cada erro identificado em `Erros_Identificados.md`. Cada item traz o trecho original, a versão corrigida e a justificativa da mudança.

---

## Correções de Sintaxe / Documentação

---

### Correção ERRO-S01 — Intervalo de atualização GPS

**Antes:**
> "Recepção e armazenamento das coordenadas dos veículos em intervalos regulares."

**Depois:**
> "Recepção e armazenamento das coordenadas dos veículos em intervalos de **10 segundos** durante operação ativa. Quando o veículo fica parado por mais de 5 minutos, o intervalo passa para **60 segundos**, reduzindo o volume de dados gerados sem perder a visibilidade. O intervalo mínimo configurável é de **5 segundos**, para situações que exigem maior precisão."

**Justificativa:**
Sem um número, essa linha não serve para nada na prática. O intervalo de 10 segundos foi escolhido por ser um equilíbrio razoável entre precisão e volume de dados — com 200 veículos operando 18 horas por dia, já chegamos a cerca de 1,3 milhão de registros GPS diários. O modo adaptativo (ativo vs. espera) é uma prática comum em sistemas de rastreamento e evita desperdício de armazenamento e banda quando o veículo não está se movendo.

---

### Correção ERRO-S02 — Metodologia de desenvolvimento

**Antes:**
> "Metodologia: Nenhuma em específico foi aplicada"

**Depois:**
> "Metodologia: **desenvolvimento incremental informal**, com algumas práticas de metodologias ágeis aplicadas de forma não estruturada:
> - Controle de versão via Git/GitHub com separação por funcionalidade
> - Entregas organizadas por módulos independentes
> - Revisão da documentação a cada novo artefato produzido
>
> Nenhuma metodologia formal (Scrum, Kanban, XP) foi seguida integralmente, mas o uso de controle de versão e a decomposição modular já caracterizam um processo minimamente estruturado."

**Justificativa:**
Dizer "nenhuma metodologia" enquanto se usa GitHub e organiza o trabalho em módulos é uma contradição. Descrever o processo como ele de fato aconteceu — mesmo que informal — é mais honesto e mais útil para quem precisar entender como o projeto foi conduzido.

---

### Correção ERRO-S03 — Diagrama UML com identificação do tipo

**Antes:**
> "Diagrama simplificado em UML representando os três atores principais: Veículo, Operador e Passageiro."

**Depois:**
> "**Diagrama de Casos de Uso (UML)** representando os três atores principais e suas interações com o sistema:
>
> - **Veículo** → envia posição GPS; recebe comandos de rota
> - **Operador** → monitora a frota; emite alertas; gerencia rotas; acessa relatórios
> - **Passageiro** → consulta ETA; verifica lotação; recebe avisos de desvio
>
> O arquivo está disponível em `Diagrama.png`. Para o fluxo interno entre os módulos, ver `Design.md`."

**Justificativa:**
Identificar o tipo de diagrama é o mínimo para que ele seja útil. O Diagrama de Casos de Uso foi escolhido por ser adequado para a fase conceitual — ele mostra o que o sistema faz e para quem, sem precisar entrar em detalhes de como.

---

### Correção ERRO-S04 — Pasta `src/` com contexto

**Antes:**
```
└── src/     # (Não foi realizado)
```

**Depois:**
```
└── src/     # [FASE FUTURA] Ainda não implementado.
             # Ordem de prioridade para a próxima entrega:
             # 1. gps_collector.py   → coleta de dados GPS
             # 2. eta_calculator.py  → cálculo de ETA
             # 3. alert_manager.py   → gerenciador de alertas
             # 4. api/               → endpoints REST para os painéis
```

**Justificativa:**
Um diretório vazio sem contexto não comunica nada — nem o estado atual nem o que vem a seguir. Indicar a ordem de prioridade transforma essa linha em um mini-roadmap, que é mais útil do que um simples "não foi feito".

---

## Correções de Lógica

---

### Correção ERRO-L01 — Algoritmo de ETA com casos extremos

**Antes:**
> "Baseado em velocidade média histórica por trecho + dados de tráfego em tempo real."

**Depois:**

```
ALGORITMO CalcularETA(veiculo, parada_destino):

  1. Obter posição_atual e velocidade_atual do veículo

  2. SE velocidade_atual == 0 E tempo_parado < 3 minutos:
       ETA = (distancia_restante / velocidade_media_historica_trecho) + tempo_parado
       status = "ESTIMATIVA COM VEÍCULO PARADO"

  3. SE velocidade_atual == 0 E tempo_parado >= 3 minutos:
       ETA = INDEFINIDO
       status = "VEÍCULO POSSIVELMENTE EM PANE OU PARADA TÉCNICA"
       emitir_alerta(tipo=OPERACIONAL, prioridade=ALTA)

  4. SE dados_trafego_disponíveis == FALSO:
       ETA = distancia_restante / velocidade_media_historica_trecho
       status = "ESTIMATIVA SEM DADOS DE TRÁFEGO (PODE VARIAR)"

  5. SE historico_insuficiente (< 7 dias de dados):
       usar velocidade_media_padrao_categoria_veiculo como fallback
       status = "ESTIMATIVA BASEADA EM MÉDIA PADRÃO"

  6. SE veiculo FORA DA ROTA cadastrada:
       ETA = INDEFINIDO
       status = "VEÍCULO FORA DE ROTA — ETA INDISPONÍVEL"
       acionar detecção de desvio (ver ERRO-L02)

  7. SENÃO (caso normal):
       ETA = (distancia_restante / velocidade_atual) * fator_correcao_trafego
       status = "ESTIMATIVA NORMAL"

  RETORNAR (ETA, status)
```

**Justificativa:**
O algoritmo original só funcionava quando tudo corria bem. O problema mais grave era a divisão por zero quando o veículo está parado (distância / velocidade = 0), que quebraria o sistema. Os demais cenários garantem que o sistema continue funcionando em modo degradado — com um aviso claro ao usuário sobre o nível de confiabilidade da estimativa.

---

### Correção ERRO-L02 — Buffer geométrico com parâmetros definidos

**Antes:**
> "Comparação entre coordenada atual e buffer geométrico da rota cadastrada."

**Depois:**
> **Parâmetros do algoritmo de detecção de desvio:**
>
> - **Raio do buffer:** 50 metros em zona urbana densa / 150 metros em zona periférica ou rural
> - **Tolerância a ruído GPS:** desvios abaixo de 20 metros são ignorados (margem de erro típica de GPS em ambiente urbano)
> - **Confirmação de desvio:** o veículo precisa estar fora do buffer em pelo menos **3 leituras consecutivas** (30 segundos) para disparar o alerta — evitando falsos positivos por oscilação de sinal
> - **Prazo do alerta:** emitido em até **45 segundos** após a confirmação
> - **Exceções:** manobras em terminais e pátios têm zonas de buffer expandidas, cadastradas manualmente pelo operador

**Justificativa:**
A técnica de exigir 3 leituras consecutivas antes de disparar o alerta vem de sistemas de monitoramento industrial — é o que chamam de "debouncing". Sem isso, qualquer pico momentâneo de sinal geraria um falso alerta. Os valores de 50m e 150m refletem a diferença real entre a precisão de GPS em área urbana densa (onde prédios interferem no sinal) e em áreas mais abertas.

---

### Correção ERRO-L03 — Fila de alertas completa

**Antes:**
> "Fila com prioridade baseada no tipo e urgência do evento (falha mecânica > atraso > lotação)."

**Depois:**

| Prioridade | Tipo de Evento | Quando dispara | Quem recebe |
|---|---|---|---|
| P0 — CRÍTICO | Acidente / Emergência | Acionamento manual pelo motorista | Operador + Central de Emergência |
| P0 — CRÍTICO | Perda total de sinal GPS (> 5 min) | Automático | Operador |
| P1 — ALTO | Falha mecânica | Sensor ou relato do motorista | Operador + Manutenção |
| P1 — ALTO | Desvio de rota confirmado | 3 leituras fora do buffer | Operador |
| P2 — MÉDIO | Atraso acima de 10 minutos | ETA vs. horário programado | Operador + Passageiros da linha |
| P2 — MÉDIO | Veículo parado > 3 min sem justificativa | Automático | Operador |
| P3 — BAIXO | Lotação máxima atingida | Sensor de ocupação | Passageiros (aviso na interface) |
| P3 — BAIXO | Atraso entre 3 e 10 minutos | ETA vs. horário programado | Passageiros (aviso na interface) |

> **Empate:** dois eventos de mesma prioridade são ordenados pelo horário de detecção (o mais antigo primeiro). Eventos P0 interrompem qualquer processamento em andamento.

**Justificativa:**
A versão original ignorava acidentes e perda de sinal GPS — dois dos cenários mais críticos em um sistema de transporte público. A definição de limiares numéricos (10 minutos para P2, 3 minutos para parada suspeita) elimina a subjetividade e permite que o sistema tome decisões sem depender de interpretação humana a cada evento.

---

### Correção ERRO-L04 — Estratégia de consistência de dados

**Antes:**
> Problema identificado, mas sem solução proposta.

**Depois:**
> **Estratégia adotada: arquitetura orientada a eventos com consistência eventual**
>
> - O módulo de Coleta GPS publica eventos em um **barramento de mensagens** (ex: Apache Kafka ou Redis Pub/Sub)
> - O Painel do Operador e a Interface do Passageiro consomem os eventos de forma independente, a partir do mesmo barramento
> - A defasagem máxima tolerada entre os dois é de **2 segundos**
> - Se um consumidor falhar, ele retoma o processamento a partir do último evento confirmado — sem perder dados
> - O estado do veículo exibido é sempre o último evento publicado, nunca uma consulta direta ao banco

**Justificativa:**
A arquitetura de pub/sub (publicar/assinar) é a forma mais comum de resolver esse problema em sistemas distribuídos. O principal benefício é o desacoplamento: o módulo GPS não precisa saber que existem painéis consumindo os dados, e a falha de um consumidor não afeta os outros. Isso também facilita escalar cada parte do sistema de forma independente no futuro.

---

## Correções de Execução / Planejamento

---

### Correção ERRO-E01 — Plano de fallback para APIs externas

**Antes:**
> Dependências de APIs externas sem nenhum plano de contingência.

**Depois:**
> **O que acontece quando cada API cai:**
>
> | API | Comportamento do sistema |
> |---|---|
> | Google Maps / HERE (tráfego) | ETA calculado só com velocidade histórica; usuário é informado que a estimativa pode variar |
> | Sistema municipal de frotas | UrbanTrack opera com dados próprios; sincronização retomada automaticamente quando disponível |
> | GPS do veículo | Última posição conhecida mantida por até 60 segundos; depois disso, veículo marcado como "sem sinal" |
>
> **Princípio:** o sistema nunca falha por completo — ele opera com funcionalidade reduzida e avisa o usuário sobre o que está indisponível.

**Justificativa:**
Em serviços públicos, não existe justificativa para o sistema inteiro parar porque uma API terceira ficou fora do ar. Esse princípio — operar de forma degradada em vez de falhar completamente — é chamado de *graceful degradation* e é padrão em sistemas críticos.

---

### Correção ERRO-E02 — Arquitetura para sustentar 99,9% de disponibilidade

**Antes:**
> Meta de 99,9% declarada sem nenhuma estratégia técnica.

**Depois:**
> **O que é necessário para atingir 99,9% de disponibilidade:**
>
> - **Servidores redundantes:** pelo menos 2 instâncias de cada serviço em zonas diferentes, operando ao mesmo tempo
> - **Balanceamento de carga:** requisições direcionadas automaticamente para instâncias saudáveis
> - **Verificação de saúde:** checagem automática a cada 30 segundos; instâncias com problema são retiradas do ar automaticamente
> - **Banco de dados:** configuração primary-replica com troca automática em até 30 segundos em caso de falha
> - **Monitoramento:** alertas quando latência passar de 500ms ou taxa de erros ultrapassar 1%
> - **Atualizações sem downtime:** deploy incremental ou em horário de baixo uso (00h–04h)
> - **Plano de recuperação:** tempo máximo para voltar ao ar (RTO): 15 minutos; dados perdidos no máximo dos últimos 5 minutos (RPO)

**Justificativa:**
99,9% de disponibilidade são no máximo 8,7 horas de downtime por ano. Sem redundância, qualquer reinicialização de servidor ou falha de hardware já ultrapassa esse limite. As práticas listadas são o mínimo para um serviço classificado como infraestrutura pública.

---

### Correção ERRO-E03 — Controles de segurança definidos

**Antes:**
> Menção ao princípio de menor privilégio, sem controles concretos.

**Depois:**
> **Controles de segurança por camada:**
>
> **Acesso e autenticação:**
> - Operadores autenticados via JWT com expiração de 8 horas
> - Interface do passageiro: acesso público, sem login
> - Perfis definidos: `PASSAGEIRO` / `OPERADOR` / `GESTOR` / `ADMIN`
> - Cada perfil acessa apenas o que é necessário para sua função (princípio do menor privilégio aplicado de fato)
>
> **Proteção dos dados:**
> - Toda comunicação via HTTPS (TLS 1.2+)
> - Dados GPS armazenados com criptografia em repouso (AES-256)
> - Logs de acesso ao painel do operador retidos por 90 dias
>
> **LGPD:**
> - Dados de localização dos veículos são operacionais, não pessoais — mas logs de acesso dos operadores são tratados como dados pessoais
> - Dados GPS brutos descartados após 180 dias; relatórios agregados mantidos indefinidamente
>
> **Proteção contra ataques:**
> - Limite de 60 requisições por minuto por IP nas APIs públicas
> - Validação de entrada em todos os endpoints
> - Proteção contra SQL Injection via ORM parametrizado

**Justificativa:**
Citar o princípio de menor privilégio sem implementá-lo é como colocar um aviso de segurança na porta e deixar a janela aberta. A LGPD se aplica aqui porque o sistema envolve dados de serviço público e logs de operadores (pessoas físicas). O não cumprimento pode resultar em multas além dos riscos operacionais.

---

## Resumo das correções

| Código | Tipo | Principal mudança |
|---|---|---|
| ERRO-S01 | Sintaxe | Intervalo GPS definido: 10s (ativo) / 60s (espera) |
| ERRO-S02 | Sintaxe | Metodologia descrita como incremental informal |
| ERRO-S03 | Sintaxe | Tipo de diagrama UML especificado (Casos de Uso) |
| ERRO-S04 | Sintaxe | `src/` com ordem de prioridade de implementação |
| ERRO-L01 | Lógica | Algoritmo de ETA com 6 cenários de fallback |
| ERRO-L02 | Lógica | Buffer de 50m/150m com confirmação em 3 leituras |
| ERRO-L03 | Lógica | 8 tipos de alerta com limiares numéricos definidos |
| ERRO-L04 | Lógica | Arquitetura pub/sub com consistência eventual |
| ERRO-E01 | Execução | Política de fallback para cada API dependente |
| ERRO-E02 | Execução | Arquitetura ativo-ativo com plano de recuperação |
| ERRO-E03 | Execução | Controles de segurança por camada + conformidade LGPD |
