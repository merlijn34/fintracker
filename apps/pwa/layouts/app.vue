<script setup lang="ts">
import { SidebarProvider, SidebarTrigger } from "@/components/ui/sidebar";
</script>

<template>
  <div
    class="dark relative min-h-screen overflow-hidden bg-background text-foreground"
  >
    <SidebarProvider>
      <AppSidebar />
      <main>
        <!-- <SidebarTrigger /> -->
        <slot />
      </main>
    </SidebarProvider>
  </div>
</template>
