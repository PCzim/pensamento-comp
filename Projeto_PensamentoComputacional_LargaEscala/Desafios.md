# Desafios – Problemas Identificados e Soluções Propostas

## 1. Escalabilidade em Tempo Real

**Problema:**
O sistema precisa processar atualizações de GPS de centenas de veículos simultaneamente (a cada 5 segundos por veículo), sem que isso cause atraso perceptível na interface do passageiro.

**Impacto:**
Uma frota de 300 veículos gera aproximadamente 60 eventos por segundo de forma contínua. Em horários de pico, falhas de desempenho afetam diretamente o serviço público.

**Solução Proposta:**
- Utilizar um broker de mensagens (ex: Apache Kafka ou MQTT) para desacoplar a coleta de dados do processamento.
- Processar as atualizações em workers paralelos, escalonáveis horizontalmente.
- Armazenar posições em banco de dados de séries temporais (ex: InfluxDB ou TimescaleDB), otimizado para esse tipo de carga.

---

## 2. Consistência de Dados Distribuídos

**Problema:**
O estado de cada veículo precisa ser consistente entre o painel do operador e a visualização do passageiro. Dados desatualizados ou conflitantes podem causar decisões erradas do operador e frustração do passageiro.

**Impacto:**
Se um veículo muda de estado (ex: entra em manutenção) e isso não reflete imediatamente nas interfaces, o sistema perde confiabilidade.

**Solução Proposta:**
- Adotar o padrão de evento único de origem (event sourcing): toda mudança de estado passa por um registro central antes de ser distribuída.
- Utilizar WebSockets para enviar atualizações em tempo real às interfaces sem necessidade de polling constante.

---

## 3. Segurança e Privacidade dos Dados Operacionais

**Problema:**
O sistema lida com dados de localização em tempo real de veículos públicos, informações de operadores e, indiretamente, padrões de deslocamento da população.

**Impacto:**
Vazamento de dados ou acesso não autorizado pode comprometer a segurança operacional da frota e violar regulamentações como a LGPD.

**Solução Proposta (baseada nos princípios de Saltzer & Schroeder):**
- **Menor privilégio:** cada perfil de usuário acessa apenas o que precisa (passageiro não vê dados operacionais; operador não acessa dados de outros operadores).
- **Economia de mecanismo:** autenticação centralizada e simples, via token JWT.
- **Separação de privilégios:** ações críticas (desativar rota, registrar falha) exigem confirmação de um segundo operador.
- Dados de localização de passageiros **não são coletados** – o sistema é anônimo para o usuário final.

---

## 4. Integração com Fontes Externas

**Problema:**
O sistema depende de fontes externas para funcionar corretamente: APIs de tráfego, equipamentos GPS de fabricantes diferentes e, potencialmente, sistemas municipais de gestão de transporte.

**Impacto:**
Uma API externa fora do ar pode invalidar os cálculos de ETA. Equipamentos GPS com protocolos diferentes dificultam a padronização da coleta.

**Solução Proposta:**
- Criar uma camada de adaptadores (padrão Adapter) para normalizar os dados de diferentes modelos de GPS para um formato interno único.
- Implementar fallback no cálculo de ETA: se a API de tráfego estiver indisponível, usar apenas a velocidade histórica média.
- Utilizar o padrão GTFS para estrutura de rotas, garantindo compatibilidade com sistemas municipais.

---

## 5. Alta Disponibilidade

**Problema:**
O transporte coletivo opera 7 dias por semana, muitas vezes 24 horas. Qualquer interrupção no sistema afeta diretamente o serviço público e os passageiros.

**Impacto:**
Uma queda de 30 minutos no sistema durante o horário de pico impacta milhares de pessoas.

**Solução Proposta:**
- Arquitetura com redundância: múltiplas instâncias do serviço de coleta e processamento.
- Monitoramento ativo com alertas automáticos para a equipe técnica (ex: Grafana + PagerDuty).
- Banco de dados com replicação e backups automáticos a cada hora.
- Meta de disponibilidade: **99,9%** (equivale a menos de 9 horas de indisponibilidade por ano).

---

## Resumo

| # | Desafio                        | Estratégia Principal                         |
|---|--------------------------------|----------------------------------------------|
| 1 | Escalabilidade em tempo real   | Broker de mensagens + workers paralelos      |
| 2 | Consistência de dados          | Event sourcing + WebSockets                  |
| 3 | Segurança e privacidade        | Princípios de Saltzer & Schroeder + LGPD     |
| 4 | Integração com fontes externas | Camada de adaptadores + fallback de ETA      |
| 5 | Alta disponibilidade           | Redundância + monitoramento ativo            |
