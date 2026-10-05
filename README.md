# Macaquinho Virtual 2.0
## Especificação inicial

## 1. Visão geral

O **Macaquinho Virtual 2.0** será um simulador 2D de um único macaquinho vivendo em um ambiente criado e manipulado pelo espectador.

A proposta é criar um **sandbox de comportamento artificial**: o ambiente possui regras e consequências programadas, enquanto o macaquinho começa sem conhecer o mapa e aprende por experiência usando **Reinforcement Learning / Q-learning**.

O espectador poderá modificar o ambiente durante o experimento. O agente deverá observar as mudanças, avaliar novamente suas opções e, quando necessário, abandonar o plano atual e recalcular.

A arquitetura deve ser genérica. O ambiente inicial será pequeno, mas a intenção é permitir novos experimentos e objetos futuramente sem reescrever o núcleo do aprendizado.

---

## 2. Escopo inicial

O primeiro mundo será uma grade **5x5** usada também para os cálculos:

```javascript
const grid = [
  [0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0]
];
```

A arquitetura não deve ficar presa a 5x5. O tamanho deve ser configurável (`width` e `height`) para permitir 10x10, 20x20 etc. sem alterar o núcleo.

---

## 3. Movimento

O macaquinho pode se mover, em cada instante, apenas para:

- cima;
- baixo;
- esquerda;
- direita.

Não há diagonais.

Também deve existir uma ação `STAY` para permitir que o agente permaneça parado quando isso for vantajoso.

---

## 4. Regra de ocupação e interação

O macaquinho ocupa uma única célula.

### Área de choque

A área de choque é um tipo de **chão perigoso**. O macaquinho pode entrar na própria célula.

Ao entrar:

- recebe choque;
- perde felicidade;
- perde vida;
- registra a experiência negativa;
- é devolvido para uma posição segura/anterior segundo a regra do ambiente.

O choque não deve ensinar instantaneamente que a célula é ruim. O agente deve precisar de várias experiências para construir uma aversão significativa.

### Banana, droga e objetos de interação

Banana, droga e outros objetos não precisam ser ocupados pelo macaquinho. A interação será feita quando ele estiver **em uma célula adjacente**.

Isso permite separar claramente `posição do objeto` de `posição do agente`.

---

## 5. Objetos iniciais

### Banana

Propriedades:

```json
{
  "type": "banana",
  "position": [1, 5],
  "quantity": 3
}
```

`quantity = 0` significa quantidade infinita.

Efeitos iniciais:

- reduz fome;
- aumenta um pouco a felicidade.

### Droga

Propriedades semelhantes às da banana, também com quantidade configurável e `0 = infinito`.

Efeitos iniciais:

- aumenta muito a felicidade;
- aumenta dependência;
- uso excessivo pode reduzir vida.

Dependência inicial sugerida:

```text
dependencia = 0..100
```

A dependência deve influenciar progressivamente o valor das decisões, e não ser apenas um gatilho binário do tipo `if (dependencia > X) procurarDroga()`.

Com dependência alta, o agente poderá:

- procurar droga mesmo quando ela estiver distante;
- procurar droga mesmo quando não houver droga disponível no mapa;
- ignorar comida que não esteja perto;
- aceitar mais riscos para tentar chegar à droga;
- reduzir outras atividades em favor da busca.

### Área de choque

Propriedades:

```json
{
  "type": "shock",
  "position": [3, 4]
}
```

Efeitos iniciais:

- reduz felicidade;
- reduz vida.

---

## 6. Objetos adicionados pelo espectador

O espectador poderá inserir objetos no ambiente durante a simulação.

Ao adicionar um objeto, deverá escolher:

1. tipo;
2. posição na grade;
3. quantidade, quando aplicável;
4. configuração do efeito, quando aplicável.

Exemplo de fluxo:

```text
Adicionar objeto
      ↓
Escolher BANANA
      ↓
Escolher quantidade
      ↓
Escolher célula
      ↓
Objeto entra no ambiente
      ↓
O agente percebe a mudança e pode recalcular
```

A arquitetura deve permitir novos objetos sem modificar o algoritmo de aprendizado.

---

## 7. Show Details

A interface terá um botão **Show Details**.

Ele deverá apresentar uma representação JSON do cenário e das regras relevantes.

Exemplo:

```json
{
  "banana": {
    "position": [1, 5],
    "quantity": 3
  },
  "shock": {
    "position": [3, 4]
  }
}
```

Futuramente o JSON poderá incluir:

- objetos;
- posições;
- quantidades;
- efeitos;
- estado atual dos recursos;
- configurações do experimento;
- memória espacial do agente;
- outros dados úteis para depuração.

---

## 8. Controle da simulação

A interface terá:

```text
RODAR
AVANÇAR 1 INSTANTE

1x
5x
25x
200x
```

E os contadores:

```text
DIA: X
INSTANTE: Y
```

`Avançar 1 instante` executa exatamente um passo.

As velocidades apenas aceleram a execução da simulação; não devem mudar a lógica do aprendizado.

---

## 9. Ciclo de decisão desejado

A lógica principal deverá seguir este fluxo:

```text
ESTADO ATUAL
     ↓
ANALISA O AMBIENTE INTEIRO
     ↓
CALCULA O VALOR ESPERADO DE CADA DESTINO
     ↓
GERA PROBABILIDADE PARA CADA DESTINO
     ↓
ESCOLHE UM DESTINO
     ↓
PLANEJA O CAMINHO
     ↓
COMEÇA A ANDAR
```

O agente deve considerar, entre outras coisas:

- posição atual;
- fome;
- energia;
- felicidade;
- vida;
- dependência;
- depressão;
- recursos conhecidos;
- histórico de experiências;
- riscos conhecidos;
- distância;
- caminhos possíveis;
- recompensa esperada.

---

## 10. Tabela de probabilidade por destino

O sistema deve conseguir produzir uma tabela para **todas as células do ambiente**.

Exemplo conceitual:

```json
{
  "1,1": 0.02,
  "1,2": 0.01,
  "1,3": 0.04,
  "1,4": 0.00,
  "1,5": 0.01,

  "2,1": 0.03,
  "2,2": 0.05,
  "2,3": 0.20,
  "2,4": 0.01,
  "2,5": 0.01,

  "3,1": 0.04,
  "3,2": 0.06,
  "3,3": 0.05,
  "3,4": 0.03,
  "3,5": 0.02,

  "4,1": 0.01,
  "4,2": 0.02,
  "4,3": 0.01,
  "4,4": 0.01,
  "4,5": 0.01,

  "5,1": 0.04,
  "5,2": 0.02,
  "5,3": 0.03,
  "5,4": 0.01,
  "5,5": 0.01
}
```

Esses valores representarão a tendência atual do agente em relação a cada destino.

Importante: esta tabela deve ser tratada como uma **representação da política/avaliação atual**, e não como se fosse automaticamente a Q-table clássica. A Q-table e a probabilidade por destino são conceitos que devem ficar separados na implementação para que seja possível depurar e evoluir o sistema.

---

## 11. Aprendizado por Reinforcement Learning

A primeira implementação validável deverá utilizar **Q-learning**.

A experiência fundamental é:

```text
estado
+
ação
+
recompensa
+
próximo estado
```

Exemplo:

```text
Estado:
muita fome + posição 3,3

Ação:
andar para cima

Resultado:
aproximou-se de uma banana

Recompensa:
positiva

Próximo estado:
posição nova + necessidades atualizadas
```

O agente atualiza suas estimativas e usa essas experiências para tomar decisões futuras.

---

## 12. Aprendizado versus planejamento

Não confundir o Q-learning com o planejador de caminho.

A arquitetura desejada possui duas camadas:

### Camada de aprendizado

Aprende valores com base nas experiências e consequências.

### Camada de planejamento

Recebe um destino escolhido e transforma o destino em uma sequência de movimentos.

Exemplo:

```text
Destino: (1,5)

UP
UP
RIGHT
RIGHT
```

Assim, o agente pode ter uma intenção de destino sem precisar tratar cada movimento como uma decisão completamente isolada.

O planejador também deve conseguir recalcular quando o mundo mudar.

---

## 13. Replanejamento dinâmico

Se houver uma mudança relevante no cenário, o plano atual deve ser invalidado ou reavaliado.

Exemplos:

- nova banana adicionada;
- banana removida;
- droga adicionada;
- droga consumida/esgotada;
- choque adicionado;
- choque removido;
- caminho alterado/bloqueado;
- estado interno alterado de forma relevante;
- evento inesperado.

Fluxo:

```text
PLANO ATUAL
    ↓
AMBIENTE MUDOU?
    ↓
   SIM
    ↓
ABANDONA/REAVALIA PLANO
    ↓
OBSERVA NOVAMENTE
    ↓
RECALCULA
```

---

## 14. Memória espacial

O agente deverá manter memória associada às células.

Exemplo:

```json
{
  "3,4": {
    "shock_experiences": 4,
    "positive_experiences": 0,
    "negative_value": 3.8
  }
}
```

A memória deve representar experiências acumuladas, e não uma regra fixa como:

```text
3,4 = proibido
```

Isso é importante para permitir que o agente explore antes de consolidar uma aversão.

---

## 15. Aprendizado gradual do choque

Exemplo desejado:

### Primeira vez

```text
entra na célula
→ choque
→ perde vida/felicidade
→ volta
```

A célula passa a ter uma experiência negativa, mas ainda pode ser explorada no futuro.

### Repetição

Novos choques acumulam evidência negativa.

A tendência de escolher a região começa a cair.

### Choque removido

Quando o choque é removido, o agente não deve apagar automaticamente a memória.

```text
choque removido
      ↓
memória negativa permanece
      ↓
célula ainda pode ser evitada
      ↓
novas experiências positivas/neutras
      ↓
valor antigo pode ser atualizado gradualmente
```

Esse comportamento é necessário para testar memória, adaptação e esquecimento.

---

## 16. Exploração

O agente não deve ser perfeitamente racional desde o início.

Ele precisa explorar o ambiente e testar ações mesmo quando existe uma preferência aparente.

No experimento do choque, isso permite:

```text
não conhece o perigo
→ entra
→ toma choque
→ registra experiência
→ ainda pode testar novamente
→ aprende gradualmente
→ começa a evitar
```

A exploração deve diminuir conforme o agente aprende, mas não necessariamente chegar a zero.

---

## 17. Labirinto e droga

O ambiente deverá permitir um experimento no qual a droga esteja atrás de um caminho indireto.

Exemplo conceitual:

```text
S  X  X  X  X
0  X  0  0  0
0  X  0  X  D
0  0  0  X  X
X  X  0  0  0
```

`S = posição inicial`
`D = droga`
`X = obstáculo/perigo`
`0 = espaço transitável`

O agente deve conseguir descobrir que o caminho correto pode inicialmente levá-lo **para longe da droga**, mas ainda assim terminar no destino de maior valor esperado.

Não avaliar apenas a distância geométrica. O valor deve incorporar recompensas futuras e experiências acumuladas.

---

## 18. Conflito entre fome e droga

Um experimento importante será:

```text
fome alta
dependência alta
droga distante
banana próxima
```

Dependência baixa pode favorecer comida.

Dependência crescente pode aumentar o valor esperado da droga.

Dependência muito alta pode produzir comportamento no qual o agente:

- ignora uma recompensa alimentar relativamente fraca;
- procura droga;
- aceita riscos maiores;
- continua procurando mesmo quando a droga não está disponível.

A decisão deve continuar sendo consequência da política/aprendizado e dos estados do agente, e não um script rígido que determine a ação final.

---

## 19. Dependência e ausência da droga

A dependência pode influenciar o estado interno e as recompensas futuras.

Quando a dependência estiver alta e a droga não aparecer durante algum tempo, podem ocorrer gradualmente:

- queda de felicidade;
- queda de energia;
- maior prioridade de busca pela droga;
- menor valor atribuído a recompensas pequenas;
- maior tolerância a risco.

A ausência da droga também deve permitir um teste importante: mesmo sem nenhum objeto `droga` no mapa, um agente dependente pode continuar procurando por algum tempo devido às expectativas aprendidas.

---

## 20. Depressão

A primeira versão pode possuir:

```text
depressao = 0..100
```

A depressão pode ser influenciada por combinações de:

- felicidade muito baixa;
- perda prolongada de uma fonte de recompensa importante;
- dependência elevada;
- ausência de estímulos desejados;
- histórico de experiências negativas.

Possíveis efeitos comportamentais:

- menor exploração;
- menor movimentação;
- menor interesse em objetos distantes;
- maior permanência em um local;
- alimentação somente quando a comida estiver muito próxima;
- redução geral de atividade.

Evitar criar somente regras binárias como `if depressao > 80 -> ficar parado`. Sempre que possível, usar o estado de depressão como parte da observação/estado que influencia a decisão aprendida.

---

## 21. Aprendizado de regiões com recursos

O agente deverá aprender padrões espaciais.

Exemplo: se o espectador colocar bananas repetidamente em uma região do lado direito do mapa, o agente deve acumular experiências positivas naquela região.

Com o tempo, isso pode gerar:

- maior frequência de visitas ao lado direito;
- desvios para essa região;
- preferência por destinos próximos;
- permanência maior naquela região.

O objetivo é que o mapa adquira valor através da experiência.

---

## 22. Recursos removidos

Remover um recurso do mapa não deve apagar automaticamente a memória do agente.

Exemplo:

```text
durante vários dias:
região direita = muita banana

depois:
bananas removidas
```

O agente pode continuar indo para lá por algum tempo porque possui uma expectativa antiga.

Novas experiências devem atualizar essa expectativa gradualmente.

O mesmo vale para o choque: remover o perigo não deve apagar instantaneamente uma aversão aprendida.

---

## 23. Separar estado, memória e aprendizado

Não misturar três conceitos:

### Estado atual

O que está acontecendo agora.

Exemplo:

```json
{
  "position": [3, 3],
  "fome": 50,
  "energia": 80,
  "vida": 100,
  "felicidade": 50,
  "dependencia": 0,
  "depressao": 0,
  "age": 0
}
```

### Memória ambiental

Experiências associadas a células e objetos.

Exemplo:

```text
3,4 → várias experiências negativas
1,5 → várias experiências positivas relacionadas à comida
```

### Q-table / parâmetros aprendidos

Valores utilizados pelo algoritmo de Reinforcement Learning para estimar ações futuras.

Esses componentes devem ficar separados para facilitar depuração e evolução.

---

## 24. Experimentos iniciais

### Experimento 1 — Banana fixa

Colocar banana repetidamente na mesma região e observar se o agente aprende a frequentá-la.

### Experimento 2 — Banana variável

Mover a banana entre regiões e medir a adaptação.

### Experimento 3 — Choque

Adicionar área perigosa e observar quantas experiências são necessárias para formar aversão.

### Experimento 4 — Remoção do choque

Remover o choque e observar por quanto tempo a memória negativa persiste.

### Experimento 5 — Labirinto

Colocar droga atrás de uma rota longa e observar se o agente descobre a rota indireta.

### Experimento 6 — Dependência

Observar mudanças de comportamento conforme a dependência aumenta.

### Experimento 7 — Fome x droga

Comparar o valor de uma banana próxima com uma droga distante em diferentes níveis de dependência.

### Experimento 8 — Ausência da droga

Remover a droga após dependência alta e observar busca persistente e mudanças no comportamento.

### Experimento 9 — Depressão

Criar perda prolongada de recompensas e observar redução de atividade.

---

## 25. Estrutura sugerida

Uma organização inicial possível:

```text
src/
│
├── core/
│   ├── simulation.js
│   ├── agent.js
│   ├── state.js
│   ├── actions.js
│   └── time.js
│
├── environment/
│   ├── environment.js
│   ├── grid.js
│   ├── objects.js
│   └── effects.js
│
├── learning/
│   ├── qLearning.js
│   ├── policy.js
│   ├── exploration.js
│   ├── memory.js
│   └── valueMap.js
│
├── planning/
│   ├── pathPlanner.js
│   └── destinationEvaluator.js
│
├── experiments/
│   └── basicSandbox.js
│
└── ui/
    ├── renderer.js
    ├── controls.js
    └── detailsPanel.js
```

Essa é uma proposta inicial, não uma exigência. O critério principal é manter o código simples, modular e compreensível.

---

## 26. Prioridade de implementação

### Fase 1 — Mundo

- grid 5x5;
- macaquinho;
- movimento;
- tempo;
- dias;
- instantes;
- interface básica.

### Fase 2 — Objetos

- banana;
- quantidade;
- posição;
- interação adjacente;
- área de choque.

### Fase 3 — Aprendizado

- Q-learning;
- exploração;
- recompensas;
- memória por célula;
- atualização dos valores.

### Fase 4 — Destino e planejamento

- avaliação global do ambiente;
- tabela de probabilidade por destino;
- escolha de destino;
- planejamento de caminho;
- replanejamento após mudanças.

### Fase 5 — Droga e dependência

- felicidade;
- dependência;
- tolerância a risco;
- busca da droga;
- conflito com fome;
- efeitos da ausência.

### Fase 6 — Depressão

- variável de depressão;
- redução gradual de atividade;
- efeito sobre exploração e busca.

### Fase 7 — Sandbox

- adicionar/remover objetos durante a execução;
- Show Details;
- edição do cenário;
- experimentos repetíveis;
- métricas.

### Roadmap futuro

- fêmea/companheira;
- vínculo social;
- reprodução;
- efeitos da perda da companheira;
- vocalização;
- expressões;
- comportamento social;
- múltiplos agentes;
- novos ambientes.

---

## 27. Fêmea/companheira — futuro

Não implementar na primeira versão, mas deixar a arquitetura preparada.

Possibilidades:

- segundo agente;
- movimentação própria;
- interação social;
- aproximação e afastamento;
- reprodução;
- formação de preferência/vínculo;
- ganho de felicidade através da interação;
- perda da companheira afetando estado emocional e comportamento.

A intensidade da relação deve crescer através de experiências acumuladas, permitindo comparar um agente no início do vínculo com um agente que passou muito tempo com a companheira.

---

## 28. Comunicação, expressão e deslocamento — futuro

Posteriormente incluir:

### Vocalização

- alerta;
- medo/aflição;
- ameaça;
- contato;
- alimento;
- apaziguamento.

### Expressões

- medo;
- raiva;
- felicidade;
- tristeza;
- curiosidade;
- relaxamento;
- tensão.

### Deslocamento

A grade continuará podendo ser usada como estrutura lógica, enquanto a representação visual do movimento poderá tornar-se mais fluida.

---

## 29. Outros elementos para o sandbox — sugestões futuras

A arquitetura deve permitir adicionar futuramente:

- água;
- abrigo;
- alimentos diferentes;
- objeto desconhecido;
- obstáculos;
- áreas de conforto;
- áreas de risco;
- fontes de calor/frio;
- objetos móveis;
- estímulos sociais;
- mudanças de ambiente feitas pelo espectador.

Um objeto desconhecido pode ser particularmente interessante:

```text
objeto desconhecido
        ↓
curiosidade
        ↓
aproximação
        ↓
interação
        ↓
consequência
        ↓
aprendizado
```

---

## 30. Métricas

Registrar pelo menos:

- posição ao longo do tempo;
- ação escolhida;
- recompensa;
- fome;
- energia;
- felicidade;
- vida;
- dependência;
- depressão;
- experiências positivas por célula;
- experiências negativas por célula;
- destino escolhido;
- probabilidade de cada destino;
- caminho planejado;
- choques sofridos;
- vezes que procurou cada recurso;
- tempo gasto parado;
- quantidade de exploração.

A finalidade é permitir verificar se houve aprendizado, e não apenas observar uma animação.

---

## 31. Princípio fundamental do projeto

O ambiente conhece as regras.

O agente aprende as consequências.

O espectador cria condições.

O agente observa.

O agente escolhe.

O ambiente responde.

O agente recebe recompensa.

O agente atualiza suas expectativas.

O agente tenta novamente.

A meta é observar comportamentos que não foram escritos diretamente como scripts, mas que surjam da interação entre:

```text
ESTADO INTERNO
+
MEMÓRIA
+
AMBIENTE
+
RECOMPENSAS
+
EXPLORAÇÃO
+
PLANEJAMENTO
+
EXPERIÊNCIA
```

A primeira versão deve priorizar clareza, modularidade, depuração e capacidade de experimentar. Complexidade adicional só deve ser adicionada depois que o aprendizado básico estiver comprovadamente funcionando.
