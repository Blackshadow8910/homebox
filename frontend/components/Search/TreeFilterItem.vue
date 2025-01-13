<template>
  <div :class="{ hidden: filtered }">
    <label class="label flex cursor-pointer justify-between bg-base-100 pl-1 pr-4 hover:bg-base-200">
      <div class="flex items-center">
        <div
          v-if="len > 0"
          class="flex items-center gap-1 rounded"
          :class="{ 'hover cursor-pointer': 'bg-base-200' }"
          @click="collapsed = !collapsed"
        >
          <label
            class="swap swap-rotate"
            :class="{
              'swap-active': !collapsed,
            }"
          >
            <MdiChevronRight name="mdi-chevron-right" class="swap-off size-5" />
            <MdiChevronDown name="mdi-chevron-down" class="swap-on size-5" />
          </label>
        </div>
        <div v-else class="size-5"></div>

        <span class="label-text mr-2" :class="{ 'font-semibold': searchMatches }">
          <slot name="display" v-bind="{ item: props.item }">
            {{ item.name }}
          </slot>
        </span>
      </div>
      <div class="flex items-center">
        <div
          class="mx-1 w-0.5 bg-primary"
          style="transition: height ease-out 0.2s"
          :class="{ 'h-3': hasSelectedChild, 'h-0': !hasSelectedChild }"
        ></div>
        <input v-model="isSelected" type="checkbox" :value="item" class="checkbox checkbox-primary checkbox-sm" />
      </div>
    </label>
    <div v-if="len > 0" :class="{ hidden: collapsed && !hasSearchedChild }" class="pl-2">
      <TreeFilterItem
        v-for="(v, i) in item.children"
        ref="children"
        :key="v.id"
        v-model="childrenSelected[i]"
        :item="v"
      ></TreeFilterItem>
    </div>
  </div>
</template>

<script setup lang="ts">
  import type { TreeItem } from "~~/lib/api/types/data-contracts";
  import type { ShallowRef } from "vue";
  import MdiChevronDown from "~icons/mdi/chevron-down";
  import MdiChevronRight from "~icons/mdi/chevron-right";

  type Props = {
    item: TreeItem;
    isSelected?: boolean;
    collapsed?: boolean;
  };

  type ExposedProps = {
    item: TreeItem;
    selected: TreeItem[];
    isSelected: boolean;
    filtered: boolean;
    hasSelectedChild: boolean;
    hasSearchedChild: boolean;
  };

  const props = withDefaults(defineProps<Props>(), {
    isSelected: false,
    collapsed: true,
  });

  const searchFold = inject<Ref<string>>("searchFold");

  const collapsed = ref(props.collapsed);
  const searchMatches = computed(
    () => !!searchFold?.value && props.item.name.toLowerCase().includes(searchFold?.value || "")
  );
  const len = computed(() => props.item.children.length ?? 0);
  const isSelected = ref(props.isSelected);

  // Whether item should be hidden by search query
  const filtered = computed(() => !!searchFold?.value && !hasSearchedChild.value);

  const children = ref<ExposedProps[]>([]);
  const childrenSelected = ref<TreeItem[][]>([]);
  const selfSelected = computed<TreeItem[]>(() => (isSelected.value ? [shallowRef(props.item).value] : []));
  const selected = computed<TreeItem[]>(() => {
    return childrenSelected.value.flat(1).concat(selfSelected.value);
  });

  const hasSelectedChild = computed<boolean>(() => {
    return isSelected.value || children.value.find(v => v.hasSelectedChild) != null;
  });
  const hasSearchedChild = computed<boolean>(() => {
    return searchMatches.value || children.value.find(v => v.hasSearchedChild) != null;
  });

  const modelValue = defineModel<TreeItem[]>({ default: [] });

  watch(isSelected, (val, oldVal) => {
    if (val === oldVal) return;

    for (const child of children.value) {
      child.isSelected = val;
    }
  });

  // Update tree if modelValue is changed
  watch(modelValue, val => {
    for (let i = 0; i < len.value; i++) {
      const child = children.value[i];

      const nested = child.item.children;
      const relevantVals = val.filter(v => nested.includes(v));
      if (child.selected.length !== relevantVals.length && !relevantVals.every(v => child.selected.includes(v)))
        childrenSelected.value[i] = relevantVals;
    }
    isSelected.value = val.includes(props.item);
  });

  // update modelValue with selected
  watch(selected, val => (modelValue.value = val));

  defineExpose({
    item: props.item,
    selected, // selected values in subtree
    isSelected, // whether this item itself is selected
    filtered, // whether is hidden by search
    hasSelectedChild,
    hasSearchedChild,
  });
</script>
