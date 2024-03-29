<script>
	import { setContext } from "svelte"
	import { createEventDispatcher } from "svelte"
	import { writable } from "svelte/store"

	export let form = {}
	export let sharedValidationFn

	let formElement
	let errors = writable({})

	const dispatch = createEventDispatcher()

	const isFormValid = () => {
		return !Object.values($errors).some(field => Object.values(field).some(errorObject => errorObject.error))
	}

	const validateField = (field, value) => {
		form[field]?.validators &&
			form[field].validators.forEach(fn => {
				const error = fn(value)
				errors.update(e => {
					e[field] = { ...e[field], ...error }
					return e
				})
			})
	}

	const validateForm = (data, fn = null) => {
		Object.keys(data).forEach(field => validateField(field, data[field]))

		if (fn) {
			let error = fn(data)
			errors.update(e => {
				e.shared = { ...e.shared, ...error }
				return e
			})
		}
	}

	const onSubmit = e => {
		const formData = new FormData(e.target)

		const data = {}

		for (let field of formData) {
			const [key, value] = field
			data[key] = value ? Number(value) : null
		}

		validateForm(data, sharedValidationFn)

		return dispatch("submit", { valid: isFormValid(), data })
	}

	export const reset = () => formElement.reset()

	setContext("form", { errors })
</script>

<form on:submit={onSubmit} bind:this={formElement}>
	<slot />
</form>

<style>
	form {
		position: relative;
		display: grid;
		grid-template-columns: 1fr 1fr 1fr;
		column-gap: 1rem;

		border-bottom: 1px solid var(--color-neutral-400);

		padding-bottom: 4rem;
	}

	@media (min-width: 50rem) {
		form {
			padding-block-end: 3rem;

			grid-template-columns: repeat(4, 1fr);
			column-gap: 2rem;
		}
	}
</style>
