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
import { createClient } from "@supabase/supabase-js";

const supabaseUrl = process.env.NUXT_PUBLIC_SUPABASE_URL || "";
const supabaseKey = process.env.NUXT_PUBLIC_SUPABASE_ANON_KEY || "";
const supabase = createClient(supabaseUrl, supabaseKey);

const spinboxValues = ref([0, 0, 0]);
const table = "spinboxes";

async function fetchSpinboxValues() {
  const { data } = await supabase
    .from(table)
    .select("value")
    .order("id", { ascending: true });
  if (data) {
    spinboxValues.value = data.map((row: any) => row.value);
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
  await supabase
    .from(table)
    .update({ value })
    .eq("id", idx + 1);
}
</script>

<style scoped>
.spinbox {
  font-size: 1.5rem;
}
</style>
