<template lang="pug">
.textbox(:class='{\
	"empty"      : value === "",\
	"show-label" : ( focused && value !== "" ),\
	"has-error"  : ( displayError !== "" ),\
	focused,\
}')
	p.label.noselect(@click='focus') {{ label }}
	.text-wrapper
		textarea(
			ref='textarea'
			v-model='value'
			:placeholder='placeholder'
			:class='classList'
			@input='resizeInput'
			@focus='focused = true; emitEvent( "focus", $event )'
			@blur='focused = false; alertBlur( $event )'
			@keydown.enter.exact='emitEvent( "enter", $event )'
		)
	p.text-error {{ displayError }}
</template>

<script>
export default {
	name : 'textbox',

	props : {
		initialValue : {
			default : () => ''
		},
		placeholder : {
			default : () => 'Type here to input text'
		},
		label : {
			default : () => 'input'
		},
		classList : {
			default : () => ''
		},
		error : {
			default : () => ''
		},
	},

	data : () => ( {
		focused      : false,
		value        : '',
		displayError : '',
	} ),

	created() {
		this.value = this.initialValue;
	},

	mounted() {
		this.$nextTick( () => this.resizeInput() );
	},

	watch : {

		error( e ) {
			this.displayError = e;
		},

		value( value ) {
			this.displayError = '';
			this.$emit( 'value', value );

			const { textarea } = this.$refs;
			this.$emit( 'change', textarea );
		},

		initialValue( val ) {

			// without this line, you trigger an infinite loop
			// of emitting, hearing a change, emitting, hearing a change,
			// forever
			if ( val !== this.value ) {
				this.value = val;

				this.$nextTick( () => this.resizeInput() );
			}

		}

	},

	methods : {

		resizeInput() {

			const { textarea } = this.$refs;

			this.$emit( 'change', textarea );

			if ( !textarea ) {
				return;
			}

			this.$emit( 'change', textarea );

			const { height } = textarea.style;

			let heightNumeric = parseInt( height.substr( 0, height.length - 2 ), 10 );

			while ( heightNumeric === textarea.scrollHeight ) {

				heightNumeric -= 5;
				textarea.style.height = `${heightNumeric}px`;

			}

			textarea.style.height = `${textarea.scrollHeight}px`;

		},

		focus() {
			this.$refs.textarea.focus();
		},

		alertBlur( e ) {
			this.$emit( 'blur', e );
		},

		emitEvent( eventName, e = null ) {
			if ( e instanceof window.Event ) {
				e.preventDefault();
			}

			this.$emit( eventName, e );
		}
	}
};
</script>

<style lang='scss'>
$primaryColor : #2E5491 !default;
$grey         : #4a5155 !default;
$lightGrey    : #ebf1f7 !default;
$error        : #dc3737 !default;

.textbox {
	transition: padding 0.5s ease;
	position: relative;
	cursor: pointer;
	will-change: padding-top;

	&.min-padding {

		.text-wrapper {
			padding: 9px;
		}

		&.left-align {

			p.label {
				left: 9px;
			}
		}
	}

	&.left-align {

		p.label {
			left: 15px;
			transform: translateY(-50%);
		}
	}

	&.focused {

		.text-wrapper {
			border: 1px dashed $primaryColor !important;
		}
	}

	&.empty {

		.text-wrapper {
			border: 1px dashed $lightGrey;
		}
	}

	&.show-label,
	&:hover:not(.empty) {

		p.label {
			top: -2px;
			max-height: 18px;
			opacity: 1;
		}

		.text-wrapper {
			border: 1px dashed $lightGrey;
		}
	}

	&.has-error {

		.text-wrapper {
			border: 1px dashed $error;
		}

		p.text-error {
			bottom: 1px;
			max-height: 12px;
			opacity: 1;
		}
	}

	.text-wrapper {
		padding: 14px;
		border-radius: 5px;
		border: 1px dashed transparent;
		transition: border 0.5s ease;

		&:hover {
			border: 1px dashed $lightGrey;
		}

		textarea {
			background-color: none;
			border: none;
			outline: none;
			width: 100%;
			background-color: transparent;
			cursor: pointer;
			display: inherit;

			&.h1 {
				font-family: 'Trocchi', open-sans;
				font-size: 36px;
				line-height: 120%;
				color: $primaryColor;
			}

			&.h2 {
				font-family: 'Trocchi', open-sans;
				font-size: 24px;
				line-height: 120%;
				color: $primaryColor;
			}

			&.h3 {
				font-family: 'Trocchi', open-sans;
				font-size: 18px;
				line-height: 120%;
				color: $primaryColor;
			}

			&.p {
				font-family: 'Work Sans', open-sans;
				font-size: 18px;
				line-height: 150%;
				color: $grey;

				&.small {
					font-size: 14px;
				}
			}
		}
	}

	p.text-error,
	p.label {
		position: absolute;
		// max-width: calc(100% - 60px);
		max-height: 0px;
		overflow: hidden;
		white-space: nowrap;
		text-overflow: ellipsis;
		padding: 2px;
		background-color: white;
		font-weight: 600;
		letter-spacing: 2px;
		text-transform: uppercase;
		transition: max-height 0.5s ease, top 0.5s ease, opacity 0.5s ease;
		opacity: 0;
		font-size: 12px;
		left: 50%;
	}

	p.text-error {
		bottom: 22px;
		color: $error;
		transform: translate(-50%,50%);
		padding: 0 2px;
		display: flex;
		align-items: center;
	}

	p.label {
		top: 10px;
		color: $primaryColor;
		cursor: pointer;
		transform: translate(-50%,-50%);
	}
}
</style>
