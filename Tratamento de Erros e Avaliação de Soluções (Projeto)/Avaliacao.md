# Avaliação da Solução Final – UrbanTrack

> Reflexão sobre o projeto após o processo de identificação e correção de erros, avaliando clareza, eficiência e escalabilidade.

---

## 1. Clareza

### O que melhorou

Boa parte dos problemas do projeto original não era técnica — era de comunicação. Expressões como "intervalos regulares", "buffer geométrico" e "alta disponibilidade" davam a impressão de que algo estava definido, quando na verdade eram apenas placeholders.

Depois das correções, o projeto ficou mais claro por três razões:

- Cada algoritmo agora tem comportamento definido tanto para o cenário normal quanto para os casos extremos. Alguém que não participou do projeto consegue entender o que acontece quando o veículo para, quando o GPS perde sinal ou quando dois alertas chegam ao mesmo tempo.
- Os módulos têm responsabilidades que não se sobrepõem. A separação entre "Coleta GPS" (dado bruto) e "Cálculo de ETA" (informação processada) já existia no original, mas ficou mais sólida com as interfaces definidas entre os módulos.
- O nível de detalhe é consistente ao longo do documento — não mistura decisões arquiteturais de alto nível com detalhes de implementação no mesmo parágrafo.

### O que ainda falta

O maior gap de clareza é o `Diagrama.png` — ele é referenciado em vários momentos, mas ainda não foi criado. Para um sistema com vários módulos e três atores principais, um diagrama visual faz uma diferença grande no entendimento.

**Nota: 7/10** — boa após as correções, mas incompleta sem o diagrama.

---

## 2. Eficiência

### Como os algoritmos se saem

**ETA:** usar velocidade média histórica por trecho é uma escolha inteligente. Calcular tudo em tempo real seria mais preciso, mas muito mais pesado computacionalmente — e desnecessário para um serviço de transporte urbano, onde as variações de tráfego num mesmo trecho seguem padrões razoavelmente previsíveis.

**Detecção de desvio:** a técnica de exigir 3 leituras consecutivas antes de disparar o alerta é simples e eficiente. O custo é mínimo — uma comparação geométrica por leitura GPS — e elimina a maioria dos falsos positivos causados por ruído de sinal.

**Fila de alertas:** a estrutura de fila com prioridade tem custo O(log n) para inserção e O(1) para retirada do elemento mais urgente. Para o volume de alertas esperado em uma frota de algumas centenas de veículos, isso é mais do que suficiente.

### O ponto de atenção

Com 200 veículos operando 18 horas por dia em intervalos de 10 segundos, o sistema gera cerca de **1,3 milhão de registros GPS por dia**. Isso não é um problema insolúvel, mas exige uma estratégia pensada de armazenamento: compressão, particionamento por data e uma política clara de retenção. O documento define o prazo de retenção (180 dias para dados brutos), mas não especifica o tipo de banco de dados — que faz diferença real aqui.

**Nota: 7,5/10** — algoritmos bem dimensionados para o problema; armazenamento precisa de atenção.

---

## 3. Escalabilidade

### O que está bem pensado

A decisão mais importante para escalabilidade foi a **arquitetura orientada a eventos com barramento de mensagens** (adicionada na correção do ERRO-L04). Ela resolve três problemas de uma vez:

- Cada módulo pode ser escalado de forma independente. O serviço de coleta GPS pode ter 10 instâncias enquanto o de relatórios tem apenas 1.
- Novos consumidores de dados podem ser adicionados sem alterar nada no módulo de coleta. Um futuro módulo de análise preditiva de falhas mecânicas, por exemplo, só precisaria se inscrever no barramento.
- O barramento absorve picos de atualização GPS sem que o banco de dados seja impactado diretamente.

A decomposição modular original do projeto também foi uma boa decisão. Módulos independentes são mais fáceis de escalar, testar e substituir no futuro.

### O que ainda precisa de definição

O **banco de dados** é o gargalo mais provável em produção, e o documento não define qual tipo usar. Para o padrão do UrbanTrack — muitas escritas de GPS, leituras distribuídas para vários clientes simultâneos — um banco de dados de séries temporais (como InfluxDB ou TimescaleDB) seria mais adequado do que um banco relacional convencional. Essa é uma decisão que precisa ser tomada antes da implementação.

A meta de **99,9% de disponibilidade** é factível com a arquitetura descrita, mas requer infraestrutura com redundância real — o que tem custo. Para uma frota de 200 veículos, esse custo provavelmente se justifica. Para frotas menores, valeria avaliar se uma arquitetura mais simples já seria suficiente.

**Nota: 7/10** — bem estruturada conceitualmente; falta a decisão sobre o banco de dados.

---

## 4. O que o processo revelou

O projeto original tinha uma base conceitual boa. A decomposição em módulos, a identificação de padrões de mercado (GTFS, SCADA) e o reconhecimento antecipado dos desafios são sinais de que o pensamento computacional foi aplicado de forma genuína. O problema não estava na visão — estava na especificação.

Existe uma diferença importante entre **identificar um desafio** e **propor uma solução para ele**. O documento original fazia bem a primeira parte, mas raramente avançava para a segunda. Essa atividade forçou essa transição.

Três lições ficaram mais claras depois desse processo:

**Vagueza não é abstração.** Abstração é uma simplificação intencional — você descarta o que não importa, mas mantém o que é essencial. Vagueza é a ausência de decisão disfarçada de abstração. "Buffer geométrico" sem uma dimensão é vago; "buffer de 50 metros com confirmação em 3 leituras" é uma abstração com substância.

**Documentação ambígua vira bug de código.** Toda incerteza na especificação vai ser resolvida pelo desenvolvedor na hora de implementar — e não necessariamente da forma que o arquiteto pretendia. Especificar bem é uma forma de proteger a integridade do sistema antes mesmo de existir código.

**Casos extremos importam tanto quanto o caminho principal.** O algoritmo de ETA funcionava perfeitamente quando tudo corria bem. O problema é que em produção, as coisas nem sempre correm bem. O tratamento dos casos extremos (veículo parado, sem histórico, fora de rota) é o que separa um sistema confiável de um sistema que quebra na primeira segunda-feira chuvosa.

---

## 5. Avaliação consolidada

| Dimensão | Nota | Observação |
|---|---|---|
| Clareza | 7/10 | Boa após correções; falta o diagrama visual |
| Eficiência | 7,5/10 | Algoritmos bem escolhidos; tipo de banco de dados não definido |
| Escalabilidade | 7/10 | Arquitetura orientada a eventos é ponto forte; BD precisa de decisão |
| **Média** | **7,2/10** | Projeto viável e bem fundamentado para a fase conceitual |

O UrbanTrack revisado é um projeto consistente e realizável. Não está completo — há decisões que ainda precisam ser tomadas antes da implementação — mas pelo menos agora é honesto sobre o que está definido e o que ainda está em aberto. Essa honestidade, combinada com as especificações concretas introduzidas pelas correções, coloca o projeto em uma posição muito melhor para a próxima fase.
