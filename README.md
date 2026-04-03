# -export default function NeonArenaDash() { const dailyBonus = new Date().getDate(); const { useEffect, useRef, useState } = React;

const skins = ["bg-cyan-400", "bg-green-400", "bg-pink-400"]; const [skin, setSkin] = useState(0); const [name, setName] = useState(""); const [started, setStarted] = useState(false); const [player, setPlayer] = useState({ x: 180, y: 180, hp: 5, score: 0, power: 0 }); const [enemies, setEnemies] = useState([{ x: 40, y: 40, boss: false }]); const [gems, setGems] = useState([{ x: 200, y: 80 }]); const [gameOver, setGameOver] = useState(false); const [leaderboard, setLeaderboard] = useState([]); const [coins, setCoins] = useState(100 + dailyBonus); const [stage, setStage] = useState(1); const [rarity, setRarity] = useState('Normal'); const [roomCode] = useState(Math.random().toString(36).slice(2, 8).toUpperCase()); const moveRef = useRef({ up: false, down: false, left: false, right: false });

useEffect(() => { setLeaderboard(JSON.parse(localStorage.getItem("neon_scores") || "[]")); }, []);

useEffect(() => { if (!started || gameOver) return; const loop = setInterval(() => { setPlayer((p) => { let { x, y, hp, score, power } = p; if (moveRef.current.up) y -= 10; if (moveRef.current.down) y += 10; if (moveRef.current.left) x -= 10; if (moveRef.current.right) x += 10; x = Math.max(0, Math.min(360, x)); y = Math.max(0, Math.min(360, y));

let nextScore = score;
    let nextPower = Math.min(100, power + 1);
    let hit = false;

    setGems((gs) => {
      const remain = gs.filter((g) => {
        const got = Math.abs(g.x - x) < 20 && Math.abs(g.y - y) < 20;
        if (got) nextScore += 1;
        return !got;
      });
      if (remain.length === 0) {
        remain.push({ x: Math.random() * 340, y: Math.random() * 340 });
        const isBoss = nextScore > 0 && nextScore % 10 === 0;
        setEnemies((es) => [...es, { x: Math.random() * 340, y: Math.random() * 340, boss: isBoss }]);
      }
      return remain;
    });

    setEnemies((es) =>
      es.map((e) => {
        const speed = e.boss ? 2 : 4;
        const nx = e.x + (x > e.x ? speed : -speed);
        const ny = e.y + (y > e.y ? speed : -speed);
        const range = e.boss ? 28 : 18;
        if (Math.abs(nx - x) < range && Math.abs(ny - y) < range) hit = true;
        return { ...e, x: nx, y: ny };
      })
    );

    const nextHp = hit ? hp - 1 : hp;
    if (nextHp <= 0) {
      setGameOver(true);
      const scores = [...leaderboard, { name: name || "PLAYER", score: nextScore }].sort((a,b)=>b.score-a.score).slice(0,5);
      localStorage.setItem("neon_scores", JSON.stringify(scores));
      setLeaderboard(scores);
    }
    return { x, y, hp: nextHp, score: nextScore, power: nextPower };
  });
}, 120);
return () => clearInterval(loop);

}, [started, gameOver, leaderboard, name]);

const press = (dir, val) => (moveRef.current[dir] = val);

const usePower = () => { if (player.power < 100) return; setEnemies((es) => es.slice(Math.floor(es.length / 2))); setPlayer((p) => ({ ...p, power: 0 })); };

if (!started) { return ( <div className="min-h-screen bg-black text-white flex flex-col items-center justify-center gap-4"> <h1 className="text-4xl font-bold">NEON ARENA DASH DX</h1> <input value={name} onChange={(e)=>setName(e.target.value)} placeholder="名前" className="px-4 py-2 rounded-xl text-black" /> <div className="flex gap-2">{skins.map((s,i)=><button key={i} onClick={()=>setSkin(i)} className={w-8 h-8 rounded-full ${s}} />)}</div> <div className="text-sm">Daily Bonus: +{dailyBonus} coins</div> <button onClick={()=>setStarted(true)} className="bg-cyan-400 text-black px-6 py-3 rounded-2xl font-bold">START</button> <p className="text-xs opacity-70">ROOM: {roomCode}（共有URL用コード）</p> </div> ); }

return ( <div className="min-h-screen bg-black text-white flex flex-col items-center justify-center p-4"> <div className="relative w-[400px] h-[400px] bg-zinc-900 rounded-3xl overflow-hidden"> <div className={absolute w-5 h-5 ${skins[skin]} rounded-full} style={{ left: player.x, top: player.y }} /> {enemies.map((e,i)=><div key={i} className={absolute ${e.boss ? 'w-8 h-8 bg-purple-500' : 'w-5 h-5 bg-red-500'} rounded-md} style={{ left:e.x, top:e.y }} />)} {gems.map((g,i)=><div key={i} className="absolute w-4 h-4 bg-yellow-300 rotate-45" style={{ left:g.x, top:g.y }} />)} </div>

<div className="mt-3 flex gap-4 text-sm"><span>STAGE {stage}</span><span>COIN {coins}</span><span>{rarity}</span><span>SCORE {player.score}</span><span>HP {player.hp}</span></div>
  <div className="w-64 bg-zinc-800 rounded-full h-3 mt-2"><div className="bg-cyan-400 h-3 rounded-full" style={{ width: `${player.power}%` }} /></div>
  <button onClick={() => { usePower(); setCoins((c)=>c+5); }} className="mt-2 bg-yellow-400 text-black px-4 py-2 rounded-xl">必殺技</button>

  <div className="grid grid-cols-3 gap-2 mt-4">
    <div />
    <button onTouchStart={()=>press('up',true)} onTouchEnd={()=>press('up',false)} className="bg-zinc-700 px-4 py-2 rounded-xl">↑</button>
    <div />
    <button onTouchStart={()=>press('left',true)} onTouchEnd={()=>press('left',false)} className="bg-zinc-700 px-4 py-2 rounded-xl">←</button>
    <button onTouchStart={()=>press('down',true)} onTouchEnd={()=>press('down',false)} className="bg-zinc-700 px-4 py-2 rounded-xl">↓</button>
    <button onTouchStart={()=>press('right',true)} onTouchEnd={()=>press('right',false)} className="bg-zinc-700 px-4 py-2 rounded-xl">→</button>
  </div>

  <div className="mt-4 flex gap-2">
    <button onClick={() => { setCoins(c=>c-50); setRarity('Rare'); }} className="bg-blue-500 px-3 py-2 rounded-xl">ガチャ 50</button>
    <button onClick={() => setStage(s=>s+1)} className="bg-green-500 px-3 py-2 rounded-xl">次ステージ</button>
  </div>

  <div className="mt-6 w-64">
    <h2 className="font-bold">TOP 5</h2>
    {leaderboard.map((s,i)=><div key={i} className="flex justify-between text-sm"><span>{s.name}</span><span>{s.score}</span></div>)}
  </div>

  {gameOver && <p className="mt-4 text-red-400 font-bold">GAME OVER</p>}
</div>

); }
ブロスタっぽいゲーム
