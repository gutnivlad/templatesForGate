<template>
    <div class="field">
        <div class="columns">
            <div class="column is-3"></div>
            <div class="column is-9">
                <p class="control">
                    <label
                        :for="elementId"
                        class="label"
                    >
                        <span>
                            <input
                                :id="elementId"
                                :disabled="isDisabled"
                                type="checkbox"
                                v-bind:checked="checked"
                                v-on:change="$emit('change', $event.target.checked)"
                                @click="$emit('click', [])"
                            >
                            {{ labelText }}
                        </span>
                    </label>
                </p>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'customCheckbox',
      //! v-model config, don`t touch it, mate
        model: {
            prop: 'checked',
            event: 'change'
        },
        props: {
          //! v-model prop, don`t touch it, mate
            checked: Boolean,
          // DOM element label or span text(introducing your checkbox text)
            labelText: {
                type: String,
                required: true,
            },
          // DOM element id
            elementId: {
                type: String,
                required: true,
            },
          // DOM element disabling
            isDisabled: {
                type: Boolean,
                default: false,
            },
        },
    };
</script>

<style scoped>

</style>
