<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aura Mobile - Simulador de Luxo Futurista</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Rajdhani:wght@300;500;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --gold: #d4af37;
            --dark-glass: rgba(10, 10, 15, 0.9);
            --neon-blue: #00f2ff;
        }
        body {
            background: radial-gradient(circle at center, #1a1a2e 0%, #050505 100%);
            color: #e0e0e0;
            font-family: 'Rajdhani', sans-serif;
            overflow-x: hidden;
            min-height: 100vh;
        }
        .luxury-font { font-family: 'Orbitron', sans-serif; }
        
        .glass-card {
            background: var(--dark-glass);
            backdrop-filter: blur(15px);
            border: 1px solid rgba(212, 175, 55, 0.2);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.8);
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        
        .glass-card:hover {
            border-color: var(--gold);
            transform: translateY(-10px);
            box-shadow: 0 0 20px rgba(212, 175, 55, 0.3);
        }

        .balance-glow {
            text-shadow: 0 0 10px rgba(0, 242, 255, 0.5);
        }

        .buy-btn {
            background: linear-gradient(45deg, #b8860b, #d4af37);
            color: #000;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 2px;
            transition: 0.3s;
        }

        .buy-btn:hover:not(:disabled) {
            filter: brightness(1.2);
            box-shadow: 0 0 15px var(--gold);
        }

        .buy-btn:disabled {
            background: #333;
            color: #666;
            cursor: not-allowed;
        }

        /* Animação de Scan */
        .scanline {
            width: 100%;
            height: 2px;
            background: rgba(0, 242, 255, 0.5);
            position: absolute;
            animation: scan 3s linear infinite;
            z-index: 10;
        }

        @keyframes scan {
            0% { top: 0; }
            100% { top: 100%; }
        }

        .inventory-item {
            animation: fadeIn 0.5s ease-out;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.9); }
            to { opacity: 1; transform: scale(1); }
        }
    </style>
</head>
<body class="p-4 md:p-8">
    
    <header class="max-w-6xl mx-auto flex flex-col md:flex-row justify-between items-center mb-12 gap-6">
        <div>
            <h1 class="luxury-font text-4xl md:text-5xl text-transparent bg-clip-text bg-gradient-to-r from-yellow-500 to-yellow-200">
                AURA TECH
            </h1>
            <p class="text-gray-400 tracking-widest uppercase text-sm mt-1">Sistemas de Comunicação Hiperexclusivos</p>
        </div>
        
        <div class="glass-card p-4 rounded-xl flex items-center gap-4">
            <div class="text-right">
                <p class="text-xs text-gray-400 uppercase">Créditos Disponíveis</p>
                <p id="balance" class="text-3xl font-bold text-cyan-400 balance-glow">R$ 100.000</p>
            </div>
            <div class="h-12 w-12 rounded-full border-2 border-cyan-500 flex items-center justify-center bg-cyan-900/30">
                <span class="text-xl">💎</span>
            </div>
        </div>
    </header>

    <main class="max-w-6xl mx-auto grid grid-cols-1 lg:grid-cols-3 gap-8">
        <!-- Coluna de Catálogo -->
        <div class="lg:col-span-2">
            <h2 class="luxury-font text-xl mb-6 flex items-center gap-2">
                <span class="w-8 h-[1px] bg-gold block"></span> LANÇAMENTOS 2029
            </h2>
            <div id="catalog" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <!-- Itens serão gerados via JS -->
            </div>
        </div>

        <!-- Coluna de Inventário/Status -->
        <div class="space-y-8">
            <div class="glass-card p-6 rounded-2xl">
                <h2 class="luxury-font text-lg mb-4 border-b border-gray-700 pb-2">SUA COLEÇÃO</h2>
                <div id="inventory-list" class="space-y-4 max-h-[400px] overflow-y-auto pr-2">
                    <p class="text-gray-500 italic text-center py-8">Nenhum dispositivo adquirido.</p>
                </div>
            </div>

            <div class="glass-card p-6 rounded-2xl relative overflow-hidden">
                <div class="scanline"></div>
                <h2 class="luxury-font text-lg mb-2">STATUS DA IA</h2>
                <div id="ai-status" class="text-sm text-cyan-200 font-mono">
                    > Conectando ao mainframe...<br>
                    > Verificando nível de prestígio...<br>
                    > Status: <span class="text-green-400">MEMBRO BÁSICO</span>
                </div>
            </div>
        </div>
    </main>

    <!-- Modal de Feedback -->
    <div id="modal" class="fixed inset-0 bg-black/80 hidden items-center justify-center z-50 p-4">
        <div class="glass-card max-w-sm w-full p-8 rounded-3xl text-center border-2 border-cyan-400">
            <div id="modal-icon" class="text-6xl mb-4">✨</div>
            <h3 id="modal-title" class="luxury-font text-2xl mb-2"></h3>
            <p id="modal-msg" class="text-gray-400 mb-6"></p>
            <button onclick="closeModal()" class="w-full py-3 rounded-lg bg-cyan-600 hover:bg-cyan-500 text-white font-bold transition">FECHAR PROTOCOLO</button>
        </div>
    </div>

    <script>
        let balance = 100000;
        const inventory = [];

        const phones = [
            {
                id: 1,
                name: "Aura Minimalist",
                price: 85000,
                desc: "O básico indispensável. Carcaça em titânio aeroespacial e tela de cristal de safira.",
                emoji: "📱",
                tier: "Entrada"
            },
            {
                id: 2,
                name: "Nebula Quantum",
                price: 250000,
                desc: "Processador alimentado por micro-fusão. A bateria dura 50 anos.",
                emoji: "✨",
                tier: "Premium"
            },
            {
                id: 3,
                name: "Singularity Gold",
                price: 1200000,
                desc: "Feito com ouro recuperado de asteroides. Inclui IA pessoal consciente.",
                emoji: "🔱",
                tier: "Ultra-Luxo"
            },
            {
                id: 4,
                name: "Event Horizon",
                price: 5000000,
                desc: "Tela feita de plasma sólido. O dispositivo existe parcialmente em outra dimensão.",
                emoji: "🌌",
                tier: "Exótico"
            },
            {
                id: 5,
                name: "Chronos Heirloom",
                price: 15000000,
                desc: "Edição limitada. Altera a percepção do tempo para o usuário ganhar produtividade.",
                emoji: "⏳",
                tier: "Lendário"
            }
        ];

        function formatMoney(value) {
            return value.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
        }

        function updateUI() {
            document.getElementById('balance').innerText = formatMoney(balance);
            
            const catalog = document.getElementById('catalog');
            catalog.innerHTML = '';
            
            phones.forEach(phone => {
                const canAfford = balance >= phone.price;
                const card = `
                    <div class="glass-card rounded-2xl p-6 relative flex flex-col justify-between">
                        <div class="absolute top-4 right-4 text-xs font-bold px-2 py-1 rounded bg-yellow-900/50 text-yellow-400 uppercase tracking-tighter border border-yellow-600">
                            ${phone.tier}
                        </div>
                        <div>
                            <div class="text-5xl mb-4">${phone.emoji}</div>
                            <h3 class="luxury-font text-xl text-white mb-2">${phone.name}</h3>
                            <p class="text-gray-400 text-sm mb-4 leading-relaxed">${phone.desc}</p>
                        </div>
                        <div>
                            <p class="text-2xl font-bold text-yellow-500 mb-4">${formatMoney(phone.price)}</p>
                            <button 
                                onclick="buyPhone(${phone.id})"
                                ${!canAfford ? 'disabled' : ''}
                                class="buy-btn w-full py-3 rounded-lg flex items-center justify-center gap-2">
                                ${canAfford ? 'ADQUIRIR AGORA' : 'SALDO INSUFICIENTE'}
                            </button>
                        </div>
                    </div>
                `;
                catalog.innerHTML += card;
            });

            updateInventory();
            updateAIStatus();
        }

        function updateInventory() {
            const list = document.getElementById('inventory-list');
            if (inventory.length === 0) return;

            list.innerHTML = '';
            inventory.forEach((item, index) => {
                list.innerHTML += `
                    <div class="inventory-item bg-white/5 p-3 rounded-lg border-l-4 border-yellow-500 flex justify-between items-center">
                        <div class="flex items-center gap-3">
                            <span class="text-2xl">${item.emoji}</span>
                            <div>
                                <p class="font-bold text-sm">${item.name}</p>
                                <p class="text-[10px] text-gray-500 uppercase">${item.tier}</p>
                            </div>
                        </div>
                        <span class="text-[10px] text-cyan-400">#${item.serial}</span>
                    </div>
                `;
            });
        }

        function updateAIStatus() {
            const status = document.getElementById('ai-status');
            let tierText = "MEMBRO BÁSICO";
            let color = "text-green-400";
            
            if (balance > 1000000) { tierText = "MAGNATA TECH"; color = "text-purple-400"; }
            if (inventory.length > 3) { tierText = "COLECIONADOR DE ELITE"; color = "text-yellow-400"; }
            if (inventory.find(i => i.price > 1000000)) { tierText = "ENTIDADE SUPREMA"; color = "text-cyan-400"; }

            status.innerHTML = `
                > Analisando ativos financeiros...<br>
                > Portfólio: ${inventory.length} dispositivos<br>
                > Status: <span class="${color}">${tierText}</span><br>
                > ${balance < 50000 ? 'ALERTA: LIQUIDEZ BAIXA' : 'SISTEMA ESTÁVEL'}
            `;
        }

        function buyPhone(id) {
            const phone = phones.find(p => p.id === id);
            if (balance >= phone.price) {
                balance -= phone.price;
                const serial = Math.random().toString(36).substring(7).toUpperCase();
                inventory.push({...phone, serial});
                
                showModal(
                    "ADQUISIÇÃO DE ELITE", 
                    `Parabéns. O seu ${phone.name} está sendo enviado via drone suborbital direto para sua cobertura.`,
                    "💎"
                );
                updateUI();
            }
        }

        function showModal(title, msg, icon) {
            document.getElementById('modal-title').innerText = title;
            document.getElementById('modal-msg').innerText = msg;
            document.getElementById('modal-icon').innerText = icon;
            const modal = document.getElementById('modal');
            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        function closeModal() {
            const modal = document.getElementById('modal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
        }

        // Inicializar
        window.onload = updateUI;
    </script>
</body>
</html>

