<template>
  <div class="flex flex-col items-center justify-center min-h-screen">
    <div class="flex flex-row gap-8">
      <div
        v-for="(value, idx) in spinboxValues"
        :key="idx"
        class="flex flex-row items-center w-40"
      >
        <label
          :for="`spinbox-${idx}`"
          class="mr-3 font-semibold text-lg text-right w-16"
        >
          {{ ["Breaker", "Max", "Viz"][idx] }}
        </label>
        <input
          :id="`spinbox-${idx}`"
          type="number"
          class="spinbox border rounded px-3 py-2 text-center flex-1"
          :value="value"
          @input="onInput(idx, ($event.target as HTMLInputElement)?.value)"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRuntimeConfig } from "#app";
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
    console.error(
      "Supabase fetch error:",
      error.message,
      error.details,
      error.hint
    );
    return;
  }
  console.log("Fetched data from Supabase:", data); // Debug log
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
        // Force a full page reload for all users when any spinbox is updated
        window.location.reload();
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
      console.error(
        "Supabase update error:",
        error.message,
        error.details,
        error.hint
      );
    }
  }
}
</script>

<style scoped>
label {
  display: block;
  margin-bottom: 0;
}
.spinbox {
  font-size: 1.5rem;
  margin-top: 0;
}
.flex-col {
  justify-content: flex-start;
}
.flex-row {
  align-items: flex-end;
}
</style>
