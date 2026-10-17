<template>
  <svg
    viewBox="0 0 24 24"
    :width="size"
    :height="size"
    :fill="filled ? 'currentColor' : 'none'"
    stroke="currentColor"
    stroke-width="2"
    stroke-linecap="round"
    stroke-linejoin="round"
    aria-hidden="true"
  >
    <polygon v-if="name === 'star'" points="12 3 14.8 8.9 21 9.6 16.4 13.8 17.7 20 12 16.9 6.3 20 7.6 13.8 3 9.6 9.2 8.9" />
    <path v-else-if="name === 'heart'" d="M12 20s-7-4.4-7-10a4 4 0 0 1 7-2.6A4 4 0 0 1 19 10c0 5.6-7 10-7 10z" />
    <path v-else-if="name === 'folder'" d="M3 7a2 2 0 0 1 2-2h4l2 2h8a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z" />
    <template v-else-if="name === 'plus'">
      <path d="M12 5v14" /><path d="M5 12h14" />
    </template>
    <template v-else-if="name === 'dice'">
      <rect x="4" y="4" width="16" height="16" rx="3" />
      <circle cx="9" cy="9" r="1.2" fill="currentColor" /><circle cx="15" cy="15" r="1.2" fill="currentColor" />
      <circle cx="15" cy="9" r="1.2" fill="currentColor" /><circle cx="9" cy="15" r="1.2" fill="currentColor" />
    </template>
    <template v-else-if="name === 'tuner'">
      <path d="M4 16a8 8 0 0 1 16 0" /><path d="M12 16l4-5" />
      <circle cx="12" cy="16" r="1.2" fill="currentColor" />
    </template>
    <template v-else-if="name === 'sort'">
      <path d="M7 4v16" /><path d="M4 17l3 3 3-3" /><path d="M17 20V4" /><path d="M14 7l3-3 3 3" />
    </template>
    <template v-else-if="name === 'shuffle'">
      <path d="M16 3h5v5" /><path d="M4 20L21 3" /><path d="M21 16v5h-5" /><path d="M15 15l6 6" /><path d="M4 4l5 5" />
    </template>
    <path v-else-if="name === 'chevron'" d="M6 9l6 6 6-6" />
  </svg>
</template>

<script>
export default {
  name: 'AppIcon',
  props: {
    name: { type: String, required: true },
    size: { type: [Number, String], default: 18 },
    filled: { type: Boolean, default: false }
  }
};
</script>
