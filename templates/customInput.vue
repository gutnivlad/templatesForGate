<template>
    <div class="field">
        <div class="columns">
            <div class="column is-3">
                <label class="label"
                       :for="elementId"
                       v-text="labelText"
                ></label>
            </div>
            <div class="column is-9">
                <p class="control">
                    <input
                        :id="elementId"
                        class="input"
                        type="text"
                        :class="{ 'is-danger': errors.length > 0 }"
                        v-bind:value="value"
                        v-on:input="$emit('input', $event.target.value)"
                        @keyup="$emit('keyup', [])"
                    >
                    <slot name="inputHelp"></slot>
                </p>
                <p class="help is-danger"
                   v-for="error in errors"
                   v-text="error"
                ></p>
                <p class="help is-danger"
                   v-if="htmlErrors.length"
                   v-for="error in htmlErrors"
                   v-html="error"
                ></p>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'customInput',
        props: {
            // v-model prop
            value: {
                type: String,
            },
            // text errors array
            errors: {
                type: Array,
                default: () => [],
            },
            // html errors array
            htmlErrors: {
                type: Array,
                default: () => [],
            },
            // DOM element label or span text(introducing your input text)
            labelText: {
                type: String,
                required: true,
            },
            // DOM element id
            elementId: {
                type: String,
                required: true,
            },
        },
    };
</script>

<style scoped>

</style>
