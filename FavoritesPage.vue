<template>
  <div class="container mx-auto py-8 space-y-6">
    <Card class="bg-card">
      <CardHeader>
        <CardTitle class="flex items-center gap-2">
          <Heart class="w-5 h-5" />
          Ulubione - {{ favorites.length }}
        </CardTitle>
      </CardHeader>
      <CardContent>
        <div class="mb-6">
          <div class="flex gap-4">
            <div class="flex-1">
              <FavoritesMatchSearch
                :loading="searchLoading"
                :results="searchResults"
                @search="handleSearch"
                @select="handleMatchSelect" />
            </div>
          </div>
        </div>

        <div
          v-if="isLoading"
          class="flex justify-center py-8">
          <Loader2 class="h-8 w-8 animate-spin" />
        </div>

        <div
          v-else-if="error"
          class="py-8">
          <Alert variant="destructive">
            <AlertCircle class="h-4 w-4" />
            <AlertTitle>Błąd</AlertTitle>
            <AlertDescription>{{ error }}</AlertDescription>
          </Alert>
        </div>

        <div
          v-else-if="favorites.length === 0"
          class="py-8">
          <Alert>
            <AlertCircle class="h-4 w-4" />
            <AlertTitle>Brak ulubionych</AlertTitle>
            <AlertDescription>
              Dodaj swój pierwszy ulubiony mecz używając wyszukiwarki powyżej.
            </AlertDescription>
          </Alert>
        </div>

        <div
          v-else
          class="space-y-4">
          <FavoritesMatchCard
            v-for="match in favorites"
            :key="match.favorite_id"
            :match="match"
            @edit="handleEdit"
            @remove="handleRemove" />
        </div>
      </CardContent>
    </Card>

    <FavoritesMatchModal
      v-model:open="showModal"
      :editing="!!editingMatch"
      :loading="modalLoading"
      :match="selectedMatch"
      @close="handleModalClose"
      @save="handleSave" />
  </div>
</template>

<script lang="ts" setup>
  import { ref } from 'vue';
  import { Heart, Loader2, AlertCircle } from 'lucide-vue-next';
  import { useFavorites } from '~/composables/useFavorites';

  const searchLoading = ref(false);
  const searchResults = ref([]);
  const showModal = ref(false);
  const modalLoading = ref(false);
  const selectedMatch = ref(null);
  const editingMatch = ref(null);

  const {
    favorites,
    isLoading,
    error,
    fetchFavorites,
    addToFavorites,
    removeFromFavorites,
    updateFavorite,
  } = useFavorites();

  onMounted(fetchFavorites);

  const handleSearch = async (query: string) => {
    if (!query || query.length < 3) {
      searchResults.value = [];
      return;
    }

    searchLoading.value = true;
    try {
      const response = await $fetch('/api/v1/matches', {
        params: { query },
      });
      searchResults.value = response.matches;
    } catch (e) {
      console.error('Search error:', e);
      searchResults.value = [];
    } finally {
      searchLoading.value = false;
    }
  };
  const handleMatchSelect = (match: any) => {
    selectedMatch.value = match;
    showModal.value = true;
  };

  const handleEdit = (match: any) => {
    editingMatch.value = match;
    selectedMatch.value = match;
    showModal.value = true;
  };

  const handleRemove = async (id: string) => {
    try {
      await removeFromFavorites(id);
    } catch (e) {
      console.error('Remove error:', e);
    }
  };

  const handleSave = async (data: any) => {
    modalLoading.value = true;
    try {
      if (editingMatch.value) {
        await updateFavorite(editingMatch.value.favorite_id, data);
      } else {
        await addToFavorites(data);
      }
      showModal.value = false;
    } catch (e) {
      console.error('Save error:', e);
    } finally {
      modalLoading.value = false;
    }
  };

  const handleModalClose = () => {
    selectedMatch.value = null;
    editingMatch.value = null;
  };
</script>
