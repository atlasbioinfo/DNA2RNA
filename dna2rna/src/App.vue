<script setup>
import Dna2Rna from './components/dna2rna.vue'
</script>

<template>
  <q-layout view="hHh lpR fFf" class="bg-gradient-light">

    <q-header elevated class="header-gradient">
      <q-toolbar class="q-py-sm">
        <q-icon name="biotech" size="md" class="q-mr-md" />
        <q-toolbar-title class="text-weight-bold text-h5">
          DNA2RNA Converter
        </q-toolbar-title>
        <q-space />
        <q-btn
          flat
          round
          dense
          icon="info"
          color="white"
          @click="showInfo = !showInfo"
        />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <div class="page-content">
        <div class="row justify-center q-pa-md">
          <div class="col-12 col-md-10 col-lg-8">
            <q-banner v-if="showInfo" class="bg-info text-white q-mb-lg info-banner" rounded>
              <template v-slot:avatar>
                <q-icon name="info" />
              </template>
              <div class="text-body1">
                <strong>About this tool:</strong> Convert DNA sequences to RNA, generate complementary strands,
                and perform reverse complement operations with ease.
              </div>
              <template v-slot:action>
                <q-btn flat icon="close" @click="showInfo = false" />
              </template>
            </q-banner>

            <Dna2Rna />
          </div>
        </div>
      </div>
    </q-page-container>

    <q-footer class="footer-gradient">
      <q-toolbar>
        <q-toolbar-title class="row justify-center items-center">
          <div class="text-body1">
            <q-icon name="science" class="q-mr-sm" />
            <span class="text-weight-medium">ATLAS BIOINFORMATICS TOOLS</span>
          </div>
        </q-toolbar-title>
      </q-toolbar>
    </q-footer>

  </q-layout>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const showInfo = ref(false)
    return { showInfo }
  }
}
</script>

<style>
.bg-gradient-light {
  background: linear-gradient(180deg, #f0f4f8 0%, #e2e8f0 100%);
}

.header-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.footer-gradient {
  background: linear-gradient(135deg, #434343 0%, #000000 100%);
  color: white;
}

.page-content {
  min-height: calc(100vh - 100px);
  padding: 40px 0;
}

.info-banner {
  animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>