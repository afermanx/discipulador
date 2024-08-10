<script setup>
import { ref, watch, onMounted } from 'vue';
import VuePdfEmbed from 'vue-pdf-embed';

const title = ref('Discipulado');
const page = ref(1);
const isLoading = ref(true);
const pageCount = ref(null);
const pdfSource = ref('/storage/discipulado.pdf');
const showToast = ref(false);
const toastMessage = ref('');

const handleLoad = ({ numPages }) => {
  pageCount.value = numPages;
  isLoading.value = false;
};

const handlePageCount = (numPages) => {
  pageCount.value = numPages;
};

watch(() => isLoading.value, (loading) => {
  if (!loading) {
    console.log('PDF carregado.');
  }
});

watch(() => page.value, (newPage) => {
  if (newPage > pageCount.value || newPage < 0) {
    toastMessage.value = 'Página inexistente.';
    showToast.value = true;
    page.value = Math.min(pageCount.value, Math.max(1, page.value));
     window.location.reload();

  }
});

const handleToastClose = () => {
  showToast.value = false;
};

onMounted(() => {
  setTimeout(() => handleLoad({ numPages: pageCount.value }), 2000);
});
</script>

<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" sm="10" md="8" lg="6">
        <v-card>
          <v-card-text>
            <v-btn @click="page = 1" icon rel="noopener noreferrer">
              <v-icon>mdi-home</v-icon>
            </v-btn>
          </v-card-text>
          <v-card-title>
            <span class="d-flex align-center justify-center text-h5 text-green">
              <v-icon color="green" size="36px">mdi-book</v-icon>{{ title }}
            </span>
          </v-card-title>

          <v-card-subtitle>
            <v-progress-linear
              v-if="isLoading"
              indeterminate
              color="green"
              height="4"
              class="mb-4"
            />
          </v-card-subtitle>

          <v-card-actions>
            <v-btn @click="page = Math.max(1, page - 1)" :disabled="page === 1" icon>
              <v-icon>mdi-chevron-left</v-icon>
            </v-btn>

            <v-spacer></v-spacer>

            <span>{{ page }}</span>
            <v-spacer></v-spacer>
            <span>de</span>
            <v-spacer></v-spacer>
            <span v-on:click="page = pageCount">{{ pageCount }}</span>

            <v-spacer></v-spacer>

            <v-btn @click="page = Math.min(pageCount, page + 1)" :disabled="page === pageCount" icon>
              <v-icon>mdi-chevron-right</v-icon>
            </v-btn>
          </v-card-actions>

          <v-card-text>
            <VuePdfEmbed
              :page="page"
              :source="pdfSource"
              @count="handlePageCount"
              @loaded="handleLoad"
              :loading="isLoading"
            />
          </v-card-text>
          <div class="d-flex align-center justify-center">
            <input
              type="number"
              v-model="page"
              :max="pageCount"
              min="1"
              class="page-input"
            />
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>

  <v-snackbar v-model="showToast" :timeout="3000" color="error" centered>
    {{ toastMessage }}
    <template v-slot:actions="{ close }">
      <v-btn color="white" text @click="handleToastClose">Fechar</v-btn>
    </template>
  </v-snackbar>
</template>

<style scoped>
.page-input {
  width: 60px;
  text-align: center;
  margin: 0 16px;
}
</style>
