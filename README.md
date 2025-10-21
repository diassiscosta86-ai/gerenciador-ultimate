# gerenciador-ultimate
geren-diassis-lima
<!DOCTYPE html>
<html lang="pt-PT">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestor de Banca</title>
    <!-- Carregamento do Tailwind CSS para estilização -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Carregamento do Chart.js para o Gráfico -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.3/dist/chart.umd.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap" rel="stylesheet">
    <style>
        :root {
            --color-primary: #10b981; /* Esmeralda 500 (vitória) */
            --color-secondary: #f43f5e; /* Rosa 500 (derrota) */
        }
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            transition: background-color 0.3s;
        }
        .card {
            background-color: white;
            border-radius: 12px;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 4, 0, 0.05);
            transition: background-color 0.3s, box-shadow 0.3s;
        }
        .btn-win {
            background-color: var(--color-primary);
            transition: all 0.15s;
        }
        .btn-win:hover {
            background-color: #059669; /* Esmeralda 600 */
            transform: translateY(-1px);
        }
        .btn-loss {
            background-color: var(--color-secondary);
            transition: all 0.15s;
        }
        .btn-loss:hover {
            background-color: #e11d48; /* Rosa 600 */
            transform: translateY(-1px);
        }
        /* CLASSE PARA DESTAQUE DE ERRO */
        .error-highlight {
            border-color: var(--color-secondary) !important; 
            box-shadow: 0 0 0 3px rgba(244, 63, 94, 0.5) !important;
        }
        #bankrollChartContainer {
            position: relative;
            height: 40vh; /* Altura responsiva */
            width: 100%;
        }
        .llm-loading {
            position: relative;
            pointer-events: none;
            opacity: 0.8;
        }
        .llm-loading::after {
            content: '';
            position: absolute;
            left: 50%;
            top: 50%;
            width: 16px;
            height: 16px;
            border: 2px solid #fff;
            border-top-color: transparent;
            border-radius: 50%;
            animation: spin 1s ease-in-out infinite;
            margin-left: -8px;
            margin-top: -8px;
        }
        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        /* DARK MODE STYLES: MELHOR CONTRASTE */
        .dark {
            background-color: #121212; /* Fundo muito escuro */
        }
        .dark .card {
            background-color: #1f2937; /* Cinza 800 */
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.5), 0 4px 6px -2px rgba(0, 0, 0, 0.3);
        }
        .dark body, .dark .text-gray-800, .dark .text-gray-700, .dark .text-gray-900 {
            color: #ffffff; /* Texto principal branco */
        }
        .dark .text-gray-500, .dark .text-gray-600 {
            color: #e5e7eb; /* Cinza 100 para texto secundário (muito visível) */
        }
        .dark .border-gray-400 {
            border-color: #4b5563;
        }
        .dark .bg-gray-50 {
            background-color: #1f2937;
        }
        .dark table.divide-gray-200 {
            border-color: #4b5563;
        }
        .dark .bg-pink-100 {
            background-color: #4c081e;
            color: #fca5a5;
        }
        .dark .border-gray-300 {
            border-color: #4b5563;
            background-color: #4b5563; /* Cor de fundo dos inputs escurecida */
        }
        .dark input, .dark select {
            background-color: #374151; /* Cor de fundo dos inputs */
            color: #ffffff;
            border-color: #4b5563;
        }
        .dark .table-header-bg { /* Nova classe para o cabeçalho da tabela */
            background-color: #374151; /* Mais escuro que o bg-gray-50 original */
        }
        .dark .table-body-bg {
             background-color: #1f2937;
        }
        /* Cor de texto forcada para o modo claro */
        .text-black {
            color: #000000;
        }
    </style>
</head>
<body class="p-4 sm:p-8">

    <div id="app" class="max-w-6xl mx-auto space-y-8">
        <header class="text-center p-6 card flex justify-between items-center">
            <div class="flex flex-col items-start">
                <h1 class="text-3xl font-extrabold text-gray-800">📊 Gestor de Banca</h1>
                <p class="text-gray-500 mt-1 text-sm">O seu risco e retorno são calculados por cada moeda de forma isolada.</p>
            </div>
            
            <!-- Botão/Switch de Modo Noturno -->
            <div class="flex items-center space-x-2">
                <span class="text-gray-500 text-sm">Modo Noturno</span>
                <label for="darkModeToggle" class="relative inline-flex items-center cursor-pointer">
                    <input type="checkbox" id="darkModeToggle" class="sr-only peer" onclick="toggleDarkMode()">
                    <div class="w-11 h-6 bg-gray-200 rounded-full peer peer-focus:ring-2 peer-focus:ring-indigo-500 peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-0.5 after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-indigo-600"></div>
                </label>
            </div>
        </header>
        
        <!-- ALERTA STOP LOSS/WIN (Será preenchido pelo JavaScript) -->
        <div id="stopLimitAlert">
            <!-- Alerta aparece aqui -->
        </div>

        <!-- Secção de Configurações de Risco -->
        <div class="p-6 card border-l-4 border-indigo-500">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">⚙️ Configurações da Banca Atual</h2>
            <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                
                <!-- Modo de Risco -->
                <div class="col-span-1">
                    <label for="riskModeInput" class="block text-sm font-medium text-gray-700">Modo de Risco</label>
                    <div class="mt-1 flex gap-4">
                        <label class="inline-flex items-center">
                            <input type="radio" name="riskMode" value="percentage" checked
                                   onclick="updateSettings('riskMode', 'percentage'); toggleRiskInputs('percentage')"
                                   class="form-radio text-indigo-600">
                            <span class="ml-2 text-sm text-gray-700">% da Banca</span>
                        </label>
                        <label class="inline-flex items-center">
                            <input type="radio" name="riskMode" value="fixed"
                                   onclick="updateSettings('riskMode', 'fixed'); toggleRiskInputs('fixed')"
                                   class="form-radio text-indigo-600">
                            <span class="ml-2 text-sm text-gray-700">Valor Fixo</span>
                        </label>
                    </div>
                </div>

                <!-- Input Risco % (Visível no modo 'percentage') -->
                <div id="percentageRiskGroup">
                    <label for="riskPercentageInput" class="block text-sm font-medium text-gray-700">Risco Máximo da Banca (%)</label>
                    <div class="relative mt-1 rounded-md shadow-sm">
                        <input type="number" id="riskPercentageInput" min="0.1" step="0.1"
                                class="block w-full rounded-md border-gray-300 pr-10 focus:border-indigo-500 focus:ring-indigo-500"
                                onchange="updateSettings('riskPercentage', this.value)" oninput="updateSettings('riskPercentage', this.value)">
                        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3">
                            <span class="text-gray-500">%</span>
                        </div>
                    </div>
                </div>

                <!-- Input Risco Fixo (Visível no modo 'fixed') -->
                <div id="fixedRiskGroup" class="hidden">
                    <label for="fixedUnitValueInput" class="block text-sm font-medium text-gray-700">Valor Fixo da Unidade (Risco / 3)</label>
                    <div class="relative mt-1 rounded-md shadow-sm">
                        <input type="number" id="fixedUnitValueInput" min="0.01" step="0.01"
                                class="block w-full rounded-md border-gray-300 pr-10 focus:border-indigo-500 focus:ring-indigo-500"
                                onchange="updateSettings('fixedUnitValue', this.value)" oninput="updateSettings('fixedUnitValue', this.value)">
                        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3">
                            <span id="fixedUnitSymbol" class="text-gray-500">$</span>
                        </div>
                    </div>
                </div>

                <!-- Input Porcentagem de Entrada (Visível no modo 'fixed') -->
                <div id="entryPercentageGroup" class="hidden">
                    <label for="entryPercentageInput" class="block text-sm font-medium text-gray-700">Porcentagem de Entrada (Unidade %)</label>
                    <div class="relative mt-1 rounded-md shadow-sm">
                        <input type="number" id="entryPercentageInput" min="0.1" step="0.1"
                                class="block w-full rounded-md border-gray-300 pr-10 focus:border-indigo-500 focus:ring-indigo-500"
                                onchange="updateSettings('entryPercentage', this.value)" oninput="updateSettings('entryPercentage', this.value)">
                        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3">
                            <span class="text-gray-500">%</span>
                        </div>
                    </div>
                </div>
                
                <!-- Input Retorno % -->
                <div>
                    <label for="returnPercentageInput" class="block text-sm font-medium text-gray-700">Retorno da Operação (Pay-out %)</label>
                    <div class="relative mt-1 rounded-md shadow-sm">
                        <input type="number" id="returnPercentageInput" min="0.1" step="0.1"
                                class="block w-full rounded-md border-gray-300 pr-10 focus:border-indigo-500 focus:ring-indigo-500"
                                onchange="updateSettings('returnPercentage', this.value)" oninput="updateSettings('returnPercentage', this.value)">
                        <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center pr-3">
                            <span class="text-gray-500">%</span>
                        </div>
                    </div>
                </div>

                <!-- Seletor de Moeda -->
                <div>
                    <label for="currencyInput" class="block text-sm font-medium text-gray-700">Moeda da Banca</label>
                    <select id="currencyInput" 
                            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                            onchange="switchBankroll(this.value)">
                        <option value="USD">Dólar (USD)</option>
                        <option value="EUR">Euro (€)</option>
                        <option value="BRL">Real (R$)</option>
                    </select>
                </div>

            </div>
        </div>

        <!-- Secção de Dados Chave e Regra de Risco / 3 -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 xl:grid-cols-6 gap-4">
            <!-- 1. Banca Inicial (Input) -->
            <div class="p-5 card border-l-4 border-gray-400">
                <label for="initialBankrollInput" class="block text-sm font-medium text-gray-500">Banca Inicial</label>
                <input type="number" id="initialBankrollInput" placeholder="Ex: 1000.00"
                       class="mt-1 block w-full text-xl font-semibold border-none focus:ring-0 p-0 text-gray-700"
                       onchange="updateInitialBankroll(this.value)" oninput="updateInitialBankroll(this.value)">
            </div>

            <!-- 2. Banca Atual (Display) -->
            <div class="p-5 card border-l-4 border-green-500">
                <p class="text-sm font-medium text-gray-500">Banca Atual (Saldo)</p>
                <p id="currentBankrollDisplay" class="mt-1 text-2xl font-bold text-green-600">$ 0,00</p>
            </div>

            <!-- 3. Risco Máximo Total (Calculado DINÂMICO) -->
            <div class="p-5 card border-l-4 border-red-500">
                <p class="text-sm font-medium text-gray-500" id="maxRiskLabel">Risco Máximo Total</p>
                <p id="maxRiskTotalDisplay" class="mt-1 text-2xl font-bold text-red-600">$ 0,00</p>
            </div>

            <!-- 4. Valor da Unidade (Calculado) -->
            <div class="p-5 card border-l-4 border-orange-500">
                <p class="text-sm font-medium text-gray-500">Valor da Unidade (Risco / 3)</p>
                <p id="unitStakeDisplay" class="mt-1 text-2xl font-bold text-orange-600">$ 0,00</p>
            </div>

             <!-- 5. Distância para Stop Loss -->
            <div class="p-5 card border-l-4 border-red-700">
                <p class="text-sm font-medium text-gray-500">Falta para Stop Loss (5%)</p>
                <p id="stopLossDistanceDisplay" class="mt-1 text-xl font-bold text-red-700">$ 0,00</p>
            </div>

            <!-- 6. Distância para Stop Win -->
            <div class="p-5 card border-l-4 border-green-700">
                <p class="text-sm font-medium text-gray-500">Falta para Stop Win (5%)</p>
                <p id="stopWinDistanceDisplay" class="mt-1 text-xl font-bold text-green-700">$ 0,00</p>
            </div>
        </div>
        
        <!-- Sugestão de Próximo Investimento (Nova Funcionalidade) -->
        <div class="p-5 card border-l-4 border-yellow-500 flex justify-between items-center">
            <div>
                <p class="text-sm font-medium text-gray-500">Próximo Investimento Sugerido (Gale)</p>
                <p id="nextGaleDisplay" class="mt-1 text-xl font-bold text-yellow-700">Unidade Padrão</p>
            </div>
            <!-- Botão de Reinício de Ciclo de Stops -->
            <button onclick="resetStopCycle()" 
                    class="bg-indigo-500 text-white font-medium py-2 px-4 rounded-lg shadow-md hover:bg-indigo-600 transition duration-150 text-sm flex items-center gap-1">
                🔨 Reiniciar Ciclo de Stops
            </button>
        </div>


        <!-- Métricas de Desempenho -->
        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
            <!-- 5. Total de Vitórias -->
            <div class="p-5 card border-l-4 border-green-700 text-center">
                <p class="text-sm font-medium text-gray-500">Vitórias (W)</p>
                <p id="winCountDisplay" class="mt-1 text-3xl font-bold text-green-700">0</p>
            </div>
             <!-- 6. Total de Derrotas -->
            <div class="p-5 card border-l-4 border-red-700 text-center">
                <p class="text-sm font-medium text-gray-500">Derrotas (L)</p>
                <p id="lossCountDisplay" class="mt-1 text-3xl font-bold text-red-700">0</p>
            </div>
             <!-- 7. Taxa de Acerto (Win Rate) -->
            <div class="p-5 card border-l-4 border-blue-500 text-center">
                <p class="text-sm font-medium text-gray-500">Taxa de Vitórias (W%)</p>
                <p id="winRateDisplay" class="mt-1 text-3xl font-bold text-blue-600">0.00%</p>
            </div>
        </div>
        
        <!-- Análise de Desempenho do Ciclo (NOVA SECÇÃO) -->
        <div class="p-6 card border-l-4 border-pink-500">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">🧠 Análise de Desempenho do Ciclo (LLM)</h2>
            <div id="cycleAnalysisOutput" class="p-3 mb-4 bg-gray-50 border border-gray-200 rounded-lg text-gray-600 dark:bg-gray-800 dark:text-gray-300 hidden">
                <!-- Output da análise de desempenho aqui -->
            </div>
            <button onclick="analyzeCyclePerformance()" id="analyzeCycleButton"
                    class="bg-pink-600 text-white font-medium py-2 px-4 rounded-lg shadow-md hover:bg-pink-700 hover:shadow-lg transition w-full sm:w-auto">
                ✨ Análise de Ciclo W/L (Gemini Coach)
            </button>
        </div>

        <!-- Gráfico de Crescimento da Banca -->
        <div class="p-6 card">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">📈 Evolução da Banca</h2>
            <div id="bankrollChartContainer">
                <canvas id="bankrollChart"></canvas>
            </div>
        </div>
        
        <!-- Secção de Registo Rápido (Operações) -->
        <div class="p-6 card">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">⚡ Registo Rápido (Estratégias)</h2>
            
            <!-- Linha 1: Unidade Padrão -->
            <div class="mb-6">
                <h3 class="text-lg font-semibold text-gray-700 mb-2 border-b pb-1">Unidade Padrão</h3>
                <p class="text-sm text-gray-600 mb-4">
                    Lucro/perda baseados no **Valor da Unidade** (Risco/3).
                </p>
                <div class="flex flex-wrap gap-4">
                    <!-- Vitória Padrão -->
                    <button onclick="registerAutoTransaction(true, 0)" 
                            class="btn-win text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        ✅ Vitória
                    </button>
                    <!-- Derrota Padrão -->
                    <button onclick="registerAutoTransaction(false, 0)" 
                            class="btn-loss text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        ❌ Derrota
                    </button>
                </div>
            </div>

            <!-- Linha 2: Estratégia GALE (Recuperação) -->
            <div>
                <h3 class="text-lg font-semibold text-gray-700 mb-2 border-b pb-1">Estratégia Gale (Recuperação)</h3>
                <p class="text-sm text-gray-600 mb-4 text-orange-600 font-medium">
                    O investimento é calculado para **cobrir as perdas acumuladas** e garantir o **lucro padrão da unidade**.
                </p>
                <div class="flex flex-wrap gap-4">
                    <!-- Gale 1 (Investimento: Recuperação) -->
                    <button onclick="registerAutoTransaction(true, 1)" 
                            class="btn-win text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        🔥 Gale 1 (Vitória)
                    </button>
                    <button onclick="registerAutoTransaction(false, 1)" 
                            class="btn-loss text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        🔥 Gale 1 (Derrota)
                    </button>

                    <!-- Gale 2 (Investimento: Recuperação Total) -->
                    <button onclick="registerAutoTransaction(true, 2)" 
                            class="btn-win text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        🔥 Gale 2 (Vitória)
                    </button>
                    <button onclick="registerAutoTransaction(false, 2)" 
                            class="btn-loss text-white font-medium py-3 px-6 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        🔥 Gale 2 (Derrota)
                    </button>
                </div>
            </div>
        </div>

        <!-- Tabela de Histórico de Transações -->
        <div class="p-6 card overflow-x-auto">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">🧾 Histórico de Transações</h2>
            <table class="min-w-full divide-y divide-gray-200">
                <thead class="table-header-bg">
                    <tr>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Data</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Descrição</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Investido</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Resultado</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Banca Atual</th>
                        <th class="px-3 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Ação</th>
                    </tr>
                </thead>
                <tbody id="transactionList" class="table-body-bg divide-y divide-gray-200">
                    <!-- As transações serão inseridas aqui pelo JavaScript -->
                    <tr>
                        <td colspan="6" class="py-4 text-center text-gray-500">Nenhuma transação registada.</td>
                    </tr>
                </tbody>
            </table>
        </div>

        <!-- Secção de Adicionar Transação Manual (Aportes/Retiradas) -->
        <div class="p-6 card">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">✍️ Adicionar Transação Manual (Aportes/Outros)</h2>
            <div id="llmAnalysisOutput" class="p-3 mb-4 bg-gray-50 border border-gray-200 rounded-lg text-gray-600 dark:bg-gray-800 dark:text-gray-300 hidden">
                <!-- Output da análise LLM aqui -->
            </div>
            <form id="transactionForm" class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="col-span-1 md:col-span-2">
                    <label for="description" class="block text-sm font-medium text-gray-700">Descrição</label>
                    <input type="text" id="description" required placeholder="Ex: Novo Aporte ou Operação Manual" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-emerald-500 focus:ring-emerald-500">
                </div>
                <div>
                    <label for="invested" class="block text-sm font-medium text-gray-700">Valor Investido</label>
                    <input type="number" id="invested" required min="0" step="0.01" value="0.00" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-emerald-500 focus:ring-emerald-500">
                </div>
                <div>
                    <label for="result" class="block text-sm font-medium text-gray-700">Resultado (L/P/Aporte)</label>
                    <input type="number" id="result" required step="0.01" placeholder="Ex: +500.00 (Aporte) ou -25.00 (Saque)" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-emerald-500 focus:ring-emerald-500">
                </div>
                
                <div class="md:col-span-4 flex flex-col sm:flex-row justify-end gap-2 mt-4">
                    <button type="button" onclick="analyzeRiskWithLLM(event)" id="analyzeButton"
                            class="bg-indigo-600 text-white font-medium py-2 px-4 rounded-lg shadow-md hover:bg-indigo-700 hover:shadow-lg transition w-full sm:w-auto">
                        ✨ Análise de Risco LLM
                    </button>
                    <button type="submit" class="btn-win text-white font-medium py-2 px-4 rounded-lg shadow-md hover:shadow-lg w-full sm:w-auto">
                        Registar Transação Manual
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Firebase Imports e Scripts -->
    <script type="module">
        // Importa as funções necessárias do Firebase SDK
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot, updateDoc, arrayRemove, arrayUnion } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Variáveis globais de ambiente (necessárias para o Canvas, mas ajustadas para ambientes públicos)
        // Para ambiente público/GitHub Pages, estas variáveis serão null, forçando o signInAnonymously.
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'public-app-id'; 
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null; 
        
        // API Key para a API Gemini (deixada em branco para o ambiente Canvas)
        const apiKey = "" 
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

        let db;
        let auth;
        let userId = 'loading';
        let trackerDocRef; // Será definido dinamicamente em listenForData/switchBankroll
        let unsubscribeSnapshot = null; // Para gerir a subscrição do Firestore
        
        // Dados atuais da aplicação
        let currentData = {
            initialBankroll: 0,
            transactions: [],
            riskPercentage: 5,
            fixedUnitValue: 10.00, 
            entryPercentage: 100, // Novo: Porcentagem do valor fixo para a stake
            riskMode: 'percentage', 
            returnPercentage: 87,
            currency: 'USD',
            stopCycleStart: 0 // NOVO: Timestamp para marcar o início do ciclo de stops
        };
        let isAuthReady = false;
        let bankrollChart; // Variável para a instância do Chart.js

        setLogLevel('Debug');

        // Função para alternar o Modo Noturno (Dark Mode)
        window.toggleDarkMode = () => {
            const isDark = document.body.classList.toggle('dark');
            localStorage.setItem('darkMode', isDark ? 'enabled' : 'disabled');
            // Força a atualização do gráfico para que as cores de fundo mudem
            if (bankrollChart) {
                bankrollChart.options.plugins.legend.labels.color = isDark ? '#f3f4f6' : '#1f2937';
                bankrollChart.options.scales.y.ticks.color = isDark ? '#d1d5db' : '#4b5563';
                bankrollChart.options.scales.x.ticks.color = isDark ? '#d1d5db' : '#4b5563';
                bankrollChart.update();
            }
        };

        // Carregar a preferência do Modo Noturno ao iniciar
        document.addEventListener('DOMContentLoaded', () => {
            const darkModeToggle = document.getElementById('darkModeToggle');
            if (localStorage.getItem('darkMode') === 'enabled') {
                document.body.classList.add('dark');
                if (darkModeToggle) darkModeToggle.checked = true;
            } else if (darkModeToggle) {
                darkModeToggle.checked = false;
            }
        });

        // Função para formatar números como moeda (Euro, Real ou Dólar)
        const formatCurrency = (value) => {
            if (typeof value !== 'number' || isNaN(value)) return '$ 0.00';
            
            let currencyCode = 'USD';
            let locale = 'en-US';

            if (currentData.currency === 'BRL') {
                currencyCode = 'BRL';
                locale = 'pt-BR';
            } else if (currentData.currency === 'EUR') {
                 currencyCode = 'EUR';
                 locale = 'pt-PT';
            }

            return new Intl.NumberFormat(locale, {
                style: 'currency',
                currency: currencyCode,
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            }).format(value);
        };

        // Função para alternar a visibilidade dos campos de input de risco
        window.toggleRiskInputs = (mode) => {
            document.getElementById('percentageRiskGroup')?.classList.toggle('hidden', mode === 'fixed');
            document.getElementById('fixedRiskGroup')?.classList.toggle('hidden', mode === 'percentage');
            // NOVO: Mostrar Porcentagem de Entrada APENAS no modo Fixo
            document.getElementById('entryPercentageGroup')?.classList.toggle('hidden', mode === 'percentage');

            // Atualiza o símbolo no input fixo
            const symbol = currentData.currency === 'BRL' ? 'R$' : (currentData.currency === 'EUR' ? '€' : '$');
            document.getElementById('fixedUnitSymbol').textContent = symbol;
        };

        // Função principal de inicialização do Firebase e autenticação
        const initFirebase = async () => {
            // Inicialização do Firebase (APENAS se a configuração estiver disponível)
            if (!firebaseConfig) {
                 console.warn("Aviso: Configuração do Firebase não encontrada. A aplicação irá tentar usar o login anónimo, mas não guardará dados de forma persistente no Canvas.");
                 // Se não houver config, a aplicação irá funcionar sem salvar dados no Firestore
                 isAuthReady = true;
                 userId = crypto.randomUUID();
                 const initialCurrency = document.getElementById('currencyInput').value;
                 switchBankroll(initialCurrency, true); 
                 initChart();
                 return;
            } 
            
            // Fluxo normal (Canvas)
            const app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);


            // Autenticação
            onAuthStateChanged(auth, async (user) => {
                if (user) {
                    userId = user.uid;
                } else {
                    // Tenta login com token (Canvas) ou anónimo (Público/Canvas)
                    try {
                        if (initialAuthToken) {
                            await signInWithCustomToken(auth, initialAuthToken);
                        } else {
                            // Este é o fallback para GitHub Pages ou se o token expirar
                            await signInAnonymously(auth); 
                        }
                        userId = auth.currentUser.uid;
                    } catch (error) {
                        console.error("Erro ao autenticar no Firebase. Tentando anónimo...", error);
                        // Última tentativa de fallback: login anónimo
                        try {
                           await signInAnonymously(auth);
                           userId = auth.currentUser.uid;
                        } catch (e) {
                           console.error("Falha na autenticação anónima.", e);
                           userId = crypto.randomUUID(); // Fallback para ID temporário
                        }
                    }
                }

                isAuthReady = true;
                // Inicializa com a moeda padrão (USD), lendo o seletor na UI
                const initialCurrency = document.getElementById('currencyInput').value;
                switchBankroll(initialCurrency, true); 
                initChart(); // Inicializa o gráfico
            });
        };

        // Troca o documento da banca para a moeda selecionada e carrega os dados
        window.switchBankroll = (newCurrency, isInitialLoad = false) => {
            if (!isAuthReady) return;
            
            // 1. Altera a moeda no estado local
            currentData.currency = newCurrency;

            // Se o Firebase NÃO estiver inicializado (ambiente público sem config), não fazemos nada.
            if (!db) {
                renderUI();
                updateChart();
                return;
            }

            // 2. Cancela a escuta anterior (CRUCIAL para multi-documento)
            if (unsubscribeSnapshot) {
                unsubscribeSnapshot();
                unsubscribeSnapshot = null;
            }

            // 3. Define o novo documento (Ex: main_tracker_USD, main_tracker_EUR)
            const docId = `main_tracker_${newCurrency}`;
            trackerDocRef = doc(db, `artifacts/${appId}/users/${userId}/financial_tracker`, docId);

            // 4. Inicia a escuta para o novo documento
            listenForData();

            // 5. Atualiza a UI
            if (!isInitialLoad) {
                document.getElementById('currencyInput').value = newCurrency;
            }
        };
        
        // Inicializa o Gráfico Chart.js
        const initChart = () => {
            const isDark = document.body.classList.contains('dark');
            
            // FIX: Destrói o gráfico existente antes de criar um novo, se houver.
            if (bankrollChart) {
                bankrollChart.destroy();
            }

            const ctx = document.getElementById('bankrollChart')?.getContext('2d');
            if (!ctx) return; // Segurança caso o canvas ainda não esteja pronto

            bankrollChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Início'], // Etiqueta inicial
                    datasets: [{
                        label: 'Banca Atual',
                        data: [0], // Valor inicial
                        borderColor: '#2563eb', // Azul
                        backgroundColor: 'rgba(37, 99, 235, 0.1)',
                        tension: 0.3,
                        pointRadius: 4,
                        pointHoverRadius: 6,
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: false,
                            title: {
                                display: true,
                                text: 'Valor'
                            },
                            ticks: {
                                color: isDark ? '#d1d5db' : '#4b5563', // Cor do tick
                                callback: function(value) {
                                    return formatCurrency(value);
                                }
                            }
                        },
                         x: {
                            ticks: {
                                color: isDark ? '#d1d5db' : '#4b5563', // Cor do tick
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed.y !== null) {
                                        label += formatCurrency(context.parsed.y);
                                    }
                                    return label;
                                }
                            }
                        }
                    }
                }
            });
             // Aplica cor do texto do gráfico para o dark mode
            if (bankrollChart.options.plugins.tooltip.titleColor) {
                bankrollChart.options.plugins.tooltip.titleColor = isDark ? '#f3f4f6' : '#1f2937';
            }
        };

        // Função para escutar as mudanças no Firestore em tempo real
        const listenForData = () => {
            if (!isAuthReady || !trackerDocRef || !db) return;
            
            // Atribui a função de "unsubscribe"
            unsubscribeSnapshot = onSnapshot(trackerDocRef, (docSnap) => {
                const currencyBefore = currentData.currency;
                
                if (docSnap.exists()) {
                    const data = docSnap.data();
                    currentData = {
                        initialBankroll: data.initialBankroll || 0,
                        transactions: (data.transactions || []).filter(t => t.timestamp),
                        // Carrega as configurações específicas do documento da moeda
                        riskPercentage: data.riskPercentage !== undefined ? data.riskPercentage : 5,
                        fixedUnitValue: data.fixedUnitValue !== undefined ? data.fixedUnitValue : 10.00, 
                        entryPercentage: data.entryPercentage !== undefined ? data.entryPercentage : 100, // NOVO
                        riskMode: data.riskMode || 'percentage', 
                        returnPercentage: data.returnPercentage !== undefined ? data.returnPercentage : 87,
                        currency: currencyBefore, // Mantém a moeda selecionada, que define o documento
                        stopCycleStart: data.stopCycleStart || 0 // NOVO: Carrega o timestamp de reinício
                    };
                } else {
                    // Documento não existe (é a primeira vez que usa esta moeda)
                    // Reseta para os defaults e cria o documento
                    currentData = {
                        initialBankroll: 0,
                        transactions: [],
                        riskPercentage: 5,
                        fixedUnitValue: 10.00,
                        entryPercentage: 100, // NOVO DEFAULT
                        riskMode: 'percentage',
                        returnPercentage: 87,
                        currency: currencyBefore,
                        stopCycleStart: 0
                    };
                    setDoc(trackerDocRef, currentData).catch(e => console.error(`Erro ao criar documento inicial para ${currencyBefore}:`, e));
                }
                
                // Aplica o modo de risco carregado
                toggleRiskInputs(currentData.riskMode);
                
                renderUI();
                updateChart(); // Atualiza o gráfico após renderizar a UI
                // Garante que o modo noturno se mantém no gráfico
                if (localStorage.getItem('darkMode') === 'enabled') {
                    // Aplica as cores corretas ao gráfico no dark mode
                    const isDark = true; 
                    if (bankrollChart) {
                        bankrollChart.options.plugins.legend.labels.color = isDark ? '#f3f4f6' : '#1f2937';
                        bankrollChart.options.scales.y.ticks.color = isDark ? '#d1d5db' : '#4b5563';
                        bankrollChart.options.scales.x.ticks.color = isDark ? '#d1d5db' : '#4b5563';
                        bankrollChart.update();
                    }
                }
            }, (error) => {
                console.error("Erro ao escutar dados:", error);
            });
        };

        // Atualiza os dados no Gráfico
        const updateChart = () => {
            if (!bankrollChart) return;
            
            // Filtra e ordena as transações pela data mais antiga primeiro para calcular a evolução
            const transactions = currentData.transactions.slice().sort((a, b) => a.timestamp - b.timestamp);
            let currentBankroll = currentData.initialBankroll;

            // 1. Prepara os dados para o gráfico
            const labels = ['Início'];
            const data = [currentBankroll];

            transactions.forEach((t, index) => {
                currentBankroll += t.result;
                labels.push(`Op. ${index + 1} (${new Date(t.timestamp).toLocaleDateString('pt-PT').slice(0, 5)})`);
                data.push(currentBankroll);
            });

            // 2. Atualiza o Chart.js
            bankrollChart.data.labels = labels;
            bankrollChart.data.datasets[0].data = data;
            bankrollChart.update();
        };

        // Reinicia o ciclo de Stop Loss/Win
        window.resetStopCycle = async () => {
            if (!isAuthReady || !trackerDocRef || !db) return;
            
            // Obtém o timestamp do reinício
            const newCycleStartTimestamp = Date.now();
            
            try {
                // Atualiza o stopCycleStart para o timestamp atual (reinicia SL/SW e W/L)
                await updateDoc(trackerDocRef, {
                    stopCycleStart: newCycleStartTimestamp,
                });
                console.log("Ciclo de Stops Reiniciado com sucesso.");
            } catch (error) {
                console.error("Erro ao reiniciar ciclo de stops:", error);
            }
        };


        // Verifica os limites de Stop Loss e Stop Win
        window.checkStopLimits = (currentBankroll, initialBankroll) => {
            const stopLimitAlert = document.getElementById('stopLimitAlert');
            const stopLossDistanceDisplay = document.getElementById('stopLossDistanceDisplay');
            const stopWinDistanceDisplay = document.getElementById('stopWinDistanceDisplay');
            
            if (stopLimitAlert) stopLimitAlert.innerHTML = ''; // Limpa alertas anteriores

            if (initialBankroll <= 0) {
                 if(stopLossDistanceDisplay) stopLossDistanceDisplay.textContent = formatCurrency(0);
                 if(stopWinDistanceDisplay) stopWinDistanceDisplay.textContent = formatCurrency(0);
                 return;
            }
            
            // --- CÁLCULO DE LIMITES DO CICLO ---
            
            const cycleStartTimestamp = currentData.stopCycleStart || 0;
            let cycleStartingBankroll = currentData.initialBankroll;
            
            // 1. Calcula o saldo no momento do reinício (ignora transações mais antigas que o cycleStartTimestamp)
            currentData.transactions.forEach(t => {
                if (t.timestamp < cycleStartTimestamp) {
                    cycleStartingBankroll += t.result;
                }
            });
            
            // 2. Define os Limites com base no Saldo de Partida do Ciclo
            const stopLossLimit = cycleStartingBankroll * 0.95; // 5% de perda
            const stopWinLimit = cycleStartingBankroll * 1.05;  // 5% de ganho
            
            // 3. Cálculos da Distância
            const differenceLoss = Math.max(0, currentBankroll - stopLossLimit);
            const differenceWin = Math.max(0, stopWinLimit - currentBankroll);

            if(stopLossDistanceDisplay) stopLossDistanceDisplay.textContent = formatCurrency(differenceLoss);
            if(stopWinDistanceDisplay) stopWinDistanceDisplay.textContent = formatCurrency(differenceWin);
            
            let alertHtml = '';

            if (currentBankroll <= stopLossLimit) {
                alertHtml = `
                    <div class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 card dark:bg-red-900 dark:text-red-300" role="alert">
                        <p class="font-bold text-lg">🛑 STOP LOSS ATINGIDO!</p>
                        <p>O seu saldo atingiu ${formatCurrency(currentBankroll)} (Limites baseados em ${formatCurrency(cycleStartingBankroll)}). Aconselha-se a parar as operações hoje.</p>
                    </div>
                `;
            } else if (currentBankroll >= stopWinLimit) {
                alertHtml = `
                    <div class="bg-green-100 border-l-4 border-green-500 text-green-700 p-4 card dark:bg-green-900 dark:text-green-300" role="alert">
                        <p class="font-bold text-lg">🎉 STOP WIN ATINGIDO!</p>
                        <p>O seu saldo atingiu ${formatCurrency(currentBankroll)} (Limites baseados em ${formatCurrency(cycleStartingBankroll)}). Meta alcançada. Aconselha-se a parar as operações.</p>
                    </div>
                `;
            }

            if (stopLimitAlert) stopLimitAlert.innerHTML = alertHtml;
        };


        // Função que renderiza a interface do utilizador com os dados atuais
        const renderUI = () => {
            const initialBankroll = currentData.initialBankroll;
            const transactions = currentData.transactions;

            // 1. Atualizar Configurações (Inputs)
            const riskInput = document.getElementById('riskPercentageInput');
            const fixedInput = document.getElementById('fixedUnitValueInput');
            const entryInput = document.getElementById('entryPercentageInput'); 
            const returnInput = document.getElementById('returnPercentageInput');
            const currencyInput = document.getElementById('currencyInput');
            const maxRiskLabel = document.getElementById('maxRiskLabel');
            const nextGaleDisplay = document.getElementById('nextGaleDisplay'); 
            
            if (riskInput && document.activeElement !== riskInput) {
                riskInput.value = currentData.riskPercentage;
            }
            if (fixedInput && document.activeElement !== fixedInput) {
                fixedInput.value = currentData.fixedUnitValue.toFixed(2);
            }
            if (entryInput && document.activeElement !== entryInput) {
                entryInput.value = currentData.entryPercentage;
            }
            if (returnInput && document.activeElement !== returnInput) {
                returnInput.value = currentData.returnPercentage;
            }
            if (currencyInput && document.activeElement !== currencyInput) {
                currencyInput.value = currentData.currency;
            }
            
            // Define o modo de risco correto nos radio buttons
            const riskModeElement = document.querySelector(`input[name="riskMode"][value="${currentData.riskMode}"]`);
            if (riskModeElement) {
                riskModeElement.checked = true;
            }

            // 2. Atualizar Banca Inicial (Input)
            const initialInput = document.getElementById('initialBankrollInput');
            if (initialInput && document.activeElement !== initialInput) { 
                initialInput.value = initialBankroll.toFixed(2);
            }

            // 3. Calcular Banca Atual (Base para o risco dinâmico)
            let currentBankroll = initialBankroll;
            transactions.forEach(t => {
                currentBankroll += t.result;
            });
            const bankrollForRisk = Math.max(0, currentBankroll); // Banca para o cálculo
            window.currentBankrollValue = currentBankroll; // Salva globalmente para o LLM
            
            // NOVO: Verifica Stop Limits E ATUALIZA DISTÂNCIAS (Baseado sempre na Banca Inicial)
            checkStopLimits(currentBankroll, initialBankroll);

            let maxRiskTotal = 0;
            let unitStake = 0;

            // 4. Cálculo baseado no Modo de Risco
            if (currentData.riskMode === 'percentage') {
                const riskRatio = currentData.riskPercentage / 100;
                maxRiskTotal = bankrollForRisk * riskRatio;
                unitStake = maxRiskTotal / 3;
                if (maxRiskLabel) { // SAFE CHECK
                    maxRiskLabel.innerHTML = `Risco Máximo Total (<span id="riskPercentLabel">${currentData.riskPercentage}%</span> da Atual) `;
                }
            } else { // fixed mode
                // Se for valor fixo, o Valor da Unidade é o Valor Fixo * Porcentagem de Entrada
                const entryRatio = currentData.entryPercentage / 100;
                unitStake = currentData.fixedUnitValue * entryRatio;
                maxRiskTotal = unitStake * 3; // O risco total é 3x a stake real
                
                // NOVO: Atualiza a label para mostrar a porcentagem de entrada no modo Fixo
                if (maxRiskLabel) { // SAFE CHECK
                    maxRiskLabel.innerHTML = `Risco Máximo Total (3x Stake: <span id="riskPercentLabel">${currentData.entryPercentage}%</span> do Fixo) `;
                }
            }

            // 5. Atualizar Labels e Displays (COM VERIFICAÇÃO DE NULL)
            document.getElementById('currentBankrollDisplay').textContent = formatCurrency(currentBankroll);
            document.getElementById('maxRiskTotalDisplay').textContent = formatCurrency(maxRiskTotal);
            document.getElementById('unitStakeDisplay').textContent = formatCurrency(unitStake);
            
            // Atualiza labels de % na UI
            const riskPercentLabel = document.getElementById('riskPercentLabel');
            if(riskPercentLabel) riskPercentLabel.textContent = `${currentData.riskPercentage}%`;
            
            const returnPercentLabel = document.getElementById('returnPercentLabel');
            if(returnPercentLabel) returnPercentLabel.textContent = `${currentData.returnPercentage}%`;


            // 6. CÁLCULO DO PRÓXIMO GALE (Clássico - Dobro)
            const autoTransactions = transactions.filter(t => t.description.includes('Unidade') || t.description.includes('Gale')).sort((a, b) => b.timestamp - b.timestamp);
            const lastAutoTransaction = autoTransactions[0];
            
            let nextInvestmentText = `Unidade Padrão: ${formatCurrency(unitStake)}`;
            
            if (lastAutoTransaction && lastAutoTransaction.result < 0) {
                // Última operação foi uma derrota
                const lastStrategyLevel = lastAutoTransaction.description.includes('Gale 1') ? 1 : (lastAutoTransaction.description.includes('Unidade') ? 0 : 2);
                
                if (lastStrategyLevel === 0) {
                    // Se a última foi Unidade (Derrota), o próximo é Gale 1
                    // Investimento = Unidade + (Unidade / Retorno)
                    const requiredInvestment = unitStake + (unitStake / (currentData.returnPercentage/100));
                    nextInvestmentText = `Gale 1 (Recuperação): ${formatCurrency(requiredInvestment)}`; 
                } else if (lastStrategyLevel === 1) {
                    // Se a última foi Gale 1 (Derrota), o próximo é Gale 2
                    // Investimento = Perda Acumulada (Unidade + Gale 1 Investido) + Lucro Padrão / Retorno
                    const previousInvestment = autoTransactions[0].investedAmount;
                    const requiredInvestment = previousInvestment + (unitStake / (currentData.returnPercentage/100));
                    nextInvestmentText = `Gale 2 (Recuperação): ${formatCurrency(requiredInvestment)}`;
                } else if (lastStrategyLevel === 2) {
                    // Fim da sequência de Gale
                    nextInvestmentText = `Sequência Gale 2 terminada. Voltar à ${formatCurrency(unitStake)}`;
                }
            } 
            // Se a última foi vitória ou não há transações, o próximo é a Unidade Padrão.

            if(nextGaleDisplay) nextGaleDisplay.innerHTML = nextInvestmentText; // Exibe o próximo investimento

            // 7. Calcular Métricas W/L (Apenas transações automáticas do CICLO ATUAL)
            let winCount = 0;
            let lossCount = 0;
            let totalAutoTransactions = 0;
            const cycleStartTimestamp = currentData.stopCycleStart || 0;

            transactions.forEach(t => {
                // Filtra apenas transações APÓS o início do ciclo de stops
                if (t.timestamp >= cycleStartTimestamp && (t.description.includes("Unidade") || t.description.includes("Gale"))) {
                    totalAutoTransactions++;
                    if (t.result > 0) {
                        winCount++;
                    } else if (t.result < 0) {
                        lossCount++;
                    }
                }
            });

            const winRate = totalAutoTransactions > 0 ? (winCount / totalAutoTransactions) * 100 : 0;

            // 8. Atualizar Métricas W/L
            document.getElementById('winCountDisplay').textContent = winCount;
            document.getElementById('lossCountDisplay').textContent = lossCount;
            document.getElementById('winRateDisplay').textContent = `${winRate.toFixed(2)}%`;

            // 9. Renderizar Tabela de Transações
            const transactionList = document.getElementById('transactionList');
            if (!transactionList) return; // SAFE CHECK para a lista
            
            transactionList.innerHTML = ''; 

            if (transactions.length === 0) {
                transactionList.innerHTML = '<tr><td colspan="6" class="py-4 text-center text-gray-500">Nenhuma transação registada.</td></tr>';
                return;
            }

            // Mapeia e inverte a ordem para mostrar o mais recente primeiro
            transactions.slice().sort((a, b) => b.timestamp - a.timestamp).forEach((t) => {
                
                // Recalcula o saldo da banca a cada linha (para o caso de edições)
                const sortedTransactions = currentData.transactions.slice().sort((a, b) => a.timestamp - b.timestamp);
                
                const originalIndex = sortedTransactions.findIndex(st => st.timestamp === t.timestamp && st.result === t.result && st.investedAmount === t.investedAmount);
                
                let rowCurrentBankroll = initialBankroll;
                if (originalIndex !== -1) {
                    for (let i = 0; i <= originalIndex; i++) {
                        rowCurrentBankroll += sortedTransactions[i].result;
                    }
                } else {
                    rowCurrentBankroll = currentBankroll; 
                }
                
                const row = document.createElement('tr');
                const resultColor = t.result >= 0 ? 'text-green-600 font-semibold' : 'text-red-600 font-semibold';
                
                // Determina a classe condicional para o valor INVESTIDO
                // CORREÇÃO: Aplicar text-black/dark:text-white APENAS para GALE, se o background estiver rosa/roxo
                const investedHighlightClass = t.description.includes("Gale") 
                    ? 'bg-pink-100 text-black dark:text-white font-medium dark:bg-pink-900' 
                    : 'text-black dark:text-white'; 
                
                let cleanDescription = t.description;


                row.innerHTML = `
                    <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-500 dark:text-gray-300">${new Date(t.timestamp).toLocaleDateString('pt-PT')}</td>
                    <td class="px-3 py-2 text-sm text-gray-900 dark:text-white">${cleanDescription}</td>
                    <td class="px-3 py-2 whitespace-nowrap text-sm ${investedHighlightClass}">
                        ${formatCurrency(t.investedAmount)}
                    </td>
                    <td class="px-3 py-2 whitespace-nowrap text-sm ${resultColor}">${formatCurrency(t.result)}</td>
                    <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-900 font-bold dark:text-white">${formatCurrency(rowCurrentBankroll)}</td>
                    <td class="px-3 py-2 whitespace-nowrap text-sm text-gray-900">
                        <button onclick="deleteTransaction(${t.timestamp}, ${t.result}, ${t.investedAmount})" 
                                class="text-red-500 hover:text-red-700 p-1 rounded-full hover:bg-red-100 dark:hover:bg-red-900 transition duration-150"
                                title="Eliminar Transação">
                            🗑️
                        </button>
                    </td>
                `;
                transactionList.appendChild(row);
            });
        };

        // --- Funções de Ação do Utilizador ---

        // Função para chamar a API Gemini e analisar a transação manual
        window.analyzeRiskWithLLM = async (event) => {
            event.preventDefault();

            const description = document.getElementById('description').value || 'Transação genérica';
            const investedAmount = parseFloat(document.getElementById('invested').value);
            const result = parseFloat(document.getElementById('result').value);
            const currentBankroll = window.currentBankrollValue;

            const outputDiv = document.getElementById('llmAnalysisOutput');
            const analyzeButton = document.getElementById('analyzeButton');

            if (isNaN(investedAmount) || isNaN(result) || currentBankroll <= 0) {
                outputDiv?.classList.remove('hidden');
                outputDiv.innerHTML = `<p class="text-red-600">⚠️ Erro: Por favor, defina a Banca Inicial e preencha os campos Investido/Resultado.</p>`;
                return;
            }

            analyzeButton?.classList.add('llm-loading');
            if(analyzeButton) analyzeButton.textContent = 'A Analisar...';
            outputDiv?.classList.add('hidden');
            if(outputDiv) outputDiv.innerHTML = '';

            // Determina o símbolo correto para o prompt
            let currencySymbol = currentData.currency === 'BRL' ? 'R$' : (currentData.currency === 'EUR' ? '€' : '$');
            
            // 1. Construir o Prompt para o LLM
            const prompt = `
                Como um analista financeiro conservador, avalie a seguinte transação em relação ao capital total.
                
                - Banca Atual (Capital Total): ${currencySymbol} ${currentBankroll.toFixed(2)}
                - Transação (Investido/Risco): ${currencySymbol} ${investedAmount.toFixed(2)}
                - Resultado Esperado/Lançado: ${currencySymbol} ${result.toFixed(2)}
                - Descrição: ${description}

                Forneça uma análise de risco concisa (máximo 4 frases) em português de Portugal.
                1. Destaque o percentual de risco (% do capital total investido nesta operação).
                2. Comente sobre o impacto desta exposição no capital total.
                3. Use um tom conservador (ex: "é um risco elevado", "abordagem sensata").
            `;
            
            // 2. Configurar a chamada da API
            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
                systemInstruction: {
                    parts: [{ text: "Você é um analista financeiro conservador e conciso. Suas respostas devem ser limitadas a 4 frases, focadas em percentual de risco e preservação de capital." }]
                },
            };

            const maxRetries = 3;
            for (let i = 0; i < maxRetries; i++) {
                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) {
                        throw new Error(`Erro HTTP: ${response.status}`);
                    }

                    const resultJson = await response.json();
                    const generatedText = resultJson.candidates?.[0]?.content?.parts?.[0]?.text || "Erro: Não foi possível obter a análise.";

                    // 3. Exibir o resultado na UI
                    if(outputDiv) outputDiv.innerHTML = `<p class="font-bold text-gray-800 dark:text-white">Análise LLM (Gemini):</p><p>${generatedText}</p>`;
                    outputDiv?.classList.remove('hidden');
                    break; // Sai do loop se for bem-sucedido

                } catch (error) {
                    console.error(`Erro na chamada LLM (Tentativa ${i + 1}):`, error);
                    if (i < maxRetries - 1) {
                        // Implementa backoff exponencial
                        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
                    } else {
                        if(outputDiv) outputDiv.innerHTML = `<p class="text-red-600">❌ Erro na API Gemini: Não foi possível realizar a análise após ${maxRetries} tentativas.</p>`;
                        outputDiv?.classList.remove('hidden');
                    }
                }
            }
            
            // 4. Resetar o botão
            analyzeButton?.classList.remove('llm-loading');
            if(analyzeButton) analyzeButton.textContent = '✨ Análise de Risco LLM';
        };
        
        // Função para chamar a API Gemini e analisar o desempenho do ciclo
        window.analyzeCyclePerformance = async () => {
            const currentBankroll = window.currentBankrollValue;
            const winCount = document.getElementById('winCountDisplay').textContent;
            const lossCount = document.getElementById('lossCountDisplay').textContent;
            const winRate = document.getElementById('winRateDisplay').textContent;

            const outputDiv = document.getElementById('cycleAnalysisOutput');
            const analyzeButton = document.getElementById('analyzeCycleButton');

            if (currentBankroll <= 0) {
                outputDiv?.classList.remove('hidden');
                outputDiv.innerHTML = `<p class="text-red-600">⚠️ Erro: Por favor, defina a Banca Inicial para calcular o desempenho.</p>`;
                return;
            }

            analyzeButton?.classList.add('llm-loading');
            if(analyzeButton) analyzeButton.textContent = 'A Analisar Desempenho...';
            outputDiv?.classList.add('hidden');
            if(outputDiv) outputDiv.innerHTML = '';

            // Determina o símbolo correto para o prompt
            let currencySymbol = currentData.currency === 'BRL' ? 'R$' : (currentData.currency === 'EUR' ? '€' : '$');
            
            // 1. Construir o Prompt para o LLM (Coach de Desempenho)
            const prompt = `
                Como um coach de gestão de capital, analise o desempenho do ciclo de operações atual.
                
                - Banca Atual: ${currencySymbol} ${currentBankroll.toFixed(2)}
                - Total de Vitórias (W): ${winCount}
                - Total de Derrotas (L): ${lossCount}
                - Taxa de Acerto (Win Rate): ${winRate}

                Forneça um feedback motivacional e uma avaliação da disciplina.
                1. Comente sobre o Win Rate e W/L.
                2. Sugira manter ou ajustar o foco na disciplina.
                3. Use um tom de coach positivo (máximo 4 frases) em português de Portugal.
            `;
            
            // 2. Configurar a chamada da API
            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
                systemInstruction: {
                    parts: [{ text: "Você é um coach de gestão de capital, motivacional e focado na disciplina. Suas respostas devem ser limitadas a 4 frases." }]
                },
            };

            const maxRetries = 3;
            for (let i = 0; i < maxRetries; i++) {
                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) {
                        throw new Error(`Erro HTTP: ${response.status}`);
                    }

                    const resultJson = await response.json();
                    const generatedText = resultJson.candidates?.[0]?.content?.parts?.[0]?.text || "Erro: Não foi possível obter a análise de desempenho.";

                    // 3. Exibir o resultado na UI
                    if(outputDiv) outputDiv.innerHTML = `<p class="font-bold text-gray-800 dark:text-white">Análise de Ciclo (Gemini Coach):</p><p>${generatedText}</p>`;
                    outputDiv?.classList.remove('hidden');
                    break; // Sai do loop se for bem-sucedido

                } catch (error) {
                    console.error(`Erro na chamada LLM (Tentativa ${i + 1}):`, error);
                    if (i < maxRetries - 1) {
                        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
                    } else {
                        if(outputDiv) outputDiv.innerHTML = `<p class="text-red-600">❌ Erro na API Gemini: Não foi possível realizar a análise após ${maxRetries} tentativas.</p>`;
                        outputDiv?.classList.remove('hidden');
                    }
                }
            }
            
            // 4. Resetar o botão
            analyzeButton?.classList.remove('llm-loading');
            if(analyzeButton) analyzeButton.textContent = '✨ Análise de Ciclo W/L (Gemini Coach)';
        };


        // Atualiza as configurações de porcentagem ou moeda no Firestore
        window.updateSettings = async (settingKey, value) => {
            if (!isAuthReady || !trackerDocRef || !db) return;

            // Se for moeda, ela já foi trocada pelo switchBankroll
            if (settingKey === 'currency') return; 

            // Atualiza os valores percentuais ou fixos
            let updateValue = value;
            if (settingKey === 'riskPercentage' || settingKey === 'returnPercentage' || settingKey === 'fixedUnitValue' || settingKey === 'entryPercentage') {
                updateValue = parseFloat(value);
                if (isNaN(updateValue) || updateValue <= 0) return;
            }

            try {
                await updateDoc(trackerDocRef, {
                    [settingKey]: updateValue
                });
            } catch (error) {
                // Se o documento não existir, tenta criar
                if (error.code === 'not-found') {
                    // Cuidado: aqui temos que ter certeza que currentData reflete o estado atual da moeda
                    const docToSet = {
                        initialBankroll: currentData.initialBankroll,
                        transactions: currentData.transactions,
                        currency: currentData.currency,
                        riskPercentage: currentData.riskPercentage,
                        fixedUnitValue: currentData.fixedUnitValue,
                        entryPercentage: currentData.entryPercentage,
                        riskMode: currentData.riskMode,
                        returnPercentage: currentData.returnPercentage,
                        stopCycleStart: currentData.stopCycleStart
                    };
                    docToSet[settingKey] = updateValue;
                    await setDoc(trackerDocRef, docToSet);
                } else {
                    console.error(`Erro ao atualizar configuração ${settingKey}:`, error);
                }
            }
        };


        // Atualiza a Banca Inicial no Firestore
        window.updateInitialBankroll = async (value) => {
            if (!isAuthReady || !trackerDocRef || !db) return;

            const newBankroll = parseFloat(value);
            if (isNaN(newBankroll) || newBankroll < 0) return;

            try {
                await updateDoc(trackerDocRef, {
                    initialBankroll: newBankroll
                });
            } catch (error) {
                // Se o documento não existir, tenta criar
                if (error.code === 'not-found') {
                     await setDoc(trackerDocRef, {
                        ...currentData, // Inclui as configurações da moeda atual
                        initialBankroll: newBankroll
                    });
                } else {
                    console.error("Erro ao atualizar banca inicial:", error);
                }
            }
        };

        // Regista uma transação automática baseada na Unidade
        window.registerAutoTransaction = async (isWin, strategyLevel) => {
            if (!isAuthReady || !trackerDocRef || !db) return;
            
            const initialInput = document.getElementById('initialBankrollInput');
            let bankrollToCheck = currentData.initialBankroll;

            // FIX DE SINCRONIZAÇÃO: Se a banca em currentData for 0, usa o valor do input e atualiza o estado local para calcular
            if (bankrollToCheck <= 0) {
                const uiBankroll = parseFloat(initialInput.value);
                if (uiBankroll > 0) {
                    // Salva o valor no Firestore
                    await updateInitialBankroll(uiBankroll);
                    // ATUALIZA O ESTADO LOCAL (Corrigido)
                    currentData.initialBankroll = uiBankroll;
                    bankrollToCheck = uiBankroll; 
                }
            }
            
            if (bankrollToCheck <= 0) {
                console.error("Erro: Banca Inicial não definida. Por favor, defina a Banca Inicial primeiro.");
                initialInput.classList.add('error-highlight');
                setTimeout(() => { initialInput.classList.remove('error-highlight'); }, 3000);
                initialInput.focus();
                return;
            }
            
            // 1. Determina a Unidade de Risco
            let unitStake = 0;
            let currentBankroll = currentData.initialBankroll;
            currentData.transactions.forEach(t => { currentBankroll += t.result; });
            const bankrollForRisk = Math.max(0, currentBankroll); 

            if (currentData.riskMode === 'percentage') {
                const riskRatio = currentData.riskPercentage / 100;
                const maxRiskTotal = bankrollForRisk * riskRatio;
                unitStake = maxRiskTotal / 3;
            } else {
                // Modo Fixo: Unidade de Risco é o Valor Fixo * Porcentagem de Entrada
                const entryRatio = currentData.entryPercentage / 100;
                unitStake = currentData.fixedUnitValue * entryRatio;
            }
            
            if (unitStake <= 0) {
                 console.warn("Valor da Unidade não definido ou zero. Não é possível calcular nova Unidade de risco.");
                 return;
            }

            const returnRatio = currentData.returnPercentage / 100;
            const standardProfit = unitStake * returnRatio; // Lucro padrão de uma vitória de unidade
            let investedAmount = unitStake;
            let description = '';
            let result; 
            
            // 2. Lógica GALE (Recuperação e Lucro Padrão)
            if (strategyLevel === 1 || strategyLevel === 2) {
                
                // 2.1 CÁLCULO DAS PERDAS ACUMULADAS
                let accumulatedLoss = 0;
                if (strategyLevel === 1) { 
                    accumulatedLoss = unitStake; 
                } else if (strategyLevel === 2) {
                    // Investimento do Gale 1 (para perda)
                    const gale1Investment = unitStake / returnRatio;
                    // Total perdido antes do Gale 2: Perda da Unidade + Perda do Gale 1
                    accumulatedLoss = unitStake + gale1Investment; 
                }
                
                // 2.2 Determina o Investimento Necessário (Stake)
                // Stake = (Perda Acumulada + Lucro Padrão) / Retorno Percentual
                investedAmount = (accumulatedLoss + standardProfit) / returnRatio;

                // Arredonda para 2 casas decimais
                investedAmount = parseFloat(investedAmount.toFixed(2));
                
                // 2.3 Determina o Resultado
                if (isWin) {
                    // CORREÇÃO FINAL: Resultado no histórico = Lucro Padrão + Perdas Acumuladas (para compensar)
                    result = standardProfit + accumulatedLoss; 
                    
                    description = `✅ Vitória (Gale ${strategyLevel}) (Líquido: ${formatCurrency(standardProfit)})`;
                    
                } else {
                    // Se for DERROTA no Gale
                    result = -investedAmount; // Perda total do investimento Gale
                    description = `❌ Derrota (Gale ${strategyLevel})`;
                }
            
            } else { 
                // Lógica Padrão (Unidade Única)
                description = isWin ? `✅ Vitória (Unidade)` : `❌ Derrota (Unidade)`;
                if (isWin) {
                    result = standardProfit;
                } else {
                    result = -investedAmount; // Perda da Unidade
                }
                // investedAmount já é unitStake
            }


            const newTransaction = {
                timestamp: Date.now(), 
                description,
                investedAmount: parseFloat(investedAmount.toFixed(2)), 
                result: parseFloat(result.toFixed(2)) 
            };


            try {
                await updateDoc(trackerDocRef, {
                    transactions: arrayUnion(newTransaction)
                });
            } catch (error) {
                 if (error.code === 'not-found') {
                    await setDoc(trackerDocRef, {
                        ...currentData, // Inclui as configurações
                        transactions: [newTransaction]
                    });
                } else {
                    console.error("Erro ao adicionar transação automática:", error);
                }
            }
        };

        // Função para eliminar uma transação
        window.deleteTransaction = async (timestamp, result, investedAmount) => {
            if (!isAuthReady || !trackerDocRef || !db) return;

            const transactionToDelete = currentData.transactions.find(t => 
                t.timestamp === timestamp && 
                Math.abs(t.result - result) < 0.01 && 
                Math.abs(t.investedAmount - investedAmount) < 0.01
            );
            
            if (!transactionToDelete) {
                console.error("Erro: Transação não encontrada para eliminação.");
                return;
            }

            try {
                await updateDoc(trackerDocRef, {
                    transactions: arrayRemove(transactionToDelete)
                });
                console.log("Transação eliminada com sucesso.");
            } catch (error) {
                console.error("Erro ao eliminar transação:", error);
            }
        };


        // Adiciona uma nova transação manual ao Firestore (Aportes/Outras)
        document.getElementById('transactionForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            if (!isAuthReady || !trackerDocRef || !db) return;

            const description = document.getElementById('description').value;
            const investedAmount = parseFloat(document.getElementById('invested').value);
            const result = parseFloat(document.getElementById('result').value);

            if (isNaN(investedAmount) || isNaN(result)) {
                console.error("Valores inválidos para investimento/resultado.");
                return;
            }

            const newTransaction = {
                timestamp: Date.now(),
                description,
                investedAmount,
                result 
            };

            try {
                await updateDoc(trackerDocRef, {
                    transactions: arrayUnion(newTransaction)
                });

                document.getElementById('transactionForm').reset();
            } catch (error) {
                 if (error.code === 'not-found') {
                    await setDoc(trackerDocRef, {
                        ...currentData, // Inclui as configurações
                        transactions: [newTransaction]
                    });
                    document.getElementById('transactionForm').reset();
                } else {
                    console.error("Erro ao adicionar transação:", error);
                }
            }
        });

        // Inicia o aplicativo Firebase
        window.onload = initFirebase;

    </script>
}
