<template>
<div class="fdidabkb" :class="{ center }" :style="`--height:${height};`" :key="key">
	<transition :name="$store.state.animation ? 'header' : ''" mode="out-in" appear>
		<button class="_button back" v-if="withBack && canBack" @click.stop="back()" v-tooltip="$ts.goBack"><Fa :icon="faChevronLeft"/></button>
	</transition>
	<template v-if="info">
		<div class="titleContainer">
			<div class="title">
				<Fa v-if="info.icon" :icon="info.icon" :key="info.icon" class="icon"/>
				<MkAvatar v-else-if="info.avatar" class="avatar" :user="info.avatar" :disable-preview="true"/>
				<MkUserName v-if="info.userName" :user="info.userName" :nowrap="false" class="text"/>
				<span v-else-if="info.title" class="text">{{ info.title }}</span>
			</div>
		</div>
		<div class="buttons">
			<template v-if="info.actions && showActions">
				<button v-for="action in info.actions" class="_button button" @click.stop="action.handler" v-tooltip="action.text"><Fa :icon="action.icon"/></button>
			</template>
			<button v-if="showMenu" class="_button button" @click.stop="menu"><Fa :icon="faEllipsisH"/></button>
		</div>
	</template>
</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { faChevronLeft, faCircle, faShareAlt, faEllipsisH } from '@fortawesome/free-solid-svg-icons';
import { modalMenu } from '@client/os';
import { url } from '@client/config';

export default defineComponent({
	props: {
		info: {
			required: true
		},
		withBack: {
			type: Boolean,
			required: false,
			default: true,
		},
		center: {
			type: Boolean,
			required: false,
			default: true,
		},
	},

	data() {
		return {
			canBack: false,
			showActions: false,
			height: 0,
			key: 0,
			faChevronLeft, faCircle, faShareAlt, faEllipsisH,
		};
	},

	computed: {
		showMenu() {
			if (this.info.actions != null && !this.showActions) return true;
			if (this.info.menu != null) return true;
			if (this.info.share != null) return true;
			return false;
		}
	},

	watch: {
		info() {
			this.key++;
		},

		$route: {
			handler(to, from) {
				this.canBack = (window.history.length > 0 && !['index'].includes(to.name));
			},
			immediate: true
		},
	},

	mounted() {
		this.height = this.$el.parentElement.offsetHeight + 'px';
		this.showActions = this.$el.parentElement.offsetWidth >= 500;
		new ResizeObserver((entries, observer) => {
			this.height = this.$el.parentElement.offsetHeight + 'px';
			this.showActions = this.$el.parentElement.offsetWidth >= 500;
		}).observe(this.$el);
	},

	methods: {
		back() {
			if (this.canBack) this.$router.back();
		},

		share() {
			navigator.share({
				url: url + this.info.path,
				...this.info.share,
			});
		},

		menu(ev) {
			let menu = this.info.menu ? this.info.menu() : [];
			if (!this.showActions && this.info.actions) {
				menu = [...this.info.actions.map(x => ({
					text: x.text,
					icon: x.icon,
					action: x.handler
				})), menu.length > 0 ? null : undefined, ...menu];
			}
			if (this.info.share) {
				if (menu.length > 0) menu.push(null);
				menu.push({
					text: this.$ts.share,
					icon: faShareAlt,
					action: this.share
				});
			}
			modalMenu(menu, ev.currentTarget || ev.target);
		}
	}
});
</script>

<style lang="scss" scoped>
.fdidabkb {
	&.center {
		text-align: center;

		> .titleContainer {
			margin: 0 auto;
		}
	}

	> .back {
		position: absolute;
		z-index: 1;
		top: 0;
		left: 0;
		height: var(--height);
		width: var(--height);
	}

	> .buttons {
		position: absolute;
		z-index: 1;
		top: 0;
		right: 0;

		> .button {
			height: var(--height);
			width: var(--height);
		}
	}

	> .titleContainer {
		overflow: auto;
		white-space: nowrap;
		width: calc(100% - (var(--height) * 2));

		> .title {
			display: inline-block;
			vertical-align: bottom;
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
			padding: 0 16px;
			position: relative;
			height: var(--height);

			> .icon + .text {
				margin-left: 8px;
			}

			> .avatar {
				$size: 32px;
				display: inline-block;
				width: $size;
				height: $size;
				vertical-align: bottom;
				margin: calc((var(--height) - #{$size}) / 2) 8px calc((var(--height) - #{$size}) / 2) 0;
				pointer-events: none;
			}
		}
	}
}
</style>
