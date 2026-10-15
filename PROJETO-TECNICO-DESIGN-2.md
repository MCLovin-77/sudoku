# Projeto técnico — Sudoku web baseado no Design 2

**Status:** proposta técnica para orientar a implementação  
**Design de referência:** Design 2 — “Acessibilidade · operação inclusiva”, em [prototype-accessibility.html](./prototype-accessibility.html)  
**Requisitos de produto:** [PRD.md](./PRD.md)

## 1. Objetivo e escopo

Este documento descreve como transformar o segundo mockup em um jogo web funcional. O design prioriza operação por teclado, leitura clara, alvos grandes, contraste configurável e instruções explícitas. A implementação preservará essa hierarquia visual: tabuleiro como área principal, comandos e preferências em um painel associado, progresso da campanha em local visível e mensagens anunciadas por tecnologia assistiva.

O escopo funcional é a primeira versão do PRD:

- Campanha finita de cinco fases, com três puzzles por fase.
- Sudoku clássico na fase 1; um puzzle diagonal introdutório seguido por dois clássicos na fase 2; mistura dos modos nas fases 3 a 5.
- A fase seguinte é liberada após a conclusão dos três puzzles da fase atual.
- Aumento-alvo de 20% na pontuação média interna de dificuldade entre fases.
- Tutorial clássico inicial, explicação de Sudoku diagonal antes do primeiro puzzle desse modo, dicas graduais e verificação de conflitos.
- Salvamento local sem conta, funcionamento offline depois do carregamento, temas claro/escuro, alto contraste, tamanho dos números e dois métodos de entrada.
- Anúncios limitados a um após cada grupo de três puzzles concluídos, sempre entre partidas, com compra única para removê-los.

Não fazem parte desta entrega: multiplayer, placares, conta obrigatória, sincronização entre dispositivos, campanha infinita ou variantes além do clássico e diagonal.

## 2. Princípios técnicos e decisões recomendadas

1. **A regra do jogo não depende da interface.** Validação, conflitos, solução, dicas e pontuação vivem em módulos de domínio testáveis sem navegador.
2. **Acessibilidade é requisito de estrutura.** Semântica, foco, teclado, contraste e mensagens para leitor de tela são implementados junto com os componentes, não adicionados no final.
3. **O cliente não gera puzzles para uma partida publicada.** Um processo de desenvolvimento/compilação gera e valida os puzzles; a aplicação distribui um catálogo fixo e curado.
4. **Persistência deve refletir o que realmente foi salvo.** Se o navegador negar armazenamento, a interface informa que o progresso está apenas na sessão atual.
5. **Conteúdo principal não depende da rede.** Regras, puzzles e validação funcionam localmente; anúncios e compra podem exigir conexão e nunca bloqueiam uma partida.
6. **Uma jogada inválida não destrói progresso.** O jogador pode corrigir, apagar ou desfazer sem perder a partida.

### Stack sugerida

O repositório ainda não define uma stack de aplicação. Para uma implementação sustentável, recomenda-se:

- **TypeScript + React + Vite** para composição da tela, estado e build estático.
- **CSS próprio com variáveis de design**, sem dependência de imagens ou fontes remotas para renderizar o jogo.
- **IndexedDB** para estado da campanha e partidas; preferências simples podem ficar no mesmo banco para facilitar versionamento.
- **Service Worker** para cache dos arquivos da aplicação e do catálogo curado, habilitando uso offline após o primeiro carregamento.
- **Vitest** para testes de domínio e componentes; **Playwright** para fluxos completos em tamanhos de tela distintos.

React/TypeScript é uma recomendação, não uma dependência existente no repositório. Se a equipe escolher outra stack, a separação de domínio, interface e infraestrutura descrita aqui deve ser preservada.

## 3. Arquitetura da aplicação

### 3.1 Camadas

```text
Interface React
  ├── Campanha e navegação
  ├── Tabuleiro acessível e comandos
  ├── Preferências visuais
  └── Mensagens, tutorial e diálogos
          │ ações tipadas
          ▼
Estado de jogo / casos de uso
  ├── selecionar casa e número
  ├── inserir, anotar, apagar, desfazer
  ├── pedir dica e verificar conclusão
  └── concluir puzzle / avançar campanha
          │
          ├── Domínio Sudoku e solver
          ├── Repositório local (IndexedDB)
          ├── Catálogo curado de puzzles
          └── Adaptadores de anúncios e compra
```

A camada visual não deve calcular regras Sudoku dentro de componentes. Os componentes enviam ações ao estado de jogo e recebem um estado já derivado (por exemplo, células em conflito, selecionadas ou bloqueadas).

### 3.2 Organização sugerida dos arquivos

```text
src/
  app/
    App.tsx
    routes.ts
  components/
    CampaignHeader.tsx
    SudokuGrid.tsx
    SudokuCell.tsx
    NumberPad.tsx
    GameActions.tsx
    AccessibilitySettings.tsx
    LiveAnnouncement.tsx
    Dialog.tsx
  features/
    campaign/
      campaign-model.ts
      campaign-service.ts
      CampaignProgress.tsx
    game/
      game-state.ts
      game-reducer.ts
      game-actions.ts
      hint-service.ts
    puzzle/
      puzzle-types.ts
      rules.ts
      candidates.ts
      validator.ts
      solver.ts
      difficulty.ts
      catalog.ts
    persistence/
      database.ts
      game-repository.ts
      migrations.ts
    monetization/
      ad-service.ts
      purchase-service.ts
      provider-types.ts
  styles/
    tokens.css
    themes.css
    app.css
  data/
    puzzles.json
  main.tsx
public/
  icons/
scripts/
  generate-puzzles.ts
  validate-puzzles.ts
tests/
  unit/
  integration/
  e2e/
```

Os nomes podem ser adaptados à convenção adotada pelo projeto. O objetivo da divisão é impedir que a regra, o salvamento, a compra e o layout acabem acoplados em um único componente.

## 4. Modelo de domínio

### 4.1 Tipos principais

```ts
type SudokuMode = "classic" | "diagonal";
type CellValue = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9;
type EntryMethod = "cell-first" | "digit-first";
type HintStage = 0 | 1 | 2 | 3;

interface PuzzleDefinition {
  id: string;
  phase: 1 | 2 | 3 | 4 | 5;
  order: 1 | 2 | 3;
  mode: SudokuMode;
  givens: CellValue[];       // 81 posições, zero representa casa vazia
  solution: Exclude<CellValue, 0>[]; // 81 posições
  difficultyScore: number;
  techniques: string[];     // perfil usado na calibração e auditoria
}

interface GameSession {
  puzzleId: string;
  values: CellValue[];
  notes: number[][];        // candidatos por casa
  elapsedSeconds: number;
  hintCount: number;
  mistakeChecking: "rules-only" | "solution";
  completedAt?: string;
  updatedAt: string;
}

interface Preferences {
  theme: "light" | "dark";
  highContrast: boolean;
  numberScale: number;      // faixa inicial recomendada: 85–145%
  entryMethod: EntryMethod;
  mistakeChecking: "rules-only" | "solution";
}

interface CampaignProgress {
  completedPuzzleIds: string[];
  unlockedPhase: 1 | 2 | 3 | 4 | 5;
  unlockedThemeIds: string[];
  adMilestonesShown: number[]; // marcos de puzzles concluídos já processados
}
```

As interfaces reais devem validar dados carregados do IndexedDB antes de usá-los. Um save antigo ou incompleto não deve derrubar a tela nem ser interpretado como progresso concluído.

### 4.2 Estado de interface

O estado transitório da partida deve incluir:

- Puzzle atual e sua grade.
- Casa selecionada (`row`, `column`), número selecionado no modo número-primeiro e método de entrada.
- Modo de anotações.
- Histórico de ações para desfazer.
- Conflitos de regra calculados para as entradas atuais.
- Estado da dica e mensagens de status.
- Estado de diálogo/tutorial visível.

As preferências persistentes não devem ser duplicadas em vários componentes. O estado da partida usa uma única fonte de verdade, atualizada por ações com tipo explícito (`CELL_SELECTED`, `DIGIT_SELECTED`, `VALUE_ENTERED`, `NOTE_TOGGLED`, `MOVE_UNDONE`, `HINT_REQUESTED` etc.).

## 5. Regras, validação e dificuldade

### 5.1 Validação de jogadas

`rules.ts` deve implementar funções puras:

- `getConflicts(values, mode)`: encontra repetições na linha, coluna, bloco 3×3 e, no modo diagonal, nas diagonais principais aplicáveis.
- `isSolutionComplete(values, solution)`: confirma que as 81 casas correspondem à solução.
- `getCandidateDigits(values, row, column, mode)`: calcula candidatos por regra.

Manter duas verificações separadas:

1. **Conflito de regra:** número duplicado em uma unidade relevante. É sinalizado imediatamente conforme o PRD.
2. **Divergência da solução:** entrada que não é a resposta pretendida, mesmo sem conflito imediato. Só é mostrada se a preferência de verificação da solução estiver ativa.

O estado visual não deve depender exclusivamente de vermelho ou outra cor. Usar também contorno/ícone, texto de status e, quando útil, sublinhado. Entradas iniciais são bloqueadas; casas preenchidas pelo jogador continuam editáveis e podem ser desfeitas.

### 5.2 Dicas progressivas

O serviço de dicas opera sobre o puzzle e a grade atual:

1. **Pista:** aponta uma área ou casa útil sem fornecer imediatamente o número.
2. **Explicação:** descreve a técnica e a dedução esperada.
3. **Revelação:** mostra o valor correto da casa escolhida, com confirmação visual e anúncio acessível.

Cada solicitação avança um estágio para o mesmo alvo; escolher outra casa reinicia a sequência da nova dica. A ajuda não conta como erro e não encerra a partida.

### 5.3 Curadoria e solver

Os 15 puzzles são selecionados antes do lançamento e distribuídos com a aplicação. O script de geração/validação deve:

1. Criar candidatos de puzzle no modo indicado.
2. Contar soluções com busca de backtracking, parando após encontrar uma segunda solução. Só puzzles com exatamente uma solução são aprovados.
3. Confirmar que a solução respeita linhas, colunas, blocos e, quando aplicável, diagonais.
4. Executar um solver lógico para identificar técnicas utilizadas e garantir que o puzzle é resolvível sem adivinhação pelas técnicas esperadas naquela fase.
5. Calcular a pontuação usando pesos calibráveis por técnica e passos: `score = soma(peso_da_técnica × ocorrências)`.
6. Gerar um relatório de validação; puzzles fora do perfil de fase são rejeitados ou revisados.

A média de `difficultyScore` de cada fase tem meta de `média_da_fase_n ≈ média_da_fase_(n-1) × 1,20`. A fase 1 define a escala-base. Como esta é uma meta de design, o relatório deve mostrar o desvio real e a equipe deve calibrar pesos/conjunto antes de publicar; não se deve alterar a dificuldade em tempo de jogo para forçar o número.

Distribuição fixa obrigatória:

| Fase | Conteúdo |
|---|---|
| 1 | 3 clássicos |
| 2 | 1 diagonal introdutório + 2 clássicos |
| 3 | 3 puzzles com mistura de clássico e diagonal |
| 4 | 3 puzzles com mistura de clássico e diagonal |
| 5 | 3 puzzles com mistura de clássico e diagonal |

Nas fases 3 a 5, a composição exata será definida pela curadoria. Cada fase concluída desbloqueia um tema cosmético. Opções de contraste, tema claro/escuro e tamanho de número não podem ser bloqueadas por progresso.

## 6. Implementação da interface Design 2

### 6.1 Estrutura responsiva

- **Desktop/tablet largo:** cabeçalho com identidade e controles de aparência; abaixo, área de jogo em duas colunas. Tabuleiro à esquerda e controles/configurações à direita.
- **Tela estreita:** uma coluna; informações da fase e tabuleiro primeiro, controles logo depois. Nenhum controle essencial depende de hover ou de largura de desktop.
- O tabuleiro preserva proporção quadrada e grade 9×9. O tamanho da casa acompanha a largura disponível, sem causar overflow horizontal.
- Aumentar o zoom do navegador até 200% não deve esconder ações nem exigir rolagem horizontal para completar uma jogada.

### 6.2 Tabuleiro e semântica

Implementar a grade como estrutura semântica de 9 linhas e 9 colunas: contêiner `role="grid"`, nove linhas `role="row"` e casas `role="gridcell"` com botão acessível interno. Usar foco móvel (roving `tabIndex`): apenas a casa atual recebe `tabIndex=0`, e as setas movem a seleção entre casas.

Cada casa precisa expor nome acessível que informe coordenadas, valor ou estado vazio, se é pista inicial e se há conflito. Exemplo: “Linha 4, coluna 7, vazia, selecionada”. Não anunciar a grade inteira em cada atualização.

O número da casa não pode ser o único estado. Codificar pista inicial, entrada do jogador, seleção e conflito por combinação de peso tipográfico, fundo, borda/contorno e mensagem de status. As diagonais do modo diagonal devem ter indicação persistente além da cor.

### 6.3 Teclado, apontador e entrada

Suportar:

- Setas: mover o foco/seleção na grade, com limites nas bordas.
- `1`–`9`: inserir número (ou alternar candidato se anotações estiverem ativas).
- `N`: ligar/desligar modo de anotações.
- `Backspace`/`Delete`: apagar a casa editável selecionada.
- `Ctrl+Z`/`Cmd+Z`: desfazer a última ação.
- `Escape`: fechar diálogos e cancelar modos transitórios.

Todos os comandos também têm botões visíveis e nomeados. O método “selecionar casa e depois número” é o padrão; “selecionar número e depois casas” deve seguir disponível nas preferências. O foco do teclado não pode ser removido ao renderizar novamente o tabuleiro.

### 6.4 Preferências visuais e acessibilidade

- Tema claro, escuro e alto contraste são opções independentes e persistidas localmente.
- O controle de tamanho deve ajustar os números da grade e não encolher ícones/controles de forma a torná-los ilegíveis.
- Usar contraste mínimo WCAG AA para texto e controles; verificar também os estados selecionado, conflito, foco e hover em cada tema.
- Alvos clicáveis e controles devem ter pelo menos 44×44 CSS px como padrão de produto, sempre que o layout permitir; casas são quadradas, com área clicável equivalente à própria célula.
- `:focus-visible` precisa ser bem visível, sem depender da cor do fundo.
- Respeitar `prefers-reduced-motion`; animações de seleção/transição são curtas e decorativas, nunca necessárias para entender a partida.
- Mensagens de erro e conclusão usam texto, não apenas cor ou som.

O protótipo é referência visual, não prova de conformidade. Acessibilidade deve ser validada com teclado, zoom, leitor de tela e usuários reais.

## 7. Persistência, offline e estatísticas

### 7.1 Banco local

Usar IndexedDB com stores versionadas:

- `sessions`: estado da partida atual e partidas retomáveis.
- `campaign`: puzzles concluídos, fase desbloqueada, temas liberados e marcos de anúncios processados.
- `preferences`: aparência, tamanho dos números, método de entrada e verificação de erros.

Salvar após cada ação que altera o tabuleiro, ao mudar preferências e ao concluir puzzle/fase. Para mudanças de alta frequência (por exemplo, tempo decorrido), agrupar gravações sem atrasar o salvamento de movimentos. Atualizações relacionadas à conclusão e campanha devem ocorrer na mesma transação para evitar estado inconsistente.

Ao abrir, validar o esquema e reconciliar progresso com o catálogo conhecido. Se IndexedDB estiver indisponível ou cheio, mostrar aviso persistente de que o estado poderá não sobreviver ao fechamento da aba; a partida permanece jogável.

### 7.2 Offline

O Service Worker deve cachear os arquivos da aplicação, CSS, ícones e catálogo de puzzles. Primeiro carregamento requer rede; depois de uma instalação/cache concluídos, a campanha e validação local funcionam sem conexão. Se recursos offline ainda não estiverem prontos, informar isso claramente. Anúncios e compra podem não estar disponíveis offline e não impedem o jogo.

### 7.3 Tempo e análise

Não iniciar cronômetro competitivo nem exigir rapidez. Se o tempo for exibido, deve ser uma estatística opcional baseada no tempo acumulado da sessão e pausada quando a página não está ativa. Não transmitir estatísticas para servidor sem decisão explícita de privacidade e consentimento; o PRD não exige conta nem analytics remoto.

## 8. Anúncios e compra sem conta

Definir interfaces de adaptador para não acoplar a UI a um provedor:

```ts
interface AdProvider {
  showBetweenPuzzles(): Promise<"shown" | "unavailable" | "dismissed">;
}

interface PurchaseProvider {
  startAdFreePurchase(installId: string): Promise<void>;
  restoreForInstall(installId: string): Promise<boolean>;
}
```

### Anúncios

- Incrementar a contagem apenas quando um puzzle for realmente concluído pela primeira vez.
- Agendar no máximo um anúncio nos marcos 3, 6, 9, 12 e 15 puzzles concluídos, e marcar o marco como processado antes de solicitar ao provedor para impedir duplicação por refresh.
- Apresentar somente depois da tela de conclusão, entre dois puzzles; nunca sobrepor tabuleiro ativo nem interromper uma jogada.
- Se offline, bloqueador, indisponibilidade ou erro do provedor impedir o anúncio, não bloquear avanço e não exibir erro que assuste o jogador.
- Não duplicar anúncio quando a tela de conclusão for reaberta.

### Remoção de anúncios

A interface deve explicar antes da compra que a remoção vale apenas para este navegador/dispositivo e não sincroniza. A compra precisa ser verificada pelo provedor em um endpoint seguro ou fluxo equivalente; um booleano no IndexedDB/localStorage nunca prova pagamento. Guardar entitlement localmente para a experiência offline, com validação/expiração conforme o mecanismo escolhido.

O provedor e o desenho exato de restauração ainda precisam ser escolhidos. Sem conta, limpar dados do navegador pode apagar o identificador local; o produto deve explicar essa limitação e não prometer recuperação/sincronização que não implementa. Requisitos legais, impostos e processamento de pagamento devem ser revisados antes do lançamento.

## 9. Fluxos de produto

1. **Primeiro acesso:** carregar catálogo → mostrar tutorial de regras clássicas → abrir puzzle 1 da fase 1.
2. **Jogar:** escolher casa/número → atualizar grade e status → salvar estado local → permitir desfazer/corrigir.
3. **Pedir dica:** exibir pista → ao pedir de novo, explicar técnica → permitir revelar a resposta.
4. **Concluir puzzle:** validar solução completa → gravar conclusão uma única vez → atualizar os três marcadores da fase → avaliar marco de anúncio.
5. **Concluir fase:** depois do terceiro puzzle, liberar próximo puzzle/fase e tema cosmético correspondente.
6. **Primeiro diagonal:** antes do puzzle diagonal da fase 2, explicar que as duas diagonais principais também não podem repetir números; destacar visualmente as diagonais.
7. **Retomar:** recarregar → validar estado salvo → retornar ao puzzle e preferências persistidos.
8. **Acessibilidade:** aplicar configurações sem reiniciar partida; anunciar alteração relevante no status acessível.

## 10. Plano de testes

### Unitários

- Duplicatas em linha, coluna, bloco e diagonal; ausência de conflito em casos válidos.
- Separação entre conflito de regra e divergência da solução.
- Cálculo de candidatos e conclusão.
- Dicas em cada estágio.
- Desfazer entradas e anotações sem alterar pistas iniciais.
- Progressão de campanha, desbloqueio de temas e idempotência dos anúncios.
- Solver conta zero, uma e múltiplas soluções e rejeita puzzle inválido.
- Pontuação e relatório de dificuldade por fase.

### Integração

- Salvar e restaurar partida, preferências e progresso com IndexedDB.
- Migrar dados de versão anterior ou tratar dados corrompidos/indisponíveis.
- Alternar tema/contraste/tamanho sem perder foco ou partida.
- Método de entrada casa-primeiro e número-primeiro.
- Compra validada e modo sem anúncios; provedor de anúncios indisponível não bloqueia jogo.

### Ponta a ponta e acessibilidade

- Tutorial → concluir três puzzles da fase 1 → liberar fase 2 e tema.
- Fase 2 explica a regra diagonal antes de abrir o puzzle.
- Atualizar/fechar/retomar e jogar com rede desligada após primeiro cache.
- Navegação completa por teclado, leitores de tela (NVDA + Firefox/Chrome e VoiceOver + Safari quando disponível), zoom 200%, alto contraste e movimento reduzido.
- Verificar layouts em 320, 375, 768 e 1280 CSS px; nenhum controle essencial causa rolagem horizontal.
- Validar anúncio nos marcos previstos e assegurar zero anúncios durante puzzle ativo.

### Critérios de aceite técnicos

1. O catálogo publicado contém 15 puzzles, todos com solução única validada e modo correto.
2. O progresso só libera a fase seguinte após três conclusões distintas da fase.
3. A média de dificuldade reportada pelo script busca aumento de 20% por fase e os desvios estão documentados antes da aprovação do catálogo.
4. A grade é utilizável por teclado e leitor de tela, com estado de casa e conflito anunciado sem depender de cor.
5. Estado da partida é retomável no mesmo navegador depois de recarregar, quando o armazenamento estiver disponível.
6. O jogo iniciado/cacheado continua jogável sem rede; nenhum serviço externo é necessário para validar uma jogada.
7. A versão gratuita apresenta no máximo um anúncio após cada três conclusões, nunca no meio do puzzle; indisponibilidade não bloqueia avanço.
8. Testes responsivos não detectam overflow horizontal nos tamanhos definidos, inclusive no painel de controles.

## 11. Sequência de implementação

1. **Fundação:** confirmar stack, configurar build/testes, tokens CSS e estrutura de estado.
2. **Domínio:** regras clássicas/diagonais, candidato, conflitos, solução, histórico e testes unitários.
3. **Catálogo:** solver, geração/validação em build, pontuação, curadoria e relatório das cinco fases.
4. **Tabuleiro:** semântica acessível, foco móvel, teclado, controles de toque, notas e desfazer.
5. **Experiência:** tutorial, dicas graduais, campanha, conclusão de puzzle/fase e temas desbloqueáveis.
6. **Persistência/offline:** IndexedDB, migrações, recuperação de estado e Service Worker.
7. **Monetização:** adaptadores, provedor escolhido, limites de anúncio, validação da compra e avisos de dispositivo.
8. **Polimento e lançamento:** auditoria de acessibilidade, navegadores/dispositivos, testes com jogadores, revisão de conteúdo e implantação.

Cada etapa deve terminar com critérios de aceite e testes automatizados relevantes passando antes de avançar para a seguinte.

## 12. Riscos e decisões pendentes

- **Stack e hospedagem:** ainda não existem no repositório; confirmar antes de adicionar dependências.
- **Métrica dos 20%:** os pesos e técnicas do solver precisam ser calibrados. Não tratar contagem de casas vazias como substituto da dificuldade lógica.
- **Distribuição das variantes:** o conteúdo exato das fases 3–5 permanece a cargo da curadoria.
- **Compra no navegador sem conta:** selecionar provedor e desenhar restauração para o mesmo navegador; esclarecer comportamento após limpeza de dados.
- **Anúncios em web/offline:** disponibilidade depende do provedor, consentimento e restrições de rede; falha deve ser não bloqueante.
- **Persistência do navegador:** armazenamento pode ser apagado ou recusado; comunicar isso e preservar uma sessão em memória.
- **Acessibilidade real:** o segundo mockup precisa ser validado com pessoas e tecnologias assistivas; os rótulos do protótipo são ponto de partida, não auditoria final.
- **Idioma:** este plano e os mockups estão em português brasileiro; confirmar idioma(s) de lançamento e estratégia de tradução.
