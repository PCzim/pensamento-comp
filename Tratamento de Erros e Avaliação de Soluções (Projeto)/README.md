# UrbanTrack – Sistema de Monitoramento de Frotas Urbanas

## O que é o projeto

O **UrbanTrack** é uma plataforma web para rastreamento e gestão de frotas de transporte coletivo urbano — ônibus municipais, vans escolares, transporte corporativo. A ideia é centralizar informações em tempo real para três perfis: quem opera a frota, quem usa o transporte e quem gerencia o serviço publicamente.

O sistema foi planejado para:

- Rastrear veículos em tempo real via GPS
- Calcular o tempo estimado de chegada (ETA) nas paradas
- Oferecer um painel de controle para gestores (alertas, rotas, manutenção)
- Disponibilizar consulta para passageiros (horários, lotação, desvios)
- Gerar relatórios de desempenho operacional

---

## Sobre esta entrega

Esta entrega faz parte da aula de **Tratamento de Erros e Avaliação de Soluções**.

O UrbanTrack ainda está na fase de planejamento — não há código implementado. Por isso, a análise foi feita sobre o que existe de fato: a documentação, os algoritmos descritos, as decisões de arquitetura e a estrutura do repositório. As três categorias de erro foram adaptadas para esse contexto:

| Categoria | Como foi aplicada aqui |
|---|---|
| Erro de Sintaxe | Inconsistências e ambiguidades na documentação |
| Erro de Lógica | Falhas nos algoritmos e fluxos descritos |
| Erro de Execução | Decisões de projeto que causariam problemas em produção |

---

## Estrutura do repositório

```
Projeto_UrbanTrack_LargaEscala/
│
├── README.md                  # Este arquivo
├── Erros_Identificados.md     # Registro dos erros encontrados
├── Projeto_Corrigido.md       # Correções com justificativas
├── Avaliacao.md               # Reflexão sobre clareza, eficiência e escalabilidade
├── Design.md                  # Decomposição, abstração e padrões aplicados
├── Diagrama.png               # Diagrama UML do sistema
├── Desafios.md                # Desafios identificados e soluções propostas
└── src/                       # (Não implementado nesta fase)
```

---

## Como o sistema foi pensado

### Módulos principais

| Módulo | O que faz |
|---|---|
| Coleta de Dados GPS | Recebe e armazena as coordenadas dos veículos |
| Cálculo de ETA | Processa posição, velocidade e distância para estimar chegada |
| Gestão de Rotas | Cadastra, edita e monitora linhas e trajetos |
| Painel do Operador | Interface para despachantes e gestores acompanharem a frota |
| Interface do Passageiro | Consulta pública de localização e previsão de chegada |
| Módulo de Alertas | Notificações automáticas de atrasos, desvios e falhas |

### Atores principais

```
Veículo  ──→  Operador  ──→  Passageiro
   ↑               ↑
  GPS           Alertas
```

---

## Metodologia e ferramentas

- **Metodologia:** Nenhuma aplicada formalmente
- **Ferramentas:** GitHub, Google, YouTube
- **Controle de versão:** GitHub

---

## Equipe

> *(Inserir nomes dos integrantes do grupo)*

## Instituição

> *(Inserir nome da instituição e curso)*
