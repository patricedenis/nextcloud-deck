<!--
  - SPDX-FileCopyrightText: 2020 Nextcloud GmbH and Nextcloud contributors
  - SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
	<div class="attachments-drag-zone"
		@dragover.prevent="!isDraggingOver && (isDraggingOver = true)"
		@dragleave.prevent="isDraggingOver && (isDraggingOver = false)"
		@drop.prevent="handleDropFiles">
		<slot />
		<transition name="fade" mode="out-in">
			<div v-show="isDraggingOver"
				class="dragover">
				<div class="drop-hint">
					<div class="drop-hint__icon"
						:class="{
							'icon-upload' : !isReadOnly,
							'icon-error' : isReadOnly}" />
					<h2 class="drop-hint__text">
						{{ dropHintText }}
					</h2>
				</div>
			</div>
		</transition>

		<NcModal v-if="modalShow" :name="t('deck', 'File already exists')" @close="modalShow=false">
			<div class="modal__content">
				<h2>{{ t('deck', 'File already exists') }}</h2>
				<p>
					{{ t('deck', 'A file with the name {filename} already exists.', {filename: file.name}) }}
				</p>
				<p>
					{{ t('deck', 'Do you want to overwrite it?') }}
				</p>
				<button class="primary" @click="overrideAttachment">
					{{ t('deck', 'Overwrite file') }}
				</button>
				<button @click="modalShow=false">
					{{ t('deck', 'Keep existing file') }}
				</button>
			</div>
		</NcModal>
	</div>
</template>

<script>
import { NcModal } from '@nextcloud/vue'
import attachmentUpload from '../mixins/attachmentUpload.js'
import { loadState } from '@nextcloud/initial-state'

const maxUploadSizeState = loadState('deck', 'maxUploadSize', -1)

export default {
	name: 'AttachmentDragAndDrop',
	components: { NcModal },
	mixins: [attachmentUpload],
	props: {
		cardId: {
			type: Number,
			default: null,
		},
	},
	data() {
		return {
			modalShow: false,
			file: '',
			overwriteAttachment: null,
			isDraggingOver: false,
			maxUploadSize: maxUploadSizeState,
		}
	},
	computed: {
		isReadOnly() {
			return !this.$store.getters.canEdit
		},
		dropHintText() {
			if (this.isReadOnly) {
				return t('deck', 'This board is read only')
			} else {
				return t('deck', 'Drop your files to upload')
			}
		},
	},
	methods: {
		handleDropFiles(event) {
			event.dataTransfer.dropEffect = 'copy'
			this.isDraggingOver = false
			if (this.isReadOnly) {
				return
			}
			const files = event.dataTransfer.files
			for (const file of files) {
				this.onLocalAttachmentSelected(file, 'file')
			}
			event.dataTransfer.value = ''
		},
	},
}
</script>

<style scoped lang="scss">

	.attachments-drag-zone {
		flex-grow: 1;
		position: relative;
	}

	.attachments-drag-zone.drop-upload--sidebar {
		display: flex;
		flex-direction: column;
		flex-basis: 100%;
	}

	.dragover {
		position: absolute;
		background: var(--color-primary-element-light);
		z-index: 11;
		display: flex;
		border-radius: var(--border-radius);
		opacity: .9;
		margin: auto;

		.drop-hint {
			width: 100%;
			display: flex;
			flex-direction: column;
			justify-content: center;

			&__icon {
				background-size: 32px;
				background-position: center center;
				height: 48px;
				margin-bottom: 16px;
			}

			&__text {
				line-height: 125%;
				text-align: center;
			}
		}
	}

	.drop-upload--sidebar .dragover {
		top: 20%;
		left: 10%;
		width: 80%;
		height: 60%;
		box-shadow: 0px 0px 36px var(--color-box-shadow);
	}

	.drop-upload--card .dragover {
		border: 0;
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;

		h2 {
			font-size: 13px;
			margin: 0;
		}

		.drop-hint__icon {
			margin: 0;
			background-size: 16px;
			height: 16px;
		}
	}

	.fade {
		&-enter {
			opacity: 0;
		}
		&-enter-to {
			opacity: .9;
		}
		&-leave {
			opacity: .9;
		}
		&-leave-to {
			opacity: 0;
		}
		&-enter-active,
		&-leave-active {
			transition: opacity 150ms ease-in-out;
		}
	}

	.modal__content {
		width: 25vw;
		min-width: 250px;
		height: 120px;
		text-align: center;
		margin: 20px 20px 60px 20px;
	}

</style>
