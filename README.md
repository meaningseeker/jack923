<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>My Fun Site</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-firestore-compat.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/soundfont-player/dist/soundfont-player.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Jua&family=Noto+Sans+KR:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Noto Sans KR', sans-serif; background-color: #faf8ef; }
        .title-font { font-family: 'Jua', sans-serif; }
        .project-card { transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out; }
        .project-card:hover { transform: translateY(-5px); box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05); }
        .piano { display: flex; position: relative; }
        .key.white { width: 60px; height: 240px; display: flex; justify-content: center; align-items: flex-end; border: 2px solid #a0aec0; border-bottom-left-radius: 6px; border-bottom-right-radius: 6px; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1); cursor: pointer; }
        .key.black { width: 38px; height: 140px; position: absolute; z-index: 10; display: flex; justify-content: center; align-items: flex-end; border-radius: 6px; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.2); cursor: pointer; }
        
        .game-tile {
            font-size: clamp(1.5rem, 5vw, 2.75rem);
            transition: transform 100ms ease-in-out;
            font-weight: 900;
            will-change: transform;
        }
        .tile-new { animation: appear 200ms ease-in-out; }
        .tile-merged { animation: pop 200ms ease-in-out; z-index: 20; }
        @keyframes appear { 0% { transform: scale(0); } 100% { transform: scale(1); } }
        @keyframes pop { 0% { transform: scale(1); } 50% { transform: scale(1.2); } 100% { transform: scale(1); } }
        .tile-2 { background-color: #eee4da; color: #776e65; } .tile-4 { background-color: #ede0c8; color: #776e65; } .tile-8 { background-color: #f2b179; color: #f9f6f2; } .tile-16 { background-color: #f59563; color: #f9f6f2; } .tile-32 { background-color: #f67c5f; color: #f9f6f2; } .tile-64 { background-color: #f65e3b; color: #f9f6f2; } .tile-128 { background-color: #edcf72; color: #f9f6f2; } .tile-256 { background-color: #edcc61; color: #f9f6f2; } .tile-512 { background-color: #edc850; color: #f9f6f2; } .tile-1024 { background-color: #edc53f; color: #f9f6f2; } .tile-2048 { background-color: #edc22e; color: #f9f6f2; } .tile-super { background-color: #3c3a32; color: #f9f6f2; }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef, useCallback } = React;

        // --- 아이콘 SVG 컴포넌트들 ---
        const MusicIcon = (props) => ( <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" {...props}><path d="M9 18V5l12-2v13"/><circle cx="6" cy="18" r="3"/><circle cx="18" cy="16" r="3"/></svg> );
        const GuitarIcon = (props) => ( <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" {...props}><path d="M12 8a2 2 0 1 0 0-4 2 2 0 0 0 0 4Z"/><path d="M10 18v-4.33a2 2 0 1 0-1.66-1.11L3 14v6h18v-4h-2.5"/><path d="m3.18 15.82.72-2.32"/></svg> );
        const DrumIcon = (props) => ( <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" {...props}><path d="M12 2a2 2 0 0 0-2 2v2a2 2 0 0 0 2 2h0a2 2 0 0 0 2-2V4a2 2 0 0 0-2-2Z"/><path d="M6.66 6.66a11.12 11.12 0 0 0-4.51 3.42A2 2 0 0 0 2 12v3a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2v-3a2 2 0 0 0-.15-1.92 11.13 11.13 0 0 0-4.51-3.42"/><path d="M22 15v3a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2v-3"/></svg> );
        const GamepadIcon = (props) => ( <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" {...props}><line x1="6" x2="10" y1="12" y2="12"/><line x1="8" x2="8" y1="10" y2="14"/><line x1="15" x2="15.01" y1="13" y2="13"/><line x1="18" x2="18.01" y1="11" y2="11"/><path d="M17.32 5H6.68a4 4 0 0 0-3.978 3.59c-.006.052-.01.101-.01.15v5.52A4 4 0 0 0 6.68 18h10.64a4 4 0 0 0 3.98-3.59c.006-.052.01-.101.01-.15v-5.52a4 4 0 0 0-3.98-3.59Z"/></svg> );
        const HomeIcon = (props) => ( <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" {...props}><path d="m3 9 9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/></svg> );
        const ThereminIcon = (props) => ( <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" {...props}><path d="M12 12v-2"/><path d="M12 8V4"/><path d="M12 12h2a2 2 0 0 1 2 2v0a2 2 0 0 1-2 2h-2"/><path d="M12 12h-2a2 2 0 0 0-2 2v0a2 2 0 0 0 2 2h2"/><path d="M12 22a7.7 7.7 0 0 0 5-2 7.7 7.7 0 0 0-10 0 7.7 7.7 0 0 0 5 2Z"/></svg>);
        
        // --- 컴포넌트들 ---
        
        const HubComponent = ({ setCurrentView }) => {
            return (
                <div className="container mx-auto px-4 py-12 md:py-20">
                    <header className="text-center mb-12 md:mb-16">
                        <h1 className="title-font text-5xl md:text-7xl font-bold text-gray-900">My Fun Site</h1>
                        <p className="mt-4 text-lg md:text-xl text-gray-600">상상력을 현실로 만드는 공간</p>
                    </header>
                    <main>
                        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6 md:gap-8">
                            <div onClick={() => setCurrentView('piano')} className="project-card block bg-white rounded-xl shadow-md overflow-hidden cursor-pointer"> <div className="h-48 flex items-center justify-center bg-violet-500"> <MusicIcon className="w-16 h-16 text-white"/> </div> <div className="p-6"> <h2 className="title-font text-2xl font-bold text-gray-900 mb-2">Play Piano</h2> <p className="text-gray-700">가상 피아노를 키보드로 연주해보세요.</p> </div> </div>
                            <div onClick={() => setCurrentView('guitar')} className="project-card block bg-white rounded-xl shadow-md overflow-hidden cursor-pointer"> <div className="h-48 flex items-center justify-center bg-emerald-500"> <GuitarIcon className="w-16 h-16 text-white"/> </div> <div className="p-6"> <h2 className="title-font text-2xl font-bold text-gray-900 mb-2">Play Guitar</h2> <p className="text-gray-700">간단한 코드로 기타를 연주해보세요.</p> </div> </div>
                            <div onClick={() => setCurrentView('drums')} className="project-card block bg-white rounded-xl shadow-md overflow-hidden cursor-pointer"> <div className="h-48 flex items-center justify-center bg-rose-500"> <DrumIcon className="w-16 h-16 text-white"/> </div> <div className="p-6"> <h2 className="title-font text-2xl font-bold text-gray-900 mb-2">Play Drums</h2> <p className="text-gray-700">키보드로 신나는 드럼 비트를 만들어보세요.</p> </div> </div>
                            <div onClick={() => setCurrentView('theremin')} className="project-card block bg-white rounded-xl shadow-md overflow-hidden cursor-pointer"> <div className="h-48 flex items-center justify-center bg-sky-500"> <ThereminIcon className="w-16 h-16 text-white"/> </div> <div className="p-6"> <h2 className="title-font text-2xl font-bold text-gray-900 mb-2">Play Theremin</h2> <p className="text-gray-700">마우스를 움직여 신비로운 소리를 연주하세요.</p> </div> </div>
                            <div onClick={() => setCurrentView('game-2048')} className="project-card block bg-white rounded-xl shadow-md overflow-hidden cursor-pointer col-span-1 sm:col-span-2 lg:col-span-3 xl:col-span-4"> <div className="h-48 flex items-center justify-center bg-[#f2b179]"> <GamepadIcon className="w-16 h-16 text-white"/> </div> <div className="p-6"> <h2 className="title-font text-2xl font-bold text-gray-900 mb-2">2048 Game</h2> <p className="text-gray-700">같은 숫자를 합쳐 2048 타일을 만들어보세요.</p> </div> </div>
                        </div>
                    </main>
                    <footer className="text-center mt-16 text-gray-500"> <p>&copy; 2025. My Fun Site. All rights reserved.</p> </footer>
                </div>
            );
        };
        
        const PianoComponent = ({ setCurrentView }) => {
            const [instrument, setInstrument] = useState(null);
            const activeNodes = useRef({});
            const PIANO_KEYS = [ { note: 'C4', key: 'a', type: 'white' }, { note: 'C#4', key: 'w', type: 'black' }, { note: 'D4', key: 's', type: 'white' }, { note: 'D#4', key: 'e', type: 'black' }, { note: 'E4', key: 'd', type: 'white' }, { note: 'F4', key: 'f', type: 'white' }, { note: 'F#4', key: 't', type: 'black' }, { note: 'G4', key: 'g', type: 'white' }, { note: 'G#4', key: 'y', type: 'black' }, { note: 'A4', key: 'h', type: 'white' }, { note: 'A#4', key: 'u', type: 'black' }, { note: 'B4', key: 'j', type: 'white' }, { note: 'C5', key: 'k', type: 'white' }, ];
            useEffect(() => { if (window.Soundfont) { window.Soundfont.instrument(new AudioContext(), 'acoustic_grand_piano').then(setInstrument); } }, []);
            const playNote = useCallback((note) => { if (!instrument) return; if(activeNodes.current[note]) activeNodes.current[note].stop(); const node = instrument.play(note); activeNodes.current[note] = node; }, [instrument]);
            const releaseNote = useCallback((note) => { if (!instrument || !activeNodes.current[note]) return; setTimeout(() => { if (activeNodes.current[note]) { activeNodes.current[note].stop(); delete activeNodes.current[note]; } }, 50); }, [instrument]);
            useEffect(() => { const pressedKeys = new Set(); const handleKeyDown = (e) => { if (document.activeElement.tagName !== 'BODY' || pressedKeys.has(e.key)) return; const keyInfo = PIANO_KEYS.find(k => k.key === e.key); if (keyInfo) { pressedKeys.add(e.key); playNote(keyInfo.note); } }; const handleKeyUp = (e) => { const keyInfo = PIANO_KEYS.find(k => k.key === e.key); if (keyInfo) { pressedKeys.delete(e.key); releaseNote(keyInfo.note); } }; window.addEventListener('keydown', handleKeyDown); window.addEventListener('keyup', handleKeyUp); return () => { window.removeEventListener('keydown', handleKeyDown); window.removeEventListener('keyup', handleKeyUp); }; }, [playNote, releaseNote]);
            let whiteKeyIndex = 0;
            return ( <div className="w-full flex flex-col items-center justify-center min-h-screen p-4 bg-gray-100"> <h1 className="text-4xl font-bold mb-8 title-font">Play Piano</h1> <div className="piano"> {PIANO_KEYS.map((k) => { if (k.type === 'white') { const left = whiteKeyIndex * 62; whiteKeyIndex++; return ( <button key={k.note} onMouseDown={() => playNote(k.note)} onMouseUp={() => releaseNote(k.note)} onMouseLeave={() => releaseNote(k.note)} onTouchStart={(e) => { e.preventDefault(); playNote(k.note); }} onTouchEnd={(e) => { e.preventDefault(); releaseNote(k.note); }} className={`key white bg-white transition-colors active:bg-gray-200`} style={{left: `${left}px`}}> <span className="self-end mb-2 text-gray-500 font-semibold">{k.key.toUpperCase()}</span> </button> ); } return null; })} {PIANO_KEYS.map((k) => { if (k.type === 'black') { const whiteKeyBefore = PIANO_KEYS.slice(0, PIANO_KEYS.indexOf(k)).filter(wk => wk.type === 'white').length; const left = whiteKeyBefore * 62 - 20; return ( <button key={k.note} onMouseDown={() => playNote(k.note)} onMouseUp={() => releaseNote(k.note)} onMouseLeave={() => releaseNote(k.note)} onTouchStart={(e) => { e.preventDefault(); playNote(k.note); }} onTouchEnd={(e) => { e.preventDefault(); releaseNote(k.note); }} className={`key black bg-black transition-colors text-white active:bg-gray-700`} style={{left: `${left}px`}}> <span className="self-end mb-2 text-sm font-semibold">{k.key.toUpperCase()}</span> </button> ); } return null; })} </div> <button onClick={() => setCurrentView('hub')} className="mt-8 text-violet-600 font-bold hover:underline flex items-center mx-auto"> <HomeIcon className="w-5 h-5 mr-1" /> Hub로 돌아가기 </button> </div> );
        };
        
        const GuitarComponent = ({ setCurrentView }) => {
            const [instrument, setInstrument] = useState(null);
            const CHORDS = [ { name: 'G', key: '1', notes: ['G3', 'B3', 'D4'] }, { name: 'C', key: '2', notes: ['C4', 'E4', 'G4'] }, { name: 'D', key: '3', notes: ['D4', 'F#4', 'A4'] }, { name: 'Em', key: '4', notes: ['E3', 'G3', 'B3'] }, ];
            useEffect(() => { if (window.Soundfont) { window.Soundfont.instrument(new AudioContext(), 'acoustic_guitar_steel').then(setInstrument); } }, []);
            const playChord = useCallback((notes) => { if (!instrument) return; notes.forEach((note, i) => instrument.play(note, i * 0.05, { gain: 0.8 })); }, [instrument]);
            useEffect(() => { const handleKeyDown = (e) => { if (document.activeElement.tagName !== 'BODY') return; const chordInfo = CHORDS.find(c => c.key === e.key); if (chordInfo) playChord(chordInfo.notes); }; window.addEventListener('keydown', handleKeyDown); return () => window.removeEventListener('keydown', handleKeyDown); }, [playChord]);
            return ( <div className="w-full flex flex-col items-center justify-center min-h-screen p-4 bg-gray-100"> <h1 className="text-4xl font-bold mb-8 title-font">Play Guitar</h1> <div className="space-y-4"> <p className="text-center text-gray-600">키보드 1, 2, 3, 4 또는 버튼을 눌러 코드를 연주하세요.</p> <div className="grid grid-cols-2 gap-4"> {CHORDS.map(chord => ( <button key={chord.name} onClick={() => playChord(chord.notes)} className="bg-emerald-500 text-white rounded-lg shadow-lg p-8 text-4xl font-bold hover:bg-emerald-600 transition-transform active:scale-95"> {chord.name} <span className="block text-lg mt-1">({chord.key})</span> </button> ))} </div> </div> <button onClick={() => setCurrentView('hub')} className="mt-8 text-violet-600 font-bold hover:underline flex items-center mx-auto"> <HomeIcon className="w-5 h-5 mr-1" /> Hub로 돌아가기 </button> </div> );
        };

        const DrumComponent = ({ setCurrentView }) => {
            const audioContext = useRef(null);
            useEffect(() => { audioContext.current = new (window.AudioContext || window.webkitAudioContext)(); }, []);
            const playSound = (type) => { if (!audioContext.current) return; const ctx = audioContext.current; if (type === 'kick') { const osc = ctx.createOscillator(); osc.type = 'sine'; osc.frequency.setValueAtTime(150, ctx.currentTime); osc.frequency.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.1); const gain = ctx.createGain(); gain.gain.setValueAtTime(1, ctx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.1); osc.connect(gain); gain.connect(ctx.destination); osc.start(); osc.stop(ctx.currentTime + 0.1); } else if (type === 'snare') { const bufferSize = ctx.sampleRate; const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate); const output = buffer.getChannelData(0); for (let i = 0; i < bufferSize; i++) { output[i] = Math.random() * 2 - 1; } const noise = ctx.createBufferSource(); noise.buffer = buffer; const gain = ctx.createGain(); gain.gain.setValueAtTime(1, ctx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.15); noise.connect(gain); gain.connect(ctx.destination); noise.start(); noise.stop(ctx.currentTime + 0.15); } else if (type === 'hihat') { const osc = ctx.createOscillator(); osc.type = 'square'; const bandpass = ctx.createBiquadFilter(); bandpass.type = 'bandpass'; bandpass.frequency.value = 10000; const gain = ctx.createGain(); gain.gain.setValueAtTime(1, ctx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.05); osc.connect(bandpass); bandpass.connect(gain); gain.connect(ctx.destination); osc.start(); osc.stop(ctx.currentTime + 0.05); }};
            const DRUM_PADS = [ { name: 'Kick', key: 'a', color: 'bg-rose-500', sound: 'kick' }, { name: 'Snare', key: 's', color: 'bg-rose-600', sound: 'snare' }, { name: 'Hi-Hat', key: 'd', color: 'bg-rose-400', sound: 'hihat' }, ];
            useEffect(() => { const handleKeyDown = (e) => { if (document.activeElement.tagName !== 'BODY') return; const padInfo = DRUM_PADS.find(p => p.key === e.key); if (padInfo) playSound(padInfo.sound); }; window.addEventListener('keydown', handleKeyDown); return () => window.removeEventListener('keydown', handleKeyDown); }, []);
            return ( <div className="w-full flex flex-col items-center justify-center min-h-screen p-4 bg-gray-100"> <h1 className="text-4xl font-bold mb-8 title-font">Play Drums</h1> <div className="space-y-4"> <p className="text-center text-gray-600">키보드 A, S, D 또는 패드를 눌러 드럼을 연주하세요.</p> <div className="flex space-x-4"> {DRUM_PADS.map(pad => ( <button key={pad.name} onClick={() => playSound(pad.sound)} className={`${pad.color} text-white rounded-lg shadow-lg p-8 text-2xl font-bold hover:opacity-80 transition-transform active:scale-95`}> {pad.name} <span className="block text-lg mt-1">({pad.key.toUpperCase()})</span> </button>))} </div> </div> <button onClick={() => setCurrentView('hub')} className="mt-8 text-violet-600 font-bold hover:underline flex items-center mx-auto"> <HomeIcon className="w-5 h-5 mr-1" /> Hub로 돌아가기 </button> </div> );
        };
        
        const ThereminComponent = ({ setCurrentView }) => {
            const audioCtx = useRef(null);
            const oscillator = useRef(null);
            const gainNode = useRef(null);
            const [isPlaying, setIsPlaying] = useState(false);
            const [freq, setFreq] = useState(0);
            const [vol, setVol] = useState(0);

            useEffect(() => { audioCtx.current = new (window.AudioContext || window.webkitAudioContext)(); return () => { if(oscillator.current) oscillator.current.stop(); if(audioCtx.current) audioCtx.current.close(); } }, []);
            const startSound = () => { if (isPlaying || !audioCtx.current) return; const ctx = audioCtx.current; oscillator.current = ctx.createOscillator(); gainNode.current = ctx.createGain(); oscillator.current.type = 'sine'; oscillator.current.connect(gainNode.current); gainNode.current.connect(ctx.destination); gainNode.current.gain.setValueAtTime(0, ctx.currentTime); oscillator.current.start(); setIsPlaying(true); };
            const stopSound = () => { if (!isPlaying || !gainNode.current) return; const ctx = audioCtx.current; gainNode.current.gain.exponentialRampToValueAtTime(0.00001, ctx.currentTime + 0.04); setIsPlaying(false); };
            const handleMouseMove = (e) => { if (!isPlaying || !oscillator.current || !gainNode.current) return; const { clientX, clientY, currentTarget } = e; const { left, top, width, height } = currentTarget.getBoundingClientRect(); const frequency = ((clientX - left) / width) * 800 + 200; const volume = 1 - ((clientY - top) / height); oscillator.current.frequency.setTargetAtTime(frequency, audioCtx.current.currentTime, 0.01); gainNode.current.gain.setTargetAtTime(volume, audioCtx.current.currentTime, 0.01); setFreq(Math.round(frequency)); setVol(Math.round(volume * 100)); };
            return ( <div className="w-full flex flex-col items-center justify-center min-h-screen p-4 bg-gray-900 text-white"> <h1 className="text-4xl font-bold mb-4 title-font">Play Theremin</h1> <p className="mb-4">박스 안에서 마우스를 누르고 움직여보세요.</p> <div className="w-full max-w-2xl h-96 bg-gradient-to-br from-sky-800 to-indigo-900 rounded-lg shadow-2xl cursor-pointer" onMouseDown={startSound} onMouseUp={stopSound} onMouseLeave={stopSound} onMouseMove={handleMouseMove}></div> <div className="mt-4 text-center text-2xl font-mono"> <span>Freq: {freq} Hz</span> <span className="mx-4">|</span> <span>Vol: {vol} %</span> </div> <button onClick={() => setCurrentView('hub')} className="mt-8 text-sky-400 font-bold hover:underline flex items-center mx-auto"> <HomeIcon className="w-5 h-5 mr-1" /> Hub로 돌아가기 </button> </div> );
        };

        const Game2048Component = ({ setCurrentView }) => {
            const [score, setScore] = useState(0);
            const [bestScore, setBestScore] = useState(() => parseInt(localStorage.getItem('bestScore2048')) || 0);
            const [grid, setGrid] = useState([]);
            const [isGameOver, setIsGameOver] = useState(false);
            const isLocked = useRef(false);
            const audioCtx = useRef(null);
            const nextId = useRef(1);

            const playMergeSound = useCallback(() => { if (!audioCtx.current) return; const osc = audioCtx.current.createOscillator(); const gain = audioCtx.current.createGain(); osc.type = 'sine'; osc.frequency.setValueAtTime(440, audioCtx.current.currentTime); osc.frequency.exponentialRampToValueAtTime(880, audioCtx.current.currentTime + 0.05); gain.gain.setValueAtTime(0.1, audioCtx.current.currentTime); gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.current.currentTime + 0.1); osc.connect(gain); gain.connect(audioCtx.current.destination); osc.start(audioCtx.current.currentTime); osc.stop(audioCtx.current.currentTime + 0.1); }, []);
            
            const addNumber = useCallback((currentGrid) => { let emptyCells = []; currentGrid.forEach((cell, i) => { if (cell.value === 0) emptyCells.push(i); }); if (emptyCells.length > 0) { let newGrid = [...currentGrid]; let randomIndex = emptyCells[Math.floor(Math.random() * emptyCells.length)]; newGrid[randomIndex] = { value: Math.random() < 0.9 ? 2 : 4, id: nextId.current++, isNew: true }; return newGrid; } return currentGrid; }, []);

            const startNewGame = useCallback(() => { let newGrid = Array(16).fill(null).map(() => ({ value: 0, id: 0, isNew: false, isMerged: false })); newGrid = addNumber(newGrid); newGrid = addNumber(newGrid); setGrid(newGrid); setScore(0); setIsGameOver(false); setBestScore(localStorage.getItem('bestScore2048') || 0); }, [addNumber]);

            useEffect(() => { audioCtx.current = new (window.AudioContext || window.webkitAudioContext)(); startNewGame(); }, [startNewGame]);

            const move = useCallback((direction) => {
                if (isLocked.current) return;
                isLocked.current = true;
                
                let tempGrid = JSON.parse(JSON.stringify(grid.map(c => ({...c, isNew: false, isMerged: false}))));
                let tempScore = score;
                let hasChanged = false;
                const isVertical = direction === 'up' || direction === 'down';
                const isReversed = direction === 'right' || direction === 'down';

                for (let i = 0; i < 4; i++) {
                    const lineIndexes = isVertical ? [i, i + 4, i + 8, i + 12] : [i * 4, i * 4 + 1, i * 4 + 2, i * 4 + 3];
                    let line = lineIndexes.map(index => tempGrid[index]);
                    
                    if (isReversed) line.reverse();
                    
                    let filtered = line.filter(cell => cell.value !== 0);
                    for (let j = 0; j < filtered.length - 1; j++) {
                        if (filtered[j].value === filtered[j+1].value) {
                            const newValue = filtered[j].value * 2;
                            tempScore += newValue;
                            playMergeSound();
                            filtered[j] = { id: nextId.current++, value: newValue, isMerged: true };
                            filtered.splice(j + 1, 1);
                        }
                    }
                    const newLine = Array(4).fill({value: 0, id: 0});
                    filtered.forEach((cell, idx) => newLine[idx] = cell);
                    if(isReversed) newLine.reverse();

                    lineIndexes.forEach((index, j) => {
                        if (tempGrid[index].value !== newLine[j].value || tempGrid[index].id !== newLine[j].id) hasChanged = true;
                        tempGrid[index] = newLine[j];
                    });
                }
                
                if (hasChanged) {
                    const newGridWithTile = addNumber(tempGrid);
                    setGrid(newGridWithTile);
                    setScore(tempScore);
                     if (tempScore > bestScore) { localStorage.setItem('bestScore2048', tempScore); setBestScore(tempScore); }
                }
                
                setTimeout(() => { isLocked.current = false; }, 150);
            }, [grid, score, bestScore, addNumber, playMergeSound]);

            useEffect(() => {
                const handleKeyDown = (e) => {
                    const directions = { 'ArrowLeft': 'left', 'ArrowRight': 'right', 'ArrowUp': 'up', 'ArrowDown': 'down' };
                    if (directions[e.key]) { e.preventDefault(); move(directions[e.key]); }
                };
                window.addEventListener('keydown', handleKeyDown);
                return () => window.removeEventListener('keydown', handleKeyDown);
            }, [move]);
            
            return ( <div className="w-full flex flex-col items-center justify-center min-h-screen p-4 bg-[#faf8ef]"> <header className="w-full max-w-md mx-auto flex justify-between items-center mb-4"> <h1 className="text-6xl md:text-7xl font-bold text-[#776e65]">2048</h1> <div className="flex space-x-2"> <div className="bg-[#bbada0] p-2 rounded-lg text-center"> <div className="text-[#eee4da] text-sm font-bold">SCORE</div> <div className="text-white text-2xl font-bold">{score}</div> </div> <div className="bg-[#bbada0] p-2 rounded-lg text-center"> <div className="text-[#eee4da] text-sm font-bold">BEST</div> <div className="text-white text-2xl font-bold">{bestScore}</div> </div> </div> </header> <div className="w-full max-w-md mx-auto flex justify-end mb-4"> <button onClick={startNewGame} className="bg-[#8f7a66] text-white font-bold py-2 px-4 rounded-lg hover:bg-[#9e8b78] transition-colors">New Game</button> </div> <main className="relative bg-[#bbada0] p-3 rounded-md w-full max-w-md mx-auto"> <div className="relative w-full" style={{paddingBottom: '100%'}}> <div className="absolute inset-0 grid grid-cols-4 grid-rows-4 gap-3"> {Array(16).fill(0).map((_, i) => <div key={`bg-${i}`} className="bg-[#cdc1b4] rounded-md"></div>)} </div> <div className="absolute inset-0"> {grid.map((cell, i) => { if (cell.value === 0) return null; const x = i % 4; const y = Math.floor(i / 4); const tileValueClass = cell.value > 2048 ? 'super' : cell.value; const tileClass = `game-tile tile-${tileValueClass} ${cell.isNew ? 'tile-new' : ''} ${cell.isMerged ? 'tile-merged' : ''}`; const style = { transform: `translate(calc(${x * 25}% + ${x * 0.75}rem), calc(${y * 25}% + ${y * 0.75}rem))`, width: 'calc(25% - 0.75rem)', height: 'calc(25% - 0.75rem)' }; return <div key={cell.id} className={`${tileClass} absolute flex justify-center items-center rounded-md`} style={style}>{cell.value}</div> })} </div> </div> {isGameOver && ( <div className="absolute inset-0 bg-white/70 flex flex-col justify-center items-center rounded-md"> <h2 className="text-5xl font-bold text-[#776e65]">Game Over!</h2> <button onClick={startNewGame} className="mt-4 bg-[#8f7a66] text-white font-bold py-2 px-4 rounded-lg hover:bg-[#9e8b78] transition-colors">Try Again</button> </div> )} </main> <footer className="w-full max-w-md mx-auto mt-4 text-center text-gray-500"> <p>Use your <strong>arrow keys</strong> or <strong>swipe</strong> to move the tiles.</p> <button onClick={() => setCurrentView('hub')} className="mt-4 text-violet-600 font-bold hover:underline flex items-center mx-auto"> <HomeIcon className="w-5 h-5 mr-1" /> Hub로 돌아가기 </button> </footer> </div> );
        };

        function App() {
            const [currentView, setCurrentView] = useState('hub');
            const renderContent = () => {
                switch(currentView) {
                    case 'hub': return <HubComponent setCurrentView={setCurrentView} />;
                    case 'piano': return <PianoComponent setCurrentView={setCurrentView} />;
                    case 'guitar': return <GuitarComponent setCurrentView={setCurrentView} />;
                    case 'drums': return <DrumComponent setCurrentView={setCurrentView} />;
                    case 'theremin': return <ThereminComponent setCurrentView={setCurrentView} />;
                    case 'game-2048': return <Game2048Component setCurrentView={setCurrentView} />;
                    default: return <HubComponent setCurrentView={setCurrentView} />;
                }
            };
            return <div className="bg-[#f8f9fa] min-h-screen">{renderContent()}</div>
        }

        ReactDOM.render(<App />, document.getElementById('root'));
    </script>
</body>
</html>
