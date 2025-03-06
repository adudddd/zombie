import React, { useState, useEffect, useCallback } from 'react';
import { Play, Pause, SkipForward, RotateCcw } from 'lucide-react';

const ZombieOutbreakSimulator = () => {
  // Grid configuration
  const rows = 15;
  const cols = 15;
  
  // State variables
  const [grid, setGrid] = useState([]);
  const [isRunning, setIsRunning] = useState(false);
  const [tickSpeed, setTickSpeed] = useState(500); // milliseconds
  const [selectedTool, setSelectedTool] = useState('human');
  const [stats, setStats] = useState({
    humans: 0,
    zombies: 0,
    barricades: 0,
    cured: 0,
    dead: 0,
    safehouses: 0
  });
  const [log, setLog] = useState([]);

  // Cell states and tools
  const CELL_TYPES = {
    EMPTY: 0,
    HUMAN: 1,
    ZOMBIE: 2,
    BARRICADE: 3,
    CURED: 4,
    DEAD: 5,
    SAFEHOUSE: 6
  };

  const TOOLS = [
    { id: 'human', label: 'Add Human', icon: '👨' },
    { id: 'zombie', label: 'Add Zombie', icon: '🧟' },
    { id: 'barricade', label: 'Add Barricade', icon: '🚧' },
    { id: 'cure', label: 'Add Cure Station', icon: '💉' },
    { id: 'weapon', label: 'Add Weapon', icon: '🔫' },
    { id: 'safehouse', label: 'Add Safehouse', icon: '🏠' },
    { id: 'clear', label: 'Clear Cell', icon: '❌' }
  ];

  // Utility to add log entries (limit to last 10 events)
  const addLog = (message) => {
    setLog(prevLog => {
      const newLog = [...prevLog, message];
      return newLog.length > 10 ? newLog.slice(newLog.length - 10) : newLog;
    });
  };

  // Initialize grid
  const initializeGrid = useCallback(() => {
    const newGrid = Array(rows)
      .fill()
      .map(() =>
        Array(cols).fill({
          type: CELL_TYPES.EMPTY,
          hasWeapon: false,
          hasCure: false,
          timeToCure: 0,
          timeInfected: 0,
          hasMoved: false // for tracking human movement within a tick
        })
      );
    
    // Add more initial humans
    for (let i = 0; i < 30; i++) {
      const row = Math.floor(Math.random() * rows);
      const col = Math.floor(Math.random() * cols);
      newGrid[row][col] = {
        ...newGrid[row][col],
        type: CELL_TYPES.HUMAN
      };
    }
    
    // Add a few humans with weapons
    for (let i = 0; i < 5; i++) {
      const row = Math.floor(Math.random() * rows);
      const col = Math.floor(Math.random() * cols);
      if (newGrid[row][col].type === CELL_TYPES.HUMAN || newGrid[row][col].type === CELL_TYPES.EMPTY) {
        newGrid[row][col] = {
          ...newGrid[row][col],
          type: CELL_TYPES.HUMAN,
          hasWeapon: true
        };
      }
    }
    
    // Add multiple initial zombies for a more interesting outbreak
    for (let i = 0; i < 3; i++) {
      const zombieRow = Math.floor(Math.random() * rows);
      const zombieCol = Math.floor(Math.random() * cols);
      newGrid[zombieRow][zombieCol] = {
        ...newGrid[zombieRow][zombieCol],
        type: CELL_TYPES.ZOMBIE,
        timeInfected: 0
      };
    }
    
    setGrid(newGrid);
    updateStats(newGrid);
    setLog([]);
  }, [rows, cols, CELL_TYPES]);

  // Update statistics
  const updateStats = (currentGrid) => {
    const newStats = {
      humans: 0,
      zombies: 0,
      barricades: 0,
      cured: 0,
      dead: 0,
      safehouses: 0
    };
    
    for (let r = 0; r < rows; r++) {
      for (let c = 0; c < cols; c++) {
        const cell = currentGrid[r][c];
        if (cell.type === CELL_TYPES.HUMAN) newStats.humans++;
        else if (cell.type === CELL_TYPES.ZOMBIE) newStats.zombies++;
        else if (cell.type === CELL_TYPES.BARRICADE) newStats.barricades++;
        else if (cell.type === CELL_TYPES.CURED) newStats.cured++;
        else if (cell.type === CELL_TYPES.DEAD) newStats.dead++;
        else if (cell.type === CELL_TYPES.SAFEHOUSE) newStats.safehouses++;
      }
    }
    
    setStats(newStats);
  };

  // Simulation tick logic refactored into a function
  const processTick = useCallback(() => {
    setGrid(prevGrid => {
      // Clone grid for modifications
      const newGrid = JSON.parse(JSON.stringify(prevGrid));

      // Reset movement flags for humans
      for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
          newGrid[r][c].hasMoved = false;
        }
      }
      
      const directions = [
        [-1, 0], [1, 0], [0, -1], [0, 1],
        [-1, -1], [-1, 1], [1, -1], [1, 1]
      ];
      const cardinalDirections = [
        [-1, 0], [1, 0], [0, -1], [0, 1]
      ];

      // Process zombies and cure logic
      for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
          const cell = prevGrid[r][c];

          // Zombie behavior
          if (cell.type === CELL_TYPES.ZOMBIE) {
            if (cell.timeInfected > 50) {
              newGrid[r][c] = { ...cell, type: CELL_TYPES.DEAD };
              addLog(`Zombie at (${r}, ${c}) died of decay.`);
              continue;
            }
            
            newGrid[r][c].timeInfected = cell.timeInfected + 1;
            let hasAttacked = false;
            
            // Attempt to infect adjacent cells in all 8 directions
            for (const [dr, dc] of directions) {
              const nr = r + dr;
              const nc = c + dc;
              if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                const neighbor = prevGrid[nr][nc];
                // Infect humans or safehouse humans with adjusted probability
                if (neighbor.type === CELL_TYPES.HUMAN || neighbor.type === CELL_TYPES.SAFEHOUSE) {
                  hasAttacked = true;
                  // If human has weapon, chance to kill the zombie
                  if (neighbor.hasWeapon && Math.random() < 0.6) {
                    newGrid[r][c] = { ...cell, type: CELL_TYPES.DEAD };
                    newGrid[nr][nc].hasWeapon = false;
                    addLog(`Human with weapon at (${nr}, ${nc}) killed zombie at (${r}, ${c}).`);
                  } else {
                    // Safehouse provides extra protection: lower infection probability
                    const infectionChance = neighbor.type === CELL_TYPES.SAFEHOUSE ? 0.2 : 0.8;
                    if (Math.random() < infectionChance) {
                      newGrid[nr][nc] = {
                        ...neighbor,
                        type: CELL_TYPES.ZOMBIE,
                        timeInfected: 0
                      };
                      addLog(`Human at (${nr}, ${nc}) infected by zombie at (${r}, ${c}).`);
                    }
                  }
                }
              }
            }
            
            // If no attack occurred, move randomly (using cardinal directions)
            if (!hasAttacked && Math.random() < 0.3) {
              const moveDir = cardinalDirections[Math.floor(Math.random() * cardinalDirections.length)];
              const nr = r + moveDir[0];
              const nc = c + moveDir[1];
              if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                const targetCell = prevGrid[nr][nc];
                if (targetCell.type === CELL_TYPES.EMPTY) {
                  newGrid[nr][nc] = { ...cell };
                  newGrid[r][c] = {
                    type: CELL_TYPES.EMPTY,
                    hasWeapon: false,
                    hasCure: false,
                    timeToCure: 0,
                    timeInfected: 0
                  };
                }
              }
            }
          }
          
          // Cure station logic for humans with cure
          if (cell.type === CELL_TYPES.HUMAN && cell.hasCure) {
            for (const [dr, dc] of cardinalDirections) {
              const nr = r + dr;
              const nc = c + dc;
              if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                const neighbor = prevGrid[nr][nc];
                if (neighbor.type === CELL_TYPES.ZOMBIE && neighbor.timeInfected < 10) {
                  if (Math.random() < 0.4) {
                    newGrid[nr][nc] = {
                      ...neighbor,
                      type: CELL_TYPES.CURED,
                      timeInfected: 0
                    };
                    newGrid[r][c].hasCure = false;
                    addLog(`Cure from (${r}, ${c}) transformed zombie at (${nr}, ${nc}).`);
                  }
                }
              }
            }
          }
          
          // Cured humans help cure adjacent zombies
          if (cell.type === CELL_TYPES.CURED) {
            for (const [dr, dc] of cardinalDirections) {
              const nr = r + dr;
              const nc = c + dc;
              if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                const neighbor = prevGrid[nr][nc];
                if (neighbor.type === CELL_TYPES.ZOMBIE && neighbor.timeInfected < 10) {
                  if (Math.random() < 0.2) {
                    newGrid[nr][nc] = {
                      ...neighbor,
                      type: CELL_TYPES.CURED,
                      timeInfected: 0
                    };
                    addLog(`Cured human at (${r}, ${c}) helped cure zombie at (${nr}, ${nc}).`);
                  }
                }
              }
            }
          }
        }
      }
      
      // Process human fleeing behavior
      for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
          const cell = prevGrid[r][c];
          if (cell.type === CELL_TYPES.HUMAN && !cell.hasMoved) {
            // Check adjacent cells for a zombie
            let zombieNearby = false;
            for (const [dr, dc] of cardinalDirections) {
              const nr = r + dr;
              const nc = c + dc;
              if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                if (prevGrid[nr][nc].type === CELL_TYPES.ZOMBIE) {
                  zombieNearby = true;
                  break;
                }
              }
            }
            // With 30% chance, try to flee to an adjacent empty cell
            if (zombieNearby && Math.random() < 0.3) {
              const emptySpots = [];
              for (const [dr, dc] of cardinalDirections) {
                const nr = r + dr;
                const nc = c + dc;
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && prevGrid[nr][nc].type === CELL_TYPES.EMPTY) {
                  emptySpots.push([nr, nc]);
                }
              }
              if (emptySpots.length > 0) {
                const [newR, newC] = emptySpots[Math.floor(Math.random() * emptySpots.length)];
                newGrid[newR][newC] = { ...cell, hasMoved: true };
                newGrid[r][c] = {
                  type: CELL_TYPES.EMPTY,
                  hasWeapon: false,
                  hasCure: false,
                  timeToCure: 0,
                  timeInfected: 0,
                  hasMoved: false
                };
                addLog(`Human at (${r}, ${c}) fled to (${newR}, ${newC}).`);
              }
            }
          }
        }
      }
      
      updateStats(newGrid);
      return newGrid;
    });
  }, [rows, CELL_TYPES, cardinalDirections, directions, addLog]);

  // Automatic simulation ticks
  useEffect(() => {
    let intervalId;
    if (isRunning) {
      intervalId = setInterval(() => {
        processTick();
      }, tickSpeed);
    }
    return () => clearInterval(intervalId);
  }, [isRunning, tickSpeed, processTick]);

  // Initialize on first render
  useEffect(() => {
    initializeGrid();
  }, [initializeGrid]);
  
  // Handle cell click to place/remove items
  const handleCellClick = (r, c) => {
    setGrid(prevGrid => {
      const newGrid = JSON.parse(JSON.stringify(prevGrid));
      const cell = newGrid[r][c];
      
      switch (selectedTool) {
        case 'human':
          newGrid[r][c] = { ...cell, type: CELL_TYPES.HUMAN, hasWeapon: false, hasCure: false };
          break;
        case 'zombie':
          newGrid[r][c] = { ...cell, type: CELL_TYPES.ZOMBIE, timeInfected: 0 };
          break;
        case 'barricade':
          newGrid[r][c] = { ...cell, type: CELL_TYPES.BARRICADE };
          break;
        case 'cure':
          if (cell.type === CELL_TYPES.HUMAN) {
            newGrid[r][c] = { ...cell, hasCure: true };
          }
          break;
        case 'weapon':
          if (cell.type === CELL_TYPES.HUMAN) {
            newGrid[r][c] = { ...cell, hasWeapon: true };
          }
          break;
        case 'safehouse':
          // Convert a human cell to safehouse for added protection
          if (cell.type === CELL_TYPES.HUMAN) {
            newGrid[r][c] = { ...cell, type: CELL_TYPES.SAFEHOUSE, hasWeapon: false, hasCure: false };
          }
          break;
        case 'clear':
          newGrid[r][c] = {
            type: CELL_TYPES.EMPTY,
            hasWeapon: false,
            hasCure: false,
            timeToCure: 0,
            timeInfected: 0,
            hasMoved: false
          };
          break;
        default:
          break;
      }
      
      updateStats(newGrid);
      return newGrid;
    });
  };
  
  // Render cell with appropriate styling
  const renderCell = (cell, r, c) => {
    let bgColor = 'bg-gray-100';
    let content = '';
    
    switch (cell.type) {
      case CELL_TYPES.HUMAN:
        bgColor = 'bg-green-200';
        content = '👨';
        if (cell.hasWeapon) content = '👨‍🔫';
        if (cell.hasCure) content = '👨‍⚕️';
        break;
      case CELL_TYPES.ZOMBIE:
        bgColor = 'bg-red-300';
        content = '🧟';
        break;
      case CELL_TYPES.BARRICADE:
        bgColor = 'bg-gray-600';
        content = '🚧';
        break;
      case CELL_TYPES.CURED:
        bgColor = 'bg-blue-200';
        content = '😇';
        break;
      case CELL_TYPES.DEAD:
        bgColor = 'bg-gray-400';
        content = '💀';
        break;
      case CELL_TYPES.SAFEHOUSE:
        bgColor = 'bg-purple-300';
        content = '🏠';
        break;
      default:
        break;
    }
    
    return (
      <div 
        key={`${r}-${c}`}
        className={`h-8 w-8 border border-gray-300 flex items-center justify-center cursor-pointer ${bgColor}`}
        onClick={() => handleCellClick(r, c)}
        title={`Row: ${r}, Col: ${c}`}
      >
        {content}
      </div>
    );
  };
  
  return (
    <div className="p-4 max-w-4xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">Zombie Outbreak Simulator</h1>
      
      {/* Controls */}
      <div className="mb-4 flex gap-4 items-center">
        <button 
          className="px-3 py-1 bg-blue-500 text-white rounded flex items-center gap-1"
          onClick={() => setIsRunning(!isRunning)}
        >
          {isRunning ? <><Pause size={16} /> Pause</> : <><Play size={16} /> Start</>}
        </button>
        
        <button 
          className="px-3 py-1 bg-gray-500 text-white rounded flex items-center gap-1"
          onClick={processTick}
        >
          <SkipForward size={16} /> Step
        </button>
        
        <button 
          className="px-3 py-1 bg-red-500 text-white rounded flex items-center gap-1"
          onClick={() => {
            setIsRunning(false);
            initializeGrid();
          }}
        >
          <RotateCcw size={16} /> Reset
        </button>
        
        <div className="ml-4">
          <label className="mr-2">Speed:</label>
          <input 
            type="range" 
            min="100" 
            max="1000" 
            step="100" 
            value={tickSpeed} 
            onChange={(e) => setTickSpeed(Number(e.target.value))}
            className="w-32"
          />
          <span className="ml-2">{tickSpeed}ms</span>
        </div>
      </div>
      
      {/* Tools */}
      <div className="mb-4 flex gap-2 flex-wrap">
        {TOOLS.map(tool => (
          <button
            key={tool.id}
            className={`px-3 py-1 border rounded flex items-center gap-1 ${selectedTool === tool.id ? 'bg-blue-100 border-blue-500' : 'bg-white border-gray-300'}`}
            onClick={() => setSelectedTool(tool.id)}
          >
            <span>{tool.icon}</span> {tool.label}
          </button>
        ))}
      </div>
      
      {/* Statistics */}
      <div className="mb-4 p-3 bg-gray-100 rounded flex gap-4 flex-wrap">
        <div><span className="font-bold">👨 Humans:</span> {stats.humans}</div>
        <div><span className="font-bold">🧟 Zombies:</span> {stats.zombies}</div>
        <div><span className="font-bold">🚧 Barricades:</span> {stats.barricades}</div>
        <div><span className="font-bold">😇 Cured:</span> {stats.cured}</div>
        <div><span className="font-bold">💀 Dead:</span> {stats.dead}</div>
        <div><span className="font-bold">🏠 Safehouses:</span> {stats.safehouses}</div>
      </div>
      
      {/* Simulation Log */}
      <div className="mb-4 p-3 bg-green-50 rounded text-sm max-h-40 overflow-y-auto">
        <p className="font-bold">Simulation Log:</p>
        <ul className="list-disc ml-5">
          {log.map((entry, idx) => (
            <li key={idx}>{entry}</li>
          ))}
        </ul>
      </div>
      
      {/* Instructions */}
      <div className="mb-4 p-3 bg-yellow-100 rounded text-sm">
        <p><strong>Instructions:</strong> Select a tool and click on the grid to place items. Start the simulation to watch the outbreak spread.</p>
        <ul className="list-disc ml-5 mt-2">
          <li>Zombies actively hunt humans and can move around</li>
          <li>Humans with weapons (👨‍🔫) can fight back with a 60% chance of killing a zombie</li>
          <li>Humans with cure stations (👨‍⚕️) can heal zombies that haven't been infected too long</li>
          <li>Cured humans (😇) become immune and can help cure others</li>
          <li>Barricades (🚧) block zombie movement</li>
          <li>Safehouses (🏠) provide extra protection; zombies only have a 20% chance to infect them</li>
          <li>Humans may flee when zombies are nearby!</li>
        </ul>
      </div>
      
      {/* Grid */}
      <div className="inline-block border border-gray-400">
        {grid.map((row, r) => (
          <div key={r} className="flex">
            {row.map((cell, c) => renderCell(cell, r, c))}
          </div>
        ))}
      </div>
    </div>
  );
};

export default ZombieOutbreakSimulator;
