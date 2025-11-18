<script setup>
import { copyToClipboard, useQuasar } from 'quasar';
import { ref } from 'vue'

const q = useQuasar()
const dna_seq = ref('')
const selectedOperation = ref('reverse')
const result = ref('')
const showResult = ref(false)
const aminoAcidSequence = ref('')
const rnaSequence = ref('')
const isTranslation = ref(false)

const operations = [
  { label: 'Only Reverse Strand', value: 'reverse', desc: 'Reverse the DNA sequence' },
  { label: 'Reverse Complement', value: 'reverseComplement', desc: 'Reverse and complement DNA' },
  { label: 'Translation', value: 'translation', desc: 'Translate to amino acid sequence' }
]

// Codon table for translation (standard genetic code)
const codonTable = {
  'UUU': 'F', 'UUC': 'F', 'UUA': 'L', 'UUG': 'L',
  'UCU': 'S', 'UCC': 'S', 'UCA': 'S', 'UCG': 'S',
  'UAU': 'Y', 'UAC': 'Y', 'UAA': '*', 'UAG': '*',
  'UGU': 'C', 'UGC': 'C', 'UGA': '*', 'UGG': 'W',
  'CUU': 'L', 'CUC': 'L', 'CUA': 'L', 'CUG': 'L',
  'CCU': 'P', 'CCC': 'P', 'CCA': 'P', 'CCG': 'P',
  'CAU': 'H', 'CAC': 'H', 'CAA': 'Q', 'CAG': 'Q',
  'CGU': 'R', 'CGC': 'R', 'CGA': 'R', 'CGG': 'R',
  'AUU': 'I', 'AUC': 'I', 'AUA': 'I', 'AUG': 'M',
  'ACU': 'T', 'ACC': 'T', 'ACA': 'T', 'ACG': 'T',
  'AAU': 'N', 'AAC': 'N', 'AAA': 'K', 'AAG': 'K',
  'AGU': 'S', 'AGC': 'S', 'AGA': 'R', 'AGG': 'R',
  'GUU': 'V', 'GUC': 'V', 'GUA': 'V', 'GUG': 'V',
  'GCU': 'A', 'GCC': 'A', 'GCA': 'A', 'GCG': 'A',
  'GAU': 'D', 'GAC': 'D', 'GAA': 'E', 'GAG': 'E',
  'GGU': 'G', 'GGC': 'G', 'GGA': 'G', 'GGG': 'G'
}

const processSequence = () => {
  if (!dna_seq.value.trim()) {
    q.notify({
      message: 'Please enter a DNA sequence',
      color: 'warning',
      icon: 'warning',
      position: 'top',
      timeout: 2000
    })
    return
  }

  const sequence = dna_seq.value.toUpperCase().replace(/\s/g, '')
  isTranslation.value = false

  switch (selectedOperation.value) {
    case 'reverse':
      // Only reverse the sequence
      result.value = sequence.split('').reverse().join('')
      break

    case 'reverseComplement':
      // Complement then reverse
      const comp = sequence.replace(/[ATCGNU]/g, function(match) {
        return { 'A': 'T', 'T': 'A', 'C': 'G', 'G': 'C', 'N': 'N', 'U': 'A' }[match];
      })
      result.value = comp.split('').reverse().join('')
      break

    case 'translation':
      // Convert DNA to RNA first
      const rna = sequence.replace(/T/g, 'U')
      rnaSequence.value = rna

      // Split into codons and translate
      let aminoAcids = []
      for (let i = 0; i < rna.length; i += 3) {
        const codon = rna.substr(i, 3)
        if (codon.length === 3) {
          const aa = codonTable[codon] || 'X'
          aminoAcids.push(aa)
        }
      }

      aminoAcidSequence.value = aminoAcids.join('')
      isTranslation.value = true
      break
  }

  showResult.value = true

  q.notify({
    message: 'Sequence processed successfully!',
    color: 'positive',
    icon: 'check_circle',
    position: 'top',
    timeout: 1500
  })
}

const copyResult = () => {
  if (result.value) {
    copyToClipboard(result.value).then(() => {
      q.notify({
        message: 'Copied to clipboard',
        color: 'green-4',
        icon: 'content_copy',
        position: 'center',
        timeout: 1000
      })
    })
  }
}

const copyAminoAcids = () => {
  if (aminoAcidSequence.value) {
    copyToClipboard(aminoAcidSequence.value).then(() => {
      q.notify({
        message: 'Amino acid sequence copied!',
        color: 'green-4',
        icon: 'content_copy',
        position: 'center',
        timeout: 1000
      })
    })
  }
}

const clearAll = () => {
  dna_seq.value = ''
  result.value = ''
  showResult.value = false
  aminoAcidSequence.value = ''
  rnaSequence.value = ''
  isTranslation.value = false
  selectedOperation.value = 'reverse'
}
</script>

<template>
  <div class="dna-converter">
    <q-card class="converter-card" flat bordered>
      <q-card-section class="bg-gradient-purple text-white">
        <div class="text-h5 text-weight-bold">
          <q-icon name="science" size="md" class="q-mr-sm" />
          DNA/RNA Sequence Converter
        </div>
        <div class="text-subtitle2 q-mt-xs">
          Enter your DNA sequence and select an operation
        </div>
      </q-card-section>

      <q-separator />

      <q-card-section class="q-pa-lg">
        <!-- Input Section -->
        <div class="input-section">
          <q-input
            v-model="dna_seq"
            filled
            type="textarea"
            label="DNA Sequence"
            placeholder="Enter DNA sequence (e.g., ATCGATCG)"
            hint="Supported bases: A, T, C, G, N, U"
            rows="4"
            class="sequence-input"
          >
            <template v-slot:prepend>
              <q-icon name="input" color="primary" />
            </template>
          </q-input>
        </div>

        <!-- Operation Selection -->
        <div class="operation-section q-mt-lg">
          <div class="text-subtitle1 text-weight-medium q-mb-md">
            <q-icon name="tune" class="q-mr-sm" />
            Select Operation
          </div>

          <q-option-group
            v-model="selectedOperation"
            :options="operations"
            color="purple"
            class="operation-options"
          >
            <template v-slot:label="opt">
              <div class="operation-label">
                <div class="text-weight-medium">{{ opt.label }}</div>
                <div class="text-caption text-grey-7">{{ opt.desc }}</div>
              </div>
            </template>
          </q-option-group>
        </div>

        <!-- Action Buttons -->
        <div class="action-buttons q-mt-lg row q-gutter-md">
          <q-btn
            @click="processSequence"
            label="Submit"
            icon="play_arrow"
            color="purple"
            size="lg"
            unelevated
            class="col submit-btn"
          />
          <q-btn
            @click="clearAll"
            label="Clear"
            icon="clear"
            color="grey-7"
            size="lg"
            outline
            class="col-auto"
          />
        </div>

        <!-- Result Section -->
        <q-slide-transition>
          <div v-if="showResult" class="result-section q-mt-xl">
            <q-separator class="q-mb-lg" />

            <div class="text-subtitle1 text-weight-medium q-mb-md">
              <q-icon name="check_circle" color="positive" class="q-mr-sm" />
              Result
            </div>

            <!-- Translation Result -->
            <q-card v-if="isTranslation" bordered flat class="result-card translation-card">
              <q-card-section>
                <div class="translation-container">
                  <!-- RNA Sequence with codon separators -->
                  <div class="rna-row">
                    <span class="direction-label">5'</span>
                    <div class="codons-display">
                      <template v-for="(codon, index) in rnaSequence.match(/.{1,3}/g)" :key="index">
                        <span class="codon">{{ codon }}</span>
                        <span v-if="index < rnaSequence.match(/.{1,3}/g).length - 1" class="codon-separator">|</span>
                      </template>
                    </div>
                    <span class="direction-label">3'</span>
                  </div>

                  <!-- Amino Acid Sequence -->
                  <div class="amino-acid-row">
                    <span class="amino-spacer"></span>
                    <div class="amino-acids-display">
                      <span v-for="(aa, index) in aminoAcidSequence.split('')" :key="index" class="amino-acid">
                        {{ aa }}
                      </span>
                    </div>
                    <span class="amino-spacer"></span>
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-actions align="right">
                <q-btn
                  @click="copyAminoAcids"
                  label="Copy Amino Acids"
                  icon="content_copy"
                  color="primary"
                  flat
                />
              </q-card-actions>
            </q-card>

            <!-- Regular Result (Reverse / Reverse Complement) -->
            <q-card v-else bordered flat class="result-card">
              <q-card-section>
                <div class="result-content">
                  <div class="result-with-direction">
                    <span class="direction-label">5'</span>
                    <span class="result-text">{{ result }}</span>
                    <span class="direction-label">3'</span>
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-actions align="right">
                <q-btn
                  @click="copyResult"
                  label="Copy Result"
                  icon="content_copy"
                  color="primary"
                  flat
                />
              </q-card-actions>
            </q-card>
          </div>
        </q-slide-transition>
      </q-card-section>
    </q-card>
  </div>
</template>

<style scoped>
.dna-converter {
  width: 100%;
}

.converter-card {
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.bg-gradient-purple {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.sequence-input {
  font-family: 'Courier New', monospace;
}

.sequence-input :deep(textarea) {
  font-family: 'Courier New', monospace;
  font-size: 16px;
  letter-spacing: 2px;
}

.operation-section {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
}

.operation-options :deep(.q-radio) {
  margin-bottom: 16px;
}

.operation-label {
  margin-left: 8px;
}

.submit-btn {
  font-weight: bold;
  letter-spacing: 1px;
}

.result-section {
  animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.result-card {
  background: #f8f9fa;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
}

.result-content {
  background: white;
  padding: 16px;
  border-radius: 6px;
  border-left: 4px solid #667eea;
}

.result-text {
  font-family: 'Courier New', monospace;
  font-size: 16px;
  letter-spacing: 2px;
  word-break: break-all;
  line-height: 1.8;
  color: #333;
}

.result-with-direction {
  display: flex;
  align-items: center;
  gap: 12px;
}

.direction-label {
  font-family: 'Courier New', monospace;
  font-weight: bold;
  font-size: 18px;
  color: #667eea;
  min-width: 30px;
}

/* Translation specific styles */
.translation-card {
  background: linear-gradient(to bottom, #f8f9fa 0%, #ffffff 100%);
}

.translation-container {
  font-family: 'Courier New', monospace;
  padding: 20px 0;
}

.rna-row {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 2px solid #e0e0e0;
}

.codons-display {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 4px;
  flex: 1;
  background: white;
  padding: 12px;
  border-radius: 6px;
  border: 1px solid #d0d0d0;
}

.codon {
  font-size: 16px;
  letter-spacing: 3px;
  font-weight: 600;
  color: #2c3e50;
  background: #e8f4fd;
  padding: 4px 8px;
  border-radius: 4px;
}

.codon-separator {
  color: #667eea;
  font-weight: bold;
  margin: 0 4px;
  font-size: 18px;
}

.amino-acid-row {
  display: flex;
  align-items: center;
  gap: 12px;
}

.amino-spacer {
  min-width: 30px;
}

.amino-acids-display {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  flex: 1;
  padding: 12px;
  background: linear-gradient(to right, #fff5f5, #ffe5e5);
  border-radius: 6px;
  border: 2px solid #ffcccb;
}

.amino-acid {
  font-size: 18px;
  font-weight: bold;
  color: #c7254e;
  background: white;
  padding: 8px 12px;
  border-radius: 6px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  min-width: 40px;
  text-align: center;
  border: 2px solid #f9f2f4;
}
</style>