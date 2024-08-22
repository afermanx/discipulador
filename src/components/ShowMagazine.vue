<script setup>
import { ref, watch, onMounted, computed } from 'vue';
import VuePdfEmbed from 'vue-pdf-embed';

const title = ref('Discipulado');
const displayedPage = ref(1); // Page displayed to the user
const isLoading = ref(true);
const pdfPageCount = ref(null); // Actual PDF page count
const pdfSource = ref('/storage/discipulado.pdf');
const showToast = ref(false);
const toastMessage = ref('');
const touchStartX = ref(0);
const toPage = ref(null);

// Convert displayed page number to PDF page number
const displayedPageToPdfPage = (displayedPage) => {
  return displayedPage; // Offset by 2 pages
};

// Convert PDF page number to displayed page number
const pdfPageToDisplayedPage = (pdfPage) => {
  return pdfPage >= 3 ? pdfPage - 2 : 0; // Offset by 2 pages, but 0 for first two pages
};

const handleLoad = ({ numPages }) => {
  pdfPageCount.value = numPages;
  isLoading.value = false;
};

const handlePageCount = (numPages) => {
  pdfPageCount.value = numPages;
};

watch(() => isLoading.value, (loading) => {
  if (!loading) {
    console.log('PDF carregado.');
  }
});

watch(() => displayedPage.value, (newPage) => {
  const actualPage = displayedPageToPdfPage(newPage);

  if (actualPage > pdfPageCount.value -2 || actualPage < 0) {
    toastMessage.value = 'Página inexistente.';
    showToast.value = true;
    displayedPage.value = 1;
    toPage.value = null;
  }
});

watch(() => toPage.value, (newPage) => {3
  if (newPage) {
    goToPage();
  }
});

const handleTouchStart = (e) => {
  touchStartX.value = e.changedTouches[0].screenX;
};

const goToPage = () => {
  if (toPage.value < 1 || toPage.value > pdfPageCount.value) {
    toastMessage.value = 'Página inexistente.';
    showToast.value = true;
  }
  displayedPage.value = Math.min(pdfPageToDisplayedPage(pdfPageCount.value), parseInt(toPage.value, 10)) + 2;
}

const handleTouchEnd = (e) => {
  const touchEndX = e.changedTouches[0].screenX;
  if (touchStartX.value - touchEndX > 50) {
    displayedPage.value = Math.min(pdfPageToDisplayedPage(pdfPageCount.value), displayedPage.value + 1);
  } else if (touchStartX.value - touchEndX < -50) {
    displayedPage.value = Math.max(1, displayedPage.value - 1);
  }
};

const handleToastClose = () => {
  showToast.value = false;
};

onMounted(() => {
  setTimeout(() => handleLoad({ numPages: pdfPageCount.value }), 2000);
});
</script>


<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" sm="10" md="8" lg="6">
        <v-card>
          <v-card-text>
            <v-btn @click="displayedPage = 1" icon>
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
            <v-btn @click="displayedPage = Math.max(1, displayedPage - 1)" :disabled="displayedPage === 1" icon>
              <v-icon>mdi-chevron-left</v-icon>
            </v-btn>

            <v-spacer></v-spacer>

            <span v-if="displayedPage <= 2">0</span>
            <span v-else>{{ displayedPage - 2 }}</span>
            <v-spacer></v-spacer>
            <span>de</span>
            <v-spacer></v-spacer>
            <span v-if="pdfPageCount > 2">{{ pdfPageToDisplayedPage(pdfPageCount) }}</span>
            <span v-else>0</span>

            <v-spacer></v-spacer>

            <v-btn @click="displayedPage = Math.min(pdfPageToDisplayedPage(pdfPageCount), displayedPage + 1)" :disabled="displayedPage === pdfPageToDisplayedPage(pdfPageCount)" icon>
              <v-icon>mdi-chevron-right</v-icon>
            </v-btn>
          </v-card-actions>
          <v-card-actions>
           <v-btn
                v-if="toPage === null"
                @click="toPage = 0"
              >
                Ir para uma página
              </v-btn>
            <v-text-field
            v-else

              v-model="toPage"
              class="page-input"
              type="number"
              label="Digite uma página"
              hide-details="auto"
              @keydown.enter="goToPage"
            ></v-text-field>
          </v-card-actions>

          <v-card-text @touchstart="handleTouchStart" @touchend="handleTouchEnd">
            <VuePdfEmbed
              :page="displayedPageToPdfPage(displayedPage)"
              :source="pdfSource"
              @count="handlePageCount"
              @loaded="handleLoad"
              :loading="isLoading"
            />
          </v-card-text>

        </v-card>
      </v-col>
    </v-row>
  </v-container>

  <v-snackbar v-model="showToast" :timeout="3000" color="error" top centered>
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



<style scoped>
.page-input {
  width: 60px;
  text-align: center;
  margin: 0 16px;
}
</style>
