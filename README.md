<!doctype html>
<html lang="ar" dir="rtl" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>لعبة ترتيب الآية</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&amp;family=Tajawal:wght@400;500;700&amp;display=swap" rel="stylesheet">
  <style>
    body {
      box-sizing: border-box;
    }
    
    * {
      font-family: 'Tajawal', 'Amiri', sans-serif;
    }
    
    .word-slot {
      min-width: 55px;
      min-height: 35px;
      transition: all 0.3s ease;
    }
    
    .word-slot.empty {
      border: 2px dashed #d4a574;
      background: rgba(212, 165, 116, 0.1);
    }
    
    .word-slot.filled {
      border: 2px solid #8b5a2b;
      background: linear-gradient(135deg, #f5e6d3 0%, #e8d4bc 100%);
    }
    
    .word-btn {
      cursor: pointer;
      transition: all 0.3s ease;
      user-select: none;
    }
    
    .word-btn:hover:not(.placed):not(.wrong) {
      transform: scale(1.05);
      box-shadow: 0 4px 15px rgba(139, 90, 43, 0.3);
    }
    
    .word-btn.placed {
      opacity: 0.4;
      cursor: not-allowed;
      transform: scale(0.95);
    }
    
    .word-btn.wrong {
      animation: shake 0.5s ease-in-out;
      background: #fee2e2 !important;
      border-color: #ef4444 !important;
    }
    
    .word-btn.correct {
      animation: pulse 0.5s ease-in-out;
    }
    
    @keyframes shake {
      0%, 100% { transform: translateX(0); }
      20%, 60% { transform: translateX(-5px); }
      40%, 80% { transform: translateX(5px); }
    }
    
    @keyframes pulse {
      0%, 100% { transform: scale(1); }
      50% { transform: scale(1.1); }
    }
    
    @keyframes celebrate {
      0% { transform: scale(1) rotate(0deg); }
      25% { transform: scale(1.1) rotate(-3deg); }
      50% { transform: scale(1.2) rotate(0deg); }
      75% { transform: scale(1.1) rotate(3deg); }
      100% { transform: scale(1) rotate(0deg); }
    }
    
    .celebrate {
      animation: celebrate 0.6s ease-in-out;
    }
    
    .confetti {
      position: fixed;
      pointer-events: none;
      z-index: 1000;
    }
    
    .verse-container {
      background: linear-gradient(145deg, #fdf8f3 0%, #f5e6d3 50%, #e8d4bc 100%);
      border: 3px solid #8b5a2b;
      box-shadow: 0 10px 40px rgba(139, 90, 43, 0.2), inset 0 2px 10px rgba(255, 255, 255, 0.5);
    }
    
    .decorative-border {
      background: repeating-linear-gradient(
        90deg,
        #8b5a2b,
        #8b5a2b 10px,
        #d4a574 10px,
        #d4a574 20px
      );
      height: 4px;
    }
  </style>
  <style>@view-transition { navigation: auto; }</style>
  <script src="/_sdk/data_sdk.js" type="text/javascript"></script>
 </head>
 <body class="h-full bg-gradient-to-br from-amber-50 via-orange-50 to-yellow-100 overflow-auto">
  <div id="app-container" class="min-h-full w-full p-4 md:p-6"><!-- Header -->
   <div class="text-center mb-6">
    <div class="inline-block">
     <h1 id="game-title" class="text-2xl md:text-3xl font-bold text-amber-900 mb-2">🕊️ لعبة ترتيب الآية 🕊️</h1>
     <p class="text-amber-700 text-sm md:text-base">إنجيل متى 3: 16-17</p>
     <div class="decorative-border mt-2 rounded-full"></div>
    </div>
   </div><!-- Score and Progress -->
   <div class="flex justify-center gap-4 mb-6 flex-wrap">
    <div class="bg-white/80 backdrop-blur px-4 py-2 rounded-xl shadow-md border border-amber-200"><span class="text-amber-800 font-medium">الكلمات المرتبة: </span> <span id="score" class="text-amber-900 font-bold">0</span> <span class="text-amber-600">/ </span> <span id="total" class="text-amber-600">0</span>
    </div><button id="reset-btn" class="bg-gradient-to-r from-amber-600 to-orange-600 text-white px-4 py-2 rounded-xl shadow-md hover:from-amber-700 hover:to-orange-700 transition-all font-medium"> 🔄 إعادة اللعبة </button>
   </div><!-- Verse Container - Target Area -->
   <div class="verse-container rounded-2xl p-3 md:p-4 mb-6 max-w-3xl mx-auto">
    <div id="verse-slots" class="flex flex-wrap gap-1.5 justify-center items-center"><!-- Slots will be generated here -->
    </div>
   </div><!-- Word Bank -->
   <div class="bg-white/60 backdrop-blur rounded-2xl p-3 md:p-4 max-w-3xl mx-auto border border-amber-200 shadow-lg">
    <h2 class="text-center text-amber-800 font-bold mb-3 text-base">📜 اختر الكلمة التالية الصحيحة 📜</h2>
    <div id="word-bank" class="flex flex-wrap gap-1.5 justify-center"><!-- Shuffled words will appear here -->
    </div>
   </div><!-- Success Message -->
   <div id="success-message" class="hidden fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-50 p-4">
    <div class="bg-gradient-to-br from-amber-100 to-orange-100 p-6 md:p-8 rounded-3xl shadow-2xl text-center max-w-md border-4 border-amber-400 celebrate">
     <div class="text-5xl md:text-6xl mb-4">
      🎉✝️🎉
     </div>
     <h2 class="text-xl md:text-2xl font-bold text-amber-900 mb-3">أحسنت! ���</h2>
     <p class="text-amber-700 mb-4 leading-relaxed text-sm md:text-base">لقد رتبت الآية بنجاح!</p><button id="play-again" class="bg-gradient-to-r from-amber-500 to-orange-500 text-white px-6 py-3 rounded-xl font-bold hover:from-amber-600 hover:to-orange-600 transition-all shadow-lg"> 🔄 العب مرة أخرى </button>
    </div>
   </div>
  </div>
  <script>
    const verseText = "فَلَمَّا اعْتَمَدَ يَسُوعُ صَعِدَ لِلْوَقْتِ مِنَ الْمَاءِ وَإِذَا السَّمَاوَاتُ قَدِ انْفَتَحَتْ لَهُ فَرَأَى رُوحَ اللهِ نَازِلًا مِثْلَ حَمَامَةٍ وَآتِيًا عَلَيْهِ وَصَوْتٌ مِنَ السَّمَاوَاتِ قَائِلًا هذَا هُوَ ابْني الْحَبِيبُ الَّذِي بِهِ سُرِرْتُ";
    
    const correctWords = verseText.split(' ').filter(w => w.trim());
    let currentIndex = 0;
    let shuffledWords = [];
    let placedWords = new Set();
    
    const defaultConfig = {
      game_title: '🕊️ لعبة ترتيب الآية 🕊️',
      primary_color: '#92400e',
      secondary_color: '#f5e6d3',
      text_color: '#78350f',
      accent_color: '#d97706',
      background_color: '#fef3c7'
    };
    
    let config = { ...defaultConfig };
    
    function shuffleArray(array) {
      const shuffled = [...array];
      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
      }
      return shuffled;
    }
    
    function createSlots() {
      const slotsContainer = document.getElementById('verse-slots');
      slotsContainer.innerHTML = '';
      
      correctWords.forEach((word, index) => {
        const slot = document.createElement('div');
        slot.className = 'word-slot empty rounded-lg flex items-center justify-center px-2 py-1.5 text-amber-900 font-medium text-xs md:text-sm';
        slot.dataset.index = index;
        slot.id = `slot-${index}`;
        slotsContainer.appendChild(slot);
      });
    }
    
    function createWordBank() {
      const wordBank = document.getElementById('word-bank');
      wordBank.innerHTML = '';
      
      shuffledWords = shuffleArray([...correctWords]);
      
      shuffledWords.forEach((word, index) => {
        const btn = document.createElement('button');
        btn.className = 'word-btn bg-gradient-to-br from-amber-100 to-orange-100 border-2 border-amber-400 rounded-lg px-2 py-1.5 text-amber-900 font-medium text-xs md:text-sm shadow-md hover:shadow-lg';
        btn.textContent = word;
        btn.dataset.word = word;
        btn.dataset.index = index;
        btn.onclick = () => handleWordClick(btn, word);
        wordBank.appendChild(btn);
      });
    }
    
    function handleWordClick(btn, word) {
      if (btn.classList.contains('placed')) return;
      
      const expectedWord = correctWords[currentIndex];
      
      if (word === expectedWord) {
        // Correct word
        btn.classList.add('correct', 'placed');
        placedWords.add(word);
        
        const slot = document.getElementById(`slot-${currentIndex}`);
        slot.textContent = word;
        slot.classList.remove('empty');
        slot.classList.add('filled');
        
        currentIndex++;
        updateScore();
        
        if (currentIndex === correctWords.length) {
          setTimeout(showSuccess, 500);
        }
      } else {
        // Wrong word
        btn.classList.add('wrong');
        setTimeout(() => {
          btn.classList.remove('wrong');
        }, 500);
      }
    }
    
    function updateScore() {
      document.getElementById('score').textContent = currentIndex;
      document.getElementById('total').textContent = correctWords.length;
    }
    
    function showSuccess() {
      document.getElementById('success-message').classList.remove('hidden');
      createConfetti();
    }
    
    function createConfetti() {
      const colors = ['#f59e0b', '#d97706', '#92400e', '#fbbf24', '#f97316'];
      for (let i = 0; i < 50; i++) {
        setTimeout(() => {
          const confetti = document.createElement('div');
          confetti.className = 'confetti';
          confetti.style.left = Math.random() * 100 + 'vw';
          confetti.style.top = '-20px';
          confetti.style.width = '10px';
          confetti.style.height = '10px';
          confetti.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
          confetti.style.borderRadius = Math.random() > 0.5 ? '50%' : '0';
          confetti.style.transform = `rotate(${Math.random() * 360}deg)`;
          document.body.appendChild(confetti);
          
          let posY = -20;
          let posX = parseFloat(confetti.style.left);
          let rotation = 0;
          const speed = 2 + Math.random() * 3;
          const drift = (Math.random() - 0.5) * 2;
          
          const fall = setInterval(() => {
            posY += speed;
            posX += drift;
            rotation += 5;
            confetti.style.top = posY + 'px';
            confetti.style.left = posX + 'vw';
            confetti.style.transform = `rotate(${rotation}deg)`;
            
            if (posY > window.innerHeight) {
              clearInterval(fall);
              confetti.remove();
            }
          }, 20);
        }, i * 50);
      }
    }
    
    function resetGame() {
      currentIndex = 0;
      placedWords.clear();
      document.getElementById('success-message').classList.add('hidden');
      createSlots();
      createWordBank();
      updateScore();
    }
    
    function applyConfig(cfg) {
      const title = document.getElementById('game-title');
      if (title) {
        title.textContent = cfg.game_title || defaultConfig.game_title;
      }
    }
    
    // Initialize
    document.getElementById('reset-btn').onclick = resetGame;
    document.getElementById('play-again').onclick = resetGame;
    
    createSlots();
    createWordBank();
    updateScore();
    
    // Initialize Element SDK
    if (window.elementSdk) {
      window.elementSdk.init({
        defaultConfig: defaultConfig,
        onConfigChange: async (newConfig) => {
          config = { ...defaultConfig, ...newConfig };
          applyConfig(config);
        },
        mapToCapabilities: (cfg) => ({
          recolorables: [],
          borderables: [],
          fontEditable: undefined,
          fontSizeable: undefined
        }),
        mapToEditPanelValues: (cfg) => new Map([
          ['game_title', cfg.game_title || defaultConfig.game_title]
        ])
      });
    }
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9bcfc2f761374c05',t:'MTc2ODI1MzQ5NS4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
