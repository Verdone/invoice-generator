<script>
	import ThirdPartyResource from './ThirdPartyResource.svelte';

	const resources = [
        {
			name: 'Icons8',
			url: 'https://icons8.com',
			licenseUrl: 'https://intercom.help/icons8-7fb7577e8170/en/articles/5534926-universal-multimedia-license-agreement-for-icons8',
			licenseText:
				'The favicon for this website uses the Document icon from Icons8.',
			fullLicenseName: 'Icons8 License'
		},
		{
			name: 'SvelteKit',
			url: 'https://kit.svelte.dev/',
			licenseUrl: 'https://github.com/sveltejs/kit/blob/main/LICENSE',
			licenseText:
				'SvelteKit simplifies the process of creating full-stack web applications with the Svelte JavaScript framework.',
			fullLicenseName: 'MIT License'
		},
		{
			name: 'pdfmake',
			url: 'https://github.com/bpampuch/pdfmake/tree/master',
			licenseUrl: 'https://github.com/bpampuch/pdfmake/blob/master/LICENSE',
			licenseText:
				'Client/server side PDF printing in pure JavaScript.',
			fullLicenseName: 'MIT License'
		},
		{
			name: 'TailwindCSS',
			url: 'https://tailwindcss.com/',
			licenseUrl: 'https://github.com/tailwindlabs/tailwindcss/blob/master/LICENSE',
			licenseText:
				"Tailwind CSS is a utility-first CSS framework that's great for prototyping because it promotes rapid development and easily customizable styles.",
			fullLicenseName: 'MIT License'
		},
        {
			name: 'shadcn-svelte',
			url: 'https://github.com/huntabyte/shadcn-svelte',
			licenseUrl: 'https://github.com/huntabyte/shadcn-svelte/blob/main/LICENSE.md',
			licenseText: 'A port of the shadcn React UI library for SvelteKit.',
			fullLicenseName: 'MIT License'
		},
		{
			name: 'Vercel',
			url: 'https://vercel.com/',
			licenseUrl: 'https://github.com/vercel/vercel/blob/main/LICENSE',
			licenseText:
				'Vercel is a cloud platform for frontend development and serverless functions. It allows anyone to deploy and host their own websites/applications.',
			fullLicenseName: 'Apache License 2.0'
		}
	];
</script>

<div class="py-16 min-h-[100vh]">

    <!-- Licensing Page Header -->
    <div class="mx-auto mb-12 max-w-2xl text-center">
        <h1 class="mb-2 text-sm sm:text-base font-semibold text-green-500">Canadian Invoice Generator</h1>
        <p class="mb-6 text-3xl sm:text-5xl font-bold">Licensing</p>
    </div>

    
    <div class="py-8 px-4 mx-auto max-w-screen-xl lg:py-16 lg:px-6">
        <div class="max-w-screen-lg sm:text-lg">
            <h2 class="mb-4 text-4xl tracking-tight font-bold text-gray-900 dark:text-white">
                Copywright
            </h2>
            <p class="mb-8 font-light">
                The following third-party resources were used in the creation of this website:
            </p>

            <ul>
                {#each resources as resource (resource.name)}
                    <ThirdPartyResource {...resource} />
                {/each}
            </ul>

        </div>
    </div>

</div>