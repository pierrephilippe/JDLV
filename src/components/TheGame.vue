<template>
  <v-container class="d-flex flex-column justify-center align-center">
    <h2>Grille QR Code 25x25</h2>
    <v-text-field
      v-model="textInput"
      label="Entrez le texte"
      @input="generateQRCode"
      @keyup.enter="generateQRCode"
      class="textfield"
    ></v-text-field>


    <div class="controller d-flex flex-row mb-3">
      <v-btn @click="clear">
        <svg-icon type="mdi" :path="mdiRestart"></svg-icon>
      </v-btn>
      <v-btn @click="back" :disabled="history.length === 0">
        <svg-icon type="mdi" :path="mdiChevronLeft"></svg-icon>
      </v-btn>
      <v-btn @click="next">
        <svg-icon type="mdi" :path="mdiChevronRight"></svg-icon>
      </v-btn>
      <v-btn @click="play" :disabled="isPlaying">
        <svg-icon type="mdi" :path="mdiPlay"></svg-icon>
      </v-btn>
      <v-btn @click="pause" :disabled="!isPlaying">
        <svg-icon type="mdi" :path="mdiPause"></svg-icon>
      </v-btn>
    </div>

    <div class="compteur">
      Compteur : {{ compteur }}
    </div>

    <div class="grid">
      <div v-for="(row, rowIndex) in grid" :key="rowIndex" class="row">
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="cell"
          :class="{ full: cell === 1 }"
          @click="toggle(rowIndex, colIndex)"
        ></div>
      </div>
    </div>

    <div>
      <v-card width="27rem" class="border-0 my-3">
        <v-card-title class="text-h6 text-md-h5 text-lg-h4">Jeu de la vie</v-card-title>
        <v-card-text>
          Le jeu de la Vie est un « jeu à zéro joueur », puisqu'il ne nécessite aucune intervention du joueur lors de son déroulement.<br/>
          Il s’agit d’un automate cellulaire, un modèle où chaque état conduit mécaniquement à l’état suivant à partir de règles préétablies.<br/>
          À chaque itération, l'état d’une cellule est entièrement déterminé par l’état de ses huit cellules voisines, selon les règles suivantes :<br />
          <br />
           - Une cellule morte possédant exactement trois cellules voisines vivantes devient vivante (elle naît)<br />
           - Une cellule vivante ne possédant pas exactement deux ou trois cellules voisines vivantes meurt.<br />
        </v-card-text>
      </v-card>

    </div>
  </v-container>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import QRCode from "qrcode";
// @ts-ignore
import SvgIcon from '@jamescoyle/vue-icon';
import { mdiChevronLeft, mdiChevronRight, mdiPlay, mdiPause, mdiRestart } from '@mdi/js';

const grid = ref<number[][]>(Array(25).fill(0).map(() => Array(25).fill(0)));
const textInput = ref<string>("https://jeudelavieqrcode.netlify.app/");
const compteur = ref(0);
const history = ref<number[][][]>([]);
const isPlaying = ref(false);
let intervalId: NodeJS.Timeout | null = null;
const generateQRCode = async () => {
  compteur.value = 0;
  history.value = [];
  try {
    // Utilisation de la bibliothèque qrcode pour générer le QR code sous forme de matrice binaire
    const qrData = await QRCode.toDataURL(textInput.value, { width: 25, scale:1, margin: 0});
    console.log("qurcode")
    console.log(qrData)
    // Extraction du QR code sous forme de matrice binaire
    const image = new Image();
    image.src = qrData;
    image.onload = () => {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");
      if (ctx) {
        canvas.width = 25;
        canvas.height = 25;
        ctx.drawImage(image, 0, 0, 25, 25);

        // Récupération des données pixel
        const imageData = ctx.getImageData(0, 0, 25, 25);
        const pixels = imageData.data;

        // Remplissage de la grille avec 0 (blanc) ou 1 (noir)
        const newGrid = Array(25).fill(0).map(() => Array(25).fill(0));
        for (let row = 0; row < 25; row++) {
          for (let col = 0; col < 25; col++) {
            const index = (row * 25 + col) * 4; // Calcul de l'index dans l'image
            const r = pixels[index];
            const g = pixels[index + 1];
            const b = pixels[index + 2];
            const a = pixels[index + 3];

            // Si la couleur est noire (ou proche du noir), on met 1, sinon 0
            newGrid[row][col] = (r < 128 && g < 128 && b < 128 && a > 128) ? 1 : 0;
          }
        }
        grid.value = newGrid;
      }
    };
  } catch (error) {
    console.error("Erreur lors de la génération du QR code", error);
  }
};

onMounted(() => {
  generateQRCode();
});

// Fonction pour basculer la valeur d'une cellule entre 0 et 1
const toggle = (rowIndex: number, colIndex: number) => {
  // Inverse la valeur de la cellule
  grid.value[rowIndex][colIndex] = grid.value[rowIndex][colIndex] === 0 ? 1 : 0;
};

const clear = () => {
  compteur.value = 0;
  history.value = ([]);
  grid.value = (Array(25).fill(0).map(() => Array(25).fill(0)));
}
// Fonction pour avancer d'un pas dans le jeu de la vie
const next = () => {
  const currentGrid = JSON.parse(JSON.stringify(grid.value)); // Copie de la grille actuelle

  // Vérifier si la grille actuelle existe déjà dans l'historique
  if (history.value.some(pastGrid => JSON.stringify(pastGrid) === JSON.stringify(currentGrid))) {
    console.log("Le jeu s'est stabilisé ou est cyclique, arrêt du processus.");
    pause(); // Interrompt la progression
    return;
  }

  compteur.value++;
  history.value.push(JSON.parse(JSON.stringify(grid.value)));
  const newGrid = grid.value.map((row, rowIndex) =>
    row.map((cell, colIndex) => {
      const liveNeighbors = countLiveNeighbors(rowIndex, colIndex);
      // Règles du jeu de la vie
      if (cell === 1) {
        return liveNeighbors === 2 || liveNeighbors === 3 ? 1 : 0; // Une cellule vivante reste vivante si elle a 2 ou 3 voisins vivants
      } else {
        return liveNeighbors === 3 ? 1 : 0; // Une cellule morte devient vivante si elle a 3 voisins vivants
      }
    })
  );
  grid.value = newGrid;
};

// Fonction pour revenir à l'état précédent de la grille
const back = () => {
  compteur.value--;
  if (history.value.length > 0) {
    grid.value = history.value.pop()!;
  }
};

// Fonction pour compter le nombre de voisins vivants autour d'une cellule
const countLiveNeighbors = (rowIndex: number, colIndex: number): number => {
  let count = 0;
  const neighbors = [
    [-1, -1], [-1, 0], [-1, 1],
    [0, -1], [0, 1],
    [1, -1], [1, 0], [1, 1]
  ];
  neighbors.forEach(([dx, dy]) => {
    const newRow = rowIndex + dx;
    const newCol = colIndex + dy;
    if (newRow >= 0 && newRow < 25 && newCol >= 0 && newCol < 25) {
      count += grid.value[newRow][newCol];
    }
  });
  return count;
};

// Fonction pour démarrer la boucle du jeu (Play)
const play = () => {
  if (!isPlaying.value) {
    isPlaying.value = true;
    intervalId = setInterval(() => {
      next();
    }, 150); // Avance d'un pas toutes les 500ms
  }
};

// Fonction pour arrêter la boucle du jeu (Pause)
const pause = () => {
  if (isPlaying.value && intervalId) {
    clearInterval(intervalId);
    isPlaying.value = false;
  }
};
</script>

<style scoped lang="scss">
.textfield {
  width: 25rem;
}

.grid {
  display: grid;
  grid-template-rows: repeat(25, 1rem);
  grid-template-columns: repeat(25, 1rem);
  gap: 0px;
  border: 1px solid #000;
  // width: calc(50rem + 2px);
}

.row {
  display: contents;
}

.cell {
  display: inline-block;
  width: 1rem;
  height: 1rem;
  background-color: #fff;
  border: 0px solid #ccc;
  &.full {
    background-color: #000;
  }
}
</style>
