<template>
  <div>
    <h1 v-if="user">Welcome {{ user.email }}</h1>
    <button @click="logout" class="terminal-btn">[ Logout ]</button>
  </div>
</template>

<script setup>
const client = useSupabaseClient();
const router = useRouter();
const user = useSupabaseUser();
const logout = async () => {
  await client.auth.signOut();
  router.replace("/login");
};
</script>

<style scoped>
.terminal-btn {
  border: none;
  background: none;
  color: #39ff14;
  font-family: inherit;
  font-size: 1.1rem;
  margin-top: 1.2rem;
  margin-right: 0.5rem;
  margin-bottom: 0.5rem;
  display: inline-block;
  letter-spacing: 1px;
  cursor: pointer;
  box-shadow: none;
  transition: color 0.2s;
}
.terminal-btn:hover {
  color: #b6ffb6;
  background: none;
}
</style>
