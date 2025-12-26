<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JSON to YAML Converter</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- JS-YAML Library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/js-yaml/4.1.0/js-yaml.min.js"></script>
    <!-- Font Awesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500&family=Inter:wght@400;500;600;700&display=swap');

        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }

        .editor {
            font-family: 'JetBrains Mono', monospace;
            font-size: 14px;
            line-height: 1.5;
        }

        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
        }
        ::-webkit-scrollbar-thumb {
            background: #c1c1c1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #a8a8a8;
        }

        .pane-header {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(4px);
        }
    </style>
</head>
<body class="h-screen flex flex-col overflow-hidden text-gray-800 bg-gray-50">

    <!-- Navbar -->
    <nav class="bg-white border-b border-gray-200 px-6 py-4 flex items-center justify-between shadow-sm z-10 shrink-0">
        <div class="flex items-center gap-3">
            <div class="bg-indigo-600 text-white p-2 rounded-lg">
                <i class="fa-solid fa-code-branch"></i>
            </div>
            <h1 class="text-xl font-bold text-gray-800 tracking-tight">JSON <span class="text-gray-400 mx-1">→</span> YAML</h1>
        </div>
        <div class="flex items-center gap-3">
            <button id="sampleBtn" class="text-sm font-medium text-gray-500 hover:text-indigo-600 px-3 py-2 transition-colors">
                Load Sample
            </button>
            <button id="clearBtn" class="text-sm font-medium text-red-500 hover:text-red-700 px-3 py-2 transition-colors">
                Clear All
            </button>
        </div>
    </nav>

    <!-- Main Content -->
    <main class="flex-1 flex flex-col md:flex-row h-full overflow-hidden relative">
        
        <!-- JSON Input Section -->
        <section class="flex-1 flex flex-col border-b md:border-b-0 md:border-r border-gray-200 h-1/2 md:h-full min-h-0 bg-white relative group">
            <div class="pane-header px-4 py-3 border-b border-gray-100 flex justify-between items-center sticky top-0 z-10">
                <div class="flex items-center gap-2">
                    <span class="w-2 h-2 rounded-full bg-yellow-400"></span>
                    <h2 class="font-semibold text-sm text-gray-600 uppercase tracking-wider">Input (JSON)</h2>
                </div>
                <div id="jsonStatus" class="text-xs font-medium px-2 py-1 rounded bg-gray-100 text-gray-500 transition-colors">
                    Waiting for input...
                </div>
            </div>
            <div class="relative flex-1">
                <textarea id="jsonInput" 
                    class="editor w-full h-full resize-none p-4 focus:outline-none text-gray-700 bg-transparent"
                    placeholder='Paste your JSON here...&#10;{&#10;  "key": "value"&#10;}' 
                    spellcheck="false"></textarea>
                
                <!-- Format Button overlay -->
                <button id="formatJsonBtn" class="absolute bottom-4 right-4 bg-white shadow-md border border-gray-200 text-gray-600 hover:text-indigo-600 p-2 rounded-md opacity-0 group-hover:opacity-100 transition-all duration-200" title="Prettify JSON">
                    <i class="fa-solid fa-align-left"></i>
                </button>
            </div>
        </section>

        <!-- Action Bar (Mobile Only / Center Strip) -->
        <div class="hidden md:flex flex-col justify-center items-center w-12 bg-gray-50 border-r border-gray-200 gap-4 z-20 shrink-0">
            <div class="h-full w-[1px] bg-gray-200 absolute left-1/2 -translate-x-1/2 -z-10"></div>
            <button id="convertBtn" class="w-8 h-8 rounded-full bg-indigo-600 text-white flex items-center justify-center shadow-lg hover:scale-110 hover:bg-indigo-700 transition-all z-20" title="Convert Now">
                <i class="fa-solid fa-arrow-right"></i>
            </button>
        </div>

        <!-- YAML Output Section -->
        <section class="flex-1 flex flex-col h-1/2 md:h-full min-h-0 bg-gray-50/50 relative">
            <div class="pane-header px-4 py-3 border-b border-gray-200 flex justify-between items-center sticky top-0 z-10 bg-gray-50">
                <div class="flex items-center gap-2">
                    <span class="w-2 h-2 rounded-full bg-green-400"></span>
                    <h2 class="font-semibold text-sm text-gray-600 uppercase tracking-wider">Output (YAML)</h2>
                </div>
                <button id="copyBtn" class="flex items-center gap-2 text-xs font-medium bg-white border border-gray-200 hover:border-indigo-300 hover:text-indigo-600 px-3 py-1.5 rounded transition-all shadow-sm">
                    <i class="fa-regular fa-copy"></i>
                    <span>Copy</span>
                </button>
            </div>
            <textarea id="yamlOutput" 
                class="editor w-full h-full resize-none p-4 focus:outline-none text-gray-800 bg-transparent"
                readonly
                placeholder="YAML output will appear here..."></textarea>
            
            <!-- Toast Notification -->
            <div id="toast" class="absolute bottom-4 left-1/2 -translate-x-1/2 translate-y-20 bg-gray-800 text-white text-sm px-4 py-2 rounded-lg shadow-xl transition-transform duration-300 flex items-center gap-2">
                <i class="fa-solid fa-circle-check text-green-400"></i>
                <span id="toastMsg">Copied to clipboard!</span>
            </div>
        </section>
    </main>

    <script>
        // DOM Elements
        const jsonInput = document.getElementById('jsonInput');
        const yamlOutput = document.getElementById('yamlOutput');
        const jsonStatus = document.getElementById('jsonStatus');
        const copyBtn = document.getElementById('copyBtn');
        const clearBtn = document.getElementById('clearBtn');
        const sampleBtn = document.getElementById('sampleBtn');
        const formatJsonBtn = document.getElementById('formatJsonBtn');
        const convertBtn = document.getElementById('convertBtn'); // Only visible on desktop
        const toast = document.getElementById('toast');
        const toastMsg = document.getElementById('toastMsg');

        // State
        let debounceTimer;

        // Sample Data
        const sampleData = {
            "project": "SuperApp",
            "version": 1.0,
            "features": [
                { "name": "Login", "enabled": true },
                { "name": "Dark Mode", "enabled": false }
            ],
            "settings": {
                "timeout": 5000,
                "retries": 3,
                "environment": "production"
            },
            "owner": null
        };

        // Initialize
        function init() {
            // Focus input on load
            jsonInput.focus();
        }

        // --- Core Logic ---

        function convertToYaml() {
            const rawJson = jsonInput.value.trim();

            if (!rawJson) {
                yamlOutput.value = '';
                setStatus('waiting', 'Waiting for input...');
                return;
            }

            try {
                // Parse JSON
                const parsedObj = JSON.parse(rawJson);
                
                // Convert to YAML using js-yaml library
                // noRefs: true prevents using anchors/aliases for duplicate objects
                const yamlStr = jsyaml.dump(parsedObj, { 
                    indent: 2, 
                    lineWidth: -1, // Don't break long lines
                    noRefs: true 
                });

                yamlOutput.value = yamlStr;
                setStatus('success', 'Valid JSON');
                
                // Visual feedback on the convert button if clicked
                if(convertBtn) {
                    convertBtn.classList.add('rotate-180');
                    setTimeout(() => convertBtn.classList.remove('rotate-180'), 300);
                }

            } catch (e) {
                // Handle Errors
                yamlOutput.value = ''; // Clear output on error? Or keep previous? Let's clear to indicate sync loss.
                // Actually, let's show the error in the output for better debugging if it's a syntax error
                
                let errorMsg = "Invalid JSON:\n" + e.message;
                setStatus('error', 'Syntax Error');
                
                // We don't overwrite the output with the error message text to avoid confusion, 
                // but we could use a placeholder or a separate error div.
                // For this design, let's keep the old output but flash the status red.
            }
        }

        function setStatus(type, message) {
            jsonStatus.textContent = message;
            jsonStatus.className = 'text-xs font-medium px-2 py-1 rounded transition-colors duration-200 ';

            if (type === 'success') {
                jsonStatus.classList.add('bg-green-100', 'text-green-700');
                jsonInput.classList.remove('bg-red-50');
            } else if (type === 'error') {
                jsonStatus.classList.add('bg-red-100', 'text-red-700');
                jsonInput.classList.add('bg-red-50');
            } else {
                jsonStatus.classList.add('bg-gray-100', 'text-gray-500');
                jsonInput.classList.remove('bg-red-50');
            }
        }

        function formatJson() {
            const raw = jsonInput.value.trim();
            if(!raw) return;
            try {
                const obj = JSON.parse(raw);
                jsonInput.value = JSON.stringify(obj, null, 2);
                convertToYaml(); // Re-trigger conversion
            } catch(e) {
                setStatus('error', 'Cannot format invalid JSON');
            }
        }

        function showToast(message, isError = false) {
            toastMsg.innerText = message;
            if(isError) {
                toast.querySelector('i').className = 'fa-solid fa-circle-exclamation text-red-400';
            } else {
                toast.querySelector('i').className = 'fa-solid fa-circle-check text-green-400';
            }
            
            toast.classList.remove('translate-y-20');
            setTimeout(() => {
                toast.classList.add('translate-y-20');
            }, 3000);
        }

        // --- Event Listeners ---

        // Auto-convert with debounce
        jsonInput.addEventListener('input', () => {
            clearTimeout(debounceTimer);
            debounceTimer = setTimeout(convertToYaml, 500); // 500ms delay
        });

        // Convert Button (Desktop center button)
        if(convertBtn) {
            convertBtn.addEventListener('click', convertToYaml);
        }

        // Format Button
        formatJsonBtn.addEventListener('click', formatJson);

        // Copy Button
        copyBtn.addEventListener('click', () => {
            const content = yamlOutput.value;
            if (!content) {
                showToast("Nothing to copy", true);
                return;
            }
            
            // Fallback for iframe environments if navigator.clipboard fails
            const textArea = document.createElement("textarea");
            textArea.value = content;
            document.body.appendChild(textArea);
            textArea.select();
            try {
                document.execCommand('copy');
                showToast("YAML copied to clipboard!");
            } catch (err) {
                showToast("Failed to copy", true);
            }
            document.body.removeChild(textArea);
        });

        // Clear Button
        clearBtn.addEventListener('click', () => {
            jsonInput.value = '';
            yamlOutput.value = '';
            setStatus('waiting', 'Waiting for input...');
            jsonInput.focus();
        });

        // Sample Button
        sampleBtn.addEventListener('click', () => {
            jsonInput.value = JSON.stringify(sampleData, null, 2);
            convertToYaml();
        });

        // Run Init
        init();

    </script>
</body>
</html>
