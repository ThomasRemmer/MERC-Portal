<template>
  <div class="terminal-container">
    <h1>{{ animatedTitle }}</h1>
    <form class="animated-form" @submit.prevent="handleAnimatedSubmit">
      <div
        v-for="(field, i) in animatedFields"
        :key="field.id"
        v-show="field.show"
      >
        <div v-if="field.model !== 'displayName' || isSignUp">
          <label :for="field.id">{{ animatedLabels[i] }}</label>
          <input
            v-if="field.model === 'displayName' && isSignUp"
            :type="field.type"
            :id="field.id"
            v-model="displayName"
            required
            class="animated-input"
          />
          <input
            v-else-if="field.model === 'email'"
            :type="field.type"
            :id="field.id"
            v-model="email"
            required
            class="animated-input"
          />
          <input
            v-else-if="field.model === 'password'"
            :type="field.type"
            :id="field.id"
            v-model="password"
            required
            class="animated-input"
          />
          <span v-else class="animated-input">{{ animatedInputs[i] }}</span>
        </div>
      </div>
      <button
        v-if="animatedButtons[0].show"
        type="submit"
        class="animated-btn terminal-btn"
      >
        [ {{ isSignUp ? "Sign Up" : "Sign In" }} ]
      </button>
      <button
        v-if="animatedButtons[1].show"
        type="button"
        class="animated-btn terminal-btn"
        @click="toggleSignUp"
      >
        [
        {{
          isSignUp ? "Already have an account? sign in" : "Create a new account"
        }}
        ]
      </button>
    </form>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from "vue";
import { useRouter } from "vue-router";
import { useSupabaseUser } from "#imports";

const email = ref("");
const password = ref("");
const isSignUp = ref(false);
const client = useSupabaseClient();
const router = useRouter();
const user = useSupabaseUser();
const displayName = ref("");
const animatedTitle = ref("");

const animatedFields = ref([
  {
    label: "Name:",
    id: "displayName",
    type: "text",
    model: "displayName",
    show: false,
  },
  { label: "Email:", id: "email", type: "email", model: "email", show: false },
  {
    label: "Password:",
    id: "password",
    type: "password",
    model: "password",
    show: false,
  },
]);
const animatedButtons = ref([
  { type: "submit", text: "Sign Up", show: false },
  { type: "button", text: "Already have an account? sign in", show: false },
]);
const animatedLabels = ref(["", "", ""]);
const animatedInputs = ref(["", "", ""]);

const getTitle = () =>
  isSignUp.value ? "MERC-Portal Signup" : "MERC-Portal Login";

const animateTitle = async () => {
  const title = getTitle();
  animatedTitle.value = "";
  for (let i = 0; i < title.length; i++) {
    animatedTitle.value += title[i];
    await new Promise((r) => setTimeout(r, 32));
  }
};

onMounted(() => {
  setTimeout(async () => {
    if (!user.value) {
      const path = router.currentRoute.value.path;
      if (path !== "/login" && !path.startsWith("/debug")) {
        router.replace("/login");
        return;
      }
    }
    await animateTitle();
    animateForm();
  }, 500);
});

const signUp = async () => {
  const { error } = await client.auth.signUp({
    email: email.value,
    password: password.value,
  });
  if (!error) {
    router.push("/confirm");
    setTimeout(() => router.push("/home"), 1200);
  }
};

const signIn = async () => {
  const { error } = await client.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  });
  if (!error) {
    router.push("/confirm");
    setTimeout(() => router.push("/home"), 1200);
  }
};

function toggleSignUp() {
  isSignUp.value = !isSignUp.value;
  animatedFields.value.forEach((f) => (f.show = false));
  animatedButtons.value.forEach((b) => (b.show = false));
  animatedLabels.value = ["", "", ""];
  animatedInputs.value = ["", "", ""];
  animateTitle();
  animateForm();
}

const animateForm = async () => {
  for (let i = 0; i < animatedFields.value.length; i++) {
    animatedFields.value[i].show = true;
    await nextTick();
    const label = animatedFields.value[i].label;
    animatedLabels.value[i] = "";
    for (let j = 0; j < label.length; j++) {
      animatedLabels.value[i] += label[j];
      await new Promise((r) => setTimeout(r, 30));
    }
    animatedInputs.value[i] = "";
    for (let k = 0; k < 8; k++) {
      animatedInputs.value[i] += "_";
      await new Promise((r) => setTimeout(r, 18));
    }
    await new Promise((r) => setTimeout(r, 80));
  }
  for (let i = 0; i < animatedButtons.value.length; i++) {
    animatedButtons.value[i].show = true;
    await new Promise((r) => setTimeout(r, 120));
  }
};

async function handleAnimatedSubmit() {
  if (isSignUp.value) {
    await signUp();
  } else {
    await signIn();
  }
}
</script>

<style scoped>
.terminal-container {
  background: rgba(16, 28, 17, 0.28);
  backdrop-filter: blur(2.5px) brightness(1.08) saturate(1.2);
  -webkit-backdrop-filter: blur(2.5px) brightness(1.08) saturate(1.2);
  box-shadow: 0 0 32px 4px #39ff1422 inset, 0 0 8px 0 #39ff1444;
  color: #39ff14;
  font-family: "Share Tech Mono", "Fira Mono", "Consolas", "Monaco", monospace;
  padding: 2rem 2.5rem;
  border-radius: 8px;
  max-width: 420px;
  margin: 3rem auto;
  border: 2px solid #39ff14;
  position: relative;
  overflow: hidden;
  min-height: 480px;
  min-width: 420px;
}
.terminal-container h1 {
  color: #39ff14;
  text-align: center;
  margin-bottom: 2rem;
  letter-spacing: 2px;
  text-shadow: 0 0 8px #39ff14, 0 0 2px #39ff14;
  font-size: 1.5rem;
}
.terminal-container label {
  color: #39ff14;
  display: block;
  margin-bottom: 0.5rem;
  font-size: 1rem;
  text-shadow: 0 0 4px #39ff14;
}
.terminal-container input {
  background: transparent;
  color: #39ff14;
  border: 1.5px solid #39ff14;
  border-radius: 2px;
  padding: 0.5rem;
  width: 100%;
  margin-bottom: 1.5rem;
  font-size: 1rem;
  outline: none;
  box-shadow: 0 0 8px #39ff1433 inset;
  text-shadow: 0 0 2px #39ff14;
  transition: border 0.2s, background 0.2s;
}
.terminal-container input:-webkit-autofill,
.terminal-container input:-webkit-autofill:focus,
.terminal-container input:-webkit-autofill:hover,
.terminal-container input:-webkit-autofill:active {
  background-color: #101c11;
  background-image: none;
  color: #39ff14;
  -webkit-text-fill-color: #39ff14;
  box-shadow: 0 0 0 1000px #101c11 inset;
  border: 1.5px solid #39ff14;
  text-shadow: 0 0 2px #39ff14;
  transition: background-color 9999s ease-in, color 0.2s;
  caret-color: #39ff14;
}
input:-internal-autofill-selected {
  background-color: #101c11;
  background-image: none;
  color: #39ff14;
  -webkit-text-fill-color: #39ff14;
  box-shadow: 0 0 0 1000px #101c11 inset;
  border: 1.5px solid #39ff14;
  text-shadow: 0 0 2px #39ff14;
  transition: background-color 9999s ease-in, color 0.2s;
  caret-color: #39ff14;
}
.terminal-container button {
  background: transparent;
  color: #39ff14;
  border: 1.5px solid #39ff14;
  border-radius: 2px;
  padding: 0.5rem 1.5rem;
  margin-right: 0.5rem;
  margin-bottom: 0.5rem;
  font-size: 1rem;
  cursor: pointer;
  text-shadow: 0 0 4px #39ff14;
  transition: background 0.2s, color 0.2s, box-shadow 0.2s;
}
.terminal-container button:hover {
  background: #39ff14;
  color: #101c11;
  box-shadow: 0 0 16px #39ff14;
}
.terminal-container form {
  position: relative;
  z-index: 2;
}
.terminal-container input::placeholder {
  color: #39ff1488;
  opacity: 1;
  text-shadow: 0 0 2px #39ff14;
}
.animated-form {
  margin-top: 1.5rem;
}
.animated-form label {
  color: #39ff14;
  font-family: inherit;
  font-size: 1rem;
  margin-bottom: 0.2rem;
  display: block;
  letter-spacing: 1px;
  text-shadow: 0 0 4px #39ff14;
}
.animated-input {
  color: #39ff14;
  font-family: inherit;
  font-size: 1.1rem;
  letter-spacing: 2px;
  margin-bottom: 1.2rem;
  display: inline-block;
}
.animated-btn {
  color: #39ff14;
  font-family: inherit;
  font-size: 1.1rem;
  margin-top: 1.2rem;
  display: inline-block;
  letter-spacing: 1px;
}
.terminal-btn {
  border: none !important;
  background: none;
  padding: 0.5rem 0;
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
