<script>
	export let name;
	export let url;
	export let licenseUrl;
	export let licenseText;
	export let fullLicenseName;
</script>

<li>
	<a href={url} target="_blank" class="text-2xl text-green-500 hover:text-green-300">{name}</a>

	<p class="mb-10 font-light">
		{licenseText} It is used under the terms of the
		<a href={licenseUrl} target="_blank" class="text-green-500 hover:text-green-300"
			>{fullLicenseName}</a
		>.
	</p>
</li>
