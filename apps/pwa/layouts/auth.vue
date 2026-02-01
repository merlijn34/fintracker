<template>
  <div class="dark relative min-h-screen overflow-hidden bg-background text-foreground">
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
        <div class="mb-8 text-center">
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
        </div>

        <slot />
      </section>
    </div>
  </div>
</template>
