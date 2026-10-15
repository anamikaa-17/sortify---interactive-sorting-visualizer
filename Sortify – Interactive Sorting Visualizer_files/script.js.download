/**
 * script.js
 * Refactored into an Object-Oriented design using ES6 classes.
 * Contains MainVisualizer (the main controller) and MainVisualizerContext (the adapter).
 */

class MainVisualizerContext {
    constructor(visualizer) {
        this.visualizer = visualizer;
    }

    size() {
        return this.visualizer.array.length;
    }

    async getValue(i) {
        return this.visualizer.array[i];
    }

    updateStats(c, s) {
        this.visualizer.updateStats(c, s);
    }

    setClass(i, className) {
        this.visualizer.setClass(i, className);
    }

    async swapNodes(i, j) {
        await this.visualizer.swapNodes(i, j);
    }

    async updateHeight(i, val) {
        await this.visualizer.updateHeight(i, val);
    }

    async sleep() {
        await this.visualizer.sleep();
    }

    isAborted() {
        return this.visualizer.isStopped;
    }
}

class MainVisualizer {
    constructor() {
        // UI Elements
        this.container = document.getElementById("array-container");
        this.sizeSlider = document.getElementById("size-slider");
        this.speedSlider = document.getElementById("speed-slider");
        
        this.generateBtn = document.getElementById("generate-btn");
        this.stopBtn = document.getElementById("stop-btn");
        this.manualBtn = document.getElementById("manual-btn");
        this.customInput = document.getElementById("custom-input");
        this.loadCustomBtn = document.getElementById("load-custom-btn");
        
        this.algoSelect = document.getElementById("algo-select");
        
        this.compCountEl = document.getElementById("comp-count");
        this.swapCountEl = document.getElementById("swap-count");
        this.sizeCountEl = document.getElementById("size-count");
        this.timerStat = document.getElementById("timer-stat");
        this.timerCount = document.getElementById("timer-count");
        this.arrayTextEl = document.getElementById("array-text");

        this.prevBtn = document.getElementById("prev-btn");
        this.playPauseBtn = document.getElementById("play-pause-btn");
        this.nextBtn = document.getElementById("next-btn");

        // State Variables
        this.array = [];
        this.isSorting = false;
        this.isStopped = false;
        this.isManualMode = false;
        
        this.comparisons = 0;
        this.swaps = 0;
        this.draggedIndex = null;
        
        this.customMinVal = null;
        this.customMaxVal = null;

        this.timerInterval = null;
        this.startTime = 0;

        // Frame Capture Variables
        this.isCapturing = false;
        this.frames = [];
        this.currentFrame = 0;
        this.playTimeout = null;
        this.virtualHeights = [];
        this.virtualClasses = [];
        this.isPlaying = false;

        this.chartInstance = null;

        this.complexities = {
            bubble: { algo: "Bubble Sort", best: "O(n)", avg: "O(n²)", worst: "O(n²)", space: "O(1)" },
            selection: { algo: "Selection Sort", best: "O(n²)", avg: "O(n²)", worst: "O(n²)", space: "O(1)" },
            insertion: { algo: "Insertion Sort", best: "O(n)", avg: "O(n²)", worst: "O(n²)", space: "O(1)" },
            merge: { algo: "Merge Sort", best: "O(n log n)", avg: "O(n log n)", worst: "O(n log n)", space: "O(n)" },
            quick: { algo: "Quick Sort", best: "O(n log n)", avg: "O(n log n)", worst: "O(n²)", space: "O(log n)" },
            heap: { algo: "Heap Sort", best: "O(n log n)", avg: "O(n log n)", worst: "O(n log n)", space: "O(1)" },
            shell: { algo: "Shell Sort", best: "O(n log n)", avg: "O(n(log n)²)", worst: "O(n(log n)²)", space: "O(1)" },
            cocktail: { algo: "Cocktail Sort", best: "O(n)", avg: "O(n²)", worst: "O(n²)", space: "O(1)" }
        };

        this.init();
    }

    // Initialization
    init() {
        this.generateArray();
        this.updateComplexityInfo();
        this.bindEvents();
    }

    bindEvents() {
        this.generateBtn.addEventListener("click", () => this.generateArray());
        this.sizeSlider.addEventListener("input", () => this.generateArray());
        this.algoSelect.addEventListener("change", () => this.updateComplexityInfo());
        
        this.stopBtn.addEventListener("click", () => {
            this.isStopped = true;
            this.isPlaying = false;
            clearTimeout(this.playTimeout);
            this.finishPlayback();
        });

        if (this.playPauseBtn) {
            this.playPauseBtn.addEventListener("click", () => this.handlePlayPause());
        }

        if (this.prevBtn) {
            this.prevBtn.addEventListener("click", () => this.handlePrevFrame());
        }

        if (this.nextBtn) {
            this.nextBtn.addEventListener("click", () => this.handleNextFrame());
        }

        this.manualBtn.addEventListener("click", () => this.toggleManualMode());
        this.loadCustomBtn.addEventListener("click", () => this.handleCustomLoad());
    }

    setPlayPauseState(state) {
        if (!this.playPauseBtn) return;
        if (state === 'idle') {
            this.playPauseBtn.innerText = '\u25b6 Start Sort';
            this.playPauseBtn.classList.add('btn-primary');
            this.playPauseBtn.style.fontWeight = 'bold';
            this.playPauseBtn.disabled = false;
        } else if (state === 'sorting') {
            this.playPauseBtn.innerText = '\u23f3 Sorting...';
            this.playPauseBtn.classList.remove('btn-primary');
            this.playPauseBtn.disabled = true;
        } else if (state === 'playing') {
            this.playPauseBtn.innerText = '\u23f8 Pause';
            this.playPauseBtn.classList.add('btn-primary');
            this.playPauseBtn.disabled = false;
        } else if (state === 'paused') {
            this.playPauseBtn.innerText = '\u25b6 Resume';
            this.playPauseBtn.classList.remove('btn-primary');
            this.playPauseBtn.disabled = false;
        }
    }

    startTimer() {
        this.timerStat.style.display = "flex";
        this.startTime = Date.now();
        this.timerInterval = setInterval(() => {
            let elapsed = (Date.now() - this.startTime) / 1000;
            this.timerCount.innerText = elapsed.toFixed(1) + "s";
        }, 100);
    }

    stopTimer() {
        if (this.timerInterval) clearInterval(this.timerInterval);
    }

    analyzeArrayAndSuggest(arr) {
        if (!arr || arr.length === 0) return;
        const n = arr.length;
        
        let inversions = 0;
        let isReverse = true;
        let sorted = true;
        for (let i = 0; i < n - 1; i++) {
            if (arr[i] > arr[i + 1]) sorted = false;
            if (arr[i] < arr[i + 1]) isReverse = false;
        }
        
        for (let i = 0; i < n; i++) {
            for (let j = i + 1; j < n; j++) {
                if (arr[i] > arr[j]) inversions++;
            }
        }
        
        let maxInversions = (n * (n - 1)) / 2;
        let suggestion = "";
        let algoValue = "quick";
        
        if (sorted && n > 1) {
            suggestion = "Already sorted! Insertion Sort parses this in O(n) speed.";
            algoValue = "insertion";
        } else if (n <= 15) {
            suggestion = "Tiny dataset! Insertion Sort is exceptionally fast due to low overhead.";
            algoValue = "insertion";
        } else if (inversions <= n * 1.5) {
            suggestion = "Nearly sorted! Insertion Sort handles this beautifully.";
            algoValue = "insertion";
        } else if (isReverse || inversions > maxInversions * 0.8) {
            suggestion = "Heavily reversed data. Merge Sort securely executes in O(n log n).";
            algoValue = "merge";
        } else {
            suggestion = "Large randomized data. Quick Sort boasts highly optimal average speeds.";
            algoValue = "quick";
        }
        
        const suggEl = document.getElementById("algo-suggestion");
        const btn = document.getElementById("apply-sugg-btn");
        
        if (suggEl) suggEl.innerText = suggestion;
        if (btn) {
            btn.onclick = () => {
                if (this.isSorting) return;
                this.algoSelect.value = algoValue;
                this.algoSelect.style.boxShadow = "0 0 10px #a78bfa";
                this.algoSelect.style.borderColor = "#a78bfa";
                setTimeout(() => { 
                    this.algoSelect.style.boxShadow = ""; 
                    this.algoSelect.style.borderColor = ""; 
                }, 1000);
            };
        }
    }

    updateStats(c = 0, s = 0) {
        this.comparisons += c;
        this.swaps += s;
        this.compCountEl.innerText = this.comparisons;
        this.swapCountEl.innerText = this.swaps;
    }

    updateArrayText() {
        if (this.customMinVal !== null) {
            const originals = this.array.map(v => Math.round(((v - 5) / 90) * (this.customMaxVal - this.customMinVal) + this.customMinVal));
            this.arrayTextEl.innerText = `[ ${originals.join(", ")} ]`;
        } else {
            this.arrayTextEl.innerText = `[ ${this.array.join(", ")} ]`;
        }
    }

    // Generate a new random array
    generateArray() {
        if (this.isSorting) return;
        
        this.container.innerHTML = '';
        this.array = [];
        const size = parseInt(this.sizeSlider.value);
        this.sizeCountEl.innerText = size;
        
        this.comparisons = 0;
        this.swaps = 0;
        this.updateStats(0, 0);
        this.isStopped = false;
        this.frames = [];
        this.currentFrame = 0;
        this.customMinVal = null;
        
        for (let i = 0; i < size; i++) {
            const value = Math.floor(Math.random() * 95) + 5;
            this.array.push(value);
            this.createBar(value, size, i);
        }
        this.updateArrayText();
        this.analyzeArrayAndSuggest(this.array);
    }

    createBar(value, size, index) {
        const bar = document.createElement("div");
        bar.classList.add("array-bar");
        bar.style.height = `${value}%`;
        bar.style.width = `${100 / size}%`;
        bar.style.margin = `0 ${20 / size}px`;
        bar.dataset.index = index;
        
        // Drag & Drop Setup
        bar.draggable = this.isManualMode;
        
        bar.addEventListener("dragstart", (e) => this.handleDragStart(e));
        bar.addEventListener("dragover", (e) => this.handleDragOver(e));
        bar.addEventListener("drop", (e) => this.handleDrop(e));
        bar.addEventListener("dragend", (e) => this.handleDragEnd(e));
        bar.addEventListener("dragenter", (e) => this.handleDragEnter(e));
        bar.addEventListener("dragleave", (e) => this.handleDragLeave(e));
        
        this.container.appendChild(bar);
    }

    // Helper to pause execution or capture frame
    async sleep() {
        if (this.isCapturing) {
            this.frames.push({
                heights: [...this.virtualHeights],
                classes: [...this.virtualClasses],
                comparisons: this.comparisons,
                swaps: this.swaps,
                arrayVals: [...this.array]
            });
            return Promise.resolve();
        }
        return new Promise((resolve) => resolve());
    }

    toggleUI(disabled) {
        this.isSorting = disabled;
        this.sizeSlider.disabled = disabled;
        this.generateBtn.disabled = disabled;
        this.manualBtn.disabled = disabled;
        this.customInput.disabled = disabled;
        this.loadCustomBtn.disabled = disabled;
        
        this.stopBtn.disabled = !disabled;
        this.algoSelect.disabled = disabled;
        if (disabled) this.setPlayPauseState('sorting');
    }

    async swapNodes(i, j) {
        if (this.isCapturing) {
            let tempH = this.virtualHeights[i];
            this.virtualHeights[i] = this.virtualHeights[j];
            this.virtualHeights[j] = tempH;
        } else {
            const bars = this.container.getElementsByClassName("array-bar");
            const tempHeight = bars[i].style.height;
            bars[i].style.height = bars[j].style.height;
            bars[j].style.height = tempHeight;
        }

        const temp = this.array[i];
        this.array[i] = this.array[j];
        this.array[j] = temp;
        
        this.updateStats(0, 1);
        if (!this.isCapturing) this.updateArrayText();
    }
    
    async updateHeight(idx, heightVal) {
        if (this.isCapturing) {
            this.virtualHeights[idx] = `${heightVal}%`;
        } else {
            const bars = this.container.getElementsByClassName("array-bar");
            if (bars[idx]) bars[idx].style.height = `${heightVal}%`;
        }
        this.array[idx] = heightVal;
        if (!this.isCapturing) this.updateArrayText();
    }

    setClass(idx, className) {
        if (this.isCapturing) {
            this.virtualClasses[idx] = `array-bar ${className}`;
        } else {
            const bars = this.container.getElementsByClassName("array-bar");
            if (bars[idx]) {
                bars[idx].className = `array-bar ${className}`;
                if (this.isManualMode) bars[idx].draggable = true;
            }
        }
    }

    // --- Runner ---
    async runSort(algoStrategy) {
        if (this.isSorting || this.isManualMode) return;
        this.isStopped = false;
        this.comparisons = 0;
        this.swaps = 0;
        this.updateStats(0, 0);
        this.toggleUI(true);
        this.timerStat.style.display = "none";
        
        const bars = this.container.getElementsByClassName("array-bar");
        this.virtualHeights = [];
        this.virtualClasses = [];
        for (let i = 0; i < bars.length; i++) {
            this.virtualHeights.push(bars[i].style.height);
            this.virtualClasses.push("array-bar");
        }
        
        this.isCapturing = true;
        this.frames = [];
        this.currentFrame = 0;
        let originalArray = [...this.array];
        
        try {
            const context = new MainVisualizerContext(this);
            await algoStrategy.sort(context);
            for (let i = 0; i < this.array.length; i++) {
                this.setClass(i, "sorted");
                await this.sleep();
            }
        } catch (e) {
            console.error(e);
        }
        
        this.isCapturing = false;
        this.array = originalArray; // restore logical array so renderer uses true values from frames
        
        this.isPlaying = true;
        this.setPlayPauseState('playing');
        if (this.prevBtn) this.prevBtn.disabled = false;
        if (this.nextBtn) this.nextBtn.disabled = false;
        
        this.playFrames();
    }

    getSpeedMs() {
        return (101 - parseInt(this.speedSlider.value)) * 2;
    }

    playFrames() {
        if (!this.isPlaying || this.currentFrame >= this.frames.length || this.isStopped) {
            if (this.currentFrame >= this.frames.length || this.isStopped) {
                this.finishPlayback();
            }
            return;
        }
        this.renderFrame(this.currentFrame);
        this.currentFrame++;
        this.playTimeout = setTimeout(() => this.playFrames(), this.getSpeedMs());
    }

    finishPlayback() {
        this.isPlaying = false;
        clearTimeout(this.playTimeout);
        this.frames = [];
        this.currentFrame = 0;
        this.toggleUI(false);
        this.setPlayPauseState('idle');
        if (this.prevBtn) this.prevBtn.disabled = true;
        if (this.nextBtn) this.nextBtn.disabled = true;
        
        const bars = this.container.getElementsByClassName("array-bar");
        for (let i = 0; i < bars.length; i++) {
            let cl = bars[i].className.replace("compare", "").replace("swap", "").trim();
            bars[i].className = cl;
        }
    }

    renderFrame(index) {
        if (!this.frames[index]) return;
        const f = this.frames[index];
        const bars = this.container.getElementsByClassName("array-bar");
        for (let i = 0; i < bars.length; i++) {
            bars[i].style.height = f.heights[i];
            bars[i].className = f.classes[i];
        }
        this.compCountEl.innerText = f.comparisons;
        this.swapCountEl.innerText = f.swaps;
        
        if (this.customMinVal !== null) {
            const originals = f.arrayVals.map(v => Math.round(((v - 5) / 90) * (this.customMaxVal - this.customMinVal) + this.customMinVal));
            this.arrayTextEl.innerText = `[ ${originals.join(", ")} ]`;
        } else {
            this.arrayTextEl.innerText = `[ ${f.arrayVals.join(", ")} ]`;
        }
    }

    updateComplexityInfo() {
        const selected = this.algoSelect.value;
        this.drawComplexityChart(selected);
    }

    drawComplexityChart(selected) {
        const canvas = document.getElementById("complexity-chart");
        if (!canvas) return;
        
        const c = this.complexities[selected];
        const isOn = c.worst === "O(n)";
        const isOnLogn = c.worst.includes("log n");
        const isOn2 = c.worst.includes("n²") || c.worst.includes("(log n)²");

        const labels = Array.from({length: 20}, (_, i) => i + 1);
        
        // Calculate datasets
        const dataOn = labels.map(x => x);
        const dataOnLogn = labels.map(x => x * Math.log2(x + 1));
        const dataOn2 = labels.map(x => (x * x) / 4); // scaled slightly for visual balance
        
        if (this.chartInstance) {
            this.chartInstance.destroy();
        }
        
        // Check if Chart is defined (in case of CDN failure)
        if (typeof Chart === 'undefined') return;

        this.chartInstance = new Chart(canvas, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [
                    {
                        label: 'O(n)',
                        data: dataOn,
                        borderColor: isOn ? 'rgba(52, 211, 153, 1)' : 'rgba(52, 211, 153, 0.3)',
                        borderWidth: isOn ? 3 : 1.5,
                        borderDash: isOn ? [] : [5, 5],
                        tension: 0.4,
                        pointRadius: 0
                    },
                    {
                        label: 'O(n log n)',
                        data: dataOnLogn,
                        borderColor: isOnLogn ? 'rgba(167, 139, 250, 1)' : 'rgba(167, 139, 250, 0.3)',
                        borderWidth: isOnLogn ? 3 : 1.5,
                        borderDash: isOnLogn ? [] : [5, 5],
                        tension: 0.4,
                        pointRadius: 0
                    },
                    {
                        label: 'O(n²)',
                        data: dataOn2,
                        borderColor: isOn2 ? 'rgba(248, 113, 113, 1)' : 'rgba(248, 113, 113, 0.3)',
                        borderWidth: isOn2 ? 3 : 1.5,
                        borderDash: isOn2 ? [] : [5, 5],
                        tension: 0.4,
                        pointRadius: 0
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: {
                        labels: { color: 'rgba(255, 255, 255, 0.7)' }
                    },
                    title: {
                        display: true,
                        text: `Time Complexity Profile: ${c.algo}`,
                        color: 'rgba(255, 255, 255, 0.9)',
                        font: { size: 16 }
                    }
                },
                scales: {
                    x: {
                        title: { display: true, text: 'Data Size (n)', color: 'rgba(255, 255, 255, 0.7)' },
                        ticks: { color: 'rgba(255, 255, 255, 0.5)' },
                        grid: { color: 'rgba(255, 255, 255, 0.1)' }
                    },
                    y: {
                        title: { display: true, text: 'Operations (Time)', color: 'rgba(255, 255, 255, 0.7)' },
                        ticks: { display: false },
                        grid: { color: 'rgba(255, 255, 255, 0.1)' }
                    }
                }
            }
        });
    }

    // --- Play/Pause/Navigation Handlers ---
    handlePlayPause() {
        // Idle state — no frames yet, start a new sort
        if (this.frames.length === 0 && !this.isSorting) {
            const selected = this.algoSelect.value;
            const algorithm = window.SortingAlgorithms[selected];
            if (algorithm) {
                this.runSort(algorithm);
            }
            return;
        }
        // Playing/Paused toggle
        this.isPlaying = !this.isPlaying;
        if (this.isPlaying) {
            this.setPlayPauseState('playing');
            this.playFrames();
        } else {
            this.setPlayPauseState('paused');
            clearTimeout(this.playTimeout);
        }
    }

    handlePrevFrame() {
        if (this.isPlaying) { 
            this.isPlaying = false; 
            this.setPlayPauseState('paused'); 
            clearTimeout(this.playTimeout); 
        }
        if (this.currentFrame > 0) {
            this.currentFrame--;
            this.renderFrame(this.currentFrame);
        }
    }

    handleNextFrame() {
        if (this.isPlaying) { 
            this.isPlaying = false; 
            this.setPlayPauseState('paused'); 
            clearTimeout(this.playTimeout); 
        }
        if (this.currentFrame < this.frames.length - 1) {
            this.currentFrame++;
            this.renderFrame(this.currentFrame);
        }
    }

    // --- Manual Sort Handling ---
    toggleManualMode() {
        if (this.isSorting) return;
        this.isManualMode = !this.isManualMode;
        
        if (this.isManualMode) {
            this.generateArray(); // Start fresh array to sort before enabling manual mode
            this.manualBtn.innerText = "Manual Mode: ON";
            this.manualBtn.classList.add("active");
            this.algoSelect.disabled = true;
            if (this.playPauseBtn) this.playPauseBtn.disabled = true;
            this.timerCount.innerText = "0.0s";
            this.startTimer();
        } else {
            this.manualBtn.innerText = "Manual Mode: OFF";
            this.manualBtn.classList.remove("active");
            this.algoSelect.disabled = false;
            this.setPlayPauseState('idle');
            this.stopTimer();
            this.timerStat.style.display = "none";
        }
        
        const bars = this.container.getElementsByClassName("array-bar");
        for (let i = 0; i < bars.length; i++) {
            bars[i].draggable = this.isManualMode;
            bars[i].className = "array-bar";
        }
    }

    // --- Drag & Drop Handlers ---
    handleDragStart(e) {
        if (!this.isManualMode) { e.preventDefault(); return; }
        this.draggedIndex = parseInt(e.target.dataset.index);
        e.target.classList.add("dragging");
        e.dataTransfer.effectAllowed = "move";
    }

    handleDragOver(e) {
        if (!this.isManualMode) return;
        e.preventDefault();
        e.dataTransfer.dropEffect = "move";
    }
    
    handleDragEnter(e) {
        if (!this.isManualMode) return;
        if (e.target.classList.contains("array-bar") && parseInt(e.target.dataset.index) !== this.draggedIndex) {
            e.target.classList.add("drag-over");
        }
    }
    
    handleDragLeave(e) {
        if (!this.isManualMode) return;
        e.target.classList.remove("drag-over");
    }

    handleDrop(e) {
        if (!this.isManualMode) return;
        e.stopPropagation();
        e.preventDefault();
        
        e.target.classList.remove("drag-over");
        
        if (!e.target.classList.contains("array-bar")) return;

        const dropIndex = parseInt(e.target.dataset.index);
        
        if (this.draggedIndex !== null && this.draggedIndex !== dropIndex) {
            const bars = this.container.getElementsByClassName("array-bar");
            const tempHeight = bars[this.draggedIndex].style.height;
            bars[this.draggedIndex].style.height = bars[dropIndex].style.height;
            bars[dropIndex].style.height = tempHeight;

            const temp = this.array[this.draggedIndex];
            this.array[this.draggedIndex] = this.array[dropIndex];
            this.array[dropIndex] = temp;
            
            this.updateStats(0, 1);
            this.updateArrayText();
            this.checkIfSorted();
        }
        
        return false;
    }

    handleDragEnd(e) {
        if (!this.isManualMode) return;
        e.target.classList.remove("dragging");
        const bars = this.container.getElementsByClassName("array-bar");
        for (let i = 0; i < bars.length; i++) {
            bars[i].classList.remove("drag-over");
        }
        this.draggedIndex = null;
    }
    
    checkIfSorted() {
        let sorted = true;
        for (let i = 0; i < this.array.length - 1; i++) {
            if (this.array[i] > this.array[i + 1]) {
                sorted = false;
                break;
            }
        }
        if (sorted) {
            const bars = this.container.getElementsByClassName("array-bar");
            for (let i = 0; i < bars.length; i++) bars[i].className = "array-bar sorted";
            this.stopTimer();
            setTimeout(() => alert("Array is Sorted manually! Great job!\nTime taken: " + this.timerCount.innerText), 100);
            
            this.isManualMode = false;
            this.manualBtn.innerText = "Manual Mode: OFF";
            this.manualBtn.classList.remove("active");
            this.algoSelect.disabled = false;
            this.setPlayPauseState('idle');
            for (let i = 0; i < bars.length; i++) bars[i].draggable = false;
        }
    }

    // --- Custom Array Loader ---
    handleCustomLoad() {
        if (this.isSorting) return;
        const val = this.customInput.value;
        if (!val) return;
        
        const parsed = val.split(/[\s,]+/).map(n => parseInt(n.trim())).filter(n => !isNaN(n));
        if (parsed.length === 0) return alert("Please enter valid numbers formatted as comma or space-separated values.");
        
        let maxVal = Math.max(...parsed);
        let minVal = Math.min(...parsed);
        if (maxVal === minVal) { maxVal = minVal + 1; } // Prevent div by 0
        
        this.customMinVal = minVal;
        this.customMaxVal = maxVal;
        
        const mapped = parsed.map(n => {
            let scaled = 5 + ((n - minVal) / (maxVal - minVal)) * 90;
            return { orig: n, height: scaled };
        });
        
        if (mapped.length > 200) {
            return alert("Please enter at most 200 numbers to maintain browser performance.");
        }
        
        this.generateCustomArray(mapped);
    }

    generateCustomArray(mappedArr) {
        if (this.isSorting) return;
        
        this.container.innerHTML = '';
        this.array = mappedArr.map(m => m.height);
        const size = mappedArr.length;
        this.sizeCountEl.innerText = size;
        
        if (size >= 5 && size <= 100) this.sizeSlider.value = size;
        
        this.comparisons = 0;
        this.swaps = 0;
        this.updateStats(0, 0);
        this.isStopped = false;
        
        for (let i = 0; i < size; i++) {
            this.createBar(mappedArr[i].height, size, i);
        }
        this.updateArrayText();
        this.analyzeArrayAndSuggest(this.array);
    }
}

// Instantiate the Main Visualizer when the document is loaded
document.addEventListener("DOMContentLoaded", () => {
    window.mainVisualizer = new MainVisualizer();
});
