# PRD — Sudoku casual para web

**Status:** primeira versão para validação  
**Plataforma:** web responsiva  
**Público:** iniciantes e jogadores casuais/experientes  
**Experiência desejada:** relaxante, acessível e sem pressão

## 1. Visão do produto

Criar um jogo de Sudoku para navegador que seja fácil de começar, agradável em sessões curtas e capaz de ensinar progressivamente. A primeira versão terá uma campanha curta com puzzles clássicos e uma introdução gradual ao Sudoku diagonal. A interface deve funcionar em celular, tablet e desktop, salvar o progresso localmente e não exigir conta.

## 2. Objetivos

- Permitir que jogadores iniciantes aprendam as regras por meio de um tutorial breve e da campanha.
- Oferecer desafio crescente, com uma meta de aumento de 20% na dificuldade média entre fases.
- Permitir que o jogador retome uma partida interrompida.
- Oferecer ajuda e verificação de erros sem impor cronômetro ou pressão competitiva.
- Manter anúncios raros e fora das partidas.
- Tornar leitura e interação configuráveis para diferentes necessidades visuais e formas de controle.

## 3. Não objetivos da primeira versão

- Aplicativos nativos para Android ou iOS.
- Multiplayer, placares competitivos ou eventos sazonais.
- Streaks ou recompensas que punam a ausência do jogador.
- Variantes além do Sudoku clássico e do Sudoku diagonal.
- Campanha infinita ou conteúdo além das cinco fases definidas.
- Conta obrigatória ou sincronização de progresso entre dispositivos.

## 4. Público e necessidades

O jogo atende desde pessoas que estão aprendendo até quem já conhece Sudoku. A experiência deve permitir que iniciantes recebam instruções e ajuda sem impedir que jogadores experientes resolvam os puzzles por conta própria. O ritmo é escolhido pelo jogador: não há cronômetro obrigatório.

## 5. Estrutura e progressão

- A campanha tem **5 fases**, com **3 puzzles por fase**, totalizando 15 puzzles.
- As fases são concluídas em ordem. É necessário resolver os 3 puzzles de uma fase para desbloquear a seguinte.
- A fase 1 contém puzzles clássicos e introduz as regras básicas.
- A fase 2 contém um puzzle diagonal introdutório e dois puzzles clássicos.
- As fases 3 a 5 misturam puzzles clássicos e diagonais. A distribuição exata será definida durante a curadoria, mantendo os dois modos representados.
- Cada fase concluída desbloqueia um tema visual cosmético.
- Opções necessárias à legibilidade e acessibilidade — como alto contraste, tamanho dos números e pistas que não dependem apenas de cor — ficam disponíveis desde o início; não são recompensas bloqueadas pela campanha.
- Não há cronômetro obrigatório. O tempo pode ser registrado e exibido como estatística opcional.

## 6. Dificuldade e produção dos puzzles

### Progressão de dificuldade

A dificuldade será estimada por uma pontuação interna baseada nas técnicas lógicas necessárias para resolver cada puzzle. A média da pontuação dos puzzles de cada fase terá como meta aumentar 20% em relação à fase anterior. A fase 1 define a linha de base.

O cálculo deve considerar a complexidade e a quantidade de passos lógicos exigidos, com pesos por técnica. A escala e os pesos precisam ser calibrados com testes de resolução e sessões de teste com jogadores. O rótulo de dificuldade não deve depender apenas do número de casas inicialmente preenchidas.

### Geração e validação

Para a primeira versão, os puzzles serão gerados por um processo automatizado e selecionados para um conjunto curado incluído no jogo, em vez de depender de geração não verificada durante a partida. Cada puzzle precisa:

- Ter uma única solução válida.
- Respeitar as regras do modo clássico ou diagonal indicado.
- Ser resolvível pelas técnicas previstas para sua fase.
- Ser verificado pelo solver e pela pontuação de dificuldade antes de ser incluído.
- Ser testado quanto a soluções, dicas e conclusão da partida.

Na variante diagonal, além de linhas, colunas e blocos 3×3, cada uma das duas diagonais principais deve conter os números de 1 a 9 sem repetição. A regra adicional precisa ser explicada antes do primeiro puzzle diagonal e indicada visualmente no tabuleiro.

## 7. Requisitos funcionais

### 7.1 Onboarding e regras

- Antes da primeira partida, apresentar um tutorial interativo curto sobre linhas, colunas e blocos 3×3.
- Apresentar a regra adicional do Sudoku diagonal antes do puzzle introdutório da fase 2.
- Permitir acessar novamente as regras durante a campanha.

### 7.2 Tabuleiro e entrada

- Permitir selecionar uma casa e depois um número.
- Oferecer também o modo de selecionar um número e tocar nas casas em que ele será inserido.
- Permitir escolher o método de entrada nas configurações.
- Oferecer entrada de anotações/candidatos, distinguindo visualmente anotações de respostas definitivas.
- Permitir apagar uma entrada e desfazer ações.
- Diferenciar visualmente números iniciais, respostas inseridas, casa selecionada, conflitos e anotações.

### 7.3 Erros e dicas

- Avisar imediatamente quando uma jogada criar conflito com a regra do modo atual.
- Disponibilizar uma configuração para verificar se uma resposta diverge da solução do puzzle mesmo sem criar conflito imediato.
- Oferecer ajuda gradual: pista, explicação da lógica e opção de revelar a resposta.
- Não encerrar nem reiniciar a partida por causa de erros.

### 7.4 Campanha, estado e estatísticas

- Mostrar fases bloqueadas/desbloqueadas e o progresso nos três puzzles de cada fase.
- Salvar automaticamente a campanha e a partida em andamento no dispositivo/navegador.
- Permitir retomar uma partida após fechar ou atualizar a página, quando o armazenamento local estiver disponível.
- Registrar conclusão e tempo de resolução como estatística opcional.
- Não exigir cadastro. O progresso não sincroniza entre navegadores ou dispositivos.

### 7.5 Aparência e acessibilidade

- Oferecer tema claro, tema escuro, opção de alto contraste e ajuste do tamanho dos números.
- Não comunicar conflitos, seleção ou estado de jogo somente por cor; combinar cor com contorno, forma, ícone ou outro sinal visual.
- Manter controles e casas utilizáveis em telas pequenas, com alvos de toque e espaçamento testados em dispositivos reais.
- Adaptar o layout para celular, tablet e desktop.

### 7.6 Monetização

- O jogo é gratuito, com anúncios e uma compra única para removê-los.
- Exibir no máximo um anúncio após cada grupo de três puzzles concluídos, sempre entre partidas; nunca interromper um puzzle em andamento.
- Se um anúncio não estiver disponível, permitir que o jogador continue sem erro ou bloqueio.
- A compra de remoção de anúncios fica vinculada ao navegador/dispositivo e não sincroniza entre dispositivos. Essa limitação deve ser comunicada antes da compra.
- A implementação web deverá validar a compra com o provedor de pagamento; não usar apenas um valor editável no armazenamento local como prova de compra.

## 8. Requisitos não funcionais

- O jogo deve ser utilizável nos navegadores modernos para desktop e dispositivos móveis.
- A partida em andamento deve ser preservada em interrupções comuns, como atualizar ou fechar a página, respeitadas as limitações do armazenamento do navegador.
- O jogo deve permanecer jogável sem conexão depois que os recursos necessários tiverem sido carregados, dentro das possibilidades da implementação web.
- Preferências visuais e de controle devem ser persistidas localmente.
- O conteúdo do puzzle e as regras devem ser consistentes entre navegadores, sem depender de anúncios ou conexão para validar uma solução.

## 9. Critérios de aceitação da primeira versão

1. Um jogador pode iniciar a campanha, aprender as regras clássicas no tutorial e concluir três puzzles para liberar a fase 2.
2. A fase 2 apresenta e explica Sudoku diagonal em seu primeiro puzzle; seus outros dois puzzles são clássicos.
3. A campanha contém exatamente cinco fases e três puzzles por fase; cada fase exige a conclusão dos três puzzles anteriores.
4. A média da pontuação interna de dificuldade tem como alvo um aumento de 20% entre fases consecutivas; o conjunto final passa por validação automatizada e teste de jogo.
5. O jogador consegue alternar entre os dois métodos de entrada, usar anotações, apagar e desfazer jogadas.
6. Conflitos são sinalizados imediatamente; a verificação de divergência da solução pode ser ligada ou desligada.
7. Dicas oferecem pista, explicação e revelação; uma jogada errada não encerra a partida.
8. O progresso da campanha e da partida em andamento pode ser retomado no mesmo navegador/dispositivo sem cadastro.
9. Tamanho dos números, temas claro/escuro e alto contraste podem ser ajustados desde o início; informações importantes não dependem apenas de cor.
10. A versão gratuita não exibe mais de um anúncio após cada três puzzles concluídos e nunca exibe anúncios durante um puzzle.
11. Cada fase concluída libera um tema cosmético; configurações necessárias à acessibilidade permanecem disponíveis sem desbloqueio.

## 10. Indicadores para avaliação

- Taxa de conclusão do tutorial e da primeira fase.
- Percentual de jogadores que iniciam e concluem cada fase, identificando pontos de abandono.
- Frequência de uso de dicas, desfazer, anotações e verificação de erros.
- Taxa de retomada de partidas interrompidas.
- Erros de solução, puzzles sem solução única ou divergências na pontuação, detectados nos testes automatizados.
- Feedback de legibilidade e facilidade de uso em celular, tablet e desktop.

Os indicadores devem ser coletados de forma compatível com a decisão de não exigir conta. Qualquer análise remota de uso precisará de uma decisão explícita sobre privacidade e consentimento.

## 11. Premissas e decisões pendentes

- O idioma inicial da interface ainda precisa ser confirmado; este documento está em português.
- A escala, os pesos das técnicas e o procedimento de calibração do aumento de 20% serão definidos durante a implementação e os testes.
- A distribuição exata entre puzzles clássicos e diagonais nas fases 3 a 5 será definida na curadoria.
- A política de uso de dicas (gratuitas, ilimitadas ou limitadas) ainda precisa ser especificada.
- O mecanismo de compra e restauração no navegador depende do provedor de pagamento escolhido. A decisão atual exclui sincronização entre dispositivos.
- “Offline quando possível” depende da estratégia de cache e dos recursos externos adotados; deve ser confirmado com um teste de desconexão após o primeiro carregamento.
- A quantidade e os nomes dos temas visuais serão definidos no escopo visual, mantendo as opções de acessibilidade disponíveis no início.
