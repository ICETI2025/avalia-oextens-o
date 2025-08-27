<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Avaliação de Projetos de Extensão</title>
    
    <!-- Chosen Palette: Warm Neutrals with Blue Accent -->
    <!-- Application Structure Plan: The application is structured as a single-page linear workflow. It starts with 'Identification'. The core evaluation is a single list of criteria, with standard radio/checkbox inputs. A 'Final Score' summary card provides immediate feedback. The flow concludes with a 'Justificativa' section and action buttons. The form now submits directly to a SheetMonkey endpoint and triggers a confirmation email via EmailJS. -->
    <!-- Visualization & Content Choices: 
        - Identification: Standard HTML text/url inputs, all marked as required, linked to a progress bar.
        - Evaluation Criteria: Each criterion is a static, always-visible section.
        - Tooltips: Added info icons with CSS-based tooltips to provide contextual help.
        - Tabela 1 Criteria (e.g., 1a-1e): Radio buttons with a conditional, restyled range slider.
        - Final Score Summary: A dedicated, styled card for high visibility.
        - Library/Method: Vanilla JavaScript for all logic. Replaced native alerts with custom toast notifications. Form submission now triggers an email confirmation. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->

    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" xintegrity="sha512-iecdLmaskl7CVkqkXNQ/ZH/XLlvWZOJyj7Yy7tcenmpD1ypASozpmT/E0iPtmFIB46ZmdtAc9eNBvH0H/ZpiBw==" crossorigin="anonymous" referrerpolicy="no-referrer" />
    <script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@emailjs/browser@4/dist/email.min.js"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8f9fa;
        }
        .card {
            background-color: white;
            border-radius: 12px;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.05);
            margin-bottom: 30px;
            overflow: hidden;
        }
        .final-score-card {
            background-color: #e7f5ff;
            border-left: 5px solid #005A9C;
        }
        .btn-primary {
            background-color: #005A9C;
            transition: all 0.3s ease;
        }
        .btn-primary:hover {
            background-color: #004170;
            transform: translateY(-2px);
            box-shadow: 0 4px 10px rgba(0, 90, 156, 0.2);
        }
        .btn-secondary {
            background-color: #6c757d;
            transition: all 0.3s ease;
        }
        .btn-secondary:hover {
            background-color: #5a6268;
            transform: translateY(-2px);
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }
        input:focus, textarea:focus {
            outline: none;
            box-shadow: 0 0 0 3px rgba(0, 90, 156, 0.2);
            border-color: #005A9C;
        }
        input.error {
            border-color: #e53e3e;
            box-shadow: 0 0 0 3px rgba(229, 62, 62, 0.2);
        }
        .modal-overlay {
            transition: opacity 0.3s ease;
        }
        .modal-content {
            transition: transform 0.3s ease;
        }
        .slider-wrapper {
            display: none;
            margin-top: 1rem;
            padding-left: 1.5rem;
        }
        .slider-wrapper.visible {
            display: block;
        }
        .slider-container {
            display: flex;
            align-items: center;
            gap: 1rem;
        }
        input[type="range"] {
            -webkit-appearance: none;
            appearance: none;
            width: 100%;
            height: 6px;
            background: #e9ecef;
            border-radius: 5px;
            outline: none;
            padding: 0;
            margin: 0;
        }
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 22px;
            height: 22px;
            background: #005A9C;
            cursor: pointer;
            border-radius: 50%;
            border: 3px solid white;
            box-shadow: 0 0 5px rgba(0,0,0,0.2);
            transition: transform 0.2s ease;
        }
        input[type="range"]::-webkit-slider-thumb:hover {
            transform: scale(1.1);
        }
        input[type="range"]::-moz-range-thumb {
            width: 22px;
            height: 22px;
            background: #005A9C;
            cursor: pointer;
            border-radius: 50%;
            border: 3px solid white;
            box-shadow: 0 0 5px rgba(0,0,0,0.2);
        }
        .criterion-section {
            padding: 1.5rem 0;
            border-bottom: 1px solid #e5e7eb;
        }
        .criterion-section:last-child {
            border-bottom: none;
            padding-bottom: 0;
        }
         .criterion-section:first-child {
            padding-top: 0;
        }
        .progress-bar-container {
            background-color: #e9ecef;
            border-radius: 99px;
            padding: 2px;
            margin-bottom: 2rem;
        }
        .progress-bar {
            height: 10px;
            background-color: #005A9C;
            border-radius: 99px;
            width: 0%;
            transition: width 0.4s ease-in-out;
        }
        #toast-notification {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background-color: #28a745;
            color: white;
            padding: 1rem 1.5rem;
            border-radius: 8px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
            transform: translateX(120%);
            transition: transform 0.5s ease;
            z-index: 1001;
        }
        #toast-notification.error {
             background-color: #dc3545;
        }
        #toast-notification.show {
            transform: translateX(0);
        }
        .tooltip {
            position: relative;
            cursor: pointer;
            color: #adb5bd;
            margin-left: 8px;
        }
        .tooltip:before {
            content: attr(data-tooltip);
            position: absolute;
            bottom: 125%;
            left: 50%;
            transform: translateX(-50%);
            background-color: #343a40;
            color: #fff;
            padding: 8px 12px;
            border-radius: 6px;
            font-size: 0.85rem;
            white-space: nowrap;
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.3s, visibility 0.3s;
            z-index: 10;
        }
        .tooltip:hover:before {
            opacity: 1;
            visibility: visible;
        }
        input[type="radio"], input[type="checkbox"] {
            accent-color: #005A9C;
        }
    </style>
</head>
<body class="text-gray-800">

    <header class="bg-white text-gray-800 shadow-md sticky top-0 z-50">
        <div class="container mx-auto px-6 py-3 flex justify-between items-center">
            <div class="flex items-center">
                <img src="https://i.imgur.com/8L1sY5b.png" alt="Logo ICETI" class="h-12">
            </div>
            <h1 class="text-xl sm:text-2xl font-medium text-gray-700 text-right">Avaliação de Projetos de Extensão</h1>
        </div>
    </header>

    <main class="container mx-auto px-6 py-8">
        <form id="evaluationForm" action="https://api.sheetmonkey.io/form/c5NVMH1aTBaAihMSwceVVq" method="post">
            
            <div class="card">
                <div class="card-header bg-gray-50 p-4 border-b border-gray-200">
                    <h2 class="text-xl font-semibold text-[#005A9C]"><i class="fas fa-user-tie mr-2"></i> Identificação</h2>
                </div>
                <div class="p-6">
                    <div class="progress-bar-container">
                        <div id="progress-bar" class="progress-bar"></div>
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div class="md:col-span-2">
                            <label for="evaluatorName" class="block text-sm font-medium text-gray-700 mb-1">Avaliador(a):</label>
                            <input type="text" id="evaluatorName" name="Avaliador" required class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm">
                        </div>
                        <div>
                            <label for="proponentName" class="block text-sm font-medium text-gray-700 mb-1">Proponente:</label>
                            <input type="text" id="proponentName" name="Proponente" required class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm">
                        </div>
                         <div>
                            <label for="email" class="block text-sm font-medium text-gray-700 mb-1">E-mail:</label>
                            <input type="email" id="email" name="Email" required class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm">
                        </div>
                        <div class="md:col-span-2">
                            <label for="projectTitle" class="block text-sm font-medium text-gray-700 mb-1">Título:</label>
                            <input type="text" id="projectTitle" name="Titulo_Projeto" required class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm">
                        </div>
                        <div>
                            <label for="campus" class="block text-sm font-medium text-gray-700 mb-1">Campus:</label>
                            <input type="text" id="campus" name="Campus" required class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm">
                        </div>
                        <div>
                            <label for="course" class="block text-sm font-medium text-gray-700 mb-1">Curso:</label>
                            <input type="text" id="course" name="Curso" required class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm">
                        </div>
                        <div class="md:col-span-2">
                            <label for="lattesLink" class="block text-sm font-medium text-gray-700 mb-1">Link para Currículo Lattes:</label>
                            <input type="url" id="lattesLink" name="Link_Lattes" placeholder="https://" required class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm">
                        </div>
                    </div>
                </div>
            </div>

            <div class="card">
                 <div class="p-4 flex justify-between items-center bg-gray-800 text-white rounded-t-lg">
                    <h2 class="text-xl font-semibold"><i class="fas fa-clipboard-list mr-2"></i> Pontuação do Projeto</h2>
                    <div class="text-lg font-bold">Total: <span id="tabela1-total" class="text-yellow-300">0.0</span></div>
                </div>
                <div class="p-6">
                    <!-- Critério 1 -->
                    <div class="criterion-section">
                        <p class="font-semibold mb-3 text-lg text-gray-700 flex items-center">1. Diretrizes da extensão (Máx: 50) <i class="fas fa-info-circle tooltip" data-tooltip="Avalie se o projeto atende às 5 diretrizes do FORPROEXT."></i></p>
                        <div class="space-y-6 pl-4">
                            <div class="sub-criterion" data-criterion="1a">
                                <p>a) Interação dialógica</p>
                                <p class="text-sm text-gray-600 mt-2">Este critério se aplica ao projeto?</p>
                                <div class="flex space-x-4 mt-1">
                                    <label class="flex items-center"><input type="radio" name="Aplica_1a" value="sim" class="mr-2"> Sim</label>
                                    <label class="flex items-center"><input type="radio" name="Aplica_1a" value="nao" checked class="mr-2"> Não</label>
                                </div>
                                <div class="slider-wrapper" id="slider-wrapper-1a">
                                    <label class="text-sm text-gray-600">Avalie de 0 a 10:</label>
                                    <div class="slider-container">
                                        <input type="range" id="slider-1a" min="0" max="10" value="0" class="w-full slider-input">
                                        <span class="font-semibold text-[#005A9C] w-16 text-center" id="slider-value-1a">0 pts</span>
                                    </div>
                                </div>
                            </div>
                            <div class="sub-criterion" data-criterion="1b">
                                <p>b) Interdisciplinaridade</p>
                                <p class="text-sm text-gray-600 mt-2">Este critério se aplica ao projeto?</p>
                                <div class="flex space-x-4 mt-1">
                                    <label class="flex items-center"><input type="radio" name="Aplica_1b" value="sim" class="mr-2"> Sim</label>
                                    <label class="flex items-center"><input type="radio" name="Aplica_1b" value="nao" checked class="mr-2"> Não</label>
                                </div>
                                <div class="slider-wrapper" id="slider-wrapper-1b">
                                    <label class="text-sm text-gray-600">Avalie de 0 a 10:</label>
                                    <div class="slider-container">
                                        <input type="range" id="slider-1b" min="0" max="10" value="0" class="w-full slider-input">
                                        <span class="font-semibold text-[#005A9C] w-16 text-center" id="slider-value-1b">0 pts</span>
                                    </div>
                                </div>
                            </div>
                            <div class="sub-criterion" data-criterion="1c">
                                <p>c) Relação com ensino e pesquisa</p>
                                <p class="text-sm text-gray-600 mt-2">Este critério se aplica ao projeto?</p>
                                <div class="flex space-x-4 mt-1">
                                    <label class="flex items-center"><input type="radio" name="Aplica_1c" value="sim" class="mr-2"> Sim</label>
                                    <label class="flex items-center"><input type="radio" name="Aplica_1c" value="nao" checked class="mr-2"> Não</label>
                                </div>
                                <div class="slider-wrapper" id="slider-wrapper-1c">
                                    <label class="text-sm text-gray-600">Avalie de 0 a 10:</label>
                                    <div class="slider-container">
                                        <input type="range" id="slider-1c" min="0" max="10" value="0" class="w-full slider-input">
                                        <span class="font-semibold text-[#005A9C] w-16 text-center" id="slider-value-1c">0 pts</span>
                                    </div>
                                </div>
                            </div>
                            <div class="sub-criterion" data-criterion="1d">
                                <p>d) Impacto na formação do estudante</p>
                                <p class="text-sm text-gray-600 mt-2">Este critério se aplica ao projeto?</p>
                                <div class="flex space-x-4 mt-1">
                                    <label class="flex items-center"><input type="radio" name="Aplica_1d" value="sim" class="mr-2"> Sim</label>
                                    <label class="flex items-center"><input type="radio" name="Aplica_1d" value="nao" checked class="mr-2"> Não</label>
                                </div>
                                <div class="slider-wrapper" id="slider-wrapper-1d">
                                    <label class="text-sm text-gray-600">Avalie de 0 a 10:</label>
                                    <div class="slider-container">
                                        <input type="range" id="slider-1d" min="0" max="10" value="0" class="w-full slider-input">
                                        <span class="font-semibold text-[#005A9C] w-16 text-center" id="slider-value-1d">0 pts</span>
                                    </div>
                                </div>
                            </div>
                            <div class="sub-criterion" data-criterion="1e">
                                <p>e) Transformação social</p>
                                <p class="text-sm text-gray-600 mt-2">Este critério se aplica ao projeto?</p>
                                <div class="flex space-x-4 mt-1">
                                    <label class="flex items-center"><input type="radio" name="Aplica_1e" value="sim" class="mr-2"> Sim</label>
                                    <label class="flex items-center"><input type="radio" name="Aplica_1e" value="nao" checked class="mr-2"> Não</label>
                                </div>
                                <div class="slider-wrapper" id="slider-wrapper-1e">
                                    <label class="text-sm text-gray-600">Avalie de 0 a 10:</label>
                                    <div class="slider-container">
                                        <input type="range" id="slider-1e" min="0" max="10" value="0" class="w-full slider-input">
                                        <span class="font-semibold text-[#005A9C] w-16 text-center" id="slider-value-1e">0 pts</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <!-- Critério 2 -->
                    <div class="criterion-section">
                        <p class="font-semibold mb-3 text-lg text-gray-700 flex items-center">2. Ações principais (Máx: 40) <i class="fas fa-info-circle tooltip" data-tooltip="Local de realização das atividades principais do projeto."></i></p>
                        <div class="space-y-2 pl-4">
                            <label class="flex items-center"><input type="radio" name="Acoes_Principais" value="40" class="mr-2"> São realizadas na comunidade externa à Unespar (40 pts)</label>
                            <label class="flex items-center"><input type="radio" name="Acoes_Principais" value="10" class="mr-2"> São realizadas nos espaços da Unespar com foco na comunidade externa (10 pts)</label>
                            <label class="flex items-center"><input type="radio" name="Acoes_Principais" value="0" checked class="mr-2"> Nenhuma das opções</label>
                        </div>
                    </div>
                    <!-- Critério 3 -->
                    <div class="criterion-section">
                        <p class="font-semibold mb-3 text-lg text-gray-700 flex items-center">3. Relações com setores sociais (Máx: 15) <i class="fas fa-info-circle tooltip" data-tooltip="Número de setores sociais com os quais o projeto dialoga."></i></p>
                        <div class="space-y-2 pl-4">
                            <label class="flex items-center"><input type="radio" name="Setores_Sociais" value="15" class="mr-2"> Com mais de dois setores (15 pts)</label>
                            <label class="flex items-center"><input type="radio" name="Setores_Sociais" value="10" class="mr-2"> Com dois setores (10 pts)</label>
                            <label class="flex items-center"><input type="radio" name="Setores_Sociais" value="5" class="mr-2"> Com um setor (5 pts)</label>
                            <label class="flex items-center"><input type="radio" name="Setores_Sociais" value="0" checked class="mr-2"> Não identifica setor (0 pts)</label>
                        </div>
                    </div>
                    <!-- Critério 5 -->
                    <div class="criterion-section">
                        <p class="font-semibold mb-3 text-lg text-gray-700 flex items-center">5. Professores orientadores (Máx: 15) <i class="fas fa-info-circle tooltip" data-tooltip="Quantidade de professores envolvidos além do coordenador."></i></p>
                        <div class="space-y-2 pl-4">
                            <label class="flex items-center"><input type="radio" name="Professores_Orientadores" value="15" class="mr-2"> Mais de 03 (15 pts)</label>
                            <label class="flex items-center"><input type="radio" name="Professores_Orientadores" value="10" class="mr-2"> De 01 a 03 (10 pts)</label>
                            <label class="flex items-center"><input type="radio" name="Professores_Orientadores" value="0" checked class="mr-2"> Nenhum</label>
                        </div>
                    </div>
                    <!-- Critério 6 -->
                    <div class="criterion-section">
                        <p class="font-semibold mb-3 text-lg text-gray-700 flex items-center">6. Ações contempladas (Máx: 20) <i class="fas fa-info-circle tooltip" data-tooltip="Selecione as áreas de atuação do projeto. A pontuação é somada."></i></p>
                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-2 pl-4">
                            <label class="flex items-center"><input type="checkbox" name="Acoes_Contempladas" value="Inclusao_Social" class="mr-2"> Inclusão Social (5 pts)</label>
                            <label class="flex items-center"><input type="checkbox" name="Acoes_Contempladas" value="Desenvolvimento_Economico_Social" class="mr-2"> Desenvolvimento Econômico e Social (5 pts)</label>
                            <label class="flex items-center"><input type="checkbox" name="Acoes_Contempladas" value="Meio_Ambiente" class="mr-2"> Defesa e preservação do meio ambiente (5 pts)</label>
                            <label class="flex items-center"><input type="checkbox" name="Acoes_Contempladas" value="Patrimonio_Cultural" class="mr-2"> Defesa da memória e do patrimônio cultural (5 pts)</label>
                        </div>
                    </div>
                    <!-- Critério 7 -->
                    <div class="criterion-section">
                        <p class="font-semibold mb-3 text-lg text-gray-700 flex items-center">7. Ações com grupos sociais específicos (Máx: 10) <i class="fas fa-info-circle tooltip" data-tooltip="Verifique se o projeto atende grupos à margem das ações tradicionais."></i></p>
                        <div class="space-y-2 pl-4">
                            <label class="flex items-center"><input type="radio" name="Grupos_Sociais" value="10" class="mr-2"> Sim (10 pts)</label>
                            <label class="flex items-center"><input type="radio" name="Grupos_Sociais" value="0" checked class="mr-2"> Não (0 pts)</label>
                        </div>
                    </div>
                </div>
            </div>

            <div class="card final-score-card">
                 <div class="p-4 flex justify-between items-center">
                    <h2 class="text-xl font-semibold text-[#004170]">Pontuação Final do Projeto:</h2>
                    <div class="text-2xl font-bold text-[#004170]"><span id="final-score-display">0.0</span></div>
                </div>
            </div>

            <div class="card">
                <div class="card-header bg-gray-50 p-4 border-b border-gray-200">
                    <h2 class="text-xl font-semibold text-[#005A9C]"><i class="fas fa-pen-alt mr-2"></i> Observações / Justificativa Final</h2>
                </div>
                <div class="p-6">
                    <label for="observacoes" class="block text-sm font-medium text-gray-700 mb-1">Forneça uma justificativa para a pontuação atribuída.</label>
                    <textarea id="observacoes" name="Observacoes" rows="6" maxlength="500" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm"></textarea>
                    <div class="text-right text-sm text-gray-500 mt-1"><span id="char-count">0</span> / 500</div>
                </div>
            </div>
            
            <!-- Hidden fields for submission -->
            <input type="hidden" name="Protocolo" id="hidden-protocol-number">
            <input type="hidden" name="Pontuacao_Final" id="hidden-total-score">
            <input type="hidden" name="Pontuacao_1a" id="hidden-score-1a" value="0">
            <input type="hidden" name="Pontuacao_1b" id="hidden-score-1b" value="0">
            <input type="hidden" name="Pontuacao_1c" id="hidden-score-1c" value="0">
            <input type="hidden" name="Pontuacao_1d" id="hidden-score-1d" value="0">
            <input type="hidden" name="Pontuacao_1e" id="hidden-score-1e" value="0">
            <input type="hidden" name="Created" value="x-sheetmonkey-current-date-time" />


            <div class="flex justify-end gap-4 mt-6">
                <button type="button" id="previewBtn" class="btn-secondary text-white font-bold py-2 px-4 rounded-md shadow-lg"><i class="fas fa-eye mr-2"></i> Visualizar Avaliação</button>
                <button type="submit" id="submitBtn" class="btn-primary text-white font-bold py-2 px-4 rounded-md shadow-lg"><i class="fas fa-paper-plane mr-2"></i> Enviar Avaliação</button>
            </div>

        </form>
    </main>

    <footer class="bg-gray-800 text-white mt-12">
        <div class="container mx-auto px-6 py-4 text-center text-sm">
            <p>&copy; 2025 ICETI - Instituto Cesumar de Ciência, Tecnologia e Inovação | Todos os direitos reservados.</p>
        </div>
    </footer>

    <div id="previewModal" class="modal-overlay fixed inset-0 bg-black bg-opacity-60 flex items-center justify-center p-4 opacity-0 invisible z-50">
        <div class="modal-content bg-white rounded-lg shadow-2xl w-full max-w-md p-8 relative transform scale-95">
            <button id="closeModalBtn" class="absolute top-4 right-4 text-gray-400 hover:text-gray-600 text-2xl">&times;</button>
            <h2 class="text-2xl font-bold text-[#005A9C] mb-6">Resumo da Avaliação</h2>
            <div class="space-y-4 text-lg">
                <div class="flex justify-between items-center">
                    <span class="text-gray-600">Pontuação do Projeto:</span>
                    <span id="modal-total-t1" class="font-bold text-gray-800">0.0</span>
                </div>
                <hr class="my-4">
                <div class="flex justify-between items-center text-xl">
                    <span class="font-semibold">PONTUAÇÃO FINAL AVALIADA:</span>
                    <strong id="modal-grand-total" class="text-2xl text-[#005A9C]">0.0</strong>
                </div>
                 <div class="flex justify-between items-center mt-4 pt-4 border-t text-base">
                    <span class="text-gray-600">Nº do Protocolo:</span>
                    <span id="modal-protocol-number" class="font-bold text-gray-800"></span>
                </div>
            </div>
            <div class="mt-8 flex justify-end">
                <button type="button" id="confirmSubmitBtn" class="btn-primary text-white font-bold py-2 px-6 rounded-md shadow-lg"><i class="fas fa-check mr-2"></i> Confirmar e Enviar</button>
            </div>
        </div>
    </div>
    
    <div id="toast-notification"></div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // --- CONFIGURAÇÃO DO EMAILJS ---
            const EMAILJS_SERVICE_ID = 'service_8jtw634';
            const EMAILJS_TEMPLATE_ID = 'template_1idnftc';
            const EMAILJS_PUBLIC_KEY = 'C3k0CDn3Or2_TP7jg';
            
            emailjs.init(EMAILJS_PUBLIC_KEY);

            const form = document.getElementById('evaluationForm');
            const tabela1TotalDisplay = document.getElementById('tabela1-total');
            const finalScoreDisplay = document.getElementById('final-score-display');
            const observacoesTextarea = document.getElementById('observacoes');
            const charCountSpan = document.getElementById('char-count');
            const previewModal = document.getElementById('previewModal');
            const previewBtn = document.getElementById('previewBtn');
            const closeModalBtn = document.getElementById('closeModalBtn');
            const confirmSubmitBtn = document.getElementById('confirmSubmitBtn');
            const modalTotalT1 = document.getElementById('modal-total-t1');
            const modalGrandTotal = document.getElementById('modal-grand-total');
            const progressBar = document.getElementById('progress-bar');
            const requiredFields = Array.from(form.querySelectorAll('[required]'));
            const hiddenTotalScore = document.getElementById('hidden-total-score');
            const modalProtocolNumber = document.getElementById('modal-protocol-number');
            const hiddenProtocolNumber = document.getElementById('hidden-protocol-number');
            let protocolNumber = null;

            function generateProtocolNumber() {
                const date = new Date();
                const year = date.getFullYear();
                const month = String(date.getMonth() + 1).padStart(2, '0');
                const day = String(date.getDate()).padStart(2, '0');
                const randomPart = Math.random().toString(36).substring(2, 6).toUpperCase();
                return `ICETI-${year}${month}${day}-${randomPart}`;
            }

            function showToast(message, isError = false) {
                const toast = document.getElementById('toast-notification');
                toast.textContent = message;
                toast.classList.remove('error');
                if(isError) {
                    toast.classList.add('error');
                }
                toast.classList.add('show');
                setTimeout(() => {
                    toast.classList.remove('show');
                }, 4000);
            }

            function calculateTabela1Score() {
                let total = 0;
                
                let item1Total = 0;
                form.querySelectorAll('.sub-criterion[data-criterion]').forEach(criterion => {
                    const criterionId = criterion.dataset.criterion;
                    const simRadio = form.querySelector(`input[name="Aplica_${criterionId}"][value="sim"]`);
                    if (simRadio && simRadio.checked) {
                        const slider = document.getElementById(`slider-${criterionId}`);
                        item1Total += parseFloat(slider.value);
                    }
                });
                total += Math.min(item1Total, 50);

                form.querySelectorAll('input[name="Acoes_Principais"]:checked, input[name="Setores_Sociais"]:checked, input[name="Professores_Orientadores"]:checked, input[name="Grupos_Sociais"]:checked').forEach(el => total += parseFloat(el.value));
                
                let item6Score = 0;
                form.querySelectorAll('input[name="Acoes_Contempladas"]:checked').forEach(el => item6Score += 5);
                total += Math.min(item6Score, 20);

                return total;
            }

            function updateAllScores() {
                const totalT1 = calculateTabela1Score();
                tabela1TotalDisplay.textContent = totalT1.toFixed(1);
                finalScoreDisplay.textContent = totalT1.toFixed(1);
                hiddenTotalScore.value = totalT1.toFixed(1);
                return { totalT1 };
            }

            function updateCharCounter() {
                charCountSpan.textContent = observacoesTextarea.value.length;
            }

            function updateProgressBar() {
                const filledFields = requiredFields.filter(field => field.value.trim() !== '').length;
                const progress = (filledFields / requiredFields.length) * 100;
                progressBar.style.width = `${progress}%`;
            }

            function validateForm() {
                let isValid = true;
                requiredFields.forEach(field => {
                    field.classList.remove('error');
                    if (!field.value.trim()) {
                        field.classList.add('error');
                        isValid = false;
                    }
                });
                if (!isValid) {
                    showToast('Por favor, preencha todos os campos obrigatórios.', true);
                }
                return isValid;
            }

            function openModal() {
                if (!validateForm()) return;
                
                if (!protocolNumber) {
                    protocolNumber = generateProtocolNumber();
                }
                hiddenProtocolNumber.value = protocolNumber;
                modalProtocolNumber.textContent = protocolNumber;

                const { totalT1 } = updateAllScores();
                modalTotalT1.textContent = totalT1.toFixed(1);
                modalGrandTotal.textContent = totalT1.toFixed(1);
                previewModal.classList.remove('invisible', 'opacity-0');
                previewModal.querySelector('.modal-content').classList.remove('scale-95');
            }

            function closeModal() {
                previewModal.classList.add('opacity-0');
                previewModal.querySelector('.modal-content').classList.add('scale-95');
                setTimeout(() => previewModal.classList.add('invisible'), 300);
            }
            
            function sendConfirmationEmail(data) {
                const templateParams = {
                    protocolo: data.Protocolo,
                    proponente: data.Proponente,
                    titulo_projeto: data.Titulo_Projeto,
                    email_destinatario: data.Email,
                };

                emailjs.send(EMAILJS_SERVICE_ID, EMAILJS_TEMPLATE_ID, templateParams)
                    .then(function(response) {
                       console.log('SUCCESS!', response.status, response.text);
                       showToast(`E-mail de confirmação enviado para ${data.Email}`, false);
                    }, function(error) {
                       console.log('FAILED...', error);
                       showToast('Falha ao enviar e-mail de confirmação.', true);
                    });
            }

            function handleFormSubmit(event) {
                event.preventDefault();
                if (!validateForm()) return;
                
                if (!protocolNumber) {
                    protocolNumber = generateProtocolNumber();
                }
                hiddenProtocolNumber.value = protocolNumber;
                
                updateAllScores(); 
                
                const formData = new FormData(form);
                const dataForEmail = Object.fromEntries(formData.entries());
                
                fetch(form.action, {
                    method: 'POST',
                    body: formData
                })
                .then(response => {
                    if (response.ok) {
                        showToast(`Avaliação enviada! Protocolo: ${protocolNumber}`, false);
                        sendConfirmationEmail(dataForEmail);
                        closeModal();
                        form.reset(); 
                        updateAllScores();
                        updateProgressBar();
                        protocolNumber = null; 
                    } else {
                        showToast('Ocorreu um erro ao enviar para a planilha.', true);
                    }
                })
                .catch(error => {
                    console.error('Error:', error);
                    showToast('Ocorreu um erro de rede ao enviar o formulário.', true);
                });
            }

            form.querySelectorAll('.sub-criterion[data-criterion]').forEach(criterion => {
                const criterionId = criterion.dataset.criterion;
                const radios = form.querySelectorAll(`input[name="Aplica_${criterionId}"]`);
                const sliderWrapper = document.getElementById(`slider-wrapper-${criterionId}`);
                const slider = document.getElementById(`slider-${criterionId}`);
                const sliderValue = document.getElementById(`slider-value-${criterionId}`);
                const hiddenScoreInput = document.getElementById(`hidden-score-${criterionId}`);
                
                radios.forEach(radio => {
                    radio.addEventListener('change', (e) => {
                        if (e.target.value === 'sim') {
                            sliderWrapper.classList.add('visible');
                        } else {
                            sliderWrapper.classList.remove('visible');
                            slider.value = 0;
                            sliderValue.textContent = '0 pts';
                            hiddenScoreInput.value = 0;
                        }
                        updateAllScores();
                    });
                });

                slider.addEventListener('input', (e) => {
                    sliderValue.textContent = `${e.target.value} pts`;
                    hiddenScoreInput.value = e.target.value;
                    updateAllScores();
                });
            });

            form.addEventListener('input', (event) => {
                if (event.target.matches('input[type="number"], textarea, input[type="range"]')) {
                    updateAllScores();
                }
                if (event.target.id === 'observacoes') {
                    updateCharCounter();
                }
                if(event.target.hasAttribute('required')) {
                    updateProgressBar();
                }
            });

             form.addEventListener('change', (event) => {
                if (event.target.matches('input[type="radio"], input[type="checkbox"]')) {
                    updateAllScores();
                }
            });
            
            previewBtn.addEventListener('click', openModal);
            closeModalBtn.addEventListener('click', closeModal);
            confirmSubmitBtn.addEventListener('click', handleFormSubmit);
            form.addEventListener('submit', handleFormSubmit);
            previewModal.addEventListener('click', (event) => {
                if (event.target === previewModal) closeModal();
            });

            updateAllScores();
            updateCharCounter();
            updateProgressBar();
        });
    </script>
</body>
</html>
