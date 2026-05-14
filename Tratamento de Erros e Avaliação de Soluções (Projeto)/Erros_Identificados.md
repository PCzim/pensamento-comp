# Erros Identificados – UrbanTrack

> O projeto está em fase conceitual, sem código implementado. Os erros foram identificados na documentação, nos algoritmos descritos e nas decisões de arquitetura — e classificados nas três categorias da atividade, adaptadas a esse contexto.

---

## Categoria 1 — Erros de Sintaxe / Documentação

Problemas de inconsistência ou falta de informação na documentação. Em um projeto real, esse tipo de erro gera interpretações diferentes entre os membros da equipe e pode comprometer a implementação.

---

### ERRO-S01 — Intervalo de atualização GPS sem definição

**Localização:** Módulo "Coleta de Dados GPS"

**Trecho problemático:**
> "Recepção e armazenamento das coordenadas dos veículos em intervalos regulares."

**Problema:**
"Intervalos regulares" não diz nada. Sem um número, não é possível dimensionar o banco de dados, calcular o consumo de rede dos modems GPS ou definir a precisão do ETA. 1 segundo e 30 segundos têm impactos completamente diferentes na infraestrutura.

**Impacto:** Alto — afeta diretamente a arquitetura de dados e o cálculo de ETA.

---

### ERRO-S02 — Contradição entre "sem metodologia" e uso de GitHub

**Localização:** Seção "Metodologia de Desenvolvimento"

**Trecho problemático:**
> "Metodologia: Nenhuma em específico foi aplicada"
> "Controle de Versão: GitHub"

**Problema:**
Usar controle de versão já é, por definição, uma prática metodológica. Dizer que não existe metodologia enquanto se usa GitHub é uma contradição. O correto seria descrever o processo como ele realmente aconteceu — mesmo que informal.

**Impacto:** Médio — gera confusão sobre como o projeto foi conduzido.

---

### ERRO-S03 — Diagrama UML sem identificação do tipo

**Localização:** Seção "Abstração" e estrutura do repositório

**Trecho problemático:**
> "Diagrama simplificado em UML representando os três atores principais."
> `└── Diagrama.png`

**Problema:**
UML tem mais de dez tipos de diagrama diferentes. Sem dizer qual foi usado (Casos de Uso? Sequência? Classes?), o arquivo `.png` fica sem contexto — qualquer pessoa que abrir não vai entender o que está vendo nem por que aquele diagrama foi escolhido.

**Impacto:** Médio — compromete o entendimento do design pelo restante da equipe.

---

### ERRO-S04 — Pasta `src/` listada sem prazo ou responsável

**Localização:** Estrutura do repositório

**Trecho problemático:**
> `└── src/     # (Não foi realizado)`

**Problema:**
Listar um diretório vazio sem indicar quando ou por quem será implementado cria uma expectativa falsa. Não fica claro se é um entregável desta fase ou de uma fase futura.

**Impacto:** Baixo agora — mas problemático quando o projeto avançar.

---

## Categoria 2 — Erros de Lógica

Falhas nos algoritmos descritos que, se implementados como estão, produziriam resultados incorretos ou quebrariam em situações reais.

---

### ERRO-L01 — Algoritmo de ETA sem tratamento de casos extremos

**Localização:** Seção "Algoritmos" — Algoritmo de ETA

**Trecho problemático:**
> "Baseado em velocidade média histórica por trecho + dados de tráfego em tempo real."

**Problema:**
O algoritmo descreve apenas o cenário ideal. Na prática, existem situações que ele ignora completamente:

| Situação ignorada | O que aconteceria |
|---|---|
| Veículo parado (velocidade = 0) | Divisão por zero — o sistema quebraria |
| Dados de tráfego indisponíveis | ETA calculado com dado errado, sem avisar o usuário |
| Veículo fora da rota cadastrada | ETA calculado para a rota errada |
| Primeiro dia de operação | Sem histórico, sem dados para calcular |

**Impacto:** Alto — o sistema exibiria valores absurdos ou travaria em produção.

---

### ERRO-L02 — Buffer geométrico sem dimensão definida

**Localização:** Seção "Algoritmos" — Algoritmo de Detecção de Desvio

**Trecho problemático:**
> "Comparação entre coordenada atual e buffer geométrico da rota cadastrada."

**Problema:**
"Buffer geométrico" não pode ser implementado sem um número. Algumas perguntas que ficam sem resposta:
- Qual o raio? (50m? 200m?)
- É fixo ou varia por tipo de zona?
- Como distinguir um desvio real de um erro de sinal GPS?
- Em quanto tempo o alerta precisa ser emitido após a detecção?

Um buffer muito pequeno gera falsos alertas o tempo todo; um buffer muito grande deixa desvios reais passarem batido.

**Impacto:** Alto — o módulo de alertas se tornaria não confiável.

---

### ERRO-L03 — Fila de alertas com hierarquia incompleta

**Localização:** Seção "Algoritmos" — Algoritmo de Prioridade de Alertas

**Trecho problemático:**
> "Fila com prioridade baseada no tipo e urgência do evento (falha mecânica > atraso > lotação)."

**Problema:**
A hierarquia cobre apenas 3 tipos de evento, mas um sistema de transporte público lida com muito mais do que isso. Questões sem resposta:

- Onde entra um **acidente**?
- Onde entra a **perda de sinal GPS** (que pode indicar roubo)?
- O que acontece quando **dois alertas de mesma prioridade** chegam ao mesmo tempo?
- A partir de quantos minutos um atraso vira um alerta?

**Impacto:** Alto — alertas críticos poderiam ser ignorados ou nunca chegar ao operador.

---

### ERRO-L04 — Consistência de dados identificada como problema, mas sem solução

**Localização:** Seção "Desafios Identificados"

**Trecho problemático:**
> "Garantir que o estado de cada veículo seja consistente entre o painel do operador e a visualização do passageiro."

**Problema:**
O desafio foi identificado corretamente, mas o documento não propõe nenhuma solução. Sem um mecanismo definido, é possível que o operador veja o veículo "em rota" enquanto o passageiro vê "parado" — ou qualquer combinação de estados desatualizados.

**Impacto:** Alto — afeta diretamente a confiabilidade percebida pelo usuário.

---

## Categoria 3 — Erros de Execução / Planejamento

Omissões ou decisões que, se o sistema fosse para produção como está, causariam falhas graves.

---

### ERRO-E01 — Sem plano para quando as APIs externas caírem

**Localização:** Desafio "Integração com fontes externas"

**Trecho problemático:**
> "APIs de tráfego (como Google Maps Platform ou HERE), sistemas municipais de gestão de frotas e equipamentos GPS heterogêneos."

**Problema:**
O sistema depende de APIs externas, mas não há nenhum plano para quando elas ficarem fora do ar. Se a API de tráfego cair, o ETA continua funcionando? Com qual dado? Se não, o que o passageiro vê?

**Impacto:** Crítico — em um serviço público, a queda de uma API terceira não pode derrubar o sistema inteiro.

---

### ERRO-E02 — Meta de 99,9% de disponibilidade sem arquitetura para sustentá-la

**Localização:** Desafio "Disponibilidade contínua"

**Trecho problemático:**
> "O sistema precisa de alta disponibilidade (próximo de 99,9%)."

**Problema:**
99,9% significa no máximo 8,7 horas de downtime por ano. Mas o documento não menciona nenhuma das práticas necessárias para chegar lá: redundância de servidores, balanceamento de carga, plano de recuperação a desastres ou monitoramento contínuo. Declarar um SLA sem a arquitetura correspondente é só uma intenção — não um requisito realizável.

**Impacto:** Crítico — o sistema não sustentaria essa meta sem essas definições.

---

### ERRO-E03 — Segurança citada como princípio, sem controles definidos

**Localização:** Desafio "Segurança e privacidade"

**Trecho problemático:**
> "Aplicação dos princípios de menor privilégio (Saltzer & Schroeder)."

**Problema:**
Citar um princípio teórico não é o mesmo que ter segurança. O documento não define quem pode acessar o quê, como a autenticação funciona, se os dados GPS são criptografados ou como o sistema se enquadra na LGPD — que se aplica aqui, já que envolve dados operacionais de serviço público.

**Impacto:** Crítico — o sistema estaria vulnerável e potencialmente fora da lei.

---

## Resumo

| Código | Categoria | Problema | Impacto |
|---|---|---|---|
| ERRO-S01 | Sintaxe/Doc | Intervalo GPS não definido | Alto |
| ERRO-S02 | Sintaxe/Doc | Contradição sobre metodologia | Médio |
| ERRO-S03 | Sintaxe/Doc | Tipo do diagrama UML não especificado | Médio |
| ERRO-S04 | Sintaxe/Doc | `src/` sem prazo ou responsável | Baixo |
| ERRO-L01 | Lógica | ETA sem tratamento de casos extremos | Alto |
| ERRO-L02 | Lógica | Buffer geométrico sem dimensão | Alto |
| ERRO-L03 | Lógica | Fila de alertas incompleta | Alto |
| ERRO-L04 | Lógica | Consistência de dados sem solução proposta | Alto |
| ERRO-E01 | Execução | Sem fallback para APIs externas | Crítico |
| ERRO-E02 | Execução | SLA sem arquitetura correspondente | Crítico |
| ERRO-E03 | Execução | Segurança citada mas não especificada | Crítico |
