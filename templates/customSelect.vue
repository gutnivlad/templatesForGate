<template>
    <div class="field">
        <div class="columns">
            <div class="column is-3">
                <label
                    :for="elementId"
                    class="label"
                    v-text="labelText"></label>
            </div>
            <div class="column is-9">
                <div class="control is-expanded">
                    <div class="select is-fullwidth">
                        <select
                            :disabled="isDisabled"
                            :id="elementId"
                            class="select"
                            v-bind:value="selected"
                            v-on:input="myInput"
                        >
                            <option
                                v-if="defaultValue"
                                :value="defaultValue.value"
                                v-text="defaultValue.text"
                                selected
                                disabled
                            ></option>
                            <option v-for="option in selectOptions"
                                    :value="option.id"
                                    v-text="option.name"
                            ></option>
                        </select>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: 'customSelect',
        // v-model config
        model: {
            prop: 'selected',
            event: 'input'
        },
        props: {
            // v-model prop. You can change type, but don`t forget to look in model config
            selected: Number,
            // DOM element label or span text(introducing your checkbox text)
            labelText: {
                type: String,
                required: true,
            },
            selectOptions: {
                type: Array,
                default: () => [],
            },
            elementId: {
                type: String,
                required: true,
            },
            defaultValue: {
                type: Object,
            },
            isDisabled: {
                type: Boolean,
                default: false
            },
        },
        methods: {
            myInput(event) {
                const nubmeredValue = Number.parseInt(event.target.value);
                if (typeof nubmeredValue === 'number' && nubmeredValue === nubmeredValue){
                    this.$emit('input', nubmeredValue);
                    return true;
                }
                this.$emit('input', event.target.value);
            },
        },

    };
</script>

<style scoped>

</style>
