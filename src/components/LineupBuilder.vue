<template>
    <div class="soccer-lineup">
      <div class="controls">
        <!-- Save Button -->
        <button @click="saveLineup">Save Lineup</button>
  
        <!-- Formation Selector -->
        <div class="formation-selector">
          <label for="formation">Formation:</label>
          <select id="formation" v-model="selectedFormation" @change="applyFormation">
            <option value="4-4-2">4-4-2</option>
            <option value="3-5-2">3-5-2</option>
            <option value="4-3-3">4-3-3</option>
          </select>
        </div>
  
        <!-- Color Picker for Team Circle -->
        <div class="color-picker">
          <label for="teamColor">Team Color:</label>
          <input type="color" id="teamColor" v-model="teamColor" @change="redrawCanvas" />
        </div>
  
        <!-- Color Picker for Circle Outline -->
        <div class="color-picker">
          <label for="circleOutlineColor">Circle Outline:</label>
          <input type="color" id="circleOutlineColor" v-model="circleOutlineColor" @change="redrawCanvas" />
        </div>
  
        <!-- Color Picker for Number Color -->
        <div class="color-picker">
          <label for="numberColor">Number Color:</label>
          <input type="color" id="numberColor" v-model="numberColor" @change="redrawCanvas" />
        </div>
  
        <!-- Color Picker for Field Color -->
        <div class="color-picker">
          <label for="fieldColor">Field Color:</label>
          <input type="color" id="fieldColor" v-model="fieldColor" @change="redrawCanvas" />
        </div>
  
        <!-- Color Picker for Player Name Color -->
        <div class="color-picker">
          <label for="nameColor">Player Name Color:</label>
          <input type="color" id="nameColor" v-model="nameColor" @change="redrawCanvas" />
        </div>
  
        <!-- Input for Renaming Player -->
        <div v-if="selectedPlayer" class="rename-input">
          <input
            ref="renameInput"
            v-model="selectedPlayer.name"
            @blur="redrawCanvas"
            @keyup.enter="redrawCanvas"
            placeholder="Enter player name"
          />
        </div>
      </div>
  
      <!-- Canvas for Lineup -->
      <canvas ref="lineupCanvas" class="lineupCanvas"></canvas>
    </div>
  </template>

  
  <script>
 export default {
  name: 'SoccerLineup',
  data() {
    return {
      selectedFormation: '4-4-2',
      players: [],
      formations: {
        '4-3-3': [
          { x: 250, y: 640, number: 1, role: 'GK', name: 'Player 1' },
          { x: 300, y: 500, number: 2, role: 'DF', name: 'Player 2' },
          { x: 200, y: 500, number: 3, role: 'DF', name: 'Player 3' },
          { x: 400, y: 450, number: 4, role: 'DF', name: 'Player 4' },
          { x: 100, y: 450, number: 5, role: 'DF', name: 'Player 5' },
          { x: 325, y: 350, number: 6, role: 'MF', name: 'Player 6' },
          { x: 175, y: 350, number: 7, role: 'MF', name: 'Player 7' },
          { x: 250, y: 400, number: 8, role: 'MF', name: 'Player 8' },
          { x: 400, y: 250, number: 10, role: 'MF', name: 'Player 10' },
          { x: 100, y: 250, number: 11, role: 'FW', name: 'Player 11' },
          { x: 250, y: 200, number: 9, role: 'FW', name: 'Player 9' },
        ],
        '4-4-2': [
          { x: 250, y: 640, number: 1, role: 'GK', name: 'Player 1' },
          { x: 300, y: 500, number: 2, role: 'DF', name: 'Player 2' },
          { x: 200, y: 500, number: 3, role: 'DF', name: 'Player 3' },
          { x: 400, y: 450, number: 4, role: 'DF', name: 'Player 4' },
          { x: 100, y: 450, number: 5, role: 'DF', name: 'Player 5' },
          { x: 300, y: 350, number: 6, role: 'MF', name: 'Player 6' },
          { x: 100, y: 300, number: 7, role: 'MF', name: 'Player 7' },
          { x: 200, y: 350, number: 8, role: 'MF', name: 'Player 8' },
          { x: 400, y: 300, number: 10, role: 'MF', name: 'Player 10' },
          { x: 200, y: 200, number: 11, role: 'FW', name: 'Player 11' },
          { x: 300, y: 200, number: 9, role: 'FW', name: 'Player 9' },
        ],
      },
      selectedPlayer: null,
      isMouseDown: false,
      teamColor: '#0F43FF', // Default team circle color
      circleOutlineColor: '#fff', // Default circle outline color
      numberColor: '#fff', // Default number color
      fieldColor: '#0f360f', // Default field color
      nameColor: '#fff', // Default player name color
    };
  },
  mounted() {
    this.setupHighResolutionCanvas(); // Initialize high-res canvas
    this.applyFormation();
    this.setupCanvas();
  },
  methods: {
    setupHighResolutionCanvas() {
      const canvas = this.$refs.lineupCanvas;
      const scale = window.devicePixelRatio || 1;
      const originalWidth = 500;
      const originalHeight = 700;

      // Set internal dimensions (scaled)
      canvas.width = originalWidth * scale;
      canvas.height = originalHeight * scale;

      // Set display dimensions (CSS)
      canvas.style.width = `${originalWidth}px`;
      canvas.style.height = `${originalHeight}px`;

      // Scale the drawing context
      const ctx = canvas.getContext('2d');
      ctx.scale(scale, scale);
    },
    // Draw the soccer field
    drawField() {
      const canvas = this.$refs.lineupCanvas;
      const ctx = canvas.getContext('2d');

      // Field background
      ctx.fillStyle = this.fieldColor;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // Field outline
      ctx.strokeStyle = '#aaa';
      ctx.lineWidth = 3;
      ctx.strokeRect(8, 8, canvas.width / 2 - 16, canvas.height / 2 - 16);

      // Center line
      ctx.beginPath();
      ctx.moveTo(8, canvas.height / 4);
      ctx.lineTo(canvas.width / 2 - 8, canvas.height / 4);
      ctx.stroke();

      // Center circle
      ctx.beginPath();
      ctx.arc((canvas.width / 2) / 2, (canvas.height / 2) / 2, 50, 0, Math.PI * 2);
      ctx.stroke();

      // Opponent Box
      ctx.strokeRect((canvas.width / 2) * 0.25, 8.5, (canvas.width / 2) * 0.5, (canvas.height / 2) * 0.19);
      ctx.strokeRect((canvas.width / 2) * 0.375, 8.5, (canvas.width / 2) * 0.25, (canvas.height / 2) * 0.07);

      // Draw a half-circle
      ctx.beginPath();
      ctx.arc((canvas.width / 2) * 0.5, (canvas.height / 2) * 0.201, (canvas.width / 2) * 0.07, 0, Math.PI);
      ctx.stroke();

      // Home Box
      ctx.strokeRect((canvas.width / 2) * 0.25, (canvas.height / 2) - 8.5, (canvas.width / 2) * 0.5, (canvas.height / 2) * -0.19);
      ctx.strokeRect((canvas.width / 2) * 0.375, (canvas.height / 2) - 8.5, (canvas.width / 2) * 0.25, (canvas.height / 2) * -0.07);

      // Draw a half-circle
      ctx.beginPath();
      ctx.arc((canvas.width / 2) * 0.5, (canvas.height / 2) * 0.799, (canvas.width / 2) * 0.07, Math.PI, 0);
      ctx.stroke();
    },
      // Draw players on the field
      drawPlayers() {
      const canvas = this.$refs.lineupCanvas;
      const ctx = canvas.getContext('2d');

      this.players.forEach(player => {
        // Draw player circle
        ctx.beginPath();
        ctx.arc(player.x, player.y, 20, 0, Math.PI * 2);
        ctx.fillStyle = this.teamColor;
        ctx.strokeStyle = this.circleOutlineColor;
        ctx.fill();
        ctx.stroke();

        // Draw player number with shadow
        ctx.fillStyle = '#000'; // Shadow color
        ctx.font = '16px Arial';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText(player.number, player.x + 1, player.y + 1);
        ctx.fillStyle = this.numberColor;
        ctx.fillText(player.number, player.x, player.y);

        // Draw player name with shadow
        ctx.fillStyle = '#000'; // Shadow color
        ctx.font = '14px Verdana';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'top';
        ctx.fillText(player.name, player.x + 1, player.y + 26);
        ctx.fillStyle = this.nameColor; // Use nameColor for name text
        ctx.fillText(player.name, player.x, player.y + 25);
      });
    },
      // Apply selected formation
      applyFormation() {
        this.players = this.formations[this.selectedFormation].map(player => ({ ...player }));
        this.redrawCanvas();
      },
      // Redraw the entire canvas
      redrawCanvas() {
        const canvas = this.$refs.lineupCanvas;
        const ctx = canvas.getContext('2d');
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        this.drawField();
        this.drawPlayers();
      },
      // Set up canvas event listeners for dragging players
      setupCanvas() {
        const canvas = this.$refs.lineupCanvas;
  
        canvas.addEventListener('mousedown', (e) => {
          this.isMouseDown = true;
          // Check if the input field is focused
          if (this.$refs.renameInput && this.$refs.renameInput === document.activeElement) {
            return; // Do nothing if the input field is focused
          }
  
          const rect = canvas.getBoundingClientRect();
          const mouseX = e.clientX - rect.left;
          const mouseY = e.clientY - rect.top;
  
          // Check if a player is clicked
          this.selectedPlayer = this.players.find(player => {
            const dx = player.x - mouseX;
            const dy = player.y - mouseY;
            return Math.sqrt(dx * dx + dy * dy) < 20; // Check if click is within player circle
          });
  
          // Redraw canvas to update the selected player
          this.redrawCanvas();
        });
  
        canvas.addEventListener('mousemove', (e) => {
          if (this.selectedPlayer && this.isMouseDown) {
            const rect = canvas.getBoundingClientRect();
            this.selectedPlayer.x = e.clientX - rect.left;
            this.selectedPlayer.y = e.clientY - rect.top;
            this.redrawCanvas();
          }
        });
  
        canvas.addEventListener('mouseup', () => {
          this.isMouseDown = false; // Set mouse state to "up"
        });
      },
      // Save the lineup as an image
      saveLineup() {
        const canvas = this.$refs.lineupCanvas;
        const link = document.createElement('a');
        link.download = 'lineup.png';
        link.href = canvas.toDataURL();
        link.click();
      },
    },
  };
  </script>
  
  <style scoped>
  .soccer-lineup {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    justify-content: center;
    padding-top: 6.8vh;
    gap: 3vw;
    width: calc(100vw - 47vw);
    margin: 0 auto;
    min-height: calc(100vh - 6.8vh);
    background-color: #9fcf30;
    box-shadow: inset 0 0 5vw 1px #406117;
  }
  .controls {
  display: flex;
  flex-direction: column;
  gap: 1.5vw;
  padding-top: 6.8vh;
  width: 11vw;
}

.color-picker, .formation-selector {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

.lineupCanvas {
    width: 500px;
    height: 700px;
}
.color-picker label {
  margin-right: 10px;
}
  canvas {
    border: 1px solid #000;
    background-color: #fff;
    box-shadow: 0 0 1vw 1px #000;
    border-radius: 0.25vw;
  }
  </style>