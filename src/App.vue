<template>
	<div class="main-container">
		<div class="presence" ref="editorPresenceElement"></div>
		<div
			class="editor-container editor-container_document-editor editor-container_include-outline editor-container_include-annotations editor-container_include-pagination"
			ref="editorContainerElement"
		>
			<div class="editor-container__menu-bar" ref="editorMenuBarElement"></div>
			<div class="editor-container__toolbar" ref="editorToolbarElement"></div>
			<div class="editor-container__editor-wrapper">
				<div class="editor-container__sidebar"><div ref="editorOutlineElement"></div></div>
				<div class="editor-container__editor">
					<div ref="editorElement">
						<ckeditor v-if="editor && config" :modelValue="config.initialData" :editor="editor" :config="config" @ready="onReady" />
					</div>
				</div>
				<div class="editor-container__sidebar"><div ref="editorAnnotationsElement"></div></div>
			</div>
		</div>
		<div class="revision-history" ref="editorRevisionHistoryElement">
			<div class="revision-history__wrapper">
				<div class="revision-history__editor" ref="editorRevisionHistoryEditorElement"></div>
				<div class="revision-history__sidebar" ref="editorRevisionHistorySidebarElement"></div>
			</div>
		</div>
	</div>
</template>

<script setup>
import { computed, ref, onMounted, watchEffect, useTemplateRef } from 'vue';
import { Ckeditor, useCKEditorCloud } from '@ckeditor/ckeditor5-vue';

const LICENSE_KEY =
	'';
const DOCUMENT_ID = 'Doc';

const CLOUD_SERVICES_TOKEN_URL =
	'';
const CLOUD_SERVICES_WEBSOCKET_URL = '';

const editorToolbar = useTemplateRef('editorToolbarElement');
const editorMenuBar = useTemplateRef('editorMenuBarElement');
const editorAnnotations = useTemplateRef('editorAnnotationsElement');
const editorPresence = useTemplateRef('editorPresenceElement');
const editorOutline = useTemplateRef('editorOutlineElement');
const editorContainer = useTemplateRef('editorContainerElement');
const editorRevisionHistory = useTemplateRef('editorRevisionHistoryElement');
const editorRevisionHistoryEditor = useTemplateRef('editorRevisionHistoryEditorElement');
const editorRevisionHistorySidebar = useTemplateRef('editorRevisionHistorySidebarElement');

const cloud = useCKEditorCloud({ version: '44.2.0', premium: true, ckbox: { version: '2.6.1' } });

const isLayoutReady = ref(false);

const editor = computed(() => {
	if (!cloud.data.value) {
		return null;
	}

	return cloud.data.value.CKEditor.DecoupledEditor;
});

const config = computed(() => {
	if (!isLayoutReady.value) {
		return null;
	}

	if (!cloud.data.value) {
		return null;
	}

	const {
		Plugin,
		ButtonView,
		Alignment,
		Autoformat,
		AutoImage,
		AutoLink,
		Autosave,
		BalloonToolbar,
		Bold,
		Bookmark,
		CKBox,
		CKBoxImageEdit,
		CloudServices,
		Code,
		CodeBlock,
		Emoji,
		Essentials,
		FindAndReplace,
		FontBackgroundColor,
		FontColor,
		FontFamily,
		FontSize,
		Heading,
		HorizontalLine,
		ImageBlock,
		ImageCaption,
		ImageEditing,
		ImageInline,
		ImageInsert,
		ImageInsertViaUrl,
		ImageResize,
		ImageStyle,
		ImageTextAlternative,
		ImageToolbar,
		ImageUpload,
		ImageUtils,
		Indent,
		IndentBlock,
		Italic,
		Link,
		LinkImage,
		List,
		ListProperties,
		Mention,
		PageBreak,
		Paragraph,
		PasteFromOffice,
		PictureEditing,
		RemoveFormat,
		SpecialCharacters,
		SpecialCharactersArrows,
		SpecialCharactersCurrency,
		SpecialCharactersEssentials,
		SpecialCharactersLatin,
		SpecialCharactersMathematical,
		SpecialCharactersText,
		Strikethrough,
		Subscript,
		Superscript,
		Table,
		TableCaption,
		TableCellProperties,
		TableColumnResize,
		TableProperties,
		TableToolbar,
		TextTransformation,
		TodoList,
		Underline
	} = cloud.data.value.CKEditor;
	const {
		CaseChange,
		Comments,
		DocumentOutline,
		ExportPdf,
		ExportWord,
		FormatPainter,
		ImportWord,
		MergeFields,
		MultiLevelList,
		Pagination,
		PasteFromOfficeEnhanced,
		PresenceList,
		RealTimeCollaborativeComments,
		RealTimeCollaborativeEditing,
		RealTimeCollaborativeRevisionHistory,
		RealTimeCollaborativeTrackChanges,
		RevisionHistory,
		SlashCommand,
		TableOfContents,
		Template,
		TrackChanges,
		TrackChangesData,
		TrackChangesPreview
	} = cloud.data.value.CKEditorPremiumFeatures;

	/**
	 * The `AnnotationsSidebarToggler` plugin adds an icon to the right side of the editor.
	 *
	 * It allows to toggle the right annotations bar visibility.
	 */
	class AnnotationsSidebarToggler extends Plugin {
		static get requires() {
			return ['AnnotationsUIs'];
		}

		static get pluginName() {
			return 'AnnotationsSidebarToggler';
		}

		init() {
			this.toggleButton = new ButtonView(this.editor.locale);

			const NON_COLLAPSE_ANNOTATION_ICON =
				'<svg viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" transform="matrix(-1,0,0,1,0,0)"><path d="M11.463 5.187a.888.888 0 1 1 1.254 1.255L9.16 10l3.557 3.557a.888.888 0 1 1-1.254 1.255L7.26 10.61a.888.888 0 0 1 .16-1.382l4.043-4.042z"></path></svg>';
			const COLLAPSE_ANNOTATION_ICON =
				'<svg viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg" transform="matrix(1,0,0,1,0,0)"><path d="M11.463 5.187a.888.888 0 1 1 1.254 1.255L9.16 10l3.557 3.557a.888.888 0 1 1-1.254 1.255L7.26 10.61a.888.888 0 0 1 .16-1.382l4.043-4.042z"/></svg>';

			const annotationsUIsPlugin = this.editor.plugins.get('AnnotationsUIs');
			const annotationsContainer = this.editor.config.get('sidebar.container');
			const sidebarContainer = annotationsContainer.parentElement;

			this.toggleButton.set({
				label: 'Toggle annotations sidebar',
				tooltip: 'Hide annotations sidebar',
				tooltipPosition: 'se',
				icon: COLLAPSE_ANNOTATION_ICON
			});

			this.toggleButton.on('execute', () => {
				// Toggle a CSS class on the annotations sidebar container to manage the visibility of the sidebar.
				annotationsContainer.classList.toggle('ck-hidden');

				// Change the look of the button to reflect the state of the annotations container.
				if (annotationsContainer.classList.contains('ck-hidden')) {
					this.toggleButton.icon = NON_COLLAPSE_ANNOTATION_ICON;
					this.toggleButton.tooltip = 'Show annotations sidebar';
					annotationsUIsPlugin.switchTo('inline');
				} else {
					this.toggleButton.icon = COLLAPSE_ANNOTATION_ICON;
					this.toggleButton.tooltip = 'Hide annotations sidebar';
					annotationsUIsPlugin.switchTo('wideSidebar');
				}

				// Keep the focus in the editor whenever the button is clicked.
				this.editor.editing.view.focus();
			});

			this.toggleButton.render();

			sidebarContainer.insertBefore(this.toggleButton.element, annotationsContainer);
		}

		destroy() {
			this.toggleButton.element.remove();

			return super.destroy();
		}
	}

	/**
	 * The `DocumentOutlineToggler` plugin adds an icon to the left side of the editor.
	 *
	 * It allows to toggle document outline visibility.
	 */
	class DocumentOutlineToggler extends Plugin {
		static get pluginName() {
			return 'DocumentOutlineToggler';
		}

		init() {
			this.toggleButton = new ButtonView(this.editor.locale);

			const DOCUMENT_OUTLINE_ICON =
				'<svg viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path d="M5 9.5a.5.5 0 0 0 .5-.5v-.5A.5.5 0 0 0 5 8H3.5a.5.5 0 0 0-.5.5V9a.5.5 0 0 0 .5.5H5Z"/><path d="M5.5 12a.5.5 0 0 1-.5.5H3.5A.5.5 0 0 1 3 12v-.5a.5.5 0 0 1 .5-.5H5a.5.5 0 0 1 .5.5v.5Z"/><path d="M5 6.5a.5.5 0 0 0 .5-.5v-.5A.5.5 0 0 0 5 5H3.5a.5.5 0 0 0-.5.5V6a.5.5 0 0 0 .5.5H5Z"/><path clip-rule="evenodd" d="M2 19a2 2 0 0 1-2-2V3a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H2Zm6-1.5h10a.5.5 0 0 0 .5-.5V3a.5.5 0 0 0-.5-.5H8v15Zm-1.5-15H2a.5.5 0 0 0-.5.5v14a.5.5 0 0 0 .5.5h4.5v-15Z"/></svg>';
			const COLLAPSE_OUTLINE_ICON =
				'<svg viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path d="M11.463 5.187a.888.888 0 1 1 1.254 1.255L9.16 10l3.557 3.557a.888.888 0 1 1-1.254 1.255L7.26 10.61a.888.888 0 0 1 .16-1.382l4.043-4.042z"/></svg>';

			const documentOutlineContainer = this.editor.config.get('documentOutline.container');
			const sidebarContainer = documentOutlineContainer.parentElement;

			this.toggleButton.set({
				label: 'Toggle document outline',
				tooltip: 'Hide document outline',
				tooltipPosition: 'se',
				icon: COLLAPSE_OUTLINE_ICON
			});

			this.toggleButton.on('execute', () => {
				// Toggle a CSS class on the document outline container to manage the visibility of the outline.
				documentOutlineContainer.classList.toggle('ck-hidden');

				// Change the look of the button to reflect the state of the document outline feature.
				if (documentOutlineContainer.classList.contains('ck-hidden')) {
					this.toggleButton.icon = DOCUMENT_OUTLINE_ICON;
					this.toggleButton.tooltip = 'Show document outline';
				} else {
					this.toggleButton.icon = COLLAPSE_OUTLINE_ICON;
					this.toggleButton.tooltip = 'Hide document outline';
				}

				// Keep the focus in the editor whenever the button is clicked.
				this.editor.editing.view.focus();
			});

			this.toggleButton.render();

			sidebarContainer.insertBefore(this.toggleButton.element, documentOutlineContainer);
		}

		destroy() {
			this.toggleButton.element.remove();

			return super.destroy();
		}
	}

	return {
		toolbar: {
			items: [
				'previousPage',
				'nextPage',
				'|',
				'revisionHistory',
				'trackChanges',
				'comment',
				'|',
				'insertMergeField',
				'previewMergeFields',
				'|',
				'formatPainter',
				'|',
				'heading',
				'|',
				'fontSize',
				'fontFamily',
				'fontColor',
				'fontBackgroundColor',
				'|',
				'bold',
				'italic',
				'underline',
				'|',
				'link',
				'insertImage',
				'insertTable',
				'|',
				'alignment',
				'|',
				'bulletedList',
				'numberedList',
				'multiLevelList',
				'todoList',
				'outdent',
				'indent', 
				'|',
				'codeBlock'
			],
			shouldNotGroupWhenFull: false
		},
		plugins: [
			Alignment,
			Autoformat,
			AutoImage,
			AutoLink,
			Autosave,
			BalloonToolbar,
			Bold,
			Bookmark,
			CaseChange,
			CKBox,
			CKBoxImageEdit,
			CloudServices,
			Code,
			CodeBlock,
			Comments,
			DocumentOutline,
			Emoji,
			Essentials,
			ExportPdf,
			ExportWord,
			FindAndReplace,
			FontBackgroundColor,
			FontColor,
			FontFamily,
			FontSize,
			FormatPainter,
			Heading,
			HorizontalLine,
			ImageBlock,
			ImageCaption,
			ImageEditing,
			ImageInline,
			ImageInsert,
			ImageInsertViaUrl,
			ImageResize,
			ImageStyle,
			ImageTextAlternative,
			ImageToolbar,
			ImageUpload,
			ImageUtils,
			ImportWord,
			Indent,
			IndentBlock,
			Italic,
			Link,
			LinkImage,
			List,
			ListProperties,
			Mention,
			MergeFields,
			MultiLevelList,
			PageBreak,
			Pagination,
			Paragraph,
			PasteFromOffice,
			PasteFromOfficeEnhanced,
			PictureEditing,
			PresenceList,
			RealTimeCollaborativeComments,
			RealTimeCollaborativeEditing,
			RealTimeCollaborativeRevisionHistory,
			RealTimeCollaborativeTrackChanges,
			RemoveFormat,
			RevisionHistory,
			SlashCommand,
			SpecialCharacters,
			SpecialCharactersArrows,
			SpecialCharactersCurrency,
			SpecialCharactersEssentials,
			SpecialCharactersLatin,
			SpecialCharactersMathematical,
			SpecialCharactersText,
			Strikethrough,
			Subscript,
			Superscript,
			Table,
			TableCaption,
			TableCellProperties,
			TableColumnResize,
			TableOfContents,
			TableProperties,
			TableToolbar,
			Template,
			TextTransformation,
			TodoList,
			TrackChanges,
			TrackChangesData,
			TrackChangesPreview,
			Underline
		],
		extraPlugins: [DocumentOutlineToggler, AnnotationsSidebarToggler],
		balloonToolbar: ['comment', '|', 'bold', 'italic', '|', 'link', 'insertImage', '|', 'bulletedList', 'numberedList'],
		cloudServices: {
			tokenUrl: CLOUD_SERVICES_TOKEN_URL,
			webSocketUrl: CLOUD_SERVICES_WEBSOCKET_URL
		},
		codeBlock: {
  			languages: [
			  	{ language: 'plaintext', label: 'Plain text' },
    			{ language: 'javascript', label: 'JavaScript' },
				{ language: 'typescript', label: 'TypeScript' },
				{ language: 'css', label: 'CSS' },
    			{ language: 'diff', label: 'Diff' }
  			]
		},
		collaboration: {
			channelId: DOCUMENT_ID
		},
		comments: {
			editorConfig: {
				extraPlugins: [Autoformat, Bold, Italic, List, Mention],
				mention: {
					feeds: [
						{
							marker: '@',
							feed: [
								/* See: https://ckeditor.com/docs/ckeditor5/latest/features/mentions.html#comments-with-mentions */
							]
						}
					]
				}
			}
		},
		documentOutline: {
			container: editorOutline.value
		},
		exportPdf: {
			stylesheets: [
				/* This path should point to application stylesheets. */
				/* See: https://ckeditor.com/docs/ckeditor5/latest/features/converters/export-pdf.html */
				'./style.css',
				/* Export PDF needs access to stylesheets that style the content. */
				'https://cdn.ckeditor.com/ckeditor5/44.2.0/ckeditor5.css',
				'https://cdn.ckeditor.com/ckeditor5-premium-features/44.2.0/ckeditor5-premium-features.css'
			],
			fileName: 'export-pdf-demo.pdf',
			converterOptions: {
				format: 'A4',
				margin_top: '20mm',
				margin_bottom: '20mm',
				margin_right: '12mm',
				margin_left: '12mm',
				page_orientation: 'portrait'
			}
		},
		exportWord: {
			stylesheets: [
				/* This path should point to application stylesheets. */
				/* See: https://ckeditor.com/docs/ckeditor5/latest/features/converters/export-word.html */
				'./style.css',
				/* Export Word needs access to stylesheets that style the content. */
				'https://cdn.ckeditor.com/ckeditor5/44.2.0/ckeditor5.css',
				'https://cdn.ckeditor.com/ckeditor5-premium-features/44.2.0/ckeditor5-premium-features.css'
			],
			fileName: 'export-word-demo.docx',
			converterOptions: {
				document: {
					orientation: 'portrait',
					size: 'A4',
					margins: {
						top: '20mm',
						bottom: '20mm',
						right: '12mm',
						left: '12mm'
					}
				}
			}
		},
		fontFamily: {
			supportAllValues: true
		},
		fontSize: {
			options: [10, 12, 14, 'default', 18, 20, 22],
			supportAllValues: true
		},
		heading: {
			options: [
				{
					model: 'paragraph',
					title: 'Paragraph',
					class: 'ck-heading_paragraph'
				},
				{
					model: 'heading1',
					view: 'h1',
					title: 'Heading 1',
					class: 'ck-heading_heading1'
				},
				{
					model: 'heading2',
					view: 'h2',
					title: 'Heading 2',
					class: 'ck-heading_heading2'
				},
				{
					model: 'heading3',
					view: 'h3',
					title: 'Heading 3',
					class: 'ck-heading_heading3'
				},
				{
					model: 'heading4',
					view: 'h4',
					title: 'Heading 4',
					class: 'ck-heading_heading4'
				},
				{
					model: 'heading5',
					view: 'h5',
					title: 'Heading 5',
					class: 'ck-heading_heading5'
				},
				{
					model: 'heading6',
					view: 'h6',
					title: 'Heading 6',
					class: 'ck-heading_heading6'
				}
			]
		},
		image: {
			toolbar: [
				'toggleImageCaption',
				'imageTextAlternative',
				'|',
				'imageStyle:inline',
				'imageStyle:wrapText',
				'imageStyle:breakText',
				'|',
				'resizeImage',
				'|',
				'ckboxImageEdit'
			]
		},
		initialData:
			'<h2>Congratulations on setting up CKEditor 5! 🎉</h2>\n<p>\n\tYou\'ve successfully created a CKEditor 5 project. This powerful text editor\n\twill enhance your application, enabling rich text editing capabilities that\n\tare customizable and easy to use.\n</p>\n<h3>What\'s next?</h3>\n<ol>\n\t<li>\n\t\t<strong>Integrate into your app</strong>: time to bring the editing into\n\t\tyour application. Take the code you created and add to your application.\n\t</li>\n\t<li>\n\t\t<strong>Explore features:</strong> Experiment with different plugins and\n\t\ttoolbar options to discover what works best for your needs.\n\t</li>\n\t<li>\n\t\t<strong>Customize your editor:</strong> Tailor the editor\'s\n\t\tconfiguration to match your application\'s style and requirements. Or\n\t\teven write your plugin!\n\t</li>\n</ol>\n<p>\n\tKeep experimenting, and don\'t hesitate to push the boundaries of what you\n\tcan achieve with CKEditor 5. Your feedback is invaluable to us as we strive\n\tto improve and evolve. Happy editing!\n</p>\n<h3>Helpful resources</h3>\n<ul>\n\t<li>📝 <a href="https://portal.ckeditor.com/checkout?plan=free">Trial sign up</a>,</li>\n\t<li>📕 <a href="https://ckeditor.com/docs/ckeditor5/latest/installation/index.html">Documentation</a>,</li>\n\t<li>⭐️ <a href="https://github.com/ckeditor/ckeditor5">GitHub</a> (star us if you can!),</li>\n\t<li>🏠 <a href="https://ckeditor.com">CKEditor Homepage</a>,</li>\n\t<li>🧑‍💻 <a href="https://ckeditor.com/ckeditor-5/demo/">CKEditor 5 Demos</a>,</li>\n</ul>\n<h3>Need help?</h3>\n<p>\n\tSee this text, but the editor is not starting up? Check the browser\'s\n\tconsole for clues and guidance. It may be related to an incorrect license\n\tkey if you use premium features or another feature-related requirement. If\n\tyou cannot make it work, file a GitHub issue, and we will help as soon as\n\tpossible!\n</p>\n',
		licenseKey: LICENSE_KEY,
		link: {
			addTargetToExternalLinks: true,
			defaultProtocol: 'https://',
			decorators: {
				toggleDownloadable: {
					mode: 'manual',
					label: 'Downloadable',
					attributes: {
						download: 'file'
					}
				}
			}
		},
		list: {
			properties: {
				styles: true,
				startIndex: true,
				reversed: true
			}
		},
		mention: {
			feeds: [
				{
					marker: '@',
					feed: [
						/* See: https://ckeditor.com/docs/ckeditor5/latest/features/mentions.html */
					]
				}
			]
		},
		menuBar: {
			isVisible: true
		},
		mergeFields: {
			definitions: 	
				[
					{
						id: 'myName',
						label: 'My Name',
						defaultValue: 'John Doe'
					},
					{
						id: 'meetingDate',
						label: 'Meeting Date',
						defaultValue: '2025-02-14'
					},
				],
		},
		pagination: {
			pageWidth: '21cm',
			pageHeight: '29.7cm',
			pageMargins: {
				top: '20mm',
				bottom: '20mm',
				right: '12mm',
				left: '12mm'
			}
		},
		placeholder: 'Type or paste your content here!',
		presenceList: {
			container: editorPresence.value
		},
		revisionHistory: {
			editorContainer: editorContainer.value,
			viewerContainer: editorRevisionHistory.value,
			viewerEditorElement: editorRevisionHistoryEditor.value,
			viewerSidebarContainer: editorRevisionHistorySidebar.value,
			resumeUnsavedRevision: true
		},
		sidebar: {
			container: editorAnnotations.value
		},
		table: {
			contentToolbar: ['tableColumn', 'tableRow', 'mergeTableCells', 'tableProperties', 'tableCellProperties']
		},
		template: {
			definitions: [
				{
					title: "Meeting Notes",
					description: "A structured template for capturing meeting discussions, decisions, and action items",
					data: `<h1 style="text-align:center;">
								Meeting Notes
							</h1>
							<figure class="table" style="width:100%;">
								<table style="border-style:none;">
									<thead>
										<tr>
											<th style="border-style:none;">
												Meeting Details
											</th>
										</tr>
									</thead>
									<tbody>
										<tr>
											<td style="border-style:none;">
												<p>
													<strong>Date:</strong> 2024-06-20
												</p>
												<p>
													<strong>Time:</strong> 10:00 AM - 11:30 AM
												</p>
												<p>
													<strong>Attendees:</strong>John Doe
												</p>
											</td>
										</tr>
									</tbody>
								</table>
							</figure>
							<h2>Agenda</h2>
							<figure class="table" style="width:100%;">
								<table>
									<thead>
										<tr>
											<th>Item</th>
											<th>Description</th>
											<th>Presenter</th>
										</tr>
									</thead>
									<tbody>
										<tr>
											<td></td>
											<td></td>
											<td></td>
										</tr>
									</tbody>
								</table>
							</figure>
							<h2>Meeting Notes</h2>
							<h2>Decisions Made</h2>
							<h2>Action Items</h2>
							<figure class="table" style="width:100%;">
								<table>
									<thead>
										<tr>
											<th>Task</th>
											<th>Owner</th>
											<th>Due Date</th>
										</tr>
									</thead>
									<tbody>
										<tr>
											<td></td>
											<td></td>
											<td></td>
										</tr>
									</tbody>
								</table>
							</figure>`
				}
			]
		}
	};
});

onMounted(() => {
	isLayoutReady.value = true;
});

function onReady(editor) {
	[...editorToolbar.value.children].forEach(child => child.remove());
	[...editorMenuBar.value.children].forEach(child => child.remove());

	editorToolbar.value.appendChild(editor.ui.view.toolbar.element);
	editorMenuBar.value.appendChild(editor.ui.view.menuBarView.element);
}

</script>
