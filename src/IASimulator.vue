<template>
    <div class="container">
        <div class="code-wrapper">
            <div class="actions">
                <button @click="runSimulation" :disabled="isRunning">Run ▶</button>
            </div>

            <div class="code">
                <textarea cols="25" :rows="lines + 1" spellcheck="false" v-model="assemblyCode"></textarea>
            </div>

            <AccumulatorDisplay :value="accumulator" />
        </div>

        <div class="ram-contents">
            <center><b>Main Memory</b></center>
            <br>
            <div v-for="(value, address) in ram" :key="address" class="ram-item">
                <div class="ram-address">{{ address }}</div>
                <div class="ram-text">{{ value }}</div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { computed, ref } from 'vue';
import AccumulatorDisplay from './AccumulatorDisplay.vue';

const ram = ref(new Array(1000).fill(null)); // Fill with initial values

ram.value[5] = 5
ram.value[6] = 7

const assemblyCode = ref('001 load m(5)\n002 add m(6)\n003 stor m(8)');
const accumulator = ref(0);
const isRunning = ref(false);
const lines = computed(() => assemblyCode.value.split('\n').length)

const runSimulation = () => {
    isRunning.value = true;
    let accumulatorValue = 0;

    const executeInstruction = (instruction, lineNumber) => {
        console.log(lines.value)
        const [programAddress, keyword, operand] = instruction.split(' ');
        let address = null;

        const instructionHandlers = {
            'load': () => {
                if (address !== null) {
                    accumulatorValue = load(address);
                }
            },
            'add': () => {
                if (address !== null) {
                    accumulatorValue += load(address); // Add the value to the accumulator
                }
            },
            'sub': () => {
                if (address !== null) {
                    accumulatorValue -= load(address); // Add the value to the accumulator
                }
            },
            'div': () => {
                if (address !== null) {
                    accumulatorValue /= load(address); // Add the value to the accumulator
                }
            },
            'mul': () => {
                if (address !== null) {
                    accumulatorValue = load(address); // Add the value to the accumulator
                }
            },
            'stor': () => {
                if (address !== null) {
                    store(address);
                }

            },

            // Add more instructions as needed
        };

        const lowercaseKeyword = keyword.toLowerCase();
        if (instructionHandlers[lowercaseKeyword]) {
            address = parseAddress(operand, lineNumber);
            instructionHandlers[lowercaseKeyword]();
        } else {
            console.error(`Unknown instruction on line ${lineNumber}: ${instruction}`);
        }
    };

    const parseAddress = (operand, lineNumber) => {
        operand = operand.toLowerCase();
        const match = operand.match(/m\((\d+)\)/);

        if (match && match[1]) {
            const address = match[1];
            if (/^\d+$/.test(address)) {
                return parseInt(address, 10);
            } else {
                console.error(`Invalid address format on line ${lineNumber}: ${operand}`);
                return null;
            }
        } else {
            console.error(`Invalid operand format on line ${lineNumber}: ${operand}`);
            return null;
        }
    };

    const load = (address) => {
        console.log('load address ', address);
        return ram.value[address]
        // Implement the 'LOAD' instruction logic here
        // Load the value from the specified memory address
        // and return the result
    };

    const store = (address) => {
        ram.value[address] = accumulator.value
    }

    const executeInstructionsSequentially = async () => {
        const instructions = assemblyCode.value.split('\n');
        let lineNumber = 1;

        for (const instruction of instructions) {
            await new Promise((resolve) => setTimeout(resolve, 1000));
            if (instruction) {
                executeInstruction(instruction, lineNumber);
                accumulator.value = accumulatorValue;
            }
            lineNumber++;
        }

        isRunning.value = false;
    };

    executeInstructionsSequentially();
};
</script>

<style>
.container {
    display: flex;
    gap: 20px;
    flex-basis: 1fr 1fr;
}

.ram-contents {
    padding: 10px;
    height: calc(100vh - 20px);
    overflow-y: scroll;
    width: 250px;
    border: 1px solid;
    display: flex;
    flex-direction: column;
}

.ram-item {
    display: flex;
    padding: 5px;
    border: 1px solid #ccc;
}

.ram-address {
    text-align: right;
    min-width: 50px;
    /* Adjust the width as needed */
    padding-right: 10px;
    background-color: #eee;
}

.ram-text {
    text-align: left;
    padding-left: 10px;
}

.code-wrapper,
.ram-contents {
    flex: auto;
}

.actions {
    margin-bottom: 10px;
}

.code textarea {
    resize: none;
}
</style>
