<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PowerShock - RTS Game</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; display: flex; justify-content: space-between; }
        #game-container { 
            display: grid;
            grid-template-columns: repeat(10, 50px);
            grid-template-rows: repeat(10, 50px);
            gap: 2px;
            margin: 20px auto;
            width: max-content;
        }
        .tile {
            width: 50px; height: 50px;
            display: flex; align-items: center; justify-content: center;
            border: 1px solid #444;
            font-size: 12px; cursor: pointer; user-select: none;
            position: relative;
        }
        .plains { background-color: lightgreen; }
        .forest { background-color: green; }
        .mountains { background-color: gray; }
        .water { background-color: lightblue; }
        .player1 { border: 3px solid blue !important; }
        .player2 { border: 3px solid red !important; }
        .army-count { position: absolute; bottom: 2px; right: 2px; font-size: 10px; color: black; background: white; padding: 2px; border-radius: 3px; }
        #action-popup, #claim-popup {
            display: none;
            position: fixed;
            top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: white; padding: 10px;
            border: 2px solid black;
            box-shadow: 3px 3px 10px rgba(0, 0, 0, 0.3);
        }
        .popup-button { margin: 5px; padding: 5px 10px; font-size: 16px; cursor: pointer; }
        #close-popup, #close-claim-popup { margin-top: 10px; display: block; background: lightgray; border: none; padding: 5px; cursor: pointer; }
        #debug-log { position: fixed; top: 0; left: 50%; transform: translateX(-50%); background: white; padding: 5px 10px; border: 2px solid black; z-index: 10; }
        #exit-attack { display: none; margin-top: 10px; background: red; color: white; padding: 5px; cursor: pointer; }

        /* Active Player Indicator */
        #active-player-indicator {
            position: fixed;
            top: 0;
            left: 10px;
            background-color: lightgray;
            padding: 5px 10px;
            border: 2px solid black;
            z-index: 10;
            font-weight: bold;
        }

        /* Swap Active Player Button */
        #swap-player-button {
            margin-top: 10px;
            padding: 10px;
            background-color: lightgray;
            border: 2px solid black;
            cursor: pointer;
            font-size: 16px;
        }

        /* Stat Sheet */
        #player1-stats, #player2-stats {
            background-color: lightgray;
            padding: 10px;
            border: 2px solid black;
            font-size: 16px;
            display: flex;
            flex-direction: column;
            justify-content: flex-start;
            margin-left: 20px;
            width: 200px;
        }

        #player1-stats h3, #player2-stats h3 {
            text-align: center;
        }

        #player1-stats div, #player2-stats div {
            margin: 5px 0;
        }
    </style>
</head>
<body>
    <div>
        <!-- Player 1 Stats -->
        <div id="player1-stats">
            <h3>Player 1 Stats</h3>
            <div>Army: <span id="player1-army">0</span></div>
            <div>Influence: <span id="player1-influence">0</span></div>
            <div>Popularity: <span id="player1-popularity">0</span></div>
            <div>Piety: <span id="player1-piety">0</span></div>
            <div>Siege Machines: <span id="player1-siege-machines">0</span></div>
            <div>Castles: <span id="player1-castles">0</span></div>
            <div>Swords: <span id="player1-swords">0</span></div>
            <div>Armor: <span id="player1-armor">0</span></div>
            <div>Wood: <span id="player1-wood">0</span></div>
            <div>Stone: <span id="player1-stone">0</span></div>
            <div>Iron: <span id="player1-iron">0</span></div>
            <div>Gold: <span id="player1-gold">0</span></div>
        </div>

        <!-- Player 2 Stats -->
        <div id="player2-stats">
            <h3>Player 2 Stats</h3>
            <div>Army: <span id="player2-army">0</span></div>
            <div>Influence: <span id="player2-influence">0</span></div>
            <div>Popularity: <span id="player2-popularity">0</span></div>
            <div>Piety: <span id="player2-piety">0</span></div>
            <div>Siege Machines: <span id="player2-siege-machines">0</span></div>
            <div>Castles: <span id="player2-castles">0</span></div>
            <div>Swords: <span id="player2-swords">0</span></div>
            <div>Armor: <span id="player2-armor">0</span></div>
            <div>Wood: <span id="player2-wood">0</span></div>
            <div>Stone: <span id="player2-stone">0</span></div>
            <div>Iron: <span id="player2-iron">0</span></div>
            <div>Gold: <span id="player2-gold">0</span></div>
        </div>
    </div>

    <div>
        <!-- Active Player Indicator -->
        <div id="active-player-indicator">Active Player: 1</div>
        <button id="swap-player-button" onclick="swapActivePlayer()">Swap Active Player</button>

        <button id="exit-attack" onclick="exitAttackMode()">Exit Attack Mode</button>
        <div id="game-container"></div>
    </div>

    <p id="debug-log">Debug Log: Ready</p>

    <div id="claim-popup">
        <p>Choose a player to claim this tile:</p>
        <button class="popup-button" onclick="claimTile(1)">Player 1</button>
        <button class="popup-button" onclick="claimTile(2)">Player 2</button>
        <button id="close-claim-popup" onclick="closeClaimPopup()">Cancel</button>
    </div>

    <div id="action-popup">
        <p>Choose an action:</p>
        <button class="popup-button" onclick="placeArmy()">Place Army</button>
        <button class="popup-button" onclick="startMoveArmy()">Move Army</button>
        <button class="popup-button" onclick="startAttack()">Attack</button>
        <button id="close-popup" onclick="closePopup()">Cancel</button>
    </div>

    <script>
        const mapSize = 10;
        const terrainTypes = ['plains', 'forest', 'mountains', 'water'];
        let grid = [];
        let selectedTile = null;
        let moveSource = null;
        let attackSource = null;
        let currentPlayer = 1;  // Set to 1 or 2
        let playerStats = {
            1: {
                army: 0,
                influence: 0,
                popularity: 0,
                piety: 0,
                siegeMachines: 0,
                castles: 0,
                swords: 0,
                armor: 0,
                wood: 0,
                stone: 0,
                iron: 0,
                gold: 0
            },
            2: {
                army: 0,
                influence: 0,
                popularity: 0,
                piety: 0,
                siegeMachines: 0,
                castles: 0,
                swords: 0,
                armor: 0,
                wood: 0,
                stone: 0,
                iron: 0,
                gold: 0
            }
        };

        function generateMap() {
            for (let y = 0; y < mapSize; y++) {
                grid[y] = [];
                for (let x = 0; x < mapSize; x++) {
                    const tile = document.createElement('div');
                    tile.classList.add('tile', terrainTypes[Math.floor(Math.random() * terrainTypes.length)]);
                    tile.dataset.x = x;
                    tile.dataset.y = y;
                    tile.addEventListener('click', () => handleTileClick(x, y));
                    grid[y].push({ tile: tile, x: x, y: y, owner: null, army: 0, armyCount: document.createElement('div') });
                    grid[y][x].armyCount.classList.add('army-count');
                    tile.appendChild(grid[y][x].armyCount); 
                    document.getElementById('game-container').appendChild(tile);
                }
            }
        }

        function handleTileClick(x, y) {
            selectedTile = { x, y };
            if (!grid[y][x].owner) {
                showClaimPopup();
            } else {
                showActionPopup();
            }
        }

        function showClaimPopup() {
            const claimPopup = document.getElementById('claim-popup');
            claimPopup.style.display = 'block';
        }

        function closeClaimPopup() {
            const claimPopup = document.getElementById('claim-popup');
            claimPopup.style.display = 'none';
        }

        function claimTile(player) {
            const tile = grid[selectedTile.y][selectedTile.x];
            tile.owner = player;
            tile.tile.classList.add(player === 1 ? 'player1' : 'player2');
            closeClaimPopup();
            showActionPopup();
        }

        function showActionPopup() {
            const popup = document.getElementById('action-popup');
            popup.style.display = 'block';
        }

        function closePopup() {
            const popup = document.getElementById('action-popup');
            popup.style.display = 'none';
        }

        function placeArmy() {
            const tile = grid[selectedTile.y][selectedTile.x];
            if (tile.owner && tile.owner !== currentPlayer) {
                alert("You can't place an army here!");
                return;
            }

            tile.army += 1;
            playerStats[currentPlayer].army += 1;
            tile.armyCount.innerText = tile.army;  // Update army count on the tile
            updateStats();
            closePopup();
        }

        function startAttack() {
    attackSource = selectedTile;
    alert("Select an adjacent tile to attack.");
    closePopup();
}

function handleTileClick(x, y) {
    selectedTile = { x, y };

    if (attackSource) {
        // Check if the clicked tile is adjacent to the attack source
        const isAdjacent = 
            (Math.abs(attackSource.x - x) === 1 && attackSource.y === y) || 
            (Math.abs(attackSource.y - y) === 1 && attackSource.x === x);

        if (isAdjacent) {
            attackTile(x, y);
        } else {
            alert("You can only attack adjacent tiles!");
        }

        attackSource = null;  // End attack mode after tile selection
        return;  // Prevent further tile actions during attack
    }

    // Handle non-attack tile actions (like claiming or placing army)
    if (!grid[y][x].owner) {
        showClaimPopup();
    } else {
        showActionPopup();
    }
}
function attack(attackerTile, defenderTile) {
    if (!attackerTile || !defenderTile || attackerTile.owner === defenderTile.owner) {
        console.error("Invalid attack.");
        return;
    }

    let attackRoll = Math.floor(Math.random() * 20) + 1;
    let defenseRoll = Math.floor(Math.random() * 20) + 1;

    console.log(`Attack Roll: ${attackRoll}`);
    console.log(`Defense Roll: ${defenseRoll}`);

    if (attackRoll > defenseRoll) {
        defenderTile.army = Math.max(0, defenderTile.army - 1); // Defender loses 1 army
        console.log(`Defender loses 1 army. Remaining: ${defenderTile.army}`);

        if (defenderTile.army === 0) {
            console.log("Defender's army is eliminated! Tile changes ownership.");
            defenderTile.owner = attackerTile.owner;
        }
    } else {
        attackerTile.army = Math.max(0, attackerTile.army - 1); // Attacker loses 1 army
        console.log(`Attack unsuccessful. Attacker loses 1 army. Remaining: ${attackerTile.army}`);
    }
}



function closeAttackMode() {
    attackSource = null;
    alert("Attack mode ended.");
}


        function startMoveArmy() {
            moveSource = selectedTile;
            alert("Select a destination tile to move your army.");
            closePopup();
        }

        function swapActivePlayer() {
            currentPlayer = currentPlayer === 1 ? 2 : 1;
            document.getElementById('active-player-indicator').innerText = `Active Player: ${currentPlayer}`;
            updateStats();
        }

        function closeAttackMode() {
            attackSource = null;
            alert("Attack mode ended.");
        }

        generateMap();
        updateStats();
    </script>
</body>
</html>
