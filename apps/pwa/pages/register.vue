<script setup lang="ts">
definePageMeta({
  layout: "auth",
});

type Invitation = {
  token: string;
  family: { name: string };
  inviter: { display_name: string };
  role: string;
};

const props = withDefaults(
  defineProps<{
    invitation?: Invitation | null;
    inviteCodeRequired?: boolean;
    selfHostedFirstLogin?: boolean;
    initialEmail?: string;
    initialInviteCode?: string;
  }>(),
  {
    invitation: null,
    inviteCodeRequired: false,
    selfHostedFirstLogin: false,
    initialEmail: "",
    initialInviteCode: "",
  },
);

const route = useRoute();

const invitation = computed(() => props.invitation ?? null);
const inviteCodeRequired = computed(() => props.inviteCodeRequired);
const selfHostedFirstLogin = computed(() => props.selfHostedFirstLogin);

const passwordVisible = ref(false);
const submitting = ref(false);
const errorMessage = ref<string>("");

const form = reactive({
  email:
    props.initialEmail ||
    (typeof route.query.email === "string" ? route.query.email : ""),
  invite_code:
    props.initialInviteCode ||
    (typeof route.query.invite === "string" ? route.query.invite : ""),
  password: "",
});

const requirements = computed(() => {
  const value = form.password || "";
  const lengthOk = value.length >= 8;
  const caseOk = /[a-z]/.test(value) && /[A-Z]/.test(value);
  const numberOk = /\d/.test(value);
  const specialOk = /[^A-Za-z0-9]/.test(value);
  return { lengthOk, caseOk, numberOk, specialOk };
});

function lineClass(ok: boolean) {
  return ok ? "bg-emerald-400" : "bg-white/10";
}

function reqTextClass(ok: boolean) {
  return ok ? "text-emerald-300" : "text-white/45";
}

async function onSubmit() {
  errorMessage.value = "";
  submitting.value = true;

  try {
    await $fetch("/api/registration", {
      method: "POST",
      body: {
        user: {
          email: form.email,
          password: form.password,
          invite_code:
            inviteCodeRequired.value && !invitation.value
              ? form.invite_code
              : undefined,
          invitation: invitation.value?.token ?? null,
        },
      },
    });

    await navigateTo("/");
  } catch (err: any) {
    errorMessage.value =
      err?.data?.error ??
      err?.data?.message ??
      (Array.isArray(err?.data?.errors) ? err.data.errors.join(", ") : "") ??
      "An error occurred. Please try again.";
  } finally {
    submitting.value = false;
  }
}
</script>

<template>
  <Card class="border-border/50 bg-background/80 backdrop-blur">
    <CardHeader class="text-center">
      <CardTitle class="text-2xl">Create an account</CardTitle>
      <CardDescription>
        Already have an account?
        <NuxtLink to="/login" class="font-medium text-primary underline-offset-4 hover:underline">
          Log in
        </NuxtLink>
      </CardDescription>
    </CardHeader>

    <CardContent class="grid gap-5">
      <div
        v-if="selfHostedFirstLogin"
        class="rounded-lg border border-border/60 bg-muted/30 px-4 py-3 text-center"
      >
        <p class="text-sm text-muted-foreground">
          Welcome! Create your account to get started.
        </p>
      </div>

      <div
        v-if="errorMessage"
        class="flex items-start gap-3 rounded-lg border border-destructive/40 bg-destructive/10 px-4 py-3"
      >
        <svg
          viewBox="0 0 24 24"
          class="mt-0.5 size-5 text-destructive"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
          aria-hidden="true"
        >
          <path
            d="M12 8v5"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
          />
          <path
            d="M12 16.5h.01"
            stroke="currentColor"
            stroke-width="3"
            stroke-linecap="round"
          />
          <path
            d="M10.3 4.35a2 2 0 0 1 3.4 0l7.1 12.3A2 2 0 0 1 19.1 20H4.9a2 2 0 0 1-1.7-3.05l7.1-12.6Z"
            stroke="currentColor"
            stroke-width="1.5"
            stroke-linejoin="round"
          />
        </svg>
        <p class="text-sm text-foreground">{{ errorMessage }}</p>
      </div>

      <form class="grid gap-4" @submit.prevent="onSubmit">
        <div class="grid gap-2">
          <Label for="email">Email</Label>
          <Input
            id="email"
            v-model.trim="form.email"
            type="email"
            autocomplete="email"
            required
            placeholder="you@example.com"
            :disabled="!!invitation"
          />
        </div>

        <div v-if="inviteCodeRequired && !invitation" class="grid gap-2">
          <Label for="invite_code">Invite code</Label>
          <Input
            id="invite_code"
            v-model.trim="form.invite_code"
            type="text"
            required
            placeholder="Enter your invite code"
          />
        </div>

        <input type="hidden" name="invitation" :value="invitation?.token ?? ''" />

        <div class="grid gap-2">
          <Label for="password">Password</Label>
          <div class="relative">
            <Input
              id="password"
              v-model="form.password"
              :type="passwordVisible ? 'text' : 'password'"
              autocomplete="new-password"
              required
              maxlength="72"
              placeholder="Create a password"
              class="pr-11"
            />
            <Button
              type="button"
              variant="ghost"
              size="icon-sm"
              class="absolute right-1 top-1/2 -translate-y-1/2"
              :aria-label="passwordVisible ? 'Hide password' : 'Show password'"
              @click="passwordVisible = !passwordVisible"
            >
              <svg
                v-if="!passwordVisible"
                viewBox="0 0 24 24"
                class="size-5"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
                aria-hidden="true"
              >
                <path
                  d="M2.5 12s3.5-7 9.5-7 9.5 7 9.5 7-3.5 7-9.5 7-9.5-7-9.5-7Z"
                  stroke="currentColor"
                  stroke-width="1.7"
                  stroke-linejoin="round"
                />
                <path
                  d="M12 15.25A3.25 3.25 0 1 0 12 8.75a3.25 3.25 0 0 0 0 6.5Z"
                  stroke="currentColor"
                  stroke-width="1.7"
                />
              </svg>
              <svg
                v-else
                viewBox="0 0 24 24"
                class="size-5"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
                aria-hidden="true"
              >
                <path
                  d="M4 4l16 16"
                  stroke="currentColor"
                  stroke-width="1.7"
                  stroke-linecap="round"
                />
                <path
                  d="M2.5 12s3.5-7 9.5-7c2.2 0 4.1.9 5.7 2"
                  stroke="currentColor"
                  stroke-width="1.7"
                  stroke-linejoin="round"
                />
                <path
                  d="M21.5 12s-3.5 7-9.5 7c-2.2 0-4.2-.9-5.8-2.1"
                  stroke="currentColor"
                  stroke-width="1.7"
                  stroke-linejoin="round"
                />
              </svg>
            </Button>
          </div>

          <div class="mt-2">
            <div class="flex gap-2">
              <div class="h-1 flex-1 rounded-full" :class="lineClass(requirements.lengthOk)" />
              <div class="h-1 flex-1 rounded-full" :class="lineClass(requirements.caseOk)" />
              <div class="h-1 flex-1 rounded-full" :class="lineClass(requirements.numberOk)" />
              <div class="h-1 flex-1 rounded-full" :class="lineClass(requirements.specialOk)" />
            </div>

            <div class="mt-3 space-y-2 text-sm">
              <div class="flex items-center gap-2" :class="reqTextClass(requirements.lengthOk)">
                <svg viewBox="0 0 24 24" class="size-4" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                  <path d="M20 6 9 17l-5-5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
                <span>At least 8 characters</span>
              </div>
              <div class="flex items-center gap-2" :class="reqTextClass(requirements.caseOk)">
                <svg viewBox="0 0 24 24" class="size-4" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                  <path d="M20 6 9 17l-5-5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
                <span>Upper &amp; lower case letters</span>
              </div>
              <div class="flex items-center gap-2" :class="reqTextClass(requirements.numberOk)">
                <svg viewBox="0 0 24 24" class="size-4" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                  <path d="M20 6 9 17l-5-5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
                <span>At least one number</span>
              </div>
              <div class="flex items-center gap-2" :class="reqTextClass(requirements.specialOk)">
                <svg viewBox="0 0 24 24" class="size-4" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                  <path d="M20 6 9 17l-5-5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
                <span>At least one special character</span>
              </div>
            </div>
          </div>
        </div>

        <Button type="submit" class="w-full" :disabled="submitting">
          {{ submitting ? "Creating account…" : "Sign up" }}
        </Button>
      </form>
    </CardContent>

    <CardFooter class="flex-col gap-2">
      <Button
        type="button"
        variant="link"
        class="h-auto px-0 text-muted-foreground"
        @click.prevent
      >
        Forgot your password?
      </Button>
    </CardFooter>
  </Card>
</template>
