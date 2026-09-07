<script setup>
import { computed } from 'vue'
import { Icon } from 'invue/core'
// The resolved wrapper, not Base/ConfirmationModal.vue directly — a
// registry swap of just `actions.ConfirmationModal` still needs to apply
// here, same rule PanelLayout follows for Sidebar/Topbar.
import ConfirmationModal from '../ConfirmationModal.vue'
import { useInvueAction } from '../../composables/useInvueAction'

// Static color map — see the parent skill's Tailwind content-scanning
// note; this can't live in a shared .js module.
const BUTTON_CLASSES = {
    gray: 'bg-gray-800 text-white hover:bg-gray-700 dark:bg-gray-700 dark:hover:bg-gray-600',
    red: 'bg-red-600 text-white hover:bg-red-500',
    green: 'bg-green-600 text-white hover:bg-green-500',
    blue: 'bg-blue-600 text-white hover:bg-blue-500',
    yellow: 'bg-yellow-500 text-white hover:bg-yellow-400',
    amber: 'bg-amber-500 text-white hover:bg-amber-400',
    sky: 'bg-sky-600 text-white hover:bg-sky-500',
    rose: 'bg-rose-600 text-white hover:bg-rose-500',
    purple: 'bg-purple-600 text-white hover:bg-purple-500',
    pink: 'bg-pink-600 text-white hover:bg-pink-500',
}

// `variant: 'ghost'` — an icon-only, no-background button for repeating an
// action once per table row (see ActionsColumn's `trigger="inline"`) —
// a solid pill per row, per action, reads as too loud/heavy at that
// density. Text-only tint, same color names, no background.
const GHOST_CLASSES = {
    gray: 'text-gray-500 hover:bg-gray-100 hover:text-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-gray-300',
    red: 'text-red-500 hover:bg-red-50 hover:text-red-700 dark:text-red-400 dark:hover:bg-red-950 dark:hover:text-red-300',
    green: 'text-green-600 hover:bg-green-50 hover:text-green-700 dark:text-green-400 dark:hover:bg-green-950 dark:hover:text-green-300',
    blue: 'text-blue-500 hover:bg-blue-50 hover:text-blue-700 dark:text-blue-400 dark:hover:bg-blue-950 dark:hover:text-blue-300',
    yellow: 'text-yellow-600 hover:bg-yellow-50 hover:text-yellow-700 dark:text-yellow-400 dark:hover:bg-yellow-950 dark:hover:text-yellow-300',
    amber: 'text-amber-600 hover:bg-amber-50 hover:text-amber-700 dark:text-amber-400 dark:hover:bg-amber-950 dark:hover:text-amber-300',
    sky: 'text-sky-500 hover:bg-sky-50 hover:text-sky-700 dark:text-sky-400 dark:hover:bg-sky-950 dark:hover:text-sky-300',
    rose: 'text-rose-500 hover:bg-rose-50 hover:text-rose-700 dark:text-rose-400 dark:hover:bg-rose-950 dark:hover:text-rose-300',
    purple: 'text-purple-500 hover:bg-purple-50 hover:text-purple-700 dark:text-purple-400 dark:hover:bg-purple-950 dark:hover:text-purple-300',
    pink: 'text-pink-500 hover:bg-pink-50 hover:text-pink-700 dark:text-pink-400 dark:hover:bg-pink-950 dark:hover:text-pink-300',
}

const props = defineProps({
    label: {
        type: String,
        default: null,
    },
    icon: {
        type: String,
        default: null,
    },
    color: {
        type: String,
        default: 'gray',
    },
    // Given, this behaves like a styled <Link> (an Inertia visit). Omitted,
    // clicking just emits `click` — for a caller that wants to run its own
    // handler (e.g. a client-only Notify() call) instead of navigating.
    url: {
        type: String,
        default: null,
    },
    method: {
        type: String,
        default: 'get',
    },
    data: {
        type: Object,
        default: () => ({}),
    },
    disabled: {
        type: Boolean,
        default: false,
    },
    requiresConfirmation: {
        type: Boolean,
        default: false,
    },
    confirmationTitle: {
        type: String,
        default: 'Are you sure?',
    },
    confirmationText: {
        type: String,
        default: null,
    },
    confirmationButtonLabel: {
        type: String,
        default: null,
    },
    variant: {
        type: String,
        default: 'solid',
    },
})

const emit = defineEmits(['click', 'success'])

const { confirming, processing, run, confirm, cancel } = useInvueAction()

const action = computed(() => ({
    url: props.url,
    method: props.method,
    data: props.data,
    disabled: props.disabled,
    requiresConfirmation: props.requiresConfirmation,
    onClick: () => emit('click'),
    onSuccess: () => emit('success'),
}))

function handleClick() {
    run(action.value)
}
</script>

<template>
    <button
        v-if="variant === 'ghost'"
        type="button"
        class="inline-flex h-7 w-7 items-center justify-center rounded-md disabled:cursor-not-allowed disabled:opacity-60"
        :class="GHOST_CLASSES[color] ?? GHOST_CLASSES.gray"
        :disabled="disabled"
        :aria-label="label"
        :title="label"
        @click="handleClick"
    >
        <Icon v-if="icon" :name="icon" class="h-4 w-4" />
        <span v-if="!icon && label">{{ label }}</span>
    </button>
    <button
        v-else
        type="button"
        class="inline-flex items-center gap-1.5 rounded-md px-3 py-1.5 text-sm font-semibold disabled:cursor-not-allowed disabled:opacity-60"
        :class="BUTTON_CLASSES[color] ?? BUTTON_CLASSES.gray"
        :disabled="disabled"
        @click="handleClick"
    >
        <Icon v-if="icon" :name="icon" class="h-4 w-4" />
        <span v-if="label">{{ label }}</span>
    </button>

    <ConfirmationModal
        :open="confirming !== null"
        :title="confirmationTitle"
        :text="confirmationText"
        :confirm-label="confirmationButtonLabel ?? label ?? 'Confirm'"
        :color="color"
        :processing="processing"
        @confirm="confirm"
        @cancel="cancel"
    />
</template>
