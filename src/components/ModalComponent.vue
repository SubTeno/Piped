<template>
    <div class="modal">
        <div @click="handleClick">
            <div class="modal-container">
                <button @click="$emit('close')"><font-awesome-icon icon="xmark" /></button>
                <slot></slot>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    mounted() {
        window.addEventListener("keydown", this.handleKeyDown);
    },
    unmounted() {
        window.removeEventListener("keydown", this.handleKeyDown);
    },
    methods: {
        handleKeyDown(event) {
            if (event.code === "Escape") {
                this.$emit("close");
            } else return;
            event.preventDefault();
        },
        handleClick(event) {
            if (event.target !== event.currentTarget) return;
            this.$emit("close");
        },
    },
};
</script>

<style>
h3 {
    @apply text-center;
}
.modal {
    @apply fixed z-50 top-0 left-0 w-full h-full bg-dark-900 bg-opacity-80 transition-opacity table;
}

.modal > div {
    @apply table-cell align-middle;
}

.modal-container {
    @apply w-fit m-auto min-w-[20vw] relative;
}

@media (max-width: 1024px) {
    .modal-container {
        @apply w-fit m-auto min-w-[40vw] relative;
    }
}

.modal-container > button {
    @apply float-right ml-40;
}
</style>
