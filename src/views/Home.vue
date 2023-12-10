<template>
  <div class="w-full h-full">
    <navbar-main />
    <div class="w-full h-full px-4 lg:px-20">
      <div class="mx-auto max-w-2xl overflow-x-auto">
        <table class="table">
          <!-- head -->
          <thead>
            <tr>
              <th></th>
              <th>Suku Min</th>
              <th>Kelompok</th>
            </tr>
            <tr>
              <th>Desimal</th>
              <th>Biner</th>
              <th>Cacah Bit</th>
            </tr>
          </thead>
          <tbody>
            <!-- row 1 -->
            <tr v-for="(item, index) in groupedData" :key="index">
              <td>{{ item.Decimal }}</td>
              <td>{{ item.Binary }}</td>
              <td>{{ item.Group }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="flex items-center justify-center lg:mx-64 my-10">
        <label class="form-control w-full">
          <div class="label">
            <span class="label-text">Input Nilai Minterm</span>
            <span class="label-text-alt mr-24">Max 0-15</span>
          </div>
          <div class="w-full flex gap-2">
            <input
              type="text"
              v-model="mintermInput"
              placeholder="Contoh 0, 2, 3, 4, 9, 10, 11, 14, 15"
              class="input input-bordered w-full"
            />
            <button @click="submitMinterm" class="btn btn-primary">Submit</button>
          </div>
          <div v-if="errorMaxinput" class="label">
            <span class="label-text text-red-600">{{ errorMaxinput }}</span>
          </div>
        </label>
      </div>

      <template v-if="mintermList.length > 0">
        <div class="mx-auto max-w-2xl overflow-x-auto my-4">
          <table class="table">
            <thead>
              <tr>
                <th>Minterm</th>
                <th>Bentuk Biner</th>
                <th>Jumlah Angka 1</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(minterm, index) in mintermList" :key="index">
                <td>{{ `m${minterm}` }}</td>
                <td>{{ decimalToBinary(minterm) }}</td>
                <td>{{ countOnes(decimalToBinary(minterm)) }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="mx-auto max-w-2xl overflow-x-auto my-4">
          <table class="table">
            <thead>
              <tr>
                <th>Kelompok</th>
                <th>Jumlah Angka 1</th>
                <th>Minterm</th>
                <th>Bentuk Biner</th>
                <th>Tanda</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(minterm, index) in mintermList" :key="index">
                <td>{{ countOnes(decimalToBinary(minterm)) }}</td>
                <td>{{ countOnes(decimalToBinary(minterm)) }}</td>
                <td>{{ `m${minterm}` }}</td>
                <td>{{ decimalToBinary(minterm) }}</td>
                <td><i class="bi bi-check-lg"></i></td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="flex items-center flex-col">
          <div class="py-2 w-[750px] sm:px-6 lg:px-8">
            <div class="shadow border-b border-zinc-200 sm:rounded-lg">
              <div class="bg-zinc-50">
                <div class="flex items-center">
                  <div class="w-1/2 p-7">
                    <span>Pasangan Minterm</span>
                  </div>
                  <div class="w-1/2 p-7">
                    <span>Hasil Pasangan</span>
                  </div>
                </div>
              </div>
              <div class="bg-white divide-y divide-zinc-200">
                <div v-for="(pair, i) in pairedMinterms" :key="i" class="flex">
                  <div class="w-1/2 p-7">
                    <div class="flex flex-col">
                      <span
                        >{{ `m${pair[0]}` }} (kelompok
                        {{ calculateGroup(decimalToBinary(pair[0])) }})</span
                      >
                      <span
                        >{{ `m${pair[1]}` }} (kelompok
                        {{ calculateGroup(decimalToBinary(pair[1])) }})</span
                      >
                    </div>
                  </div>

                  <div class="w-1/2 p-7">
                    <div class="flex flex-col">
                      <span>{{ decimalToBinary(pair[0]) }}</span>
                      <span>{{ decimalToBinary(pair[1]) }}</span>
                      <div class="border-b border-black w-10"></div>
                      <span class="font-bold">{{ calculateCombinedMinterm(pair) }}</span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="mx-auto max-w-2xl overflow-x-auto my-4 rounded-lg shadow">
          <div class="w-full h-20 flex items-center justify-center bg-zinc-50">
            <span>Kubus 1</span>
          </div>
          <template v-for="(block, i) in kubusBlocks" :key="i">
            <div class="w-full flex p-6 border-b">
              <div class="w-1/2 flex items-center justify-center flex-col">
                <span v-for="minterm in block.minterms" :key="minterm">{{
                  minterm
                }}</span>
              </div>
              <div class="w-1/2 flex items-center justify-center flex-col">
                <span v-for="binary in block.binaries" :key="binary">{{ binary }}</span>
              </div>
              <div class="w-1/2 flex items-center justify-center flex-col">
                <span v-for="sign in block.signs" :key="sign">
                  <i v-if="sign === 'check'" class="bi bi-check-lg"></i>
                  <i v-else-if="sign === 'sun'" class="bi bi-sun-fill"></i>
                </span>
              </div>
            </div>
          </template>
        </div>
      </template>
    </div>
    <footer-main />
  </div>
</template>

<script setup>
import { ref } from "vue";
import navbarMain from "/src/components/base/navbar-main.vue";
import footerMain from "/src/components/base/footer-main.vue";

const decimalArray = ref(Array.from({ length: 16 }, (_, i) => i));
const binaryArray = ref(decimalArray.value.map(decimalToBinary));

function decimalToBinary(decimal) {
  return decimal.toString(2).padStart(4, "0");
}

const groupedData = ref(
  decimalArray.value.map((item) => ({
    Decimal: item,
    Binary: decimalToBinary(item),
    Group: calculateGroup(decimalToBinary(item)),
    Included: false,
  }))
);

const mintermInput = ref("");
const mintermList = ref([]);
const errorMaxinput = ref("");
const pairedMinterms = ref([]);
const kubusBlocks = ref([]);

function calculateGroup(binary) {
  const countOnes = (binary.match(/1/g) || []).length;
  if (countOnes === 0) {
    return 0;
  } else if (countOnes === 1) {
    return 1;
  } else if (countOnes === 2) {
    return 2;
  } else if (countOnes === 3) {
    return 3;
  } else if (countOnes === 4) {
    return 4;
  }
}

function calculateMinterm(binary) {
  const countOnes = (binary.match(/1/g) || []).length;
  if (countOnes === 0) {
    return "000-";
  } else if (countOnes === 1) {
    return "001-";
  } else if (countOnes === 2) {
    return "010-";
  } else if (countOnes === 3) {
    return "011-";
  } else if (countOnes === 4) {
    return "100-";
  }
}

function calculateCombinedMinterm(pair) {
  const binary1 = decimalToBinary(pair[0]);
  const binary2 = decimalToBinary(pair[1]);

  let result = "";

  for (let i = 0; i < binary1.length; i++) {
    if (binary1[i] === binary2[i]) {
      result += binary1[i];
    } else {
      result += "-";
    }
  }

  return result;
}

function countOnes(binary) {
  return (binary.match(/1/g) || []).length;
}

function submitMinterm() {
  const mintermArray = mintermInput.value
    .split(",")
    .map((item) => parseInt(item.trim(), 10));

  mintermList.value = mintermArray.filter((item) => {
    const parsedItem = parseInt(item, 10);
    if (parsedItem >= 0 && parsedItem <= 15) {
      return true;
    } else {
      errorMaxinput.value = `Angka ${parsedItem} diabaikan karena melebihi rentang 0-15.`;
      return false;
    }
  });

  createPairedMinterms();
  updateKubusBlocks();
}

function createPairedMinterms() {
  pairedMinterms.value = [];

  for (let i = 0; i < mintermList.value.length; i++) {
    for (let j = i + 1; j < mintermList.value.length; j++) {
      pairedMinterms.value.push([mintermList.value[i], mintermList.value[j]]);
    }
  }
}

function updateKubusBlocks() {
  kubusBlocks.value = pairedMinterms.value.map((pair) => {
    return {
      minterms: [`m${pair[0]}`, `m${pair[1]}`],
      binaries: [
        decimalToBinary(pair[0]),
        decimalToBinary(pair[1]),
        calculateCombinedMinterm(pair),
      ],
      signs: ["check", "check", "check"],
    };
  });
}
</script>
