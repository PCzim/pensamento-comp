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
