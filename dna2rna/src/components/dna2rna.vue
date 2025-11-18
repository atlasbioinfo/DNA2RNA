<script setup>
import { copyToClipboard, useQuasar } from 'quasar';
import { ref } from 'vue'

const q = useQuasar()
const dna_seq = ref('')
const selectedOperation = ref('complement')
const result = ref('')
const showResult = ref(false)

const operations = [
  { label: 'Complement', value: 'complement', desc: 'DNA complementary strand' },
  { label: 'Reverse Complement', value: 'reverseComplement', desc: 'Reverse & complement DNA' },
  { label: 'RNA', value: 'rna', desc: 'Convert DNA to RNA' },
  { label: 'RNA Complement', value: 'rnaComplement', desc: 'RNA complementary strand' },
  { label: 'RNA Reverse Complement', value: 'rnaReverseComplement', desc: 'Reverse & complement RNA' }
]

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

  const sequence = dna_seq.value.toUpperCase()

  switch (selectedOperation.value) {
    case 'complement':
      result.value = sequence.replace(/[ATCGNU]/g, function(match) {
        return { 'A': 'T', 'T': 'A', 'C': 'G', 'G': 'C', 'N': 'N', 'U': 'A' }[match];
      })
      break

    case 'reverseComplement':
      const comp = sequence.replace(/[ATCGNU]/g, function(match) {
        return { 'A': 'T', 'T': 'A', 'C': 'G', 'G': 'C', 'N': 'N', 'U': 'A' }[match];
      })
      result.value = comp.split('').reverse().join('')
      break

    case 'rna':
      result.value = sequence.replace(/T/g, 'U')
      break

    case 'rnaComplement':
      result.value = sequence.replace(/T/g, 'U').replace(/[ATCGNU]/g, function(match) {
        return { 'A': 'U', 'T': 'A', 'C': 'G', 'G': 'C', 'N': 'N', 'U': 'A' }[match];
      })
      break

    case 'rnaReverseComplement':
      const rnaComp = sequence.replace(/T/g, 'U').replace(/[ATCGNU]/g, function(match) {
        return { 'A': 'U', 'T': 'A', 'C': 'G', 'G': 'C', 'N': 'N', 'U': 'A' }[match];
      })
      result.value = rnaComp.split('').reverse().join('')
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

const clearAll = () => {
  dna_seq.value = ''
  result.value = ''
  showResult.value = false
  selectedOperation.value = 'complement'
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

            <q-card bordered flat class="result-card">
              <q-card-section>
                <div class="result-content">
                  <div class="result-text">{{ result }}</div>
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
</style>