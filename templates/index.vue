<template>
    <article class="box sms_wrap mt-5 overflow-custom">
        <div class="templates_wrap">
            <div class="level custom-sm-flex">
                        <div class="control mb-sm-1">
                            <input
                              type="text"
                              ref="search"
                              class="input"
                              :placeholder="$t('templates.search')"
                              v-model="searchKey"
                            >
                        </div>
                    <div class="has-text-right buttons">
                        <button type="button" class="button is-primary w-sm-100"
                                @click="addTemplate()"
                                v-text="$t('sms.new_template')"
                        ></button>
                    </div>
            </div>
            <table class="table is-bordered is-striped is-narrow is-hoverable is-fullwidth is-responsive-table">
                <thead>
                <tr>
                    <th @click="sortField('date')">
                        <i
                          class="fa sort-arrow"
                          :class="sortType === 'date' ? 'fa-sort-down' : 'fa-sort-up'"
                        ></i>
                        {{ $t('default.date') }}
                    </th>
                    <th @click="sortField('type')">
                        <i
                            class="fa sort-arrow"
                            :class="sortType === 'type' ? 'fa-sort-down' : 'fa-sort-up'"
                        ></i>
                        {{ $t('default.template_type') }}
                    </th>
                    <th @click="sortField('name')">
                        <i
                            class="fa sort-arrow"
                            :class="sortType === 'name' ? 'fa-sort-down' : 'fa-sort-up'"
                        ></i>
                        {{ $t('default.name') }}
                    </th>
                    <th @click="sortField('sender')">
                        <i
                            class="fa sort-arrow"
                            :class="sortType === 'sender' ? 'fa-sort-down' : 'fa-sort-up'"
                        ></i>
                        {{ $t('sms.text_sender') }}
                    </th>
                    <th v-text="$t('sms.text_message')"></th>
                    <th v-text="$t('default.actions')"></th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="template in filteredTemplates">
                    <td v-text="getCreatedAtDate(template)" :data-th="$t('default.date')"></td>
                    <td v-text="template.type_id === 1 ? 'SMS' : 'Viber'" :data-th="'Тип'"></td>
                    <td class="limited-field" v-text="getTemplateName(template)" :data-th="$t('default.name')"></td>
                    <td v-text="showSender(template)" :data-th="$t('sms.text_sender')"></td>
                    <td class="limited-field" v-text="getTemplateMessage(template)" :data-th="$t('sms.text_message')"></td>
                    <td :data-th="$t('default.actions')">
                        <div class="d-flex justify-content-center">
                            <button :title="$t('templates.edit_button')" class="button is-small is-warning mx-01" @click="editTemplate(template.id)">
                                    <span class="icon is-small">
                                      <i class="fa fa-edit"></i>
                                    </span>
                            </button>
                            <button :title="$t('templates.delete_button')" class="button is-small is-danger mx-01" @click="removeTemplate(template.id)">
                                    <span class="icon is-small">
                                      <i class="fa fa-times"></i>
                                    </span>
                            </button>
                        </div>
                    </td>
                </tr>
                </tbody>
            </table>
        </div>

        <div class="pagination">
        </div>

        <modal :isActive="isModal('new_template')">
            <h1 slot="header" v-text="$t('sms.template')"></h1>

            <!--template type (isSMS)-->
            <customSelect
                v-bind:is-disabled="action_type !== 'add'"
                v-bind:element-id="'template_type'"
                v-bind:label-text="$t('templates.type_select')"
                v-bind:select-options="source_data.typeSelect"
                v-model="isSMS"
                @change.native="setTemplatesToDefault()"
            ></customSelect>

            <!--template name(имя шаблона)-->
            <customInput
                v-bind:element-id="'name'"
                v-model="currentTemplate.data.name"
                @keyup="form_errors.template_name = []"
                v-bind:label-text="$t('sms.name_template')"
                v-bind:errors="form_errors.template_name"
            ></customInput>

            <!--mailing_name(имя рассылки)-->
            <customInput
                v-bind:element-id="'mailing_name'"
                v-model="currentTemplate.data.mailing_name"
                v-bind:label-text="$t('sms.text_name')"
            ></customInput>

            <!--sender_id-->
            <customSelect
                v-bind:element-id="'sender_id'"
                v-bind:label-text="$t('sms.text_sender')"
                v-bind:select-options="computedSenders"
                v-model="currentTemplate.data.sender_id"
                v-bind:default-value="{ value: 0, text: $t('sms.default_sender') }"
            ></customSelect>

            <!--need_text(добавить текст для типа вайбер)-->
            <customCheckbox
                v-bind:element-id="'need_text'"
                v-bind:label-text="$t('viber.add_text')"
                v-show="!isSMS"
                v-model="new_viber_template.data.need_text"
            ></customCheckbox>

            <!--Type error-->
            <div class="field" v-show="!isSMS && form_errors.types.length > 0">
                <div class="columns">
                    <div class="column is-3"></div>
                    <div class="column is-9">
                        <p class="help is-danger"
                           v-for="str_error in form_errors.types"
                           v-text="str_error"
                        ></p>
                    </div>
                </div>
            </div>

            <!--Main message textarea, symbol and messages counters-->
            <div class="field" v-show="isSMS || new_viber_template.data.need_text">
                <div class="columns">
                    <div class="column is-3">
                        <label for="message" class="label" v-text="$t('sms.text_message')"></label>
                    </div>
                    <div class="column is-9">
                        <p class="control">
                            <textarea id="message" class="textarea"
                                      :class="{'is-danger': form_errors.message.length > 0}"
                                      @keyup="form_errors.message = []"
                                      @input="throttledMessage"
                                      v-model="currentTemplate.data.message"
                            ></textarea>
                        </p>
                        <p class="help has-text-right" id="message_counterViber">
                            <span v-text="countSymbols.full"></span>,
                            &nbsp;
                            <span
                                v-show="!isSMS"
                                v-text="$t('viber.notify_limit_symbols', { limit: limitMessageSymbols})"
                            ></span>
                            <span
                                v-show="isSMS"
                                v-text="countMessagesSMS"></span>
                            &nbsp;
                        </p>
                        <p class="help is-danger"
                           v-for="message_error in form_errors.message"
                           v-text="message_error"
                        ></p>
                    </div>
                </div>
            </div>

            <!--need_image(добавить изображение для типа вайбер)-->
            <customCheckbox
                v-show="!isSMS"
                v-bind:element-id="'need_image'"
                v-bind:label-text="$t('viber.add_image')"
                @click="form_errors.types = []"
                v-model="new_viber_template.data.need_image"
            ></customCheckbox>

            <!--image_url(URL картинки)-->
            <customInput
                v-bind:element-id="'image_url'"
                v-show="new_viber_template.data.need_image"
                v-model="new_viber_template.data.url_image"
                @keyup="form_errors.image = []; form_errors.image_url = [];"
                v-bind:label-text="$t('templates.image_url_label')"
                v-bind:errors="form_errors.image"
                v-bind:html-errors="form_errors.image_url"
            >
                <template v-slot:inputHelp>
                    <span class="help" v-text="$t('viber.notify_url_image')"></span>
                </template>
            </customInput>

            <!--Need button-->
            <customCheckbox
                v-show="!isSMS"
                v-bind:element-id="'need_button'"
                v-bind:label-text="$t('viber.add_button')"
                v-model="new_viber_template.data.need_button"
                @click="form_errors.types = []"
            ></customCheckbox>

            <!--button_name(название кнопки)-->
            <customInput
                v-show="new_viber_template.data.need_button"
                v-bind:element-id="'button_name'"
                v-model="new_viber_template.data.button_name"
                @keyup="form_errors.button_name = []"
                v-bind:label-text="$t('viber.text_button_name')"
                v-bind:errors="form_errors.button_name"
            >
                <template v-slot:inputHelp>
                    <span class="help" v-text="$t('viber.notify_limit_symbols', { limit: limitButtonSymbols})"></span>
                </template>
            </customInput>

            <!--button_url(URL кнопки)-->
            <customInput
                v-show="new_viber_template.data.need_button"
                v-bind:element-id="'button_url'"
                v-bind:label-text="$t('viber.text_button_url')"
                v-model="new_viber_template.data.button_url"
                @keyup="form_errors.button_url = []"
                v-bind:html-errors="form_errors.button_url"
            >
                <template v-slot:inputHelp>
                    <p class="help" v-html="$t('viber.notify_button_url')"></p>
                </template>
            </customInput>

            <!--SMS resend message textarea, symbol and messages counters-->
            <div class="field" v-show="!isSMS">
                <customCheckbox
                    v-bind:element-id="'sms_resend'"
                    v-bind:label-text="$t('viber.text_sms_resend')"
                    v-model="new_viber_template.data.sms_resend"
                ></customCheckbox>

                <div v-show="new_viber_template.data.sms_resend">

                    <!--Choose your sms_sender_id(имя отправителя СМС)-->
                    <customSelect
                        v-bind:element-id="'sms_sender_id'"
                        v-bind:label-text="$t('sms.text_sender')"
                        v-bind:select-options="source_data.senders"
                        v-model="new_viber_template.data.sms_sender_id"
                        v-bind:default-value="{ value: 0, text: $t('sms.default_sender') }"
                    ></customSelect>

                    <div class="field">
                        <div class="columns">
                            <div class="column is-3">
                                <label for="sms_message" class="label">Текст SMS</label>
                            </div>
                            <div class="column is-9">
                                <p class="control">
                                    <textarea id="sms_message" class="textarea"
                                              :class="{'is-danger': form_errors.sms_message.length > 0}"
                                              @keyup="form_errors.sms_message = []"
                                              v-model="new_viber_template.data.sms_message"
                                    ></textarea>
                                </p>
                                <p class="help has-text-right" id="message_counter_sms_resend">
                                    <span v-text="countSymbolsSMSResend.full"></span>,
                                    &nbsp;
                                    <span v-text="countMessagesSMSResend"></span>
                                    &nbsp;
                                </p>
                                <p class="help is-danger"
                                   v-for="message_error in form_errors.sms_message"
                                   v-text="message_error"
                                ></p>
                            </div>
                        </div>
                    </div>

                    <customCheckbox
                        v-bind:element-id="'translitViber'"
                        v-bind:label-text="$t('sms.text_translit')"
                        v-model="new_viber_template.data.sms_translit"
                    ></customCheckbox>

                    <!--Choose your lifetimes(срок жизни СМС)-->
                    <customSelect
                        v-bind:element-id="'sms_lifetime'"
                        v-bind:label-text="$t('sms.text_lifetime') + ' ' + $t('sms.title')"
                        v-bind:select-options="source_data.lifetimes"
                        v-bind:default-value="{ value: maxLifetimes, text: $t('sms.max_lifetime') }"
                        v-model="new_viber_template.data.sms_lifetime"
                    ></customSelect>
                </div>
            </div>

            <!--translit(для типа СМС)-->
            <customCheckbox
                v-show="isSMS"
                v-bind:element-id="'translit'"
                v-bind:label-text="$t('sms.text_translit')"
                v-model="new_sms_template.data.translit"
            ></customCheckbox>

            <!--delay_send(Запланировать рассылку)-->
            <customCheckbox
                v-bind:element-id="'delay_send'"
                v-bind:label-text="$t('sms.text_planning')"
                v-model="currentTemplate.data.delay_send"
            ></customCheckbox>

            <!--Large delay_send area, a couple of selects and fuck knows what else, mate-->
            <div v-show="currentTemplate.data.delay_send">
                <div class="field" v-show="isSMS">
                    <div class="columns">
                        <div class="column is-3">
                            <label for="type" class="label" v-text="$t('sms.text_type')"></label>
                        </div>
                        <div class="column is-9">
                            <div class="control is-expanded">
                                <div class="select is-fullwidth">
                                    <select id="type" v-model="new_sms_template.data.type">
                                        <option v-for="d_type in source_data.types"
                                                :value="d_type.value"
                                                v-text="$t(d_type.name)"
                                        ></option>
                                    </select>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="field" v-show="isSMS">
                    <div class="columns">
                        <div class="column is-3">
                            <label for="period" class="label" v-text="$t('sms.text_period')"></label>
                        </div>
                        <div class="column is-9">
                            <div class="control is-expanded">
                                <div class="select is-fullwidth">
                                    <select id="period" v-model="new_sms_template.data.period">
                                        <option
                                            v-for="period in source_data.period"
                                            :value="period.value"
                                            v-text="$t(period.name)"
                                        ></option>
                                    </select>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="field">
                    <div class="columns">
                        <div class="column is-3"></div>
                        <div class="column is-9">
                            <p class="control">
                                <label for="local_time" class="label">
                                    <span>
                                        <input id="local_time" type="checkbox" v-model="currentTemplate.data.local_time">
                                        &nbsp;{{ $i18n.t('sms.text_localtime') }}
                                    </span>
                                </label>
                            </p>
                        </div>
                    </div>
                </div>
            </div>

            <!--smooth_send(плавная отсылка)-->
            <customCheckbox
                v-bind:element-id="'smooth_send'"
                v-bind:label-text="$t('sms.text_smooth')"
                v-model="currentTemplate.data.smooth_send"
                v-bind:is-disabled="currentTemplate.data.local_time || new_sms_template.data.type === 'birthday'"
            ></customCheckbox>

            <!--smooth_time (длительность отправки плавной отсылки)-->
            <customSelect
                v-show="currentTemplate.data.smooth_send"
                v-bind:element-id="'smooth_time'"
                v-bind:label-text="$t('sms.text_smooth_time')"
                v-bind:select-options="source_data.lifetimes"
                v-model="currentTemplate.data.smooth_time"
            ></customSelect>

            <!--paused(создать остановленную рассылку)-->
            <customCheckbox
                v-bind:element-id="'paused'"
                v-bind:label-text="$t('sms.text_paused')"
                v-model="new_sms_template.data.paused"
            ></customCheckbox>

            <!--lifetime (срок жизни смс или вайбер рассылки)-->
            <customSelect
                v-bind:element-id="'lifetime'"
                v-bind:label-text="computedLifetimeLabel"
                v-bind:select-options="source_data.lifetimes"
                v-model="currentTemplate.data.lifetime"
                v-bind:default-value="{ value: maxLifetimes, text: $t('sms.max_lifetime') }"
            ></customSelect>


            <div slot="footer">
                <button class="button is-warning" @click.prevent="closeModal()" v-text="$t('default.cancel')"></button>
                <button class="button is-primary"
                        :disabled="computedDisabled"
                        @click.prevent="createTemplate()"
                        v-text="$t('default.save')"
                ></button>
            </div>
        </modal>

        <message-modal
            :isActive="isModal('message_modal')"
            :messageType="message_type"
            :timer="timer_modal"
        >
            <div v-html="message_modal"></div>
        </message-modal>

        <loading class="centered" v-show="loading"></loading>
    </article>
</template>

<script>
    // api
    import Templates from '../../api/user/templates.js';
    import Senders from '../../api/sms/senders.js';
    import ViberSenders from '../../api/viber/senders.js';
    // components
    import Modal from '../common/modal.vue';
    import MessageModal from '../common/message_modal.vue';
    import loading from '../common/loading.vue';
    import customInput from './customInput.vue';
    import customSelect from './customSelect.vue';
    import customCheckbox from './customCheckbox.vue';
    // libs
    import { defaultTo } from 'lodash';
    import Lifetimes from '../../api/sms/lifetimes.js';
    import { period, types } from '../common/variables.js';

    // mixins
    import GeneralMixin from '../../vue_mixins/general_mixin.js';

    export default {
        components: {
            modal: Modal,
            'message-modal': MessageModal,
            loading,
            customInput,
            customSelect,
            customCheckbox,
        },

        mixins: [GeneralMixin],

        created() {
            this.setTemplatesToDefault();
            // source data
            this.getTemplates();
            this.getSenders();
            this.getLifetimes();
            // loader
            this.loading = true;
        },

        mounted() {
            // modal event
            this.$events.listen('closeModal', () => {
                this.closeModal();
            });
        },

        data(){
            return {
                loading: true,
                limitMessageSymbols: 853,
                isSMS: 1,
                limitButtonSymbols: 19,
                new_sms_template: null,
                new_viber_template: null,
                action_type: '',
                sortType: 'date',
                searchKey: '',
                source_data: {
                    lifetimes: defaultTo(this.$store.get('lifetimes'), []),
                    current_user: this.$store.get('Laravel').current_user,
                    senders: defaultTo(this.$store.get('sms_senders'), []),
                    viberSenders: defaultTo(this.$store.get('viber_senders'), []),
                    templates: defaultTo(this.$store.get('templates'), []),
                    types: types,
                    period: period,
                    typeSelect: [
                        {
                            name: 'Viber',
                            id: 0,
                        },
                        {
                            name: 'SMS',
                            id: 1,
                        },
                    ],
                },
                form_errors: {
                    name: [],
                    message: [],
                    types: [],
                    sms_message: [],
                    image: [],
                    button_name: [],
                    button_url: [],
                    image_url: [],
                    template_name: [],
                }
            };
        },

        methods: {
            // helpers
            checkMessage(){
                if (this.isSMS) return true;

                let isGoodMessage = true;

                while (this.countSymbols.message_length > this.limitMessageSymbols) {
                    this.new_viber_template.data.message = this.new_viber_template.data.message.substring(0, this.new_viber_template.data.message.length - 10);
                    isGoodMessage = false;
                }

                return isGoodMessage;
            },
            setTemplatesToDefault() {
                // форматирую дату
                const formDate = new Intl.DateTimeFormat('ru', {
                    year: 'numeric',
                    month: '2-digit',
                    day: '2-digit',
                    hour: '2-digit',
                    minute: '2-digit',
                }).format(Date.now());

                this.new_viber_template = {
                    type_id: 2,
                    data: {
                        name: `Viber ${this.$i18n.t('templates.template_name')} ${formDate}`,
                        message: '',
                        sender_id: 0,
                        mailing_name: `Viber ${this.$i18n.t('templates.mailing_name')} ${formDate}`,
                        paused: false,
                        delay_send: false,
                        local_time: false,
                        smooth_send: false,
                        smooth_time: 5,
                        lifetime: this.maxLifetimes,
                        url_image: '',
                        button_name: '',
                        button_url: '',
                        sms_sender_id: 0,
                        sms_message: '',
                        sms_translit: false,
                        sms_lifetime: this.maxLifetimes,
                        need_text: false,
                        need_image: false,
                        sms_resend: false,
                    },
                };
                this.new_sms_template = {
                    type_id: 1,
                    data: {
                        name: `SMS ${this.$i18n.t('templates.template_name')} ${formDate}`,
                        mailing_name: `SMS ${this.$i18n.t('templates.mailing_name')} ${formDate}`,
                        message: '',
                        sender_id: 0,
                        translit: false,
                        paused: false,
                        delay_send: false,
                        type: 'usual',
                        period: 'single',
                        local_time: false,
                        smooth_send: false,
                        smooth_time: 5,
                        lifetime: this.maxLifetimes
                    },
                };
                this.form_errors = {
                    name: [],
                    recipients_ids: [],
                    types: [],
                    message: [],
                    sms_message: [],
                    image: [],
                    button_name: [],
                    button_url: [],
                    image_url: [],
                    template_name: []
                };
            },
            // filter and set templates to their data
            templatesFilter(array_templates) {
                if(!array_templates || !array_templates.length) {
                    this.$store.set('viberTemplates', []);
                    this.$store.set('smsTemplates', []);
                    return true;
                }
                const viber_templates = [];
                const sms_templates = [];
                array_templates.forEach(template => {
                    if(template.type_id === 1) sms_templates.push(template);
                    if(template.type_id === 2) viber_templates.push(template);
                });
                this.$store.set('viberTemplates', viber_templates);
                this.$store.set('smsTemplates', sms_templates);
            },

            // source data
            getSenders(){
                Senders.list().then((result) => {
                    this.source_data.senders = result.data.senders;
                    this.$store.set('sms_senders', result.data.senders);
                }).catch((err) => {
                    console.error('Error: ' + err.toString() )
                });
                ViberSenders.list()
                    .then((result) => {
                        this.source_data.viberSenders = result.data.senders;
                        this.$store.set('viber_senders', result.data.senders);
                    })
                    .catch((err) => {
                        console.error('Error: ' + err.toString() )
                    });
            },
            getTemplates(){
                this.loading = true;
                Templates.list().then((result) => {
                    this.source_data.templates = result.data;
                    this.$store.set('templates', result.data);
                    this.loading = false;
                    this.sortType = 'date';
                    window.setTimeout(this.templatesFilter, 700, this.source_data.templates || []);
                }).catch((err) => {
                    console.error('Error: ' + err.toString() );
                    this.showModalMessage(this.$i18n.t('templates.load_error'), 'error', 2000);
                    this.loading = false;
                });
            },
            getLifetimes(){
                Lifetimes.list().then((result) => {
                    this.source_data.lifetimes = result.data;
                    this.$store.set('lifetimes', result.data);
                    this.new_sms_template.data.lifetime = this.maxLifetimes;
                }).catch((err) => {
                    console.error('Error: ' + err.toString() )
                });
            },

            // templates
            getTemplateMessage(template) {
              return template?.data?.message || '';
            },

            showSender(template){
                let id = template?.data?.sender_id;
                if(typeof this.source_data.senders === 'undefined'){
                    return null;
                }
                let sender_name = this.$i18n.t('sms.default_sender'),
                    sender = this.source_data.senders.filter((item) => item.id === id);
                if(sender.length){
                    sender_name = (sender.pop()).name;
                }
                return sender_name
            },

            getCreatedAtDate(template) {
                const unformattedDate = template.created_at || template.updated_at || ' - ';
                if (unformattedDate === ' - ') {
                    return ' - ';
                }
                return `${ new Date(unformattedDate).toLocaleDateString() }, ${ new Date(unformattedDate).toLocaleTimeString() }`;
            },

            createTemplate(){
                    if(this.action_type === 'add'){
                        this.saveTemplate();
                    } else {
                        this.updateTemplate();
                    }
            },

            addTemplate(){
                this.setTemplatesToDefault();
                this.action_type = 'add';
                this.showModal('new_template');
            },

            checkNullFields() {
                if (!this.currentTemplate.data) {
                    this.setTemplatesToDefault();
                }
                if (!this.currentTemplate.data.name) {
                    this.currentTemplate.data.name = this.isSMS ? 'SMS Template' : 'Viber Template';
                }

                const TemplateFields = {
                    translit: false,
                    type: 'usual',
                    period: 'single',
                    message: '',
                    sender_id: 0,
                    mailing_name: '',
                    paused: false,
                    delay_send: false,
                    local_time: false,
                    smooth_send: false,
                    smooth_time: 5,
                    lifetime: this.maxLifetimes,
                    url_image: '',
                    button_name: '',
                    button_url: '',
                    sms_sender_id: 0,
                    sms_message: '',
                    sms_translit: false,
                    sms_lifetime: this.maxLifetimes,
                    need_text: false,
                    need_image: false,
                    sms_resend: false,
                };
                Object.keys(this.currentTemplate.data).forEach(key => {
                    if (this.currentTemplate.data[key] === null) {
                        this.currentTemplate.data[key] = TemplateFields[key];
                    }
                });

            },

            editTemplate(id){
                this.action_type = 'update';
                this.form_errors = {
                    name: [],
                    recipients_ids: [],
                    types: [],
                    message: [],
                    sms_message: [],
                    image: [],
                    button_name: [],
                    button_url: [],
                    image_url: [],
                    template_name: []
                };
                let template = this.source_data.templates.find((item) => item.id === id);
                if(template.data) {
                    if(Object.keys(template).length){
                        if(template.type_id === 1) {
                            this.isSMS = 1;
                            this.new_sms_template = template;
                        } else {
                            this.isSMS = 0;
                            this.new_viber_template = template;
                        }
                        this.checkNullFields();
                    }
                }
                this.showModal('new_template');
            },

            saveTemplate() {
                if (!this.validation()) {
                    return false;
                }

                Templates.store(this.currentTemplate)
                    .then(() => {
                        this.setTemplatesToDefault();
                        this.showModalMessage(this.$i18n.t(
                            'sms.messages.add_template'),
                            'success',
                            850
                        );
                        this.getTemplates();
                    })
                    .catch((err) => {
                        this.showModalMessage(this.$i18n.t('sms.messages.error_template'), 'error');
                        console.error('Error: ' + err.toString() )
                    });
                this.closeModal();
            },

            updateTemplate() {
                if (!this.validation()) {
                    return false;
                }

                Templates.update(this.currentTemplate.id, this.currentTemplate)
                    .then(() => {
                        this.setTemplatesToDefault();
                        this.showModalMessage(
                            this.$i18n.t('sms.messages.add_template'),
                            'success',
                            850
                        );
                        this.getTemplates();
                    })
                    .catch((err) => {
                        this.showModalMessage(this.$i18n.t('sms.messages.error_template'), 'error');
                        console.error('Error: ' + err.toString());
                    });
                this.closeModal();
            },

            removeTemplate(id){
                if (!id) return false;

                Templates.destroy(null, id)
                    .then(() => {
                        this.showModalMessage(this.$i18n.t(
                            'sms.messages.del_template'),
                            'success',
                            850
                        );
                        this.getTemplates();
                    })
                    .catch((err) => {
                        this.showModalMessage(this.$i18n.t('sms.messages.error_del_template'), 'error');
                        console.error('Error: ' + err.toString() )
                    });
            },

            // Validation
            validation() {
                let isValid = false;
                if (!this.isSMS) {
                    isValid = this.validationViber();
                } else {
                    isValid = this.validationSMS();
                }
                return isValid;
            },
            validationViber() {
                let errors = 0;
                this.form_errors = {
                    name: [],
                    recipients_ids: [],
                    types: [],
                    message: [],
                    sms_message: [],
                    image: [],
                    button_name: [],
                    button_url: [],
                    image_url: [],
                    template_name: []
                };

                if (!this.isValidTemplateName()) errors++;

                if(!this.new_viber_template.data.need_text && !this.new_viber_template.data.need_image && !this.new_viber_template.data.need_button){
                    errors++;
                    this.form_errors.types.push(this.$t('viber.errors.empty_type'));
                }

                if(this.new_viber_template.data.need_text) {
                    if(!this.new_viber_template.data.message || this.new_viber_template.data.message.trim() === ''){
                        errors++;
                        this.form_errors.message.push(this.$t('sms.errors.empty_message'));
                    }
                    if(this.new_viber_template.data.message.trim().length < 4){
                        errors++;
                        this.form_errors.message.push(this.$t('sms.errors.length_message'));
                    }
                    this.checkMessage();
                }
                if(this.new_viber_template.data.need_image) {
                    if (!this.new_viber_template.data.url_image) {
                        this.new_viber_template.data.url_image = '';
                    }
                    if(!this.new_viber_template.data.url_image || this.new_viber_template.data.url_image.trim() === ''){
                        errors++;
                        this.form_errors.image.push(this.$t('viber.errors.empty_image'));
                    }
                    let regexp = /^((https?:\/\/)|(viber:\/\/)|(tel:))\.*/i;
                    if(!regexp.test(this.new_viber_template.data.url_image.trim())){
                        errors++;
                        this.form_errors.image_url.push(this.$t('viber.errors.wrong_button_url'));
                    }
                }
                if(this.new_viber_template.data.need_button){
                    if (!this.new_viber_template.data.need_button) {
                        this.new_viber_template.data.need_button = '';
                    }
                    if (!this.new_viber_template.data.button_url) {
                        this.new_viber_template.data.button_url = '';
                    }
                    if(!this.new_viber_template.data.button_name || this.new_viber_template.data.button_name.trim() === ''){
                        errors++;
                        this.form_errors.button_name.push(this.$t('viber.errors.empty_button_name'));
                    }
                    if(!this.new_viber_template.data.button_url || this.new_viber_template.data.button_url.trim() === ''){
                        errors++;
                        this.form_errors.button_url.push(this.$t('viber.errors.empty_button_url'));
                    }
                    let regexp = /^((https?:\/\/)|(viber:\/\/)|(tel:))\.*/i;
                    if(!regexp.test(this.new_viber_template.data.button_url.trim())) {
                        errors++;
                        this.form_errors.button_url.push(this.$t('viber.errors.wrong_button_url'));
                    }
                }
                if(this.new_viber_template.data.sms_resend) {
                    if(!this.new_viber_template.data.sms_message || this.new_viber_template.data.sms_message.trim() === ''){
                        errors++;
                        this.form_errors.sms_message.push(this.$t('sms.errors.empty_message'));
                    }
                    if(this.new_viber_template.data.sms_message.trim().length < 4) {
                        errors++;
                        this.form_errors.sms_message.push(this.$t('sms.errors.length_message'));
                    }
                }
                return errors === 0;
            },
            validationSMS() {
                let errors = 0;
                this.form_errors = {
                    name: [],
                    recipients_ids: [],
                    types: [],
                    message: [],
                    sms_message: [],
                    image: [],
                    button_name: [],
                    button_url: [],
                    image_url: [],
                    template_name: []
                };

                if (!this.isValidTemplateName()) errors++;

                if(!this.new_sms_template.data.message || this.new_sms_template.data.message.trim() === ''){
                    errors++;
                    this.form_errors.message.push(this.$i18n.t('sms.errors.empty_message'));
                }
                if(this.new_sms_template.data.message.trim().length < 4){
                    errors++;
                    this.form_errors.message.push(this.$i18n.t('sms.errors.length_message'));
                }
                return errors === 0;
            },
            isValidTemplateName() {
                let isValid = true;
                if(!this.currentTemplate.data.name || this.currentTemplate.data.name.trim() === ''){
                    isValid = false;
                    this.form_errors.template_name.push(this.$i18n.t('sms.errors.empty_name'));
                }
                if(this.currentTemplate.data.name.trim().length < 4 || this.currentTemplate.data.name.trim().length > 60){
                    isValid = false;
                    this.form_errors.template_name.push(this.$i18n.t('sms.errors.length_name'));
                }


                let candidateName = this.currentTemplate.data.name.trim();

                const arrOfExists = this.source_data.templates.filter(temp => {
                    if (!temp.data.name) return false;
                    let existingName = temp.data.name.trim();
                    existingName = existingName.split(' ').join('').toLowerCase();
                    candidateName = candidateName.split(' ').join('').toLowerCase();
                    const sameName = existingName === candidateName;
                    let sameType = (this.isSMS === temp.type_id) || (!this.isSMS && temp.type_id === 2);
                    return sameName && sameType;
                });

                if (this.action_type === 'add' && arrOfExists.length) {
                    this.form_errors.template_name.push(this.$i18n.t('templates.errors.existing_template_name'));
                    isValid = false;
                }
                if (this.action_type === 'update' && arrOfExists.length > 1) {
                    this.form_errors.template_name.push(this.$i18n.t('templates.errors.existing_template_name'));
                    isValid = false;
                }

                return isValid;
            },

            // sort
            sortField(sortName) {
                if (!this.source_data.templates.length) return false;

                if (this.sortType === sortName){
                    this.sortType = `${sortName}Reverse`;
                    this.source_data.templates.reverse();
                    return true;
                }

                this.sortType = sortName;

                switch (this.sortType) {
                    case 'date':
                        this.source_data.templates.sort((a, b) => {
                            const aDate = a.created_at || a.updated_at || null;
                            const bDate = b.created_at || b.updated_at || null;
                            if (!aDate) return -1;
                            if (!bDate) return -1;
                            const dateA = new Date(aDate);
                            const dateB = new Date(bDate);
                            if (dateA > dateB) return 1;
                            if (dateA < dateB) return -1;
                            return 0;
                        });
                        break;
                    case 'type':
                        this.source_data.templates.sort((a, b) => {
                            if (!a.type_id) return -1;
                            if (!b.type_id) return -1;
                            if (a.type_id > b.type_id) return 1;
                            if (a.type_id < b.type_id) return -1;
                            return 0;
                        });
                        break;
                    case 'name':
                        this.source_data.templates.sort((a, b) => {
                            const nameA = a?.data?.name || 'Unnamed';
                            const nameB = b?.data?.name || 'Unnamed';
                            return nameA.localeCompare(nameB);
                        });
                        break;
                    case 'sender':
                        this.source_data.templates.sort((a, b) => {
                            const nameA = this.showSender(a);
                            const nameB = this.showSender(b);
                            return nameA.localeCompare(nameB);
                        });
                        break;
                }
            },
        },

        computed: {
            // templates list filtered
            filteredTemplates() {
                if (!this.searchKey) return this.source_data.templates || [];
                return this.source_data.templates.filter(template => template.data.name.toLowerCase().includes(this.searchKey.toLowerCase()));
            },

            computedDisabled() {
                if (this.isSMS) {
                    return this.currentTemplate.data.message.length <= 0 || this.currentTemplate.data.name.length <= 0;
                } else {
                    if (!this.new_viber_template.data.need_text && !this.new_viber_template.data.need_image && !this.new_viber_template.data.need_button){
                        return true;
                    }
                    if (this.currentTemplate.data.need_text) {
                        return this.currentTemplate.data.message.length <= 0 || this.currentTemplate.data.name.length <= 0;
                    }
                    if (this.currentTemplate.data.sms_resend) {
                        if (!this.currentTemplate.data.sms_message) return true;
                        return this.currentTemplate.data.sms_message.length <= 0 || this.currentTemplate.data.name.length <= 0;
                    }
                    if(!this.new_viber_template.data.need_text && !this.new_viber_template.data.need_image && !this.new_viber_template.data.need_button){
                        return true;
                    }
                    return this.currentTemplate.data.name.length <= 0;
                }

            },

            currentTemplate() {
                return this.isSMS ? this.new_sms_template : this.new_viber_template;
            },

            computedSenders: {
              get() {
                  return this.isSMS ? this.source_data.senders : this.source_data.viberSenders;
              },
              set(value) {
                  if (this.isSMS){
                      this.source_data.senders = value;
                  } else {
                      this.source_data.viberSenders = value;
                  }
              },
            },

            computedLifetimeLabel() {
                return this.isSMS ?
                    this.$i18n.t('sms.text_lifetime') :
                    `${this.$i18n.t('sms.text_lifetime')} ${this.$i18n.t('viber.title')}`;
            },

            // viber and sms counter on data.message
            countSymbols(){
                if (!this.currentTemplate.data.message) {
                    this.currentTemplate.data.message = '';
                    return {
                        full: 0 +' '+ this.declOfNum(this.$i18n.t('sms.decl_symbols'))(0),
                        message_length: 0,
                    };
                }

                const finalLength = this.countHelper(this.currentTemplate.data.message);

                return {
                    full: finalLength +' '+ this.declOfNum(this.$i18n.t('sms.decl_symbols'))(finalLength),
                    message_length: finalLength,
                };
            },

            //sms counter on data.message
            countMessagesSMS(){
                if (!this.currentTemplate.data.message) {
                    this.currentTemplate.data.message = '';
                }
                // http://jrgraphix.net/r/Unicode/0400-04FF
                let cyrillicPattern = /[\u0400-\u04FF]/, // or use /[а-яА-ЯЁё]/
                    number = 0,
                    cyrillicLimit = 70,
                    fromSecondMessageCyrillicLimit = 67,
                    latinLimit = 160,
                    fromSecondMessageLatinLimit = 153,
                    msgType = 'latin';

                if(cyrillicPattern.test(this.currentTemplate.data.message)){
                    msgType = 'cyrillic';
                }

                if (this.countSymbols.message_length > 0){
                    switch (msgType) {
                        case "latin":
                            number = this.countSymbols.message_length <= latinLimit ? 1 : Math.ceil(this.countSymbols.message_length / fromSecondMessageLatinLimit);
                            break;
                        case "cyrillic":
                            number = this.countSymbols.message_length <= cyrillicLimit ? 1 : Math.ceil(this.countSymbols.message_length / fromSecondMessageCyrillicLimit);
                            break;
                    }
                }

                return number +' '+ this.declOfNum(this.$i18n.t('sms.decl_messages'))(number);
            },

            //SMS Resend counters
            countSymbolsSMSResend() {
                if (!this.currentTemplate.data.sms_message) {
                    this.currentTemplate.data.sms_message = '';
                    return {
                        full: 0 +' '+ this.declOfNum(this.$i18n.t('sms.decl_symbols'))(0),
                        message_length: 0,
                    };
                }

                const finalLength = this.countHelper(this.currentTemplate.data.sms_message);

                return {
                    full: finalLength +' '+ this.declOfNum(this.$i18n.t('sms.decl_symbols'))(finalLength),
                    message_length: finalLength,
                };
            },
            countMessagesSMSResend() {
                if (!this.currentTemplate.data.sms_message) {
                    this.currentTemplate.data.sms_message = '';
                }
                // http://jrgraphix.net/r/Unicode/0400-04FF
                let cyrillicPattern = /[\u0400-\u04FF]/, // or use /[а-яА-ЯЁё]/
                    number = 0,
                    cyrillicLimit = 70,
                    fromSecondMessageCyrillicLimit = 67,
                    latinLimit = 160,
                    fromSecondMessageLatinLimit = 153,
                    msgType = 'latin';

                if(cyrillicPattern.test(this.currentTemplate.data.sms_message)){
                    msgType = 'cyrillic';
                }

                if (this.countSymbolsSMSResend.message_length > 0){
                    switch (msgType) {
                        case "latin":
                            number = this.countSymbolsSMSResend.message_length <= latinLimit ? 1 : Math.ceil(this.countSymbolsSMSResend.message_length / fromSecondMessageLatinLimit);
                            break;
                        case "cyrillic":
                            number = this.countSymbolsSMSResend.message_length <= cyrillicLimit ? 1 : Math.ceil(this.countSymbolsSMSResend.message_length / fromSecondMessageCyrillicLimit);
                            break;
                    }
                }

                return number +' '+ this.declOfNum(this.$i18n.t('sms.decl_messages'))(number);
            },
        },

        watch: {
            'new_sms_template.data.type'(d_type) {
                if(d_type === 'birthday'){
                    this.new_sms_template.data.smooth_send = false;
                    this.new_sms_template.data.period = 'day';
                }
            },
            'currentTemplate.data.local_time'(value) {
                if(value){
                    this.currentTemplate.data.smooth_send = false;
                }
            },
            'new_viber_template.data.button_name'(value) {
                if(value && value.length >= this.limitButtonSymbols){
                    this.new_viber_template.data.button_name = this.new_viber_template.data.button_name.substring(0, this.limitButtonSymbols)
                }
            },
            'currentTemplate.data.delay_send'(value) {
                if (!value){
                    this.currentTemplate.data.local_time = false;
                    if (this.currentTemplate.data.type && this.currentTemplate.data.period) {
                        this.currentTemplate.data.type = 'usual';
                        this.currentTemplate.data.period = 'single';
                    }
                }
            },
            'new_viber_template.data.need_text'(value){
                if(false === value){
                    this.new_viber_template.data.message = '';
                }
            },
            'new_viber_template.data.need_image'(value){
                if(false === value){
                    this.new_viber_template.data.url_image = '';
                }
            },
            'new_viber_template.data.need_button'(value){
                if(false === value){
                    this.new_viber_template.data.button_name = '';
                    this.new_viber_template.data.button_url = '';
                }
            },
            'new_viber_template.data.sms_resend'(value){
                if(false === value){
                    this.new_viber_template.data.sms_sender_id = 0;
                    this.new_viber_template.data.sms_message = '';
                    this.new_viber_template.data.sms_translit = false;
                    this.new_viber_template.data.sms_lifetime = this.maxLifetimes;
                }
            },
        },
    }
</script>

<style scoped>
    th {
      cursor: pointer;
      text-align: center!important;
      vertical-align: middle;
    }
    .buttons{
        justify-content: flex-end;
    }
    .centered {
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }
    .overflow-custom {
      max-height: 94vh;
      overflow-y: auto;
    }
    .sort-arrow {
        margin-right: 0.3rem;
    }
    .custom-sm-flex {
        display: flex;
    }
    @media all and (max-width: 500px), print{
        .custom-sm-flex {
            display: block;
        }
        .mb-sm-1 {
            margin-bottom: 0.5rem;
        }
        .w-sm-100 {
            width: 100%;
        }
    }
    @media all and (min-width: 1024px), print{
      .limited-field {
        max-width: 19rem;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
      }
      td:nth-child(1){
        min-width: 5rem;
        text-align: center;
      }
      td:nth-child(2){
        min-width: 4rem;
        text-align: center;
      }
    }
    .d-flex{
      display: flex;
    }
    .justify-content-center{
      justify-content: center;
    }
    .mx-01{
      margin: 0 0.1rem;
    }

</style>
