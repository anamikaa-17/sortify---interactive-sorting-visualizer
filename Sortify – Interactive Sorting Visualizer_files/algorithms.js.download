/**
 * algorithms.js
 * Implements the Strategy Pattern for Sorting Algorithms.
 * Each algorithm extends the base class SortingAlgorithm and executes its sorting logic
 * using a standardized context interface.
 */

class SortingAlgorithm {
    constructor(name) {
        this.name = name;
    }

    /**
     * Executes the sorting algorithm.
     * @param {Object} context The sorting visualizer context.
     */
    async sort(context) {
        throw new Error("Method sort() must be implemented.");
    }
}

class BubbleSort extends SortingAlgorithm {
    constructor() {
        super("Bubble Sort");
    }

    async sort(context) {
        const n = context.size();
        for (let i = 0; i < n - 1; i++) {
            for (let j = 0; j < n - i - 1; j++) {
                if (context.isAborted && context.isAborted()) return;
                context.updateStats(1, 0);
                context.setClass(j, "compare");
                context.setClass(j + 1, "compare");
                await context.sleep();

                const valJ = await context.getValue(j);
                const valJ1 = await context.getValue(j + 1);
                if (valJ > valJ1) {
                    context.setClass(j, "swap");
                    context.setClass(j + 1, "swap");
                    await context.sleep();
                    await context.swapNodes(j, j + 1);
                }

                context.setClass(j, "");
                context.setClass(j + 1, "");
            }
            context.setClass(n - i - 1, "sorted");
        }
        context.setClass(0, "sorted");
    }
}

class SelectionSort extends SortingAlgorithm {
    constructor() {
        super("Selection Sort");
    }

    async sort(context) {
        const n = context.size();
        for (let i = 0; i < n; i++) {
            if (context.isAborted && context.isAborted()) return;
            let minIdx = i;
            context.setClass(i, "compare");

            for (let j = i + 1; j < n; j++) {
                if (context.isAborted && context.isAborted()) return;
                context.updateStats(1, 0);
                context.setClass(j, "compare");
                await context.sleep();

                const valJ = await context.getValue(j);
                const valMin = await context.getValue(minIdx);
                if (valJ < valMin) {
                    if (minIdx !== i) context.setClass(minIdx, "");
                    minIdx = j;
                    context.setClass(minIdx, "swap");
                } else {
                    context.setClass(j, "");
                }
            }

            if (minIdx !== i) {
                await context.swapNodes(i, minIdx);
                context.setClass(minIdx, "");
            }
            context.setClass(i, "sorted");
        }
    }
}

class InsertionSort extends SortingAlgorithm {
    constructor() {
        super("Insertion Sort");
    }

    async sort(context) {
        const n = context.size();
        context.setClass(0, "sorted");
        
        for (let i = 1; i < n; i++) {
            if (context.isAborted && context.isAborted()) return;
            let key = await context.getValue(i);
            let j = i - 1;
            
            context.setClass(i, "compare");
            await context.sleep();

            while (j >= 0) {
                if (context.isAborted && context.isAborted()) return;
                context.updateStats(1, 0);
                const valJ = await context.getValue(j);
                if (valJ <= key) break;
                
                context.setClass(j, "swap");
                context.setClass(j + 1, "swap");
                await context.sleep();
                
                await context.updateHeight(j + 1, valJ);
                context.updateStats(0, 1);
                context.setClass(j, "sorted");
                context.setClass(j + 1, "sorted");
                j = j - 1;
            }
            await context.updateHeight(j + 1, key);
            context.updateStats(0, 1);
            context.setClass(j + 1, "sorted");
            
            for (let k = 0; k <= i; k++) context.setClass(k, "sorted");
        }
    }
}

class MergeSort extends SortingAlgorithm {
    constructor() {
        super("Merge Sort");
    }

    async sort(context) {
        await this.mergeSortRecursive(context, 0, context.size() - 1);
    }
    
    async mergeSortRecursive(context, l, r) {
        if (context.isAborted && context.isAborted()) return;
        if (l >= r) return;
        const m = l + Math.floor((r - l) / 2);
        await this.mergeSortRecursive(context, l, m);
        await this.mergeSortRecursive(context, m + 1, r);
        await this.merge(context, l, m, r);
    }

    async merge(context, l, m, r) {
        if (context.isAborted && context.isAborted()) return;
        if (context.incrementPass) context.incrementPass();
        const n1 = m - l + 1;
        const n2 = r - m;
        let left = new Array(n1);
        let right = new Array(n2);

        for (let i = 0; i < n1; i++) left[i] = await context.getValue(l + i);
        for (let j = 0; j < n2; j++) right[j] = await context.getValue(m + 1 + j);

        let i = 0, j = 0, k = l;

        while (i < n1 && j < n2) {
            if (context.isAborted && context.isAborted()) return;
            context.updateStats(1, 0);
            context.setClass(k, "compare");
            await context.sleep();

            if (left[i] <= right[j]) {
                await context.updateHeight(k, left[i]);
                i++;
            } else {
                await context.updateHeight(k, right[j]);
                j++;
            }
            context.updateStats(0, 1);
            context.setClass(k, "");
            k++;
        }

        while (i < n1) {
            if (context.isAborted && context.isAborted()) return;
            context.setClass(k, "compare");
            await context.sleep();
            await context.updateHeight(k, left[i]);
            context.updateStats(0, 1);
            context.setClass(k, "");
            i++; k++;
        }

        while (j < n2) {
            if (context.isAborted && context.isAborted()) return;
            context.setClass(k, "compare");
            await context.sleep();
            await context.updateHeight(k, right[j]);
            context.updateStats(0, 1);
            context.setClass(k, "");
            j++; k++;
        }
    }
}

class QuickSort extends SortingAlgorithm {
    constructor() {
        super("Quick Sort");
    }

    async sort(context) {
        await this.quickSortRecursive(context, 0, context.size() - 1);
    }
    
    async quickSortRecursive(context, low, high) {
        if (context.isAborted && context.isAborted()) return;
        if (low < high) {
            if (context.incrementPass) context.incrementPass();
            const pi = await this.partition(context, low, high);
            await this.quickSortRecursive(context, low, pi - 1);
            await this.quickSortRecursive(context, pi + 1, high);
        } else if (low === high) {
            context.setClass(low, "sorted");
        }
    }

    async partition(context, low, high) {
        let pivot = await context.getValue(high);
        context.setClass(high, "swap"); 
        let i = low - 1;

        for (let j = low; j < high; j++) {
            if (context.isAborted && context.isAborted()) return i + 1;
            context.updateStats(1, 0);
            context.setClass(j, "compare");
            await context.sleep();

            const valJ = await context.getValue(j);
            if (valJ < pivot) {
                i++;
                if (i !== j) {
                    await context.swapNodes(i, j);
                }
            }
            context.setClass(j, "");
        }
        
        await context.swapNodes(i + 1, high);
        context.setClass(high, "");
        context.setClass(i + 1, "sorted"); 
        return i + 1;
    }
}

class HeapSort extends SortingAlgorithm {
    constructor() {
        super("Heap Sort");
    }

    async sort(context) {
        let n = context.size();
        for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
            if (context.isAborted && context.isAborted()) return;
            await this.heapify(context, n, i);
        }
        for (let i = n - 1; i > 0; i--) {
            if (context.isAborted && context.isAborted()) return;
            context.setClass(0, "swap");
            context.setClass(i, "swap");
            await context.sleep();
            await context.swapNodes(0, i);
            context.setClass(0, "");
            context.setClass(i, "sorted");
            await this.heapify(context, i, 0);
        }
        context.setClass(0, "sorted");
    }

    async heapify(context, n, i) {
        let largest = i; 
        let l = 2 * i + 1; 
        let r = 2 * i + 2; 
        
        if (l < n) {
            if (context.isAborted && context.isAborted()) return;
            context.updateStats(1, 0);
            context.setClass(l, "compare");
            context.setClass(largest, "compare");
            await context.sleep();
            
            const valL = await context.getValue(l);
            const valLargest = await context.getValue(largest);
            if (valL > valLargest) largest = l;
            context.setClass(l, "");
            context.setClass(largest, "");
        }
        
        if (r < n) {
            if (context.isAborted && context.isAborted()) return;
            context.updateStats(1, 0);
            context.setClass(r, "compare");
            context.setClass(largest, "compare");
            await context.sleep();
            
            const valR = await context.getValue(r);
            const valLargest = await context.getValue(largest);
            if (valR > valLargest) largest = r;
            context.setClass(r, "");
            context.setClass(largest, "");
        }
        
        if (largest !== i) {
            if (context.isAborted && context.isAborted()) return;
            context.setClass(i, "swap");
            context.setClass(largest, "swap");
            await context.sleep();
            await context.swapNodes(i, largest);
            context.setClass(i, "");
            context.setClass(largest, "");
            await this.heapify(context, n, largest);
        }
    }
}

class ShellSort extends SortingAlgorithm {
    constructor() {
        super("Shell Sort");
    }

    async sort(context) {
        let n = context.size();
        for (let gap = Math.floor(n / 2); gap > 0; gap = Math.floor(gap / 2)) {
            for (let i = gap; i < n; i += 1) {
                if (context.isAborted && context.isAborted()) return;
                let temp = await context.getValue(i);
                let j;
                context.setClass(i, "compare");
                await context.sleep();
                
                for (j = i; j >= gap; j -= gap) {
                    if (context.isAborted && context.isAborted()) return;
                    context.updateStats(1, 0);
                    const valGap = await context.getValue(j - gap);
                    if (valGap <= temp) break;
                    
                    context.setClass(j - gap, "swap");
                    context.setClass(j, "swap");
                    await context.sleep();
                    await context.updateHeight(j, valGap);
                    context.updateStats(0, 1);
                    context.setClass(j - gap, "");
                    context.setClass(j, "");
                }
                await context.updateHeight(j, temp);
                context.updateStats(0, 1);
                context.setClass(i, "");
            }
        }
    }
}

class CocktailSort extends SortingAlgorithm {
    constructor() {
        super("Cocktail Sort");
    }

    async sort(context) {
        let swapped = true;
        let start = 0;
        let end = context.size();
        
        while (swapped) {
            if (context.isAborted && context.isAborted()) return;
            swapped = false;
            for (let i = start; i < end - 1; ++i) {
                if (context.isAborted && context.isAborted()) return;
                context.updateStats(1, 0);
                context.setClass(i, "compare");
                context.setClass(i + 1, "compare");
                await context.sleep();
                
                const valI = await context.getValue(i);
                const valI1 = await context.getValue(i + 1);
                if (valI > valI1) {
                    context.setClass(i, "swap");
                    context.setClass(i + 1, "swap");
                    await context.sleep();
                    await context.swapNodes(i, i + 1);
                    swapped = true;
                }
                context.setClass(i, "");
                context.setClass(i + 1, "");
            }
            if (!swapped) break;
            
            swapped = false;
            end = end - 1;
            context.setClass(end, "sorted");
            
            for (let i = end - 1; i >= start; i--) {
                if (context.isAborted && context.isAborted()) return;
                context.updateStats(1, 0);
                context.setClass(i, "compare");
                context.setClass(i + 1, "compare");
                await context.sleep();
                
                const valI = await context.getValue(i);
                const valI1 = await context.getValue(i + 1);
                if (valI > valI1) {
                    context.setClass(i, "swap");
                    context.setClass(i + 1, "swap");
                    await context.sleep();
                    await context.swapNodes(i, i + 1);
                    swapped = true;
                }
                context.setClass(i, "");
                context.setClass(i + 1, "");
            }
            context.setClass(start, "sorted");
            start = start + 1;
        }
    }
}

// Registry of algorithms to be used by any visualizer context
window.SortingAlgorithms = {
    bubble: new BubbleSort(),
    selection: new SelectionSort(),
    insertion: new InsertionSort(),
    merge: new MergeSort(),
    quick: new QuickSort(),
    heap: new HeapSort(),
    shell: new ShellSort(),
    cocktail: new CocktailSort()
};
