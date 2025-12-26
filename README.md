<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JSON to YAML Converter</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Load js-yaml library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/js-yaml/4.1.0/js-yaml.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500&family=Inter:wght@400;500;600&display=swap');
        
        body {
            font-family: 'Inter', sans-serif;
        }
        
        textarea {
            font-family: 'JetBrains Mono', monospace;
            resize: none; 
        }

        /* Custom scrollbar for better aesthetics */
        textarea::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        textarea::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        textarea::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        textarea::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
        
        .dark textarea::-webkit-scrollbar-track {
            background: #1e293b;
        }
        .dark textarea::-webkit-scrollbar-thumb {
            background: #475569;
        }

        /* Error state styling */
        .error-ring {
            box-shadow: 0 0 0 2px #ef4444;
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-900 h-screen flex flex-col overflow-hidden selection:bg-indigo-100 selection:text-indigo-900">

    <!-- Header -->
    <header class="bg-white border-b border-slate-200 px-6 py-4 flex items-center justify-between shadow-sm shrink-0 z-10">
        <div class="flex items-center gap-3">
            <div class="bg-indigo-600 text-white p-2 rounded-lg shadow-sm">
                <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12c0 1.66-4 3-9 3s-9-1.34-9-3"></path><path d="M3 5v14c0 1.66 4 3 9 3s9-1.34 9-3V5"></path></svg>
            </div>
            <div>
                <h1 class="font-bold text-lg tracking-tight text-slate-800">JSON <span class="text-slate-400 mx-1">→</span> YAML</h1>
                <p class="text-xs text-slate-500 font-medium">Instant Conversion</p>
            </div>
        </div>

        <div class="flex items-center gap-3">
             <div id="status-indicator" class="hidden px-3 py-1.5 rounded-full text-xs font-medium bg-green-100 text-green-700 items-center gap-1.5 transition-all">
                <span class="w-2 h-2 rounded-full bg-green-500"></span>
                Valid JSON
            </div>
            <button onclick="clearAll()" class="text-sm font-medium text-slate-500 hover:text-red-600 px-3 py-2 rounded-md hover:bg-red-50 transition-colors">
                Clear
            </button>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1 flex flex-col md:flex-row h-full overflow-hidden relative">
        
        <!-- Input Section -->
        <div class="flex-1 flex flex-col border-b md:border-b-0 md:border-r border-slate-200 h-1/2 md:h-full min-h-0 bg-white">
            <div class="px-4 py-2 bg-slate-50 border-b border-slate-200 flex justify-between items-center shrink-0">
                <label for="json-input" class="text-xs font-semibold text-slate-500 uppercase tracking-wider">Input: JSON</label>
                <button onclick="pasteFromClipboard()" class="text-xs flex items-center gap-1.5 bg-white border border-slate-200 hover:border-indigo-300 hover:text-indigo-600 text-slate-600 px-2 py-1 rounded shadow-sm transition-all">
                    <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path></svg>
                    Paste
                </button>
            </div>
            <div class="relative flex-1 min-h-0 group">
                <textarea 
                    id="json-input" 
                    placeholder='Paste your JSON here...
{
  "example": "data",
  "nested": {
    "values": [1, 2, 3]
  }
}' 
                    class="w-full h-full p-4 bg-white text-sm leading-relaxed outline-none text-slate-700 placeholder:text-slate-300 transition-shadow"
                    spellcheck="false"
                ></textarea>
                <!-- Error Overlay -->
                <div id="error-message" class="absolute bottom-4 left-4 right-4 bg-red-50 text-red-600 text-xs p-3 rounded-md border border-red-100 hidden shadow-lg animate-fade-in z-20">
                    <div class="font-semibold mb-1">Invalid JSON</div>
                    <span id="error-text" class="font-mono opacity-80"></span>
                </div>
            </div>
        </div>

        <!-- Output Section -->
        <div class="flex-1 flex flex-col h-1/2 md:h-full min-h-0 bg-slate-50/50">
            <div class="px-4 py-2 bg-slate-50 border-b border-slate-200 flex justify-between items-center shrink-0">
                <label for="yaml-output" class="text-xs font-semibold text-slate-500 uppercase tracking-wider">Output: YAML</label>
                <button onclick="copyToClipboard()" id="copy-btn" class="text-xs flex items-center gap-1.5 bg-indigo-600 hover:bg-indigo-700 text-white px-3 py-1 rounded shadow-sm transition-all transform active:scale-95">
                    <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path></svg>
                    Copy YAML
                </button>
            </div>
            <div class="flex-1 min-h-0 relative">
                <textarea 
                    id="yaml-output" 
                    readonly 
                    placeholder="YAML output will appear here..." 
                    class="w-full h-full p-4 bg-slate-50 text-sm leading-relaxed outline-none text-indigo-900 placeholder:text-slate-300"
                    spellcheck="false"
                ></textarea>
            </div>
        </div>
    </main>

    <script>
        // DOM Elements
        const jsonInput = document.getElementById('json-input');
        const yamlOutput = document.getElementById('yaml-output');
        const errorMessage = document.getElementById('error-message');
        const errorText = document.getElementById('error-text');
        const statusIndicator = document.getElementById('status-indicator');
        const copyBtn = document.getElementById('copy-btn');

        // Conversion Logic
        function convert() {
            const jsonValue = jsonInput.value.trim();

            // Clear output if empty
            if (!jsonValue) {
                yamlOutput.value = '';
                hideError();
                hideStatus();
                return;
            }

            try {
                // 1. Parse JSON
                const parsedObject = JSON.parse(jsonValue);

                // 2. Dump to YAML using js-yaml library
                // indent: 2 spaces, noCompatMode ensures cleaner output
                const yamlString = jsyaml.dump(parsedObject, {
                    indent: 2,
                    lineWidth: -1, // Don't wrap long lines
                    noRefs: true   // Don't use anchors/aliases if possible
                });

                // 3. Update Output
                yamlOutput.value = yamlString;
                
                // 4. Update UI State
                hideError();
                showSuccess();

            } catch (e) {
                // Handle Errors
                showError(e.message);
                // Keep the old output or clear it? 
                // Usually better to keep it so user can see last valid state, 
                // OR clear it to indicate broken state. 
                // Let's grey it out visually via CSS but not clear text? 
                // For simplicity, we'll leave the last valid output but show big error.
            }
        }

        // UI Helper Functions
        function showError(msg) {
            errorMessage.classList.remove('hidden');
            errorText.textContent = msg;
            jsonInput.parentElement.classList.add('error-ring');
            
            // Status pill
            statusIndicator.classList.remove('hidden', 'bg-green-100', 'text-green-700');
            statusIndicator.classList.add('bg-red-100', 'text-red-700');
            statusIndicator.innerHTML = '<span class="w-2 h-2 rounded-full bg-red-500"></span> Invalid JSON';
        }

        function hideError() {
            errorMessage.classList.add('hidden');
            jsonInput.parentElement.classList.remove('error-ring');
        }

        function showSuccess() {
            statusIndicator.classList.remove('hidden', 'bg-red-100', 'text-red-700');
            statusIndicator.classList.add('bg-green-100', 'text-green-700');
            statusIndicator.innerHTML = '<span class="w-2 h-2 rounded-full bg-green-500"></span> Valid JSON';
        }

        function hideStatus() {
            statusIndicator.classList.add('hidden');
        }

        function clearAll() {
            jsonInput.value = '';
            yamlOutput.value = '';
            hideError();
            hideStatus();
            jsonInput.focus();
        }

        async function pasteFromClipboard() {
            try {
                const text = await navigator.clipboard.readText();
                jsonInput.value = text;
                convert(); // Trigger conversion immediately
            } catch (err) {
                // Fallback or permission denied
                console.error('Failed to read clipboard contents: ', err);
                // Simple alert fallback since we are in an iframe usually
                alert("Could not access clipboard. Please press Ctrl+V / Cmd+V to paste.");
                jsonInput.focus();
            }
        }

        function copyToClipboard() {
            if (!yamlOutput.value) return;
            
            yamlOutput.select();
            document.execCommand('copy');
            
            // Visual feedback
            const originalText = copyBtn.innerHTML;
            copyBtn.innerHTML = `
                <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"></polyline></svg>
                Copied!
            `;
            copyBtn.classList.add('bg-green-600', 'hover:bg-green-700');
            copyBtn.classList.remove('bg-indigo-600', 'hover:bg-indigo-700');

            setTimeout(() => {
                copyBtn.innerHTML = originalText;
                copyBtn.classList.remove('bg-green-600', 'hover:bg-green-700');
                copyBtn.classList.add('bg-indigo-600', 'hover:bg-indigo-700');
            }, 2000);
        }

        // Event Listeners
        // Debounce input to prevent flashing on every keystroke
        let debounceTimer;
        jsonInput.addEventListener('input', () => {
            clearTimeout(debounceTimer);
            debounceTimer = setTimeout(convert, 300);
        });

        // Initialize
        jsonInput.focus();

    </script>
</body>
</html>
