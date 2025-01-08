<template>
  <div :class="{ hidden: filtered }">
    <label class="label flex cursor-pointer justify-between pl-1 pr-4 bg-base-100 hover:bg-base-200">
      <div class="flex items-center">
        <div v-if="len > 0"
          class="flex items-center gap-1 rounded"
          :class="{ 'cursor-pointer hover':'bg-base-200' }"
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

        <span class="label-text mr-2" :class="{ underline: searchMatches }">
          <slot name="display" v-bind="{ item: props.item }">
            {{ item.name }}
          </slot>
        </span>
      </div>
    <div class="flex items-center">
      <div class="w-0.5 mx-1 bg-primary" 
        style="transition: height ease-out 0.2s;" 
        :class="{ 'h-3': hasSelectedChild, 'h-0': !hasSelectedChild }"></div>
      <input v-model="isSelected" type="checkbox" :value="item" class="checkbox checkbox-primary checkbox-sm" />
    </div>
    </label>
    <div v-if="len > 0" :class="{ hidden: collapsed && !hasSearchedChild }" class="pl-2 bg-neutral-300">
      <TreeFilterItem
      ref="children"
      v-for="(v, i) in item.children"
      v-model="childrenSelected[i]"
      :key="v.id"
      :item="v"
      ></TreeFilterItem>
    </div>
  </div>
</template>

<script setup lang="ts">
  import type { TreeItem } from '~~/lib/api/types/data-contracts';
  import MdiChevronDown from "~icons/mdi/chevron-down";
  import MdiChevronRight from "~icons/mdi/chevron-right";

  type Props = {
    item: TreeItem,
    isSelected?: boolean,
    collapsed?: boolean
  }

  type ExposedProps = {
    item: TreeItem,
    selected: Ref<TreeItem[]>,
    isSelected: Ref<boolean>,
    filtered: Ref<boolean>,
    hasSelectedChild: Ref<boolean>,
    hasSearchedChild: Ref<boolean>,
  }

  const props = withDefaults(defineProps<Props>(), {
    isSelected: false,
    collapsed: true,
  });

  const searchFold = inject<Ref<string>>("searchFold");

  const collapsed = ref(props.collapsed);
  const searchMatches = computed(() => !!searchFold?.value && props.item.name.toLowerCase().includes(searchFold?.value || ""));
  const len = computed(() => props.item.children.length ?? 0);
  const isSelected = ref(props.isSelected);

  // Whether item should be hidden by search query
  const filtered = computed(() => !!searchFold?.value && !hasSearchedChild.value)
  
  const children = ref<ExposedProps[]>([]);
  const childrenSelected = ref<TreeItem[][]>([]);
  const selfSelected = computed(() => isSelected ? [props.item.id] : [])
  const selected = computed<TreeItem[]>(() => {
      return childrenSelected.value.flat(1).concat(isSelected.value ? [props.item] : []);
  });

  const hasSelectedChild = computed<boolean>(() => isSelected.value || children.value.find(v => v.hasSelectedChild) != null)
  const hasSearchedChild = computed<boolean>(() => searchMatches.value || children.value.find(v => v.hasSearchedChild) != null)

  const emit = defineEmits(["update:modelValue"]);
  const modelValue = defineModel<TreeItem[]>({default: []});

  // Toggles all children if collapsed (Maybe always is better?)
  watch(isSelected, (val, oldVal) => {
    if (val == oldVal) return;
    //if (!collapsed.value) return;

    for (let child of children.value) {
      child.isSelected = val;
    }
  })

  // Update tree if modelVale is changed
  watch(modelValue, val => {
    for (let i = 0; i < len.value; i++) {
      let child = children.value[i];

      let nested = child.item.children;
      let childVals = val.filter(v => nested.includes(v));
      if (child.selected.length != childVals.length && !childVals.every(v => child.selected))
        childrenSelected.value[i] = childVals;
    }
    isSelected.value = val.includes(props.item);
  })

  // update modelValue with selected
  watch(selected, val => modelValue.value = val);

  defineExpose<ExposedProps>({
    item: props.item,
    selected, // selected values in subtree
    isSelected, // whether this item itself is selected
    filtered, // whether is hidden by search
    hasSelectedChild,
    hasSearchedChild,
  })
</script>