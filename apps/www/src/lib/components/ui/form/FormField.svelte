<script lang="ts">
	import { tick } from "svelte";

	import { cn } from "$lib/utils";
	import type { SuperForm } from "sveltekit-superforms/client";
	import type { AnyZodObject } from "zod";
	import type { SuperFormPath } from ".";
	import { superFormField, superFormFieldProxy } from ".";

	let className: string | undefined | null = undefined;
	export { className as class };

	type T = $$Generic<AnyZodObject>;

	export let form: SuperForm<T>;
	export let name: SuperFormPath<T>;
	export let id: string | null | undefined = String(name);
	export let tag = "div";

	const { value, errors, constraints } = superFormFieldProxy(form, name);

	const fieldName = name.match(/^[a-zA-Z]\w*/);
	if (!fieldName) throw new Error("Invalid form field name.");

	$: field = superFormField({
		id,
		name: fieldName[0],
		errors: $errors,
		constraints: $constraints,
		value
	});

	let uuid: string | null = null;

	$: fieldWithListeners = () => {
		const uuidWasNull = uuid === null;
		uuid = uuidWasNull ? Math.random().toString(36).slice(2) : uuid;
		if (typeof window !== "undefined" && uuidWasNull) {
			tick().then(() => {
				const el = document.querySelector(`[data-uuid="${uuid}"]`);
				if (!el) return;

				const cb = (e: Event) => {
					const target = e.target as HTMLInputElement;
					field.value?.set(target.value);
				};

				el.addEventListener("keydown", cb);
				el.addEventListener("keyup", cb);
				el.addEventListener("change", cb);
			});
		}

		return {
			...field,
			constraints: undefined,
			errors: undefined,
			value: $value,
			"data-uuid": uuid
		};
	};
</script>

<svelte:element this={tag} class={cn("grid gap-2", className)} {...$$restProps}>
	<slot {field} {fieldWithListeners} />
</svelte:element>
