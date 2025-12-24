!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EchoScript - Live Speech to Text</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc;
        }

        .visualizer-bar {
            transition: height 0.1s ease;
        }

        .pulse {
            animation: pulse-animation 2s infinite;
        }

        @keyframes pulse-animation {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.05); opacity: 0.8; }
            100% { transform: scale(1); opacity: 1; }
        }

        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: transparent;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 10px;
        }
    </style>
</head>
<body class="min-h-screen text-slate-800">

    <div class="max-w-5xl mx-auto px-4 py-8 md:py-12">
        <!-- Header -->
        <header class="text-center mb-10">
            <div class="inline-flex items-center justify-center w-16 h-16 bg-blue-600 text-white rounded-2xl shadow-lg mb-4">
                <i class="fas fa-microphone-alt text-2xl"></i>
            </div>
            <h1 class="text-4xl font-bold tracking-tight text-slate-900">EchoScript</h1>
            <p class="text-slate-500 mt-2 text-lg">Transform your voice into text instantly.</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            
            <!-- Sidebar / Controls -->
            <div class="lg:col-span-1 space-y-6">
                <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-100">
                    <h2 class="text-sm font-semibold uppercase tracking-wider text-slate-400 mb-4">Settings</h2>
                    
                    <div class="space-y-4">
                        <div>
                            <label class="block text-sm font-medium text-slate-700 mb-1">Language</label>
                            <select id="languageSelect" class="w-full bg-slate-50 border border-slate-200 rounded-xl px-4 py-2.5 outline-none focus:ring-2 focus:ring-blue-500 transition-all">
                                <option value="en-US">English (US)</option>
                                <option value="en-GB">English (UK)</option>
                                <option value="es-ES">Spanish</option>
                                <option value="fr-FR">French</option>
                                <option value="de-DE">German</option>
                                <option value="zh-CN">Chinese (Mandarin)</option>
                                <option value="ja-JP">Japanese</option>
                                <option value="pt-BR">Portuguese</option>
                            </select>
                        </div>

                        <div class="flex items-center justify-between p-3 bg-slate-50 rounded-xl">
                            <span class="text-sm font-medium text-slate-700">Auto-scroll</span>
                            <label class="relative inline-flex items-center cursor-pointer">
                                <input type="checkbox" id="autoScrollToggle" class="sr-only peer" checked>
                                <div class="w-11 h-6 bg-slate-200 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"></div>
                            </label>
                        </div>
                    </div>
                </div>

                <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-100 h-64 flex flex-col">
                    <h2 class="text-sm font-semibold uppercase tracking-wider text-slate-400 mb-4">History</h2>
                    <div id="historyList" class="flex-grow overflow-y-auto custom-scrollbar space-y-2 text-sm text-slate-500 italic">
                        No previous recordings in this session.
                    </div>
                </div>
            </div>

            <!-- Main Editor -->
            <div class="lg:col-span-2 space-y-6">
                <!-- Visualizer Area -->
                <div id="visualizerContainer" class="hidden bg-blue-50 rounded-2xl p-4 flex items-center justify-center gap-1 h-16 mb-4">
                    <div class="visualizer-bar w-1 bg-blue-400 rounded-full h-2"></div>
                    <div class="visualizer-bar w-1 bg-blue-500 rounded-full h-4"></div>
                    <div class="visualizer-bar w-1 bg-blue-600 rounded-full h-8"></div>
                    <div class="visualizer-bar w-1 bg-blue-500 rounded-full h-4"></div>
                    <div class="visualizer-bar w-1 bg-blue-400 rounded-full h-2"></div>
                </div>

                <div class="relative bg-white rounded-3xl shadow-sm border border-slate-100 overflow-hidden">
                    <div id="statusBadge" class="absolute top-4 right-4 flex items-center gap-2 px-3 py-1 bg-slate-100 text-slate-500 rounded-full text-xs font-bold uppercase tracking-widest">
                        <span class="w-2 h-2 rounded-full bg-slate-400"></span> Ready
                    </div>

                    <textarea id="transcriptArea" 
                        class="w-full h-96 p-8 pt-12 text-lg leading-relaxed bg-transparent outline-none resize-none custom-scrollbar" 
                        placeholder="Click 'Start Listening' and speak clearly..."></textarea>
                    
                    <div class="p-4 bg-slate-50 border-t border-slate-100 flex flex-wrap gap-3">
                        <button id="copyBtn" class="flex items-center gap-2 px-4 py-2 bg-white border border-slate-200 rounded-xl text-slate-600 hover:bg-slate-100 transition-colors text-sm font-medium">
                            <i class="far fa-copy"></i> Copy
                        </button>
                        <button id="downloadBtn" class="flex items-center gap-2 px-4 py-2 bg-white border border-slate-200 rounded-xl text-slate-600 hover:bg-slate-100 transition-colors text-sm font-medium">
                            <i class="fas fa-download"></i> Download .txt
                        </button>
                        <button id="clearBtn" class="flex items-center gap-2 px-4 py-2 bg-white border border-slate-200 rounded-xl text-red-500 hover:bg-red-50 transition-colors text-sm font-medium">
                            <i class="far fa-trash-alt"></i> Clear
                        </button>
                    </div>
                </div>

                <!-- Primary Action Button -->
                <button id="mainToggleBtn" class="w-full py-5 bg-blue-600 hover:bg-blue-700 text-white rounded-3xl shadow-xl shadow-blue-200 flex items-center justify-center gap-3 text-xl font-bold transition-all transform active:scale-95">
                    <i id="mainBtnIcon" class="fas fa-microphone"></i>
                    <span id="mainBtnText">Start Listening</span>
                </button>
            </div>
        </div>

        <footer class="mt-16 text-center text-slate-400 text-sm">
            <p>Requires a modern browser with Web Speech API support (Chrome, Edge, Safari).</p>
        </footer>
    </div>

    <!-- Notification Toast -->
    <div id="toast" class="fixed bottom-8 left-1/2 -translate-x-1/2 px-6 py-3 bg-slate-900 text-white rounded-full shadow-2xl opacity-0 pointer-events-none transition-all duration-300 translate-y-4">
        Text copied to clipboard!
    </div>

    <script>
        // DOM Elements
        const mainToggleBtn = document.getElementById('mainToggleBtn');
        const mainBtnIcon = document.getElementById('mainBtnIcon');
        const mainBtnText = document.getElementById('mainBtnText');
        const transcriptArea = document.getElementById('transcriptArea');
        const statusBadge = document.getElementById('statusBadge');
        const visualizerContainer = document.getElementById('visualizerContainer');
        const visualizerBars = document.querySelectorAll('.visualizer-bar');
        const languageSelect = document.getElementById('languageSelect');
        const historyList = document.getElementById('historyList');
        const copyBtn = document.getElementById('copyBtn');
        const downloadBtn = document.getElementById('downloadBtn');
        const clearBtn = document.getElementById('clearBtn');
        const toast = document.getElementById('toast');
        const autoScrollToggle = document.getElementById('autoScrollToggle');

        // State variables
        let isRecording = false;
        let recognition = null;
        let finalTranscript = '';
        let audioContext, analyser, dataArray, source;

        // Initialize Speech Recognition
        function initRecognition() {
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            
            if (!SpeechRecognition) {
                alert("Sorry, your browser doesn't support speech recognition.");
                mainToggleBtn.disabled = true;
                return false;
            }

            recognition = new SpeechRecognition();
            recognition.continuous = true;
            recognition.interimResults = true;
            recognition.lang = languageSelect.value;

            recognition.onstart = () => {
                isRecording = true;
                updateUI(true);
                startVisualizer();
            };

            recognition.onerror = (event) => {
                console.error("Speech recognition error", event.error);
                stopRecording();
                showToast("Error: " + event.error);
            };

            recognition.onend = () => {
                if (isRecording) {
                    recognition.start();
                } else {
                    updateUI(false);
                    stopVisualizer();
                }
            };

            recognition.onresult = (event) => {
                let interimTranscript = '';
                for (let i = event.resultIndex; i < event.results.length; ++i) {
                    if (event.results[i].isFinal) {
                        finalTranscript += event.results[i][0].transcript + ' ';
                    } else {
                        interimTranscript += event.results[i][0].transcript;
                    }
                }
                
                transcriptArea.value = finalTranscript + interimTranscript;
                
                if (autoScrollToggle.checked) {
                    transcriptArea.scrollTop = transcriptArea.scrollHeight;
                }
            };

            return true;
        }

        // Toggle Recording
        function toggleRecording() {
            if (!recognition && !initRecognition()) return;

            if (isRecording) {
                stopRecording();
            } else {
                startRecording();
            }
        }

        function startRecording() {
            recognition.lang = languageSelect.value;
            recognition.start();
        }

        function stopRecording() {
            isRecording = false;
            recognition.stop();
            if (transcriptArea.value.trim().length > 0) {
                addToHistory(transcriptArea.value.substring(0, 50) + "...");
            }
        }

        function updateUI(recording) {
            if (recording) {
                mainToggleBtn.classList.replace('bg-blue-600', 'bg-red-500');
                mainToggleBtn.classList.replace('hover:bg-blue-700', 'hover:bg-red-600');
                mainBtnIcon.classList.replace('fa-microphone', 'fa-stop');
                mainBtnText.textContent = "Stop Recording";
                statusBadge.innerHTML = '<span class="w-2 h-2 rounded-full bg-red-500 pulse"></span> Listening...';
                statusBadge.classList.replace('text-slate-500', 'text-red-500');
                visualizerContainer.classList.remove('hidden');
                languageSelect.disabled = true;
            } else {
                mainToggleBtn.classList.replace('bg-red-500', 'bg-blue-600');
                mainToggleBtn.classList.replace('hover:bg-red-600', 'hover:bg-blue-700');
                mainBtnIcon.classList.replace('fa-stop', 'fa-microphone');
                mainBtnText.textContent = "Start Listening";
                statusBadge.innerHTML = '<span class="w-2 h-2 rounded-full bg-slate-400"></span> Ready';
                statusBadge.classList.replace('text-red-500', 'text-slate-500');
                visualizerContainer.classList.add('hidden');
                languageSelect.disabled = false;
            }
        }

        async function startVisualizer() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                audioContext = new (window.AudioContext || window.webkitAudioContext)();
                source = audioContext.createMediaStreamSource(stream);
                analyser = audioContext.createAnalyser();
                analyser.fftSize = 32;
                source.connect(analyser);
                dataArray = new Uint8Array(analyser.frequencyBinCount);
                
                function draw() {
                    if (!isRecording) return;
                    requestAnimationFrame(draw);
                    analyser.getByteFrequencyData(dataArray);
                    
                    visualizerBars.forEach((bar, index) => {
                        const val = dataArray[index % dataArray.length];
                        const height = Math.max(8, (val / 255) * 64);
                        bar.style.height = `${height}px`;
                    });
                }
                draw();
            } catch (err) {
                console.warn("Visualizer failed:", err);
            }
        }

        function stopVisualizer() {
            if (audioContext) {
                audioContext.close();
            }
        }

        function addToHistory(snippet) {
            if (historyList.innerText.includes("No previous recordings")) {
                historyList.innerHTML = "";
            }
            const div = document.createElement('div');
            div.className = "p-3 bg-slate-50 rounded-xl hover:bg-slate-100 cursor-pointer transition-colors border border-transparent hover:border-slate-200 group flex justify-between items-center";
            const time = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
            div.innerHTML = `
                <div class="truncate flex-grow mr-2">
                    <span class="text-[10px] block font-bold text-slate-400">${time}</span>
                    <span class="text-slate-700">${snippet}</span>
                </div>
                <i class="fas fa-chevron-right text-slate-300 opacity-0 group-hover:opacity-100 transition-opacity"></i>
            `;
            historyList.prepend(div);
        }

        copyBtn.onclick = () => {
            const text = transcriptArea.value;
            if (!text) return;
            const textArea = document.createElement("textarea");
            textArea.value = text;
            document.body.appendChild(textArea);
            textArea.select();
            document.execCommand('copy');
            document.body.removeChild(textArea);
            showToast("Copied to clipboard!");
        };

        downloadBtn.onclick = () => {
            const text = transcriptArea.value;
            if (!text) return;
            const blob = new Blob([text], { type: 'text/plain' });
            const url = window.URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `echoscript-${new Date().getTime()}.txt`;
            a.click();
            window.URL.revokeObjectURL(url);
        };

        clearBtn.onclick = () => {
            if (confirm("Are you sure you want to clear the transcript?")) {
                transcriptArea.value = "";
                finalTranscript = "";
            }
        };

        function showToast(message) {
            toast.textContent = message;
            toast.classList.remove('opacity-0', 'pointer-events-none', 'translate-y-4');
            setTimeout(() => {
                toast.classList.add('opacity-0', 'pointer-events-none', 'translate-y-4');
            }, 2500);
        }

        mainToggleBtn.onclick = toggleRecording;
        languageSelect.onchange = () => {
            if (isRecording) {
                stopRecording();
                setTimeout(startRecording, 300);
            }
        };

        window.onload = () => {
            if (!(window.SpeechRecognition || window.webkitSpeechRecognition)) {
                transcriptArea.placeholder = "Your browser does not support Speech Recognition. Please use Chrome, Edge or Safari.";
                mainToggleBtn.classList.add('opacity-50', 'cursor-not-allowed');
                mainToggleBtn.disabled = true;
            }
        };
    </script>
</body>
</html>
