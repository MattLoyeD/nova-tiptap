<template>
    <default-field
        :field="currentField"
        :errors="errors"
        :full-width-content="true"
        :show-help-text="showHelpText"
    >
        <template #field>
            <div style="position: relative; top: 0; left: 0">
                <div
                    v-show="!currentField.readonly"
                    class="w-full bg-gray-100 rounded overflow-break dark:bg-gray-700"
                    style="z-index: 10; position: sticky; top: 0; left: 0"
                >
                    <div class="p-1">
                        <div
                            v-for="button in buttons"
                            :key="'button-' + button"
                            :class="{
                                'inline-block': button != 'br',
                            }"
                        >
                            <template v-if="button == '|'">
                                <button
                                    class="w-[1px] h-6 relative top-2 mx-1 bg-gray-400"
                                ></button>
                            </template>

                            <template v-else-if="button == 'br'"> </template>

                            <template v-else-if="button == 'heading'">
                                <heading-buttons
                                    :headingLevels="headingLevels"
                                    :mode="mode"
                                    :editor="editor"
                                >
                                </heading-buttons>
                            </template>

                            <template v-else-if="button == 'link'">
                                <link-button
                                    :editor="editor"
                                    :button="button"
                                    :field="currentField"
                                    :mode="mode"
                                    :fileDisk="fileDisk"
                                    :filePath="filePath"
                                >
                                </link-button>
                            </template>

                            <template v-else-if="button == 'image'">
                                <image-button
                                    :editor="editor"
                                    :button="button"
                                    :field="currentField"
                                    :mode="mode"
                                    :imageDisk="imageDisk"
                                    :imagePath="imagePath"
                                >
                                </image-button>
                            </template>

                            <template v-else-if="button == 'placeholderBlock'">
                                <placeholder-block-button
                                    :editor="editor"
                                    :button="button"
                                    :field="currentField"
                                    :mode="mode"
                                >
                                </placeholder-block-button>
                            </template>

                            <template v-else-if="button == 'contentBlock'">
                                <content-block-button
                                    :editor="editor"
                                    :button="button"
                                    :field="currentField"
                                    :mode="mode"
                                    :imageDisk="imageDisk"
                                    :imagePath="imagePath"
                                >
                                </content-block-button>
                            </template>

                            <template v-else-if="button == 'textAlign'">
                                <text-align-buttons
                                    :editor="editor"
                                    :mode="mode"
                                    :alignments="alignments"
                                    :alignElements="alignElements"
                                    :defaultAlignment="defaultAlignment"
                                >
                                </text-align-buttons>
                            </template>

                            <template v-else-if="button == 'rtl'">
                                <rtl-button :editor="editor" :mode="mode">
                                </rtl-button>
                            </template>

                            <template v-else-if="button == 'history'">
                                <history-buttons :editor="editor" :mode="mode">
                                </history-buttons>
                            </template>

                            <template v-else-if="button == 'editHtml'">
                                <base-button
                                    :isActive="mode == 'html'"
                                    :clickMethod="switchMode"
                                    :icon="['fas', 'file-code']"
                                    :title="__('edit html')"
                                >
                                </base-button>
                            </template>

                            <template v-else-if="button == 'color'">
                                <color-button
                                    :editor="editor"
                                    :colors="colors"
                                    mode="color"
                                >
                                </color-button>
                            </template>

                            <template v-else-if="button == 'backgroundColor'">
                                <color-button
                                    :editor="editor"
                                    :colors="backgroundColors"
                                    mode="backgroundColor"
                                >
                                </color-button>
                            </template>

                            <template v-else-if="button == 'tableAlternative'">
                                <table-button
                                    :editor="editor"
                                    :table-cell-background-colors="
                                        tableCellBackgroundColors
                                    "
                                    :table-cell-border-colors="
                                        tableCellBorderColors
                                    "
                                ></table-button>
                            </template>

                            <template v-else>
                                <normal-button
                                    :editor="editor"
                                    :button="button"
                                    :mode="mode"
                                >
                                </normal-button>
                            </template>
                        </div>
                    </div>

                    <div
                        class="flex items-center rounded"
                        style="z-index: 10"
                        v-if="tableIsActive"
                    >
                        <table-buttons :editor="editor"> </table-buttons>
                    </div>
                </div>

                <div
                    class="w-full pt-2 pb-2 mt-3 nova-tiptap-editor form-input form-input-bordered"
                    :style="cssProps"
                    v-show="mode == 'editor'"
                >
                    <editor-content :editor="editor" />
                </div>

                <div class="w-full px-0 mt-3" v-show="mode == 'html'">
                    <edit-html :theme="htmlTheme" v-model="htmlModeValue" />
                </div>
            </div>
        </template>
    </default-field>
</template>

<script>
    import { Editor, EditorContent, VueNodeViewRenderer } from "@tiptap/vue-3";

    import Text from "@tiptap/extension-text";

    import Blockquote from "@tiptap/extension-blockquote";
    import Bold from "@tiptap/extension-bold";
    import BulletList from "@tiptap/extension-bullet-list";
    import Code from "@tiptap/extension-code";
    import CodeBlock from "@tiptap/extension-code-block";
    import CodeBlockLowlight from "@tiptap/extension-code-block-lowlight";
    import Highlight from "@tiptap/extension-highlight";
    import HorizontalRule from "@tiptap/extension-horizontal-rule";
    import Italic from "@tiptap/extension-italic";
    import Link from "@tiptap/extension-link";
    import ListItem from "@tiptap/extension-list-item";
    import OrderedList from "@tiptap/extension-ordered-list";
    import Strike from "@tiptap/extension-strike";
    import Subscript from "@tiptap/extension-subscript";
    import Superscript from "@tiptap/extension-superscript";
    import TextStyle from "@tiptap/extension-text-style";
    import Underline from "@tiptap/extension-underline";
    import { Color } from "@tiptap/extension-color";

    import Heading from "@tiptap/extension-heading";
    import TextAlign from "@tiptap/extension-text-align";
    import History from "@tiptap/extension-history";
    import Document from "@tiptap/extension-document";

    import Table from "@tiptap/extension-table";
    import TableRow from "@tiptap/extension-table-row";
    import TableCell from "../extensions/TableCell";
    import TableHeader from "@tiptap/extension-table-header";

    import Paragraph from "@tiptap/extension-paragraph";
    import HardBreak from "@tiptap/extension-hard-break";
    import Placeholder from "@tiptap/extension-placeholder";

    import Image from "@tiptap/extension-image";
    import Dropcursor from "@tiptap/extension-dropcursor";
    import ImageResize from 'tiptap-extension-resize-image';

    import LinkButton from "./buttons/LinkButton";
    import NormalButton from "./buttons/NormalButton";
    import HeadingButtons from "./buttons/HeadingButtons";
    import TableButtons from "./buttons/TableButtons";
    import TextAlignButtons from "./buttons/TextAlignButtons";
    import RtlButton from "./buttons/RtlButton";
    import HistoryButtons from "./buttons/HistoryButtons";
    import ImageButton from "./buttons/ImageButton";
    import PlaceholderBlockButton from "./buttons/PlaceholderBlockButton";
    import ContentBlockButton from "./buttons/ContentBlockButton";
    import BaseButton from "./buttons/BaseButton.vue";
    import ColorButton from "./buttons/ColorButton.vue";
    import TableButton from "./buttons/TableButton.vue";

    import CodeBlockComponent from "./CodeBlockComponent";
    import EditHtml from "./EditHtml";

    import Gapcursor from "@tiptap/extension-gapcursor";

    import { lowlight } from "lowlight";
    import pretty from "pretty";

    import buttonHovers from "../mixins/buttonHovers";
    import contentSanitizer from "../mixins/contentSanitizer";

    import { DependentFormField, HandlesValidationErrors } from "laravel-nova";

    import PlaceholderBlockExtension from "../extensions/PlaceholderBlockExtension.js";
    import VideoContentBlockExtension from "./content-blocks/VideoContentBlockExtension.js";
    import GalleryContentBlockExtension from "./content-blocks/GalleryContentBlockExtension.js";
    import BackgroundColorExtension from "../extensions/BackgroundColor.js";

    export default {
        mixins: [
            DependentFormField,
            HandlesValidationErrors,
            buttonHovers,
            contentSanitizer,
        ],

        props: ["resourceName", "resourceId", "field"],

        components: {
            TableButton,
            ColorButton,
            EditorContent,
            LinkButton,
            NormalButton,
            HeadingButtons,
            TableButtons,
            TextAlignButtons,
            RtlButton,
            HistoryButtons,
            ImageButton,
            PlaceholderBlockButton,
            ContentBlockButton,
            EditHtml,
            BaseButton,
        },

        data() {
            return {
                editor: null,
                mode: "editor",
                htmlModeValue: "",
                placeholder: "",
                imageDisk: "",
                imagePath: "",
                fileDisk: "",
                filePath: "",
                initialFieldValue: null,
                defaultColors: [
                    "#000000",
                    "#ff133b",
                    "#0000ff",
                    "#008000",
                    "#ffff00",
                    "#ffa500",
                ],
            };
        },

        watch: {
            htmlModeValue: function (val) {
                this.editor.commands.setContent(val);
                this.updateValue(this.editor.getHTML());
            },
            "currentField.readonly": function (val) {
                if (this.editor) {
                    this.editor.setEditable(!val);
                }
            },
        },

        computed: {
            contentWithTrailingParagraph() {
                let sanitizedValue = this.sanitizeTiptapContent(this.value);

                if (
                    _.isString(sanitizedValue) &&
                    _.endsWith(_.trim(sanitizedValue), "content-block>")
                ) {
                    return sanitizedValue + "<p></p>";
                }

                return sanitizedValue;
            },
            buttons() {
                let tmpButtons = this.currentField.buttons
                    ? this.currentField.buttons
                    : ["bold", "italic"];

                return _.map(tmpButtons, function (button) {
                    return button == "|" || button == "br"
                        ? button
                        : _.camelCase(button);
                });
            },

            headingLevels() {
                return this.currentField.headingLevels
                    ? this.currentField.headingLevels
                    : [1, 2, 3];
            },

            colors() {
                return this.currentField.colors
                    ? this.currentField.colors
                    : this.defaultColors;
            },

            backgroundColors() {
                return this.currentField.backgroundColors
                    ? this.currentField.backgroundColors
                    : this.defaultColors;
            },

            tableCellBackgroundColors() {
                return this.currentField.tableCellBackgroundColors
                    ? this.currentField.tableCellBackgroundColors
                    : this.defaultColors;
            },

            tableCellBorderColors() {
                return this.currentField.tableCellBorderColors
                    ? this.currentField.tableCellBorderColors
                    : this.defaultColors;
            },

            alignments() {
                return this.currentField.alignments
                    ? this.currentField.alignments
                    : ["start", "center", "end", "justify"];
            },

            tableIsActive() {
                if (this.buttons.indexOf("table") > -1) {
                    return this.editor ? this.editor.isActive("table") : false;
                }
                return false;
            },

            alignElements() {
                return this.currentField.alignElements
                    ? this.currentField.alignElements
                    : ["heading", "paragraph"];
            },

            defaultAlignment() {
                return this.currentField.defaultAlignment
                    ? this.currentField.defaultAlignment
                    : "left";
            },

            cssProps() {
                return {
                    "--text-align": this.defaultAlignment,
                };
            },

            htmlTheme() {
                return this.currentField.htmlTheme
                    ? this.currentField.htmlTheme
                    : "";
            },

            saveAsJson() {
                return this.currentField.saveAsJson
                    ? this.currentField.saveAsJson
                    : false;
            },
        },

        methods: {
            updateValue(value) {
                this.value = this.sanitizeTiptapContent(value);
            },

            switchMode() {
                if (this.mode == "html") {
                    this.editor.commands.setContent(this.htmlModeValue);
                    this.updateValue(this.editor.getHTML());
                    this.mode = "editor";
                } else {
                    this.htmlModeValue = pretty(this.editor.getHTML());
                    this.mode = "html";
                }
            },

            onSyncedField() {
                if (this.editor) {
                    this.editor.setOptions({
                        editable: !this.currentField.readonly,
                    });
                }
                if (!this.initialFieldValue) {
                    this.htmlModeValue = this.value;
                }
            },
        },

        mounted() {
            this.placeholder = this.currentField.placeholder
                ? this.currentField.placeholder
                : this.currentField.extraAttributes
                ? this.currentField.extraAttributes.placeholder
                : "";

            if (
                this.currentField.imageSettings &&
                this.currentField.imageSettings.path
            ) {
                this.imagePath = this.currentField.imageSettings.path;
            }
            if (
                this.currentField.imageSettings &&
                this.currentField.imageSettings.disk
            ) {
                this.imageDisk = this.currentField.imageSettings.disk;
            }
            if (
                this.currentField.fileSettings &&
                this.currentField.fileSettings.path
            ) {
                this.filePath = this.currentField.fileSettings.path;
            }
            if (
                this.currentField.fileSettings &&
                this.currentField.fileSettings.disk
            ) {
                this.fileDisk = this.currentField.fileSettings.disk;
            }

            let extensions = [
                Document,
                Bold,
                Code,
                Italic,
                Highlight,
                PlaceholderBlockExtension.extend({
                    addAttributes() {
                        return {
                            ...this.parent?.(),
                            "data-key": {
                                default: "bogus",
                            },
                        };
                    },
                }),
                VideoContentBlockExtension,
                GalleryContentBlockExtension,
                Link.extend({
                    addAttributes() {
                        return {
                            ...this.parent?.(),
                            "tt-mode": {
                                default: "url",
                            },
                            class: String,
                            rel: String,
                            title: String,
                            download: String,
                        };
                    },
                }),
                Strike,
                TextStyle,
                Color,
                BackgroundColorExtension,
                Underline,
                Subscript,
                Superscript,

                Heading.configure({
                    levels: this.headingLevels,
                }).extend({
                    addAttributes() {
                        return {
                            ...this.parent?.(),
                            dir: String,
                        };
                    },
                }),
                Blockquote.extend({
                    addAttributes() {
                        return {
                            ...this.parent?.(),
                            dir: String,
                        };
                    },
                }),
                BulletList.extend({
                    addAttributes() {
                        return {
                            ...this.parent?.(),
                            dir: String,
                        };
                    },
                }),
                HorizontalRule,
                ListItem,
                OrderedList.extend({
                    addAttributes() {
                        return {
                            ...this.parent?.(),
                            dir: String,
                        };
                    },
                }),
                HardBreak,
                Paragraph.extend({
                    addAttributes() {
                        return {
                            ...this.parent?.(),
                            dir: String,
                            data: String,
                        };
                    },
                }),
                Table.configure({
                    resizable: true,
                }),
                TableRow,
                TableCell,
                TableHeader,
                TextAlign.configure({
                    types: this.alignElements,
                    alignments: this.alignments,
                    defaultAlignment: this.defaultAlignment,
                }),
                History,
                Text,
                Gapcursor,
                Placeholder.configure({
                    placeholder: this.placeholder,
                }),
                Image.extend({
                    addAttributes() {
                        return {
                            ...this.parent?.(),
                            "tt-mode": {
                                default: "file",
                            },
                            "tt-link-url": {
                                default: "",
                            },
                            "tt-link-target": {
                                default: "",
                            },
                            "tt-link-mode": {
                                default: "url",
                            },
                            class: String,
                            title: String,
                            alt: String,
                        };
                    },
                }),
                Dropcursor,
                ImageResize,
            ];

            if (
                this.buttons.includes("codeBlock") &&
                this.currentField.syntaxHighlighting
            ) {
                extensions.push(
                    CodeBlockLowlight.extend({
                        addNodeView() {
                            return VueNodeViewRenderer(CodeBlockComponent);
                        },
                    }).configure({
                        lowlight,
                    })
                );
            } else if (this.buttons.includes("codeBlock")) {
                extensions.push(CodeBlock);
            }

            const context = this;

            this.editor = new Editor({
                extensions: extensions,
                content: this.contentWithTrailingParagraph,
                editable: !this.currentField.readonly,
                onCreate() {
                    try {
                        let content = JSON.parse(context.value);
                        let sanitizedContent =
                            context.sanitizeTiptapContent(content);
                        this.commands.setContent(sanitizedContent);
                    } catch {}
                },
                onUpdate() {
                    if (context.saveAsJson) {
                        let jsonContent = this.getJSON();
                        // Sanitize each text node in the JSON content
                        const sanitizeJsonContent = (node) => {
                            if (node.type === "text" && node.text) {
                                node.text = context.sanitizeTiptapContent(
                                    node.text
                                );
                            }
                            if (node.content) {
                                node.content.forEach(sanitizeJsonContent);
                            }
                            return node;
                        };
                        jsonContent.content.forEach(sanitizeJsonContent);
                        context.updateValue(
                            JSON.stringify(jsonContent.content)
                        );
                    } else {
                        context.updateValue(this.getHTML());
                    }
                },
            });

            this.initialFieldValue = this.value;
        },

        beforeDestroy() {
            this.editor.destroy();
        },
    };
</script>

<style lang="scss">
    .nova-tiptap-editor {
        padding-bottom: 20px;
        padding-top: 20px;

        .ProseMirror-focused {
            outline: none;
        }

        img.ProseMirror-selectednode {
            outline: 3px solid var(--primary-30);
        }

        .ProseMirror {
            p.is-editor-empty:first-child::before {
                content: attr(data-placeholder);
                float: left;
                color: #ced4da;
                pointer-events: none;
                height: 0;
            }

            p,
            h1,
            h2,
            h3,
            h4,
            h5,
            h6,
            blockquote,
            ul,
            ol,
            table,
            pre {
                margin-top: 1em;
                line-height: 1.5em;
            }

            h1 {
                font-size: 3em;
            }
            h2 {
                font-size: 2.4em;
            }
            h3 {
                font-size: 1.8em;
            }
            h4 {
                font-size: 1.5em;
            }
            h5 {
                font-size: 1.3em;
            }
            h6 {
                font-size: 1.1em;
            }

            h1,
            h2,
            h3,
            h4,
            h5,
            h6 {
                text-align: var(--text-align);
            }

            a {
                color: #0ea5e9;
                text-decoration: underline;
            }

            pre {
                padding-top: 5px;
                padding-bottom: 5px;
                padding-left: 12px;
                padding-right: 12px;
                background-color: #3c4b5f;
                color: white;
                border-radius: 0.125rem;
            }

            p:first-child,
            h1:first-child,
            h2:first-child,
            h3:first-child,
            h4:first-child,
            h5:first-child,
            h6:first-child,
            blockquote:first-child,
            ul:first-child,
            ol:first-child,
            table:first-child,
            pre:first-child {
                margin-top: 0;
            }

            blockquote {
                display: block;
                margin-top: 1.5em;
                margin-bottom: 1.5em;
                padding-left: 15px;
                border-left: 3px solid #dddddd;
            }

            a {
                pointer-events: none;
            }

            ul {
                padding-left: 16px;

                li {
                    list-style: disc;
                }
            }

            ol {
                padding-left: 16px;

                li {
                    list-style: numeric;
                }
            }

            hr {
                border-top: 1px solid #dddddd;
                margin-top: 20px;
                margin-bottom: 10px;
            }

            .tableWrapper {
                margin-top: 15px;
                overflow-x: auto;
            }

            .resize-cursor {
                cursor: ew-resize;
                cursor: col-resize;
            }

            table {
                border-collapse: collapse;
                table-layout: fixed;
                width: 100%;
                overflow: hidden;

                td,
                th {
                    min-width: 1em;
                    border: 2px solid #dddddd;
                    padding: 3px 5px;
                    vertical-align: top;
                    box-sizing: border-box;
                    position: relative;

                    > * {
                        margin-bottom: 0;
                    }
                }

                th {
                    font-weight: bold;
                    text-align: left;
                    background-color: #fafafa;
                }

                .selectedCell:after {
                    z-index: 2;
                    position: absolute;
                    content: "";
                    left: 0;
                    right: 0;
                    top: 0;
                    bottom: 0;
                    background: rgba(200, 200, 255, 0.4);
                    pointer-events: none;
                }

                .column-resize-handle {
                    position: absolute;
                    right: -2px;
                    top: 0;
                    bottom: -2px;
                    width: 4px;
                    background-color: #adf;
                    pointer-events: none;
                }
            }
        }
    }
</style>
