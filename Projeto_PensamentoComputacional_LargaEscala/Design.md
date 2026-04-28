# Design – Decomposição, Abstração e Padrões Aplicados

## 1. Decomposição

O sistema UrbanTrack foi decomposto em módulos independentes, cada um com responsabilidade bem definida:

### 1.1 Módulo de Coleta de Dados GPS
- Recebe coordenadas dos veículos a cada 5 segundos via protocolo MQTT.
- Armazena as posições em banco de dados de séries temporais.
- Entradas: latitude, longitude, ID do veículo, timestamp.
- Saídas: registro persistido + evento publicado para os demais módulos.

### 1.2 Módulo de Cálculo de ETA
- Processa a posição atual do veículo e calcula o tempo estimado de chegada em cada parada seguinte.
- Considera velocidade média histórica por trecho e condições de tráfego.
- Entradas: posição atual, rota cadastrada, dados de tráfego.
- Saídas: ETA em minutos para cada parada da linha.

### 1.3 Módulo de Gestão de Rotas
- Permite cadastrar, editar e desativar linhas e trajetos.
- Define as paradas, a sequência e o horário previsto de cada linha.
- Entradas: dados inseridos pelo operador.
- Saídas: rotas disponíveis para os demais módulos consumirem.

### 1.4 Módulo de Alertas
- Monitora eventos críticos: atrasos acima de 10 minutos, desvio de rota e falhas reportadas pelo motorista.
- Classifica os alertas por prioridade e os encaminha ao painel do operador.
- Entradas: eventos dos módulos de GPS, ETA e veículos.
- Saídas: notificações priorizadas no painel.

### 1.5 Painel do Operador
- Interface web para despachantes e gestores de frota.
- Exibe mapa em tempo real, lista de alertas e relatórios de desempenho.

### 1.6 Interface do Passageiro
- Aplicação web/mobile pública.
- Exibe localização dos veículos na linha consultada e ETA nas paradas.

---

## 2. Abstração

### 2.1 Entidades Principais (simplificadas)

| Entidade   | Atributos relevantes para o sistema         | Atributos ignorados                  |
|------------|---------------------------------------------|--------------------------------------|
| Veículo    | ID, posição atual, linha, estado            | Marca, modelo, consumo de combustível|
| Rota       | ID, lista de paradas, horário previsto       | Histórico de alterações antigas      |
| Parada     | ID, coordenadas, nome                       | Estrutura física, acessibilidade     |
| Passageiro | (anônimo) – apenas consulta                 | Dados pessoais                       |
| Operador   | ID, nome, permissões                        | Dados de RH                          |

### 2.2 Estados do Veículo

```
[Em Garagem] → [Em Rota] → [Na Parada] → [Em Rota] → ... → [Em Garagem]
                    ↓
              [Em Manutenção]
```

### 2.3 Diagrama de Camadas

```
┌──────────────────────────────────────┐
│         Interface (Passageiro)        │  ← Consulta pública
├──────────────────────────────────────┤
│         Painel (Operador)             │  ← Gestão e controle
├──────────────────────────────────────┤
│  ETA │ Alertas │ Rotas │ Relatórios   │  ← Lógica de negócio
├──────────────────────────────────────┤
│         Coleta de Dados GPS           │  ← Entrada de dados brutos
└──────────────────────────────────────┘
```

---

## 3. Reconhecimento de Padrões

| Padrão identificado           | Referência / Inspiração                          |
|-------------------------------|--------------------------------------------------|
| Atualização de posição em tempo real | Apps de mobilidade (Uber, Moovit, Google Maps) |
| Estrutura de rotas e paradas  | GTFS – General Transit Feed Specification        |
| Fila de alertas por prioridade| Sistemas SCADA de monitoramento industrial       |
| Autenticação por papéis       | Controle de acesso baseado em perfil (RBAC)      |
| Publicação de eventos GPS     | Padrão Publish/Subscribe (MQTT)                  |

---

## 4. Algoritmos

### 4.1 Cálculo de ETA

```
ETA(parada_destino) =
  distância_restante_até_parada / velocidade_média_trecho
  + fator_de_tráfego_em_tempo_real
```

- A velocidade média é calculada com base nos últimos 7 dias no mesmo trecho e horário.
- O fator de tráfego é obtido via API externa (ex: HERE Traffic API).

### 4.2 Detecção de Desvio de Rota

```
SE distância(posição_atual, rota_cadastrada) > 50 metros
   POR MAIS DE 30 segundos consecutivos
ENTÃO
   disparar alerta de desvio de rota
```

### 4.3 Prioridade de Alertas

```
PRIORIDADE 1 (Crítico)   → Falha mecânica reportada pelo motorista
PRIORIDADE 2 (Alto)      → Desvio de rota detectado
PRIORIDADE 3 (Médio)     → Atraso superior a 10 minutos
PRIORIDADE 4 (Baixo)     → Lotação máxima atingida
```
