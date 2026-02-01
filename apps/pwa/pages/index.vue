<script setup lang="ts"></script>

<template>
  <div class="relative min-h-screen overflow-hidden bg-black text-white">
    <div
      class="pointer-events-none absolute inset-0 opacity-70 [background:radial-gradient(900px_circle_at_50%_35%,rgba(255,255,255,0.09),transparent_60%)]"
    />
    <div
      class="pointer-events-none absolute inset-0 opacity-40 [background:radial-gradient(700px_circle_at_50%_65%,rgba(99,102,241,0.10),transparent_60%)]"
    />

    <div
      class="relative mx-auto flex min-h-screen max-w-md items-center px-6 py-16"
    >
      <section class="w-full">
        <div class="mb-10 text-center">
          <div
            class="mx-auto mb-6 grid size-14 place-items-center rounded-2xl bg-gradient-to-br from-fuchsia-500/35 via-indigo-500/25 to-cyan-400/30 ring-1 ring-white/10 shadow-[0_0_40px_rgba(59,130,246,0.15)]"
            aria-hidden="true"
          >
            <svg
              viewBox="0 0 24 24"
              class="size-7 text-white/90"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M6.5 8.25h3.75M13.75 8.25H17.5M6.5 12h11M6.5 15.75h11"
                stroke="currentColor"
                stroke-width="1.8"
                stroke-linecap="round"
              />
            </svg>
          </div>

          <h1 class="text-3xl font-medium tracking-tight sm:text-4xl">
            Welcome to fintracker
          </h1>
          <p class="mt-3 text-sm text-white/60">
            Choose how you’d like to continue.
          </p>
        </div>

        <div class="space-y-4">
          <NuxtLink
            to="/login"
            class="block w-full rounded-xl bg-white px-4 py-3 text-center text-sm font-medium text-black shadow-sm transition hover:bg-white/90 focus:outline-none focus-visible:ring-2 focus-visible:ring-white/25"
          >
            Log in
          </NuxtLink>

          <NuxtLink
            to="/register"
            class="block w-full rounded-xl border border-white/15 bg-white/5 px-4 py-3 text-center text-sm font-medium text-white transition hover:bg-white/10 focus:outline-none focus-visible:ring-2 focus-visible:ring-white/20"
          >
            Sign up
          </NuxtLink>
        </div>
      </section>
    </div>
  </div>
</template>
