<template>
  <div class="flex flex-col items-center justify-center min-h-screen">
    <div class="grid grid-cols-3 gap-8">
      <div
        v-for="(value, idx) in spinboxValues"
        :key="idx"
        class="flex flex-col items-center"
      >
        <label :for="`spinbox-${idx}`" class="mb-2"
          >Spinbox {{ idx + 1 }}</label
        >
        <input
          :id="`spinbox-${idx}`"
          type="number"
          class="spinbox border rounded px-3 py-2 text-center w-24"
          :value="value"
          @input="onInput(idx, ($event.target as HTMLInputElement)?.value)"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRuntimeConfig } from '#app';
import { createClient } from "@supabase/supabase-js";

const config = useRuntimeConfig();
const supabaseUrl = config.public.supabaseUrl;
const supabaseKey = config.public.supabaseAnonKey;
const supabase = createClient(supabaseUrl, supabaseKey);

const spinboxValues = ref<number[]>([0, 0, 0]);
const table = "spinboxes";

async function fetchSpinboxValues() {
  const { data, error } = await supabase
    .from(table)
    .select("id, values")
    .order("id", { ascending: true });
  if (error) {
    console.error('Supabase fetch error:', error.message, error.details, error.hint);
    return;
  }
  console.log('Fetched data from Supabase:', data); // Debug log
  if (data && data.length > 0) {
    const values = [0, 0, 0];
    data.forEach((row: any) => {
      if (row.id >= 1 && row.id <= 3) values[row.id - 1] = row.values;
    });
    spinboxValues.value = values;
  }
}

onMounted(async () => {
  await fetchSpinboxValues();
  supabase
    .channel("spinboxes")
    .on(
      "postgres_changes",
      { event: "UPDATE", schema: "public", table },
      (payload) => {
        fetchSpinboxValues();
      }
    )
    .subscribe();
});

async function onInput(idx: number, val: string) {
  const value = Number(val);
  // Only update if the value is different
  if (spinboxValues.value[idx] !== value) {
    const { error } = await supabase
      .from(table)
      .update({ values: value })
      .eq("id", idx + 1);
    if (!error) {
      spinboxValues.value[idx] = value;
    } else {
      console.error('Supabase update error:', error.message, error.details, error.hint);
    }
  }
}
</script>

<style scoped>
.spinbox {
  font-size: 1.5rem;
}
</style>
