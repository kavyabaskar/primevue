<template>
    <ul :ref="containerRef" :class="cx('list')" v-bind="level === 0 ? ptm('list') : ptm('sublist')">
        <template v-for="(processedOption, index) of options" :key="getOptionLabelToRender(processedOption)">
            <li
                :id="getOptionId(processedOption)"
                :class="cx('item', { processedOption })"
                role="treeitem"
                :aria-label="getOptionLabelToRender(processedOption)"
                :aria-selected="isOptionGroup(processedOption) ? undefined : isOptionSelected(processedOption)"
                :aria-expanded="isOptionGroup(processedOption) ? isOptionActive(processedOption) : undefined"
                :aria-level="level + 1"
                :aria-setsize="options.length"
                :aria-posinset="index + 1"
                v-bind="ptm('item')"
                :data-p-item-group="isOptionGroup(processedOption)"
                :data-p-highlight="isOptionActive(processedOption)"
                :data-p-focus="isOptionFocused(processedOption)"
                :data-p-disabled="isOptionDisabled(processedOption)"
            >
                <div v-ripple :class="cx('content')" @click="onOptionClick($event, processedOption)" v-bind="ptm('content')">
                    <component v-if="templates['option']" :is="templates['option']" :option="processedOption.option" />
                    <span v-else :class="cx('text')" v-bind="ptm('text')">{{ getOptionLabelToRender(processedOption) }}</span>
                    <template v-if="isOptionGroup(processedOption)">
                        <component v-if="templates['optiongroupicon']" :is="templates['optiongroupicon']" aria-hidden="true" />
                        <span v-else-if="optionGroupIcon" :class="[cx('groupIcon'), optionGroupIcon]" aria-hidden="true" v-bind="ptm('groupIcon')" />
                        <AngleRightIcon v-else :class="cx('groupIcon')" aria-hidden="true" v-bind="ptm('groupIcon')" />
                    </template>
                </div>
                <CascadeSelectSub
                    v-if="isOptionGroup(processedOption) && isOptionActive(processedOption)"
                    role="group"
                    :class="cx('sublist')"
                    :selectId="selectId"
                    :focusedOptionId="focusedOptionId"
                    :options="getOptionGroupChildren(processedOption)"
                    :activeOptionPath="activeOptionPath"
                    :level="level + 1"
                    :templates="templates"
                    :optionLabel="optionLabel"
                    :optionValue="optionValue"
                    :optionDisabled="optionDisabled"
                    :optionGroupIcon="optionGroupIcon"
                    :optionGroupLabel="optionGroupLabel"
                    :optionGroupChildren="optionGroupChildren"
                    @option-change="onOptionChange"
                    :pt="pt"
                    :unstyled="unstyled"
                    :isParentMount="mounted"
                />
            </li>
        </template>
    </ul>
</template>

<script>
import BaseComponent from 'primevue/basecomponent';
import AngleRightIcon from 'primevue/icons/angleright';
import Ripple from 'primevue/ripple';
import { DomHandler, ObjectUtils } from 'primevue/utils';

export default {
    name: 'CascadeSelectSub',
    hostName: 'CascadeSelect',
    extends: BaseComponent,
    emits: ['option-change'],
    container: null,
    props: {
        selectId: String,
        focusedOptionId: String,
        options: Array,
        optionLabel: String,
        optionValue: String,
        optionDisabled: null,
        optionGroupIcon: String,
        optionGroupLabel: String,
        optionGroupChildren: {
            type: [String, Array],
            default: null
        },
        activeOptionPath: Array,
        level: Number,
        templates: null,
        isParentMount: Boolean
    },
    data() {
        return {
            mounted: false
        };
    },
    watch: {
        isParentMount: {
            handler(newValue) {
                newValue && DomHandler.nestedPosition(this.container, this.level);
            }
        }
    },
    mounted() {
        // entering order correction when an item is selected
        (this.isParentMount || this.level === 0) && DomHandler.nestedPosition(this.container, this.level);
        this.mounted = true;
    },
    methods: {
        getOptionId(processedOption) {
            return `${this.selectId}_${processedOption.key}`;
        },
        getOptionLabel(processedOption) {
            return this.optionLabel ? ObjectUtils.resolveFieldData(processedOption.option, this.optionLabel) : processedOption.option;
        },
        getOptionValue(processedOption) {
            return this.optionValue ? ObjectUtils.resolveFieldData(processedOption.option, this.optionValue) : processedOption.option;
        },
        isOptionDisabled(processedOption) {
            return this.optionDisabled ? ObjectUtils.resolveFieldData(processedOption.option, this.optionDisabled) : false;
        },
        getOptionGroupLabel(processedOption) {
            return this.optionGroupLabel ? ObjectUtils.resolveFieldData(processedOption.option, this.optionGroupLabel) : null;
        },
        getOptionGroupChildren(processedOption) {
            return processedOption.children;
        },
        isOptionGroup(processedOption) {
            return ObjectUtils.isNotEmpty(processedOption.children);
        },
        isOptionSelected(processedOption) {
            return !this.isOptionGroup(processedOption) && this.isOptionActive(processedOption);
        },
        isOptionActive(processedOption) {
            return this.activeOptionPath.some((path) => path.key === processedOption.key);
        },
        isOptionFocused(processedOption) {
            return this.focusedOptionId === this.getOptionId(processedOption);
        },
        getOptionLabelToRender(processedOption) {
            return this.isOptionGroup(processedOption) ? this.getOptionGroupLabel(processedOption) : this.getOptionLabel(processedOption);
        },
        onOptionClick(event, processedOption) {
            this.$emit('option-change', { originalEvent: event, processedOption, isFocus: true });
        },
        onOptionChange(event) {
            this.$emit('option-change', event);
        },
        containerRef(el) {
            this.container = el;
        }
    },
    directives: {
        ripple: Ripple
    },
    components: {
        AngleRightIcon: AngleRightIcon
    }
};
</script>
