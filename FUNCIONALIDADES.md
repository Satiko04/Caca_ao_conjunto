# Funcionalidades do sistema — Missão Matemática

Documento de referência com todas as telas, mecânicas e funções JavaScript do projeto, organizado por arquivo.

## Estrutura do projeto

| Arquivo | Papel |
|---|---|
| [index.html](index.html) | Tela de login/avatar e mapa de seleção de jogos |
| [caca-ao-conjunto.html](caca-ao-conjunto.html) | Jogo 1 — Caça ao Conjunto (meteoros) |
| [diagrama-de-venn.html](diagrama-de-venn.html) | Jogo 2 — Diagrama de Venn (arrastar e soltar) |
| [pocoes-magicas.html](pocoes-magicas.html) | Jogo 3 — Laboratório de Poções Mágicas (frações) |

Não há backend: tudo roda no navegador e o progresso é salvo em `localStorage`.

## Armazenamento (localStorage)

| Chave | Conteúdo |
|---|---|
| `mq_player` | `{ name, avatar }` do jogador atual |
| `mq_sound` | `true`/`false` — som ligado/desligado |
| `mq_progress` | `{ [nomeEmMinúsculo]: { [gameId]: { best, stars } } }` — recorde e estrelas por jogador e por jogo |
| `mq_gas` | `{ [nomeEmMinúsculo]: { gas, stars } }` — banco de gás acumulado (compartilhado entre os três jogos) e quantas estrelas já foram trocadas por esse jogador |
| `mq_pocoes_niveis` | `{ [nomeEmMinúsculo]: { unlocked[5], stars[5], best[5] } }` — progresso por nível do Laboratório de Poções Mágicas |
| `mq_caca_grimorio` | `{ [nomeEmMinúsculo]: { [itemId]: true } }` — itens colecionáveis desbloqueados no Caça ao Conjunto |
| `mq_venn_grimorio` | `{ [nomeEmMinúsculo]: { [itemId]: true } }` — itens colecionáveis desbloqueados no Diagrama de Venn |

---

## index.html — Mapa de missões

### Tela de boas-vindas
- Campo de nome (`nameInput`, máx. 16 caracteres, Enter também confirma).
- Grade de 12 avatares (`AVATARS`) selecionáveis por clique.
- Botão **Decolar 🚀** (`launchBtn`) — valida nome, salva `mq_player` e toca um jingle.

### Mapa de planetas
- Cabeçalho com avatar, nome e **patente** calculada a partir do total de estrelas (`RANKS`: Cadete Espacial → Explorador(a) Estelar → Piloto Galáctico → Comandante do Universo).
- Barra de XP mostrando progresso até a próxima patente.
- Contador de estrelas totais (`⭐ totalStars`) — soma as estrelas de desempenho dos jogos **e** as estrelas trocadas por gás (`mq_gas[jogador].stars`).
- Card de cada jogo (`caca`, `venn`, `pocoes`) com: estrelas conquistadas (0–3), recorde de pontos, selo **NOVO** quando ainda não jogado, e link para o jogo.
- Unidade 2 "Expedição Galáctica": o Planeta Místico (frações) já é jogável (`pocoes-magicas.html`); os outros seis planetas seguem **bloqueados** (Oceano — decimais, Jurássico — tabelas e gráficos, Máquina — fluxogramas, Carga da Nave — volume, Doce — massa, Extremos — temperatura), reservados para conteúdo futuro.
- Botão de som (liga/desliga efeitos sonoros, salvo em `mq_sound`).
- Botão **Sair** (`exitBtn`) → modal de confirmação → apaga `mq_player` e volta à tela de boas-vindas (troca de jogador).
- Atualiza o mapa automaticamente ao voltar de um jogo pelo botão "voltar" do navegador (`pageshow`).
- Fundo com estrelas piscando e um "meteoro cadente" decorativo (CSS).

### Funções JavaScript principais

| Função | O que faz |
|---|---|
| `store.get/set/del` | Leitura/escrita segura no `localStorage` (não quebra se estiver bloqueado) |
| `beep(freq, dur, type, vol, delay)` | Gera um som simples via Web Audio API (sem arquivos de áudio) |
| `showWelcome()` | Exibe a tela de cadastro do jogador |
| `showMap()` | Carrega o jogador salvo, calcula estrelas/patente/XP e desenha o mapa |
| `launchBtn.onclick` | Valida o nome, salva o jogador e abre o mapa |
| `exitBtn` / `stayBtn` / `leaveBtn` | Abrem/fecham o modal de saída e trocam de jogador |
| `readGasBank()` / `writeGasBank(mine)` | Leem/gravam o banco de gás (`mq_gas`) do jogador atual |
| `gasTradeBtn` / `gasExchangeBtn` | Abrem a tela de troca e convertem 5000 de gás em 1 estrela |

---

## caca-ao-conjunto.html — Caça ao Conjunto (jogo 1)

**Mecânica:** meteoros caem do topo da tela trazendo um elemento (número, letra ou figura); o jogador precisa mandá-lo para o conjunto correto (portal A ou B) antes que ele atinja o chão.

### Telas (overlays)
- **Início** — regras do jogo, botão **🗺️ Mapa** e **Começar 🚀**.
- **Rodada** — apresenta o tema da rodada atual, com botão **🗺️ Mapa** e **Estou pronto(a)!** *(botão de voltar adicionado nesta tela)*.
- **Pausa** — reiniciar ou continuar.
- **Sair** — confirma abandono da partida (pontos não são salvos).
- **Fim de missão** — estrelas conquistadas, pontuação final, indicação de novo recorde, **🗺️ Mapa** e **Jogar de novo**.

### Rodadas
1. Pares × Ímpares
2. Vogais × Consoantes
3. Múltiplos de 3
4. Triângulos × Quadriláteros (figuras geradas em SVG na hora)

### Mecânicas de jogo
- Cada rodada tem 8 elementos, sem repetir o mesmo item (`uniqueGen`).
- Velocidade de queda por rodada (`time`); se o meteoro chegar ao chão sem resposta, conta como erro.
- Resposta pelos **portais A/B** (clique/toque) ou teclas **← / →**.
- Pontuação: 10 pontos base + bônus por rapidez + 5 pontos extra em combo (3+ acertos seguidos).
- No lugar do ícone fixo de estrela, o pill de pontuação mostra uma **nuvem de gás** que cresce conforme o banco acumulado se aproxima de 5000 (veja "Gás e estrelas trocáveis" abaixo).
- Combo zera ao errar; vidas (❤️❤️❤️) diminuem a cada erro ou meteoro perdido.
- Pontinhos (`dots`) mostram acerto/erro de cada item da rodada.
- Contador **✅ acertos** no HUD, somado durante toda a partida (não zera entre rodadas).
- Pausa automática ao trocar de aba (`visibilitychange`) e manual (botão ⏸️ ou tecla `P`/`Esc`).
- Ao final: 3 estrelas (venceu com todas as vidas), 2 (venceu com menos vidas), 1 (perdeu depois da 3ª rodada), 0 (perdeu antes). Recorde e estrelas são salvos por jogador. A tela de fim também mostra o **aproveitamento** (acertos de X tentativas, em %).
- Confete e som de vitória ao concluir as 4 rodadas.
- **Grimório 📖** (botão na tela inicial): coleção de 6 itens salva em `mq_caca_grimorio` — um por rodada concluída (`caca_r1`–`caca_r4`), um por vencer sem perder vida (`caca_perfeito`) e um por acertar 10 meteoros seguidos (`caca_combo`). Cada item mostra um fato curioso ao ser desbloqueado; um aviso "📖 ... desbloqueada!" aparece na tela seguinte (rodada ou fim de jogo).

### Funções JavaScript principais

| Função | O que faz |
|---|---|
| `uniqueGen(fn)` | Evita repetir o mesmo elemento na mesma rodada |
| `polygonSVG(n)` | Desenha um polígono aleatório de `n` lados em SVG |
| `paintHUD()` | Atualiza vidas e pontuação na tela |
| `paintRound()` / `showRoundIntro()` | Preenchem os dados da rodada atual (nome dos conjuntos, dicas, ícone) |
| `spawn()` | Cria um novo meteoro com um elemento sorteado |
| `loop(t)` | Laço de animação (`requestAnimationFrame`) que move o meteoro para baixo |
| `answer(side)` | Processa a resposta do jogador (acerto/erro), pontua e anima |
| `miss()` | Trata o meteoro que chegou ao chão sem resposta |
| `next(delay)` | Avança para o próximo meteoro ou rodada |
| `startRound()` / `resetGame()` / `stop()` | Controlam início, reinício e parada do jogo |
| `pause()` / `resume()` | Pausam/retomam a partida |
| `confetti()` | Efeito visual de confete na vitória |
| `endGame(won)` | Calcula estrelas, salva recorde e mostra a tela final |

---

## diagrama-de-venn.html — Diagrama de Venn (jogo 2)

**Mecânica:** arrastar (ou tocar e depois tocar no diagrama) cada elemento para a região correta de um diagrama de Venn com 2 ou 3 conjuntos.

### Telas (overlays)
Mesmo padrão do jogo 1: **Início**, **Rodada** (com botão **🗺️ Mapa** adicionado), **Pausa**, **Sair** e **Fim de missão**.

### Rodadas
1. Pares e Múltiplos de 3 — 2 conjuntos, números de 1 a 20.
2. Bichos que voam e que nadam — 2 conjuntos, animais.
3. Desafio dos 3 conjuntos — 3 conjuntos (par / múltiplo de 3 / maior que 10), números de 1 a 24.

### Mecânicas de jogo
- Os itens de cada rodada são sorteados garantindo exemplos de todas as combinações possíveis de pertencimento (`pickItems`), incluindo interseções e elementos fora de qualquer conjunto.
- Arraste com Pointer Events, ou toque no elemento e depois toque no diagrama.
- A posição do soltar é comparada com os círculos do diagrama para validar a região.
- Acerto: fixa o elemento no diagrama (com animação), soma pontos (10 + bônus de combo) e avança o progresso.
- Erro: o elemento treme, mostra uma explicação (`explain`) do porquê está errado, perde uma vida e zera o combo.
- Botão **💡 Dica** — seleciona um elemento e mostra as perguntas-guia dos conjuntos.
- Vidas, pausa, som e telas de fim seguem o mesmo esquema do jogo 1 (3/2/1/0 estrelas, recorde salvo por jogador), incluindo o contador **✅ acertos** no HUD, o aproveitamento (%) na tela final e o banco de **gás trocável por estrela**.
- **Grimório 👾** (botão na tela inicial): coleção de 6 itens salva em `mq_venn_grimorio` — um por rodada concluída (`venn_r1`–`venn_r3`), um por vencer sem perder vida (`venn_perfeito`), um por vencer sem usar a dica (`venn_semdica`) e um por acertar 8 elementos seguidos (`venn_combo`).

### Funções JavaScript principais

| Função | O que faz |
|---|---|
| `pickItems(R)` | Sorteia os elementos da rodada cobrindo todas as combinações de conjuntos |
| `drawDiagram()` | Desenha os círculos do diagrama de Venn em SVG |
| `buildTray()` | Monta a bandeja de elementos arrastáveis |
| `attachDrag(chip)` | Liga os eventos de arrastar/tocar em cada elemento |
| `tryDrop(chip, x, y)` | Verifica se o elemento foi solto na região certa |
| `correct()` / `wrong()` | Tratam acerto (fixa no diagrama, pontua) e erro (treme, explica, perde vida) |
| `explain(it)` | Gera o texto explicando a quais conjuntos o item pertence |
| `hintBtn.onclick` | Mostra a dica do elemento selecionado |
| `showRoundIntro()` / `startRound()` / `resetGame()` | Controlam o fluxo entre telas e rodadas |
| `pause()` / `resume()` | Pausam/retomam a partida |
| `confetti()` | Efeito visual de confete na vitória |
| `endGame(won)` | Calcula estrelas, salva recorde e mostra a tela final |

---

## pocoes-magicas.html — Laboratório de Poções Mágicas (jogo 3)

**Mecânica:** exercícios de frações organizados como um **mapa de 5 níveis independentes** (um por operação), onde toda fração aparece desenhada num frasco/caldeirão dividido em partes iguais (nunca só como número em linha).

### Telas (overlays)
**Início** ("Laboratório de Poções Mágicas") → **Mapa de níveis** (5 frascos, um por nível) → **Grimório** (coleção, acessível do mapa) → **Nível** (introdução com ícone, explicação e exemplo visual, botão "Já sei, bora! ▶" para pular) → **Jogo** → **Pausa/Sair** → **Fim de nível** ("Poção pronta! 🏆" ou "O caldeirão esfriou...").

### Mapa e progressão (`mq_pocoes_niveis`)
- 5 frascos no mapa, um por nível. O **nível 1 começa liberado**; completar um nível libera o próximo, salvo em `mq_pocoes_niveis` (`{ [jogador]: { unlocked[], stars[], best[] } }`). Qualquer nível liberado pode ser rejogado a qualquer momento.
- Cada nível é uma sessão independente: **5 poções, 3 vidas próprias**. Perder todas as vidas reinicia só aquele nível (`finishLevel(false)`), sem afetar o progresso dos outros.
- **Estrelas do nível**: 3 = concluiu sem perder vida, 2 = perdeu 1 ou 2 vidas, 1 = concluiu mas usou a dica (prevalece sobre as outras regras). O jogo grava sempre o melhor resultado (`Math.max`).
- **Estrelas finais** (as que aparecem no card do jogo no mapa principal, `index.html`) são a **média das estrelas dos 5 níveis**, sincronizada em `mq_progress` toda vez que um nível é concluído (`syncOverallProgress`).

### Grimório 📖
Cada nível concluído pela primeira vez desbloqueia uma poção colecionável com nome e emoji próprios (ex.: "Poção do Arroto Arco-Íris 🌈"), guardada em `GRIMOIRE` e exibida numa tela de coleção acessível pelo botão 📖 do mapa — poções de níveis ainda não concluídos aparecem como "❓ Bloqueada".

### Níveis (5 poções cada)
1. 🧪 Misturando ingredientes — adição com mesmo denominador. Resposta por **múltipla escolha** (3 frascos), com erros comuns como opção (ex.: somar os denominadores).
2. 🥄 Usando parte da poção — subtração com mesmo denominador. Também múltipla escolha (ex.: somar em vez de subtrair).
3. 🔮 Receitas de mestres — denominadores diferentes: o aluno primeiro escolhe o denominador comum entre 3 opções, depois monta o resultado num **teclado numérico (1 a 12)**.
4. 📜 Metade da receita — multiplicação ("metade de" ou "N × 1/den"), com modelo de área para fração × fração. Múltipla escolha (ex.: multiplicar só o numerador).
5. 🍶 Enchendo frascos menores — divisão (fração ÷ fração unitária ou fração ÷ inteiro), respondida também pelo **teclado numérico (1 a 12)**.

### Mecânicas de jogo
- Toda fração é renderizada como um frasco (`makeFlask`) com o número de traços igual ao denominador e o líquido preenchendo a fração do numerador, além do formato empilhado (`fracHTML`).
- **Níveis 1, 2 e 4**: múltipla escolha entre 3 frascos (`setupMC`/`checkMCAnswer`) — as duas alternativas erradas (`mcWrong`) representam confusões comuns do conteúdo daquele nível.
- **Níveis 3 e 5**: a fração é montada tocando em botões numéricos de 1 a 12 (`setupKeypad`/`buildKeypad`), sem `+`/`−`; aceita qualquer fração equivalente à correta e dá **+5 de bônus** com "Poção perfeita! 🌟" quando a resposta já está simplificada.
- Todo texto de pergunta (`recipeText`) cabe numa linha; a explicação completa só aparece depois de errar (`explain`, no toast "Ops! ...").
- Sem repetir a mesma conta na mesma sessão de um nível (`uniqueGen`).
- Vidas, combo, HUD de gás/acertos e recorde de pontuação por nível seguem o mesmo esquema dos outros dois jogos.

### Botão de dica (💡)
- Cada poção tem **uma** dica, mostrada num balão de fala de uma corujinha 🦉 (`showOwlHint`), com uma pequena animação no frasco/caldeirão que acompanha o texto (`playHintAnimation`): partes piscando (adição), a parte usada escurecida (subtração), os frascos se subdividindo no novo denominador (nível 3), a interseção destacada no modelo de área (multiplicação) ou frasquinhos de exemplo aparecendo ao lado (divisão).
- A dica usa os números da questão atual, mas nunca escreve o resultado final da conta.
- Usar a dica não tira vida nem pontos, mas cancela o bônus de "Poção perfeita! 🌟" daquela questão — a resposta certa ainda conta, só sem o `+5` extra.
- O botão continua sempre ativo (pode ser clicado de novo na mesma poção, mas o bônus de simplificada já fica cancelado desde o primeiro uso).

### Funções JavaScript principais

| Função | O que faz |
|---|---|
| `genAdd` / `genSub` / `genMixed` / `genMul` / `genDiv` | Geram as poções de cada nível (um gerador por nível), com distratores de múltipla escolha ou faixa 1–12 já embutidos conforme o nível |
| `fracHTML(n, d)` | Monta o HTML da fração no formato empilhado |
| `makeFlask(n, d, cor)` / `rebuildFlaskTicks` | Desenham o frasco/caldeirão e suas divisões |
| `renderQuestion(Q)` | Decide qual visual e painel de resposta mostrar (múltipla escolha, escolha de denominador ou teclado numérico) |
| `renderAreaGrid` | Desenha o modelo de área (grade) da multiplicação |
| `buildMCOptions()` / `setupMC()` / `checkMCAnswer()` | Montam e conferem a resposta de múltipla escolha (níveis 1, 2 e 4) |
| `buildKeypad()` / `setupKeypad()` / `checkFractionAnswer()` | Montam e conferem a resposta pelo teclado numérico (nível 3 etapa 2 e nível 5) |
| `chooseDen(v)` | Valida a escolha do denominador comum (nível 3, etapa 1) |
| `showOwlHint()` / `hideOwlHint()` / `playHintAnimation(Q)` | Mostram o balão da coruja e a animação da dica no frasco/diagrama correspondente |
| `renderExample(Q)` | Monta o exemplo visual da tela de introdução de cada nível |
| `readLevelProgress()` / `writeLevelProgress()` / `syncOverallProgress()` | Leem/gravam o progresso por nível e sincronizam a média de estrelas com `mq_progress` |
| `paintLevelMap()` / `showLevelMap()` / `paintBook()` | Desenham o mapa de níveis e o Grimório |
| `showRoundIntro()` / `startLevel()` / `finishLevel(won)` | Controlam o fluxo entre telas, inicia uma sessão de nível e calcula/salva o resultado ao terminar |

---

## Mecânicas compartilhadas entre os três jogos

- **Som**: efeitos gerados via Web Audio API (sem arquivos de áudio), com botão 🔊/🔇 que persiste em `mq_sound`.
- **Fundo estelar**: estrelas piscando geradas dinamicamente por JavaScript.
- **Fluxo de telas**: Início → Rodada → Jogo → (Pausa | Sair) → Fim.
- **Navegação para o mapa**: botão **🗺️ Mapa** disponível na tela inicial, na tela de rodada e na tela de fim de cada jogo.
- **Vidas, pontuação e combo**: 3 vidas por partida, combo de 3+ acertos seguidos dá bônus de pontos.
- **Acompanhamento de acertos**: contador ✅ no HUD durante o jogo e resumo de aproveitamento (acertos/tentativas em %) na tela final.
- **Feedback imediato**: som, texto flutuante, toast explicativo e animação (flash no portal, tremida no elemento) a cada resposta certa ou errada.
- **Tentar novamente**: botão "🔄 Reiniciar" na pausa e "Jogar de novo 🔄" na tela de fim, sempre reiniciando pontuação, vidas e acertos do zero (o banco de gás **não** é afetado — veja abaixo).
- **Estrelas de resultado**: 0 a 3 por partida, sempre salvando o melhor resultado (recorde) por jogador em `mq_progress`.
- **Grimório**: Caça ao Conjunto e Diagrama de Venn têm cada um sua própria coleção de itens (acessível pelo botão 📖 na tela inicial), desbloqueados ao completar rodadas e desafios específicos (vencer sem perder vida, sem usar dica, ou emendar uma sequência de acertos). Um aviso "📖 ... desbloqueada!" aparece na tela seguinte quando isso acontece.

### Gás e estrelas trocáveis

Cada ponto ganho em qualquer um dos dois jogos também é somado a um **banco de gás persistente**, guardado por jogador (`mq_gas`, chave = nome em minúsculo, compartilhado entre Caça ao Conjunto e Diagrama de Venn).

- **Nunca se perde**: diferente da pontuação da partida (que zera se você sair ou reiniciar), o gás é gravado no `localStorage` a cada acerto (`addGas(pts)`), então sair no meio de uma partida não desconta nada do banco.
- **Visual em jogo**: o pill de pontuação do HUD, em ambos os jogos, mostra uma nuvem que cresce conforme o banco se aproxima de 5000 — é só um indicador, sem botão de troca ali.
- **Troca manual no mapa**: o botão **➕** fica ao lado do **⭐ totalStars**, no topo do mapa (`index.html`), e pulsa em amarelo quando o jogador já tem gás suficiente. Clicar nele abre a tela "Trocar gás por estrela", com uma barra de progresso até 5000. O botão **Trocar por ⭐** só fica ativo com 5000+ de gás; cada troca desconta 5000 do banco, soma 1 à contagem de estrelas trocadas (`mq_gas[jogador].stars`) e toca o som de "estrela formada".
- **Reflexo no mapa**: as estrelas trocadas entram na soma do **⭐ totalStars**, junto com as estrelas de desempenho de cada jogo, para calcular a patente e a barra de XP.
