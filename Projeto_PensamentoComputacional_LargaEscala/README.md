# Projeto – Pensamento Computacional para Sistemas de Larga Escala

## Descrição

Este projeto foi desenvolvido como parte da disciplina **Pensamento Computacional** no curso de Engenharia de Software, com a Profa. Kadidja Valéria.

O objetivo é aplicar os conceitos de pensamento computacional e engenharia de software na concepção de um sistema de larga escala, explorando decomposição, abstração, reconhecimento de padrões e algoritmos.

\---

## Objetivos

* Relacionar engenharia de software e pensamento computacional.
* Reconhecer princípios e padrões relevantes para sistemas de larga escala.
* Identificar dificuldades reais no desenvolvimento de aplicações complexas.
* Aplicar metodologias ágeis no planejamento do projeto.

\---

## Sistema Proposto

**Nome do Sistema:** UrbanTrack – Sistema de Monitoramento de Frotas Urbanas

**Descrição:**

Uma plataforma web para rastreamento e gestão de frotas de transporte coletivo urbano (ônibus municipais, vans escolares, transporte corporativo), integrando dados em tempo real para operadores, passageiros e gestores públicos.

O sistema permite:

* Rastreamento em tempo real dos veículos via GPS.
* Cálculo dinâmico de tempo estimado de chegada (ETA) nas paradas.
* Painel de controle para gestores de frota (alertas, rotas, manutenção).
* Aplicativo de consulta para passageiros (horários, lotação, desvios de rota).
* Geração de relatórios de desempenho operacional.

\---

## Pensamento Computacional Aplicado

### Decomposição

O sistema foi dividido nos seguintes módulos independentes:

* **Coleta de Dados GPS** – Recepção e armazenamento das coordenadas dos veículos em intervalos regulares.
* **Cálculo de ETA** – Processamento da posição atual, velocidade média e distância até a próxima parada.
* **Gestão de Rotas** – Cadastro, edição e monitoramento das linhas e trajetos.
* **Painel do Operador** – Interface de controle para despachantes e gestores de frota.
* **Interface do Passageiro** – Consulta pública de localização e previsão de chegada.
* **Módulo de Alertas** – Notificações automáticas de atrasos, desvios e falhas mecânicas.

### Reconhecimento de Padrões

* O padrão de atualização de posição em tempo real é similar ao utilizado em apps como Uber e Google Maps.
* A estrutura de rotas e paradas segue o modelo adotado por sistemas como o GTFS (General Transit Feed Specification), padrão internacional de dados de transporte público.
* O fluxo de alertas automáticos segue padrões de sistemas de monitoramento industrial (SCADA).

### Abstração

* Diagrama simplificado em UML representando os três atores principais: **Veículo**, **Operador** e **Passageiro**.
* Representação da frota como uma coleção de entidades com estado (em rota, parado, em manutenção), ignorando detalhes mecânicos irrelevantes para o sistema.
* Separação clara entre a camada de dados brutos (GPS) e a camada de informação tratada (ETA, alertas).

### Algoritmos

* **Algoritmo de ETA:** Baseado em velocidade média histórica por trecho + dados de tráfego em tempo real.
* **Algoritmo de Detecção de Desvio:** Comparação entre coordenada atual e buffer geométrico da rota cadastrada.
* **Algoritmo de Prioridade de Alertas:** Fila com prioridade baseada no tipo e urgência do evento (falha mecânica > atraso > lotação).

\---

## Metodologia de Desenvolvimento

* **Metodologia:** Nenhuma em especifico foi aplicada
* **Ferramentas:** GitHub, Google, Youtube
* **Controle de Versão:** GitHub

\---

## Desafios Identificados

* **Escalabilidade em tempo real:** Processar atualizações de GPS de centenas de veículos simultaneamente sem degradar a latência da interface do passageiro.
* **Consistência de dados distribuídos:** Garantir que o estado de cada veículo seja consistente entre o painel do operador e a visualização do passageiro.
* **Segurança e privacidade:** Rastreamento de veículos envolve dados operacionais sensíveis; aplicação dos princípios de menor privilégio (Saltzer \& Schroeder).
* **Integração com fontes externas:** APIs de tráfego (como Google Maps Platform ou HERE), sistemas municipais de gestão de frotas e equipamentos GPS heterogêneos.
* **Disponibilidade contínua:** O sistema precisa de alta disponibilidade (próximo de 99,9%), pois interrupções afetam diretamente o serviço público de transporte.

\---

## 📂 Estrutura do Repositório

```
Projeto\_UrbanTrack\_LargaEscala/
│
├── README.md              # Documentação principal
├── Design.md              # Decomposição, abstração e padrões aplicados
├── Diagrama.png           # Diagrama UML ou fluxograma do sistema
├── Desafios.md            # Lista de desafios e soluções propostas
└── src/                   # (Nao foi realizado)
```

\---

## Autores

> Projeto desenvolvido para a disciplina de Pensamento Computacional - Joao Pedro Correia - 35885513.

