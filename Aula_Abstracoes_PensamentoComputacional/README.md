# UrbanTrack — Pensamento Computacional Aplicado

**Disciplina:** Pensamento Computacional — Engenharia de Software  
**Professora:** Kadidja Valéria  
**Sistema proposto:** UrbanTrack — Monitoramento de Frotas Urbanas

---

## Contexto

Este documento faz parte de uma atividade da disciplina Pensamento Computacional, cujo objetivo é aplicar os quatro pilares — decomposição, abstração, reconhecimento de padrões e algoritmos — na concepção de um sistema de larga escala.

O sistema escolhido foi o **UrbanTrack**, uma plataforma web para rastreamento e gestão de frotas de transporte coletivo urbano (ônibus municipais, vans escolares e transporte corporativo), integrando dados em tempo real para operadores, passageiros e gestores públicos.

---

## Objetivos

- Relacionar engenharia de software e pensamento computacional.
- Reconhecer princípios e padrões relevantes para sistemas de larga escala.
- Identificar dificuldades reais no desenvolvimento de aplicações complexas.
- Aplicar as etapas de descrição detalhada, representação intermediária e abstração computacional sobre uma função real do sistema.

---

## Função analisada: Cálculo de ETA

Dentre os módulos do UrbanTrack, foi escolhido o **cálculo de ETA** (Estimated Time of Arrival) como função central da análise. Essa função conecta os dados brutos de GPS à informação que o passageiro vê na tela: o tempo estimado de chegada do ônibus à próxima parada.

A escolha se justifica pela riqueza do processo — entradas concretas (coordenadas GPS, velocidade, tráfego), lógica de processamento não trivial e uma saída clara (tempo em minutos) — além de o algoritmo ser genérico o suficiente para representar qualquer sistema de estimativa de chegada.

---

## Etapa 1 — Descrição detalhada

Passo a passo de como o sistema executa o cálculo de ETA, do dispositivo físico no veículo até o número exibido na tela do passageiro.

### No veículo

1. O motorista liga o ônibus. O dispositivo GPS instalado no painel acende automaticamente junto com a ignição.
2. O chip GPS leva alguns segundos para obter sinal dos satélites e fixar a posição inicial do veículo.
3. A cada 10 segundos, o dispositivo registra latitude, longitude, velocidade instantânea e timestamp.
4. Esse pacote de dados é enviado via rede móvel (4G) para o servidor central do UrbanTrack.
5. Se o sinal cair (túnel, zona sem cobertura), o dispositivo armazena os pacotes localmente e os envia em lote quando o sinal volta.

### No servidor

6. O servidor recebe o pacote e verifica se as coordenadas estão dentro do limite geográfico esperado para aquela linha.
7. Se as coordenadas forem inválidas (leitura corrompida, salto absurdo de posição), o pacote é descartado e o sistema aguarda o próximo envio.
8. A posição válida é salva no banco de dados com o timestamp exato da coleta.
9. O módulo de ETA é acionado: ele busca qual é a próxima parada cadastrada na rota daquele veículo.
10. O sistema calcula a distância entre a posição atual e a próxima parada, usando geometria esférica sobre as coordenadas.
11. O sistema consulta o banco de histórico: qual foi a velocidade média observada neste mesmo trecho, neste horário, neste dia da semana, nas últimas semanas?
12. Em paralelo, uma chamada é feita à API de tráfego externo para saber se há lentidão ou incidente naquele trecho agora.
13. O sistema combina a velocidade histórica com o fator de tráfego atual para estimar a velocidade real de deslocamento.
14. Divide a distância pela velocidade estimada e obtém um tempo bruto em minutos.
15. Aplica um fator de ajuste baseado em atrasos históricos conhecidos naquela linha e horário.
16. O ETA final é publicado via API REST.

### No app do passageiro

17. O app consulta a API a cada 15 segundos e recebe o novo valor de ETA.
18. A tela atualiza o número de minutos exibido na parada selecionada pelo usuário.

---

## Etapa 2 — Representação intermediária

Diagrama simplificado mostrando apenas as etapas principais do fluxo. Detalhes como protocolos, formatos de dados e tratamento de erros foram removidos.

```
[Veículo transmite posição]
          |
          v
[Servidor valida os dados]
          |
          v
[Calcula distância e velocidade] <-- [Tráfego em tempo real]
          ^                      <-- [Histórico de velocidade]
          |
          v
   [Refina o ETA]
          |
          v
[App exibe ETA ao passageiro]
```

---

## Etapa 3 — Abstração computacional

Algoritmo genérico que representa o processo de forma universal — aplicável a qualquer sistema de estimativa de chegada, não só ônibus.

```
INICIO Algoritmo CalcularETA

    posicao_atual <- obter_posicao(entidade_movel)
    SE posicao_atual = INVALIDA ENTAO
        registrar_falha(entidade_movel)
        RETORNAR ERRO_POSICAO_INDISPONIVEL
    FIM SE

    proximo_destino    <- obter_proximo_ponto(entidade_movel, rota_ativa)
    distancia_restante <- calcular_distancia(posicao_atual, proximo_destino)

    velocidade_historica <- consultar_historico(entidade_movel, trecho_atual, horario_atual)
    fator_externo        <- consultar_fonte_externa(trecho_atual)
    velocidade_estimada  <- combinar(velocidade_historica, fator_externo)

    SE velocidade_estimada <= 0 ENTAO
        RETORNAR ERRO_VELOCIDADE_INVALIDA
    FIM SE

    eta_bruto      <- distancia_restante / velocidade_estimada
    fator_contexto <- calcular_fator(horario_atual, dia_semana, historico_atrasos)
    eta_refinado   <- eta_bruto * fator_contexto

    publicar(entidade_movel, proximo_destino, eta_refinado)
    RETORNAR eta_refinado

FIM Algoritmo CalcularETA
```
