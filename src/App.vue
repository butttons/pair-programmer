<template lang="pug">
h1.title Tic-tac-toe
.board-wrapper
  .board
    button.board__tile(
      v-for='tile, index of gameState',
      :key='index',
      @click='handleTileClick(index)'
    ) 
      | {{ tile !== "empty" ? tile : "" }}
.info
  button.button(@click='handleNewGame') Start new game
  .form-items
    .form-item
      label(for='player-one') Player One
      input#player-one(
        type='text',
        placeholder='Player #1',
        v-model='gamePlayerInfo.names[0]'
      )
    .form-item
      label(for='player-two') Player Two
      input#player-two(
        type='text',
        placeholder='Player #2',
        v-model='gamePlayerInfo.names[1]'
      )
  .game-info
    .text-pill
      span Currently playing:
      span {{ currentPlayerName }}
    .text-pill
      span Game state:
      span(:class='gameStateClass') {{ currentGameCondition }}
</template>
<script lang="ts">
  import { computed, defineComponent, onMounted, ref, watch } from 'vue';
  const CACHE_KEY = 'game_state';
  type TileState = 'empty' | 'x' | 'o';
  type GameCondition = 'ongoing' | 'won' | 'draw';

  const WINNING_CONDITIONS = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];

  export default defineComponent({
    name: 'App',
    setup() {
      const gamePlayerInfo = ref({
        names: ['', ''],
      });
      const gameState = ref<TileState[]>([]);
      const gameCondition = ref<GameCondition>('ongoing');
      const gameWinner = ref(-1);
      const activePlayerIndex = ref(0);

      const getPlayerName = (index: number) =>
        gamePlayerInfo.value.names[index] === ''
          ? `Player #${index + 1}`
          : gamePlayerInfo.value.names[index];

      const currentPlayerName = computed(() =>
        getPlayerName(activePlayerIndex.value),
      );
      const currentGameCondition = computed(() => {
        switch (gameCondition.value) {
          case 'ongoing':
            return 'Game is ongoing';
          case 'won':
            return `${getPlayerName(gameWinner.value)} won`;
          case 'draw':
            return 'Game is draw. Start a new one.';
          default:
            return '';
        }
      });
      const gameStateClass = computed(() => ({
        'state--won': gameCondition.value === 'won',
        'state--ongoing': gameCondition.value === 'ongoing',
        'state--draw': gameCondition.value === 'draw',
      }));

      const initGameCache = () => {
        const savedCached = localStorage.getItem(CACHE_KEY);
        if (savedCached === null) return;
        const cacheNames = JSON.parse(savedCached);
        gamePlayerInfo.value.names = cacheNames;
      };
      const initGameState = () => {
        for (let i = 0; i < 9; i++) {
          gameState.value.push('empty');
        }
      };
      const resetGameState = () =>
        gameState.value.forEach((_, index) => (gameState.value[index] = 'empty'));

      const toggleActivePlayer = () =>
        (activePlayerIndex.value = activePlayerIndex.value === 0 ? 1 : 0);

      const checkGameState = () => {
        const anyWinningCondition = WINNING_CONDITIONS.find((winIndices) => {
          const winConditionTiles = winIndices.map(
            (winIndex) => gameState.value[winIndex],
          );
          const anyEmptyTile = winConditionTiles.includes('empty');
          const areTilesSame = winConditionTiles.every(
            (winTile) => winTile !== 'empty' && winTile === winConditionTiles[0],
          );

          if (areTilesSame) return true;
          if (anyEmptyTile) return false;
        });

        const isGameWon = anyWinningCondition !== undefined;
        if (isGameWon) {
          const winningTile = gameState.value[anyWinningCondition![0]];
          gameWinner.value = winningTile === 'x' ? 0 : 1;
          gameCondition.value = 'won';
          return;
        }

        const isGameDraw = !gameState.value.includes('empty');
        if (isGameDraw) {
          gameCondition.value = 'draw';
          return;
        }
      };

      const handleTileClick = (tileIndex: number) => {
        const isEmptyTile = gameState.value[tileIndex] === 'empty';
        const isGameWon = gameCondition.value === 'won';
        if (!isEmptyTile || isGameWon) return;

        const tileState = activePlayerIndex.value === 0 ? 'x' : 'o';
        gameState.value[tileIndex] = tileState;
        checkGameState();
        toggleActivePlayer();
      };

      const handleNewGame = () => {
        resetGameState();
        activePlayerIndex.value = 0;
        gameCondition.value = 'ongoing';
        gameWinner.value = -1;
      };

      onMounted(() => {
        initGameCache();
        initGameState();
      });

      watch(
        () => gamePlayerInfo.value.names,
        (val) => {
          localStorage.setItem(CACHE_KEY, JSON.stringify(val));
        },
        { deep: true },
      );

      return {
        gamePlayerInfo,
        gameState,

        currentPlayerName,
        currentGameCondition,
        gameStateClass,

        handleNewGame,
        handleTileClick,
      };
    },
  });
</script>
<style>
  h1.title {
    text-align: center;
    color: #374151;
  }
  .board-wrapper {
    padding: 1rem;
  }
  .board {
    width: 100%;
    display: grid;
    grid-gap: 1rem;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    grid-template-row: repeat(3, minmax(0, 1fr));
  }
  @media (min-width: 768px) {
    .board {
      width: 60vw;
      margin: 0 auto;
    }
  }
  @media (min-width: 1024px) {
    .board {
      width: 50vw;
    }
  }
  .board__tile {
    width: 100%;
    height: 6rem;
    background-color: #d1d5db;
    border: 1px solid #9ca3af;
    padding: 1rem;
    border-radius: 0.5rem;
    transition: background-color 0.25s ease-in;
    cursor: pointer;
    font-size: 1rem;
    text-transform: uppercase;
  }
  .board__tile:hover {
    background-color: #9ca3af;
  }
  .info {
    text-align: center;
  }
  .info > * + * {
    margin-top: 1rem;
    margin-bottom: 1rem;
  }
  .form-items > * + *,
  .game-info > * + * {
    margin-top: 0.5rem;
    margin-bottom: 0.5rem;
  }

  button.button {
    padding: 0.5rem 1rem;
    transition: background-color 0.25s ease-in;
    background: #3b82f6;
    border: 1px solid #2563eb;
    color: white;
    border-radius: 0.25rem;
    cursor: pointer;
  }
  button.button:hover {
    background-color: #1d4ed8;
  }

  .form-item > label {
    font-size: 0.75rem;
    margin-right: 0.5rem;
  }
  .form-item > input {
    padding: 0.5rem;
    border: 1px solid #6b7280;
    border-radius: 0.25rem;
  }
  .form-item > input:focus {
    outline: none;
    border-color: #60a5fa;
  }

  .text-pill > span {
    display: inline-block;
    padding: 0.5rem;
  }
  .text-pill > span:first-child {
    background-color: #e5e7eb;
    border-top-left-radius: 0.25rem;
    border-bottom-left-radius: 0.25rem;
  }
  .text-pill > span:last-child {
    background-color: #d1d5db;
    border-top-right-radius: 0.25rem;
    border-bottom-right-radius: 0.25rem;
  }
  .text-pill > span:last-child {
    font-weight: bold;
  }
  .text-pill > span.state--won {
    background-color: #a7f3d0;
    color: #10b981;
  }
  .text-pill > span.state--ongoing {
    background-color: #bfdbfe;
    color: #3b82f6;
  }
  .text-pill > span.state--draw {
    background-color: #fde68a;
    color: #d97706;
  }
</style>