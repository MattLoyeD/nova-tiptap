<template>
  <button
        type="button"
        class="
            group relative
            p-2
            leading-none text-xs
            rounded shadow
            tiptap-button
            
        "
        style="margin: 4px; min-width: 32px; height: 32px"
        :class="{ 
            'bg-primary-500 hover:bg-primary-400 text-white hover:dark:bg-gray-900 dark:bg-gray-800 dark:text-white': isActive,
            'bg-white hover:bg-gray-200 text-black hover:dark:bg-gray-900 dark:bg-gray-800 dark:text-white': !isActive,
            'opacity-50 pointer-events-none': isDisabled,
        }"
        @mouseover="hovered = true"
        @mouseout="hovered = false"
        @click="callClickMethod"
    >
        <div 
            class="
                pointer-events-none
                absolute left-0 w-full flex justify-center
            "
            style="z-index: 100; bottom: 36px;"
            v-show="hovered"
        >
            <div 
                v-html="title"
                class="
                    whitespace-nowrap bg-black 
                    text-2xs text-white font-semibold capitalize
                    px-1 py-px rounded
                "
            >

            </div>
        </div>

        <template v-if="icon">
            <font-awesome-icon :icon="icon">
            </font-awesome-icon>
        </template>

        <template v-else-if="innerHtml">
            <span class="font-bold" v-html="innerHtml"></span>
        </template>

        <template v-else>
            <slot />
        </template>
    </button>
</template>

<script>

import { library } from '@fortawesome/fontawesome-svg-core';

import { 
    faAlignCenter,
    faAlignJustify,
    faAlignLeft,
    faPanorama,
    faAlignRight,
    faBold,
    faCode,
    faCube,
    faCubes,
    faFileCode,
    faHighlighter,
    faImage,
    faItalic,
    faLink, 
    faListOl,
    faListUl,
    faParagraph,
    faQuoteRight,
    faRedoAlt,
    faStrikethrough,
    faSubscript,
    faSuperscript,
    faTable,
    faUnderline,
    faUndoAlt,
    faUnlink,
    faA,
    faSwatchbook,
    faDropletSlash,
    faPenFancy
} from '@fortawesome/free-solid-svg-icons';

library.add(
    faAlignCenter,
    faAlignJustify,
    faAlignLeft,
    faAlignRight,
    faBold,
    faCode,
    faCube,
    faCubes,
    faFileCode,
    faHighlighter,
    faRulerHorizontal,
    faImage,
    faPanorama,
    faItalic,
    faLink, 
    faListOl,
    faListUl,
    faParagraph,
    faRightLeft,
    faQuoteRight,
    faRedoAlt,
    faStrikethrough,
    faSubscript,
    faSuperscript,
    faTable,
    faUnderline,
    faUndoAlt,
    faUnlink,
    faA,
    faSwatchbook,
    faDropletSlash,
    faPenFancy
);

import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';

export default {
    data() {
        return {
            hovered: false,
        }
    },
    props: [
        'clickMethod', 
        'clickMethodParameters',
        'title',
        'isActive',
        'isDisabled',
        'icon',
        'innerHtml',
    ],

    components: {
        FontAwesomeIcon,
    },

    methods: {
        callClickMethod() {
            let tmpParams = this.clickMethodParameters
            if (tmpParams) {
                
                if (!typeof(tmpParams) != 'object') {
                    tmpParams = [tmpParams];
                }

                this.clickMethod(...tmpParams);
            } else {
                this.clickMethod();
            }
        }
    }
}
</script>
