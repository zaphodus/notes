<template>
	<AppNavigationItem :item="item" :menu-open.sync="menuOpen" />
</template>

<script>
import {
	AppNavigationItem,
} from 'nextcloud-vue'
import NotesService from '../NotesService'

export default {
	name: 'NavigationNoteItem',

	components: {
		AppNavigationItem,
	},

	props: {
		note: {
			type: Object,
			required: true,
		},
	},

	data: function() {
		return {
			loading: {
				favorite: false,
				delete: false,
			},
			menuOpen: false,
		}
	},

	computed: {
		item() {
			let icon = ''
			if (this.note.error) {
				icon = 'nav-icon icon-error-color'
			} else if (this.note.favorite) {
				icon = 'nav-icon icon-starred'
			}
			let iconActionFavorite = this.note.favorite ? 'icon-star-dark' : 'icon-starred'
			if (this.loading.favorite) {
				iconActionFavorite += ' loading'
			}
			return {
				text: this.note.title + (this.note.unsaved ? ' *' : ''),
				icon: icon,
				router: {
					name: 'note',
					params: {
						noteId: this.note.id.toString(),
					},
				},
				utils: {
					actions: [
						{
							text: this.note.favorite ? this.t('notes', 'Remove from favorites') : this.t('notes', 'Add to favorites'),
							icon: iconActionFavorite,
							action: this.onToggleFavorite,
						},
						{
							text: NotesService.categoryLabel(this.note.category),
							icon: 'icon-files-dark',
							action: this.onCategorySelected,
						},
						{
							text: this.t('notes', 'Delete note'),
							icon: 'icon-delete' + (this.loading.delete ? ' loading' : ''),
							action: this.onDeleteNote,
						},
					],
				},
			}
		},
	},

	methods: {
		onToggleFavorite() {
			this.loading.favorite = true
			NotesService.setFavorite(this.note.id, !this.note.favorite)
				.catch(() => {
				})
				.finally(() => {
					this.loading.favorite = false
					this.menuOpen = false
				})
		},

		onCategorySelected() {
			this.menuOpen = false
			this.$emit('category-selected', this.note.category)
		},

		onDeleteNote() {
			this.loading.delete = true
			NotesService.deleteNote(this.note.id)
				.then(() => {
					this.$emit('note-deleted')
				})
				.catch(() => {
				})
				.finally(() => {
					this.loading.delete = false
					this.menuOpen = false
				})
			// TODO implement undo
		},
	},
}
</script>
