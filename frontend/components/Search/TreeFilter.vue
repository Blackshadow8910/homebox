<template>
  <div ref="el" class="dropdown" :class="{ 'dropdown-open': dropdownOpen }">
    <button ref="btn" tabindex="0" class="btn btn-xs" @click="toggle">
      {{ label }} {{ len }} <MdiChevronDown class="size-4" />
    </button>
    <div tabindex="0" class="dropdown-content mt-1 w-64 rounded-md bg-base-100 shadow">
      <div class="mb-1 px-4 pt-4 shadow-sm">
        <input v-model="search" type="text" placeholder="Search…" class="input input-bordered input-sm mb-2 w-full" />
      </div>
      <div class="max-h-72 divide-y overflow-y-auto">
        <TreeFilterItem v-for="(v, i) in props.options" :key="v.id" v-model="childrenSelected[i]" :item="v">
          <template #display>
            <slot name="display">
              {{ v.name }}
            </slot>
          </template>
        </TreeFilterItem>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import TreeFilterItem from "./TreeFilterItem.vue";
  import MdiChevronDown from "~icons/mdi/chevron-down";
  import type { TreeItem } from "~~/lib/api/types/data-contracts";
  type Props = {
    label: string;
    options: TreeItem[];
    display?: string;
    modelValue: TreeItem[];
  };

  const btn = ref<HTMLButtonElement>();

  const search = ref("");
  const searchFold = computed(() => search.value.toLowerCase());
  const dropdownOpen = ref(false);
  const el = ref();

  provide("searchFold", searchFold);

  function toggle() {
    dropdownOpen.value = !dropdownOpen.value;

    if (!dropdownOpen.value) {
      btn.value?.blur();
    }
  }

  onClickOutside(el, () => {
    dropdownOpen.value = false;
  });

  const emit = defineEmits(["update:modelValue"]);
  const props = withDefaults(defineProps<Props>(), {
    label: "",
    display: "name",
    modelValue: () => [],
    uniqueField: "id",
  });

  const len = computed(() => {
    return selected.value.length > 0 ? `(${selected.value.length})` : "";
  });

  const childrenSelected = ref([]);

  const selected = computed<TreeItem[]>(() => {
    return childrenSelected.value.flat(1);
  });
  const modelValue = useVModel(props, "modelValue", emit);

  watch(selected, val => {
    modelValue.value = val.flat(1);
  });
</script>
