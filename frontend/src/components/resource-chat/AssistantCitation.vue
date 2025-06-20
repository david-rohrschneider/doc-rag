<script setup lang="ts">
import routes from "@/api/routes";
import { useApi } from "@/api/use-api";
import { useResourceStore } from "@/stores/resource-store";
import type { Chunk } from "@/types/api";
import { storeToRefs } from "pinia";
import { Popover, Tag, Divider } from "primevue";
import { onMounted, ref, useTemplateRef } from "vue";

const { chunkId, citationNumber } = defineProps<{ chunkId: string; citationNumber: number }>();

const resourceStore = useResourceStore();
const { resources, fetchedChunks } = storeToRefs(resourceStore);

const hoverContent = ref<{ title: string; content: string; page?: number } | null>(null);

const { get } = useApi(routes.resources.get.getChunk);

const getHoverContent = async () => {
  let chunk = fetchedChunks.value.find((c) => c._id === chunkId);

  if (!chunk) {
    const response = await get<Chunk>({ routeParams: [chunkId] });
    if (!response) return;
    chunk = response.data;
    fetchedChunks.value.push(chunk);
  }

  const title = resources.value.find((r) => r._id === chunk.resource)!.title;
  hoverContent.value = {
    title,
    content: chunk.content,
    // @ts-expect-error: Maybe chunk.page_number is not defined
    page: chunk.page_number,
  };
};

const popover = useTemplateRef("popover");

const handleMouseEnter = (e: MouseEvent) => {
  if (!hoverContent.value) return;
  popover.value?.show(e);
};

const handleMouseLeave = () => popover.value?.hide();

onMounted(getHoverContent);
</script>

<template>
  <div class="assistant-citation" @mouseenter="handleMouseEnter" @mouseleave="handleMouseLeave">
    <Tag :style="{ padding: '2px 5px' }" :value="citationNumber" />
    <Popover ref="popover">
      <div v-if="hoverContent" class="hover-content">
        <h3>{{ hoverContent.title }}</h3>
        <small v-if="hoverContent.page !== undefined">Page {{ hoverContent.page + 1 }}</small>
        <Divider style="margin-top: 0.5rem; margin-bottom: 0.5rem" />
        <p>[...] {{ hoverContent.content }} [...]</p>
      </div>
    </Popover>
  </div>
</template>

<style scoped>
.assistant-citation {
  display: inline-block;
  cursor: pointer;
  vertical-align: text-bottom;
}

.hover-content {
  max-width: 400px;
  max-height: 300px;
  overflow-y: auto;
  padding: 0.5rem;
}
</style>
