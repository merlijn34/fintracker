<script setup lang="ts">
definePageMeta({
  layout: "auth",
});

const route = useRoute();

const passwordVisible = ref(false);
const submitting = ref(false);
const errorMessage = ref<string>("");

const form = reactive({
  email: typeof route.query.email === "string" ? route.query.email : "",
  password: "",
});

async function onSubmit() {
  errorMessage.value = "";
  submitting.value = true;

  try {
    await $fetch("/api/session", {
      method: "POST",
      body: {
        user: {
          email: form.email,
          password: form.password,
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
      <CardTitle class="text-2xl">Sign in</CardTitle>
      <CardDescription>
        No account?
        <NuxtLink to="/register" class="font-medium text-primary underline-offset-4 hover:underline">
          Sign up
        </NuxtLink>
      </CardDescription>
    </CardHeader>

    <CardContent class="grid gap-5">
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
          />
        </div>

        <div class="grid gap-2">
          <Label for="password">Password</Label>
          <div class="relative">
            <Input
              id="password"
              v-model="form.password"
              :type="passwordVisible ? 'text' : 'password'"
              autocomplete="current-password"
              required
              maxlength="72"
              placeholder="Enter your password"
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
        </div>

        <Button type="submit" class="w-full" :disabled="submitting">
          {{ submitting ? "Signing in…" : "Log in" }}
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
