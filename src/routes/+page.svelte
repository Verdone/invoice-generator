<script lang="ts">
    //@ts-nocheck
    import { onMount } from 'svelte';
    import { Input } from "$lib/components/ui/input";
    import { Label } from "$lib/components/ui/label/index.js";
    import toast, { Toaster } from 'svelte-french-toast';
    import * as ToggleGroup from "$lib/components/ui/toggle-group/index.js";
    import * as Table from "$lib/components/ui/table/index.js";
    import CirclePlus from "lucide-svelte/icons/circle-plus";
    import CircleMinus from "lucide-svelte/icons/circle-minus";
    import { Button } from "$lib/components/ui/button/index.js";
    import { Switch } from "$lib/components/ui/switch";
	import { Textarea } from "$lib/components/ui/textarea";
    import { FileText } from 'lucide-svelte';
    import { DateFormatter, today, getLocalTimeZone } from "@internationalized/date";
    import { ImageUp } from 'lucide-svelte';

    const removeItem = (index) => {
        if (invoiceItems.length <= 1) return;
        invoiceItems = invoiceItems.filter((_, i) => i !== index);
    };


    const df = new DateFormatter("en-US", {
        dateStyle: "long"
    });

    // Invoice Information instance variables
    let invoiceNumber = '';
    let invoiceDateIssued = today(getLocalTimeZone());
    let invoiceDueDate = today(getLocalTimeZone());
    let invoiceStatus = '';
    function getInvoiceStatusColor(status) {
        // Convert the status to lowercase
        const lowerCaseStatus = status.toLowerCase();
        
        // Determine the color based on the status
        switch (lowerCaseStatus) {
            case 'paid':
                return 'green';
            case 'overdue':
                return 'red';
            default:
                return 'gray';
        }
    }

    // Company logo
    let avatar = '';
    let fileinput;
	const onFileSelected =(e)=>{
        let image = e.target.files[0];
        let reader = new FileReader();
        reader.readAsDataURL(image);
        reader.onload = e => {
            avatar = e.target.result;
        };
    }

    // Client Information instance variables
    let clientName = '';
    let clientCompany = '';
    let clientStreetAddress = '';
    let clientCityStatePostalCode = '';
    let clientCountry = '';

    // Your Information instance variables
    let yourName = '';
    let yourCompany = '';
    let yourStreetAddress = '';
    let yourCityStatePostalCode = '';
    let yourCountry = '';
    
    // Invoice Items instance variables
    let invoiceItems = [{ description: '', quantity: 1, price: 0, applyQST: true, applyGST: true }];
    
    // Notes and TOC strings
    let notesText = '';
    let termsAndConditionsText = '';

    // errors JSON variable for really, really rudimentary validation
    let errors = {
        invoiceNumber: false,
        invoiceStatus: false,
        avatar: false,
        yourName: false,
        yourStreetAddress: false,
        yourCompany: false,
        yourCityStatePostalCode: false,
        yourCountry: false, 
        clientName: false,
        clientStreetAddress: false,
        clientCompany: false,
        clientCityStatePostalCode: false,
        clientCountry: false,
        termsAndConditionsText: false
    };

    const addItem = () => {
        invoiceItems = [...invoiceItems, { description: '', quantity: 1, price: 0, applyQST: true, applyGST: true }];
    };

    const validate = () => {
        // Validate inputs for basic invoice information
        errors.invoiceNumber = invoiceNumber.trim() === '';
        errors.invoiceStatus = invoiceStatus.trim() === '';
        errors.avatar = avatar.trim() === '';

        // Validate inputs for your information
        errors.yourName = yourName.trim() === '';
        errors.yourStreetAddress = yourStreetAddress.trim() === '';
        errors.yourCompany = yourCompany.trim() === '';
        errors.yourCityStatePostalCode = yourCityStatePostalCode.trim() === '';
        errors.yourCountry = yourCountry.trim() === '';

        // Validate inputs for client information
        errors.clientName = clientName.trim() === '';
        errors.clientStreetAddress = clientStreetAddress.trim() === '';
        errors.clientCompany = clientCompany.trim() === '';
        errors.clientCityStatePostalCode = clientCityStatePostalCode.trim() === '';
        errors.clientCountry = clientCountry.trim() === '';

        // Validate input for terms/conditions
        errors.termsAndConditionsText = termsAndConditionsText.trim() === '';

        // Validate each item
        invoiceItems.forEach((item, index) => {
            errors[`itemDescription${index}`] = item.description.trim() === '';
            errors[`itemQuantity${index}`] = isNaN(item.quantity) || item.quantity <= 0;
            
        });
    };

    const generatePDF = async () => {
        // Call the validate() method to ensure inputs are acceptable
        validate();
        const hasErrors = Object.values(errors).some(error => error);
        if (!hasErrors) {
            const pdfMake = (await import('pdfmake/build/pdfmake')).default;
            const pdfFonts = (await import('pdfmake/build/vfs_fonts')).default;
            pdfMake.vfs = pdfFonts.pdfMake.vfs;

            const dd = {
                content: [
                    {
                        columns: [
                            {
                                    image: `${avatar}`,
                                    width: 75,
                            },
                            [
                                {
                                    text: 'INVOICE',
                                    color: '#333333',
                                    width: '*',
                                    fontSize: 28,
                                    bold: true,
                                    alignment: 'right',
                                    margin: [0, 0, 0, 15],
                                },
                                {
                                    stack: [
                                        {
                                            columns: [
                                                {
                                                    text: 'Invoice No.',
                                                    color: '#aaaaab',
                                                    bold: true,
                                                    width: '*',
                                                    fontSize: 12,
                                                    alignment: 'right',
                                                },
                                                {
                                                    text: `${invoiceNumber}`,
                                                    bold: true,
                                                    color: '#333333',
                                                    fontSize: 12,
                                                    alignment: 'right',
                                                    width: 100,
                                                },
                                            ],
                                        },
                                        {
                                            columns: [
                                                {
                                                    text: 'Date Issued',
                                                    color: '#aaaaab',
                                                    bold: true,
                                                    width: '*',
                                                    fontSize: 12,
                                                    alignment: 'right',
                                                },
                                                {
                                                    text: `${df.format(invoiceDateIssued.toDate(getLocalTimeZone()))}`,
                                                    bold: true,
                                                    color: '#333333',
                                                    fontSize: 12,
                                                    alignment: 'right',
                                                    width: 100,
                                                },
                                            ],
                                        },
                                        {
                                            columns: [
                                                {
                                                    text: 'Due Date',
                                                    color: '#aaaaab',
                                                    bold: true,
                                                    width: '*',
                                                    fontSize: 12,
                                                    alignment: 'right',
                                                },
                                                {
                                                    text: `${df.format(invoiceDueDate.toDate(getLocalTimeZone()))}`,
                                                    bold: true,
                                                    color: '#333333',
                                                    fontSize: 12,
                                                    alignment: 'right',
                                                    width: 100,
                                                },
                                            ],
                                        },
                                        {
                                            columns: [
                                                {
                                                    text: 'Status',
                                                    color: '#aaaaab',
                                                    bold: true,
                                                    fontSize: 12,
                                                    alignment: 'right',
                                                    width: '*',
                                                },
                                                {
                                                    text: `${invoiceStatus.toUpperCase()}`, // var status
                                                    bold: true,
                                                    fontSize: 14,
                                                    alignment: 'right',
                                                    color: getInvoiceStatusColor(invoiceStatus), // var statusColor
                                                    width: 100,
                                                },
                                            ],
                                        },
                                    ],
                                },
                            ],
                        ],
                    },
                    {
                        columns: [
                            {
                                text: 'From',
                                color: '#aaaaab',
                                bold: true,
                                fontSize: 14,
                                alignment: 'left',
                                margin: [0, 20, 0, 5],
                            },
                            {
                                text: 'To',
                                color: '#aaaaab',
                                bold: true,
                                fontSize: 14,
                                alignment: 'left',
                                margin: [0, 20, 0, 5],
                            },
                        ],
                    },
                    {
                        columns: [
                            {
                                text: `${yourName} \n ${yourCompany}`,
                                bold: true,
                                color: '#333333',
                                alignment: 'left',
                            },
                            {
                                text: `${clientName} \n ${clientCompany}`,
                                bold: true,
                                color: '#333333',
                                alignment: 'left',
                            },
                        ],
                    },
                    {
                        columns: [
                            {
                                text: 'Address',
                                color: '#aaaaab',
                                bold: true,
                                margin: [0, 7, 0, 3],
                            },
                            {
                                text: 'Address',
                                color: '#aaaaab',
                                bold: true,
                                margin: [0, 7, 0, 3],
                            },
                        ],
                    },
                    {
                        columns: [
                            {
                                text: `${yourStreetAddress} \n ${yourCityStatePostalCode} \n   ${yourCountry}`,
                                style: 'invoiceBillingAddress',
                            },
                            {
                                text: `${clientStreetAddress} \n ${clientCityStatePostalCode} \n   ${clientCountry}`,
                                style: 'invoiceBillingAddress',
                            },
                        ],
                    },
                    '\n\n',
                    {
                        width: '100%',
                        alignment: 'center',
                        text: 'Invoice Details',
                        bold: true,
                        margin: [0, 10, 0, 10],
                        fontSize: 15,
                    },
                    {
                        layout: {
                            defaultBorder: false,
                            hLineWidth: function (i, node) {
                                return 1;
                            },
                            vLineWidth: function (i, node) {
                                return 1;
                            },
                            hLineColor: function (i, node) {
                                if (i === 1 || i === 0) {
                                    return '#bfdde8';
                                }
                                return '#eaeaea';
                            },
                            vLineColor: function (i, node) {
                                return '#eaeaea';
                            },
                            hLineStyle: function (i, node) {
                                // if (i === 0 || i === node.table.body.length) {
                                return null;
                                //}
                            },
                            // vLineStyle: function (i, node) { return {dash: { length: 10, space: 4 }}; },
                            paddingLeft: function (i, node) {
                                return 10;
                            },
                            paddingRight: function (i, node) {
                                return 10;
                            },
                            paddingTop: function (i, node) {
                                return 2;
                            },
                            paddingBottom: function (i, node) {
                                return 2;
                            },
                            fillColor: function (rowIndex, node, columnIndex) {
                                return '#fff';
                            },
                        },
                        table: {
                            headerRows: 1,
                            widths: ['*', 60, 60, 60, 60, 60],
                            body: [
                                [
                                    {
                                        text: 'DESCRIPTION',
                                        fillColor: '#eaf2f5',
                                        border: [false, true, false, true],
                                        margin: [0, 5, 0, 5],
                                        textTransform: 'uppercase',
                                    },
                                    {
                                        text: 'QTY',
                                        fillColor: '#eaf2f5',
                                        border: [false, true, false, true],
                                        margin: [0, 5, 0, 5],
                                        textTransform: 'uppercase',
                                    },
                                    {
                                        text: 'PRICE',
                                        fillColor: '#eaf2f5',
                                        border: [false, true, false, true],
                                        margin: [0, 5, 0, 5],
                                        textTransform: 'uppercase',
                                    },
                                    {
                                        text: 'QST',
                                        fillColor: '#eaf2f5',
                                        border: [false, true, false, true],
                                        margin: [0, 5, 0, 5],
                                        textTransform: 'uppercase',
                                    },
                                    {
                                        text: 'GST',
                                        fillColor: '#eaf2f5',
                                        border: [false, true, false, true],
                                        margin: [0, 5, 0, 5],
                                        textTransform: 'uppercase',
                                    },
                                    {
                                        text: 'TOTAL',
                                        border: [false, true, false, true],
                                        alignment: 'right',
                                        fillColor: '#eaf2f5',
                                        margin: [0, 5, 0, 5],
                                        textTransform: 'uppercase',
                                    },
                                ],
                                ...invoiceItems.map(item => [
                                    {
                                        text: item.description,
                                        border: [false, false, false, true],
                                        margin: [0, 5, 0, 5],
                                        alignment: 'left',
                                    },
                                    {
                                        text: item.quantity,
                                        border: [false, false, false, true],
                                        margin: [0, 5, 0, 5],
                                        alignment: 'left',
                                    },
                                    {
                                        //text: `$${item.price.toFixed(2)}`,
                                        text: `$${item.price}`,
                                        border: [false, false, false, true],
                                        margin: [0, 5, 0, 5],
                                        alignment: 'left',
                                    },
                                    {
                                        text: item.applyQST ? `$${(item.price * 0.09975).toFixed(2)}` : `0.00`,
                                        border: [false, false, false, true],
                                        margin: [0, 5, 0, 5],
                                        alignment: 'left',
                                    },
                                    {
                                        text: item.applyGST ? `$${(item.price * 0.05).toFixed(2)}` : `0.00`,
                                        border: [false, false, false, true],
                                        margin: [0, 5, 0, 5],
                                        alignment: 'left',
                                    },
                                    {
                                        text: `$${(item.quantity * item.price + (item.applyQST ? item.price * 0.09975 : 0) + (item.applyGST ? item.price * 0.05 : 0)).toFixed(2)}`,
                                        border: [false, false, false, true],
                                        fillColor: '#f5f5f5',
                                        alignment: 'right',
                                        margin: [0, 5, 0, 5],
                                    }
                                ])
                            ],
                        },
                    },
                    '\n',
                    '\n\n',
                    {
                        layout: {
                            defaultBorder: false,
                            hLineWidth: function (i, node) {
                                return 1;
                            },
                            vLineWidth: function (i, node) {
                                return 1;
                            },
                            hLineColor: function (i, node) {
                                return '#eaeaea';
                            },
                            vLineColor: function (i, node) {
                                return '#eaeaea';
                            },
                            hLineStyle: function (i, node) {
                                // if (i === 0 || i === node.table.body.length) {
                                return null;
                                //}
                            },
                            // vLineStyle: function (i, node) { return {dash: { length: 10, space: 4 }}; },
                            paddingLeft: function (i, node) {
                                return 10;
                            },
                            paddingRight: function (i, node) {
                                return 10;
                            },
                            paddingTop: function (i, node) {
                                return 3;
                            },
                            paddingBottom: function (i, node) {
                                return 3;
                            },
                            fillColor: function (rowIndex, node, columnIndex) {
                                return '#fff';
                            },
                        },
                        table: {
                            headerRows: 1,
                            widths: ['*', 'auto'],
                            body: [
                                [
                                    {
                                        text: 'Payment Subtotal',
                                        border: [false, true, false, true],
                                        alignment: 'right',
                                        margin: [0, 5, 0, 5],
                                    },
                                    {
                                        border: [false, true, false, true],
                                        text: `$${invoiceItems.reduce(
                                                (acc, item) => acc + item.quantity * item.price,
                                                0
                                            ).toFixed(2)}`,
                                        alignment: 'right',
                                        fillColor: '#f5f5f5',
                                        margin: [0, 5, 0, 5],
                                    },
                                ],
                                [
                                    {
                                        text: 'Total QST',
                                        border: [false, false, false, true],
                                        alignment: 'right',
                                        margin: [0, 5, 0, 5],
                                    },
                                    {
                                        text: `$${invoiceItems
                                                .reduce(
                                                    (acc, item) =>
                                                        acc + item.quantity * item.price * 0.09975,
                                                    0
                                                )
                                                .toFixed(2)}`,
                                        border: [false, false, false, true],
                                        fillColor: '#f5f5f5',
                                        alignment: 'right',
                                        margin: [0, 5, 0, 5],
                                    },
                                ],
                                [
                                    {
                                        text: 'Total GST',
                                        border: [false, false, false, true],
                                        alignment: 'right',
                                        margin: [0, 5, 0, 5],
                                    },
                                    {
                                        text: `$${invoiceItems
                                                .reduce(
                                                    (acc, item) =>
                                                        acc + item.quantity * item.price * 0.05,
                                                    0
                                                )
                                                .toFixed(2)}`,
                                        border: [false, false, false, true],
                                        fillColor: '#f5f5f5',
                                        alignment: 'right',
                                        margin: [0, 5, 0, 5],
                                    },
                                ],
                                [
                                    {
                                        text: 'Total Amount',
                                        bold: true,
                                        fontSize: 20,
                                        alignment: 'right',
                                        border: [false, false, false, true],
                                        margin: [0, 5, 0, 5],
                                    },
                                    {
                                        text: `CAD $${(
                                                    invoiceItems.reduce(
                                                        (acc, item) =>
                                                            acc + item.quantity * item.price,
                                                        0
                                                    ) * 1.14975
                                                ).toFixed(2)}`,
                                        bold: true,
                                        fontSize: 20,
                                        alignment: 'right',
                                        border: [false, false, false, true],
                                        fillColor: '#f5f5f5',
                                        margin: [0, 5, 0, 5],
                                    },
                                ],
                            ],
                        },
                    },
                    '\n\n',
                    {
                    text: 'NOTES',
                    style: 'notesTitle',
                    },
                    {
                    text: `${notesText ? notesText : "None."}`,
                    style: 'notesText',
                    },
                    {
                    text: 'TERMS & CONDITIONS',
                    style: 'notesTitle',
                    },
                    {
                    text: `${termsAndConditionsText}`,
                    style: 'notesText',
                    },
                ],
                styles: {
                    notesTitle: {
                    fontSize: 10,
                    bold: true,
                    margin: [0, 50, 0, 3],
                    },
                    notesText: {
                    fontSize: 10,
                    },
                },
                defaultStyle: {
                    columnGap: 20,
                    //font: 'Quicksand',
                },
            };

            pdfMake.createPdf(dd).download('invoice.pdf');

            toast.success('Your invoice has been generated.');
        } else {
            toast.error('Please fix the errors.');
        }
    };
</script>

<svelte:head>
	<title>Invoice Generator</title>
</svelte:head>

<div class="py-16 min-h-[100vh]">

    <!-- Invoice Generator Page Header -->
    <div class="mx-auto mb-12 max-w-2xl text-center">
		<h1 class="mb-2 text-sm sm:text-base font-semibold text-green-500">Canadian Invoice Generator</h1>
		<p class="mb-6 text-3xl sm:text-5xl font-bold">Create Invoices for Free</p>
		<p class="text-base sm:text-lg">
			Your invoice will be saved to your device as a PDF document.
		</p>
	</div>

    <!-- Form for Custom Invoice Parameters -->
    <form class="grid w-full items-start gap-6 overflow-auto p-4 pt-0 my-2">

        <!-- Document Information and Image Upload -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <!-- Document Information -->
            <fieldset class="grid gap-6 rounded-lg border p-4 shadow-md bg-card/30">
                <legend class="-ml-1 px-1 text-sm font-medium"> Basic Information </legend>

                <!-- Invoice Number -->
                <div class="grid gap-3">
                    <Label for="invoice-number">
                        Invoice No.
                        {#if errors.yourName}
                            <span class="text-xs text-red-500">Invoice number is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={invoiceNumber} id="invoice-number" placeholder="Enter an Invoice Number" aria-placeholder="Enter an Invoice Number or identifier." />
                </div>

                <div class="grid grid-cols-1 xl:grid-cols-2 gap-6">
                    <!-- Date Issued -->
                    <div class="grid gap-3">
                        <Label for="date-issued">
                            Date Issued (YYYY-MM-DD)
                        </Label>
                        <Input required aria-required type="date" bind:value={invoiceDateIssued}/>
                    </div>

                    <!-- Due Date -->
                    <div class="grid gap-3">
                        <Label for="due-date">
                            Due Date (YYYY-MM-DD)
                        </Label>
                        <Input required aria-required type="date" bind:value={invoiceDueDate}/>
                    </div>
                </div>

                <!-- Invoice Status -->
                <div class="grid gap-3">
                    <Label for="invoice-status">
                        Invoice Status
                        {#if errors.invoiceStatus}
                            <span class="text-xs text-red-500">Invocie status is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={invoiceStatus} id="invoice-status" placeholder="e.g. Paid / Pending / Overdue" aria-placeholder="Enter your Invoice Status (Paid, Pending, Overdue)..." />
                </div>
            </fieldset>

            <!-- Company Image Upload -->
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <fieldset class="grid gap-6 rounded-lg border p-4 shadow-md bg-card/30">
                <legend class="-ml-1 px-1 text-sm font-medium"> Your Company&apos;s Logo </legend>

                <div class="flex justify-center items-center flex-col gap-3">
                    {#if avatar}
                    <div class="container mx-auto h-[250px] w-[250px] overflow-hidden m-2">
                        <img class="object-cover w-full h-full" src="{avatar}" alt="Avatar" />
                    </div>
                    {:else}
                    <img class="flex h-[250px] w-[250px] m-2 border-[5px] border-neutral-500" src="https://www.vocaleurope.eu/wp-content/uploads/no-image.jpg" alt="Placeholder" /> 
                    
                    {/if}
                    <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
                    <div on:click={()=>{fileinput.click();}} class="flex justify-center items-center flex-col gap-3 cursor-pointer">
                        <ImageUp class="h-[50px] w-[50px]" />
                        Upload your Company's Logo
                    </div>
                    <input style="display:none" type="file" accept=".jpg, .jpeg, .png" on:change={(e)=>onFileSelected(e)} bind:this={fileinput} >
                    {#if errors.avatar}
                        <p class="text-xs text-red-500">Please upload a logo.</p>
                    {/if}
                </div>
            </fieldset>
        </div>

        <!-- Personal/Client Information -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <!-- Personal Company Information -->
            <fieldset class="grid gap-6 rounded-lg border p-4 shadow-md bg-card/30">
                <legend class="-ml-1 px-1 text-sm font-medium"> Your Company&apos;s Information </legend>

                <!-- Personal Name -->
                <div class="grid gap-3">
                    <Label for="your-name">
                        Your Name
                        {#if errors.yourName}
                            <span class="text-xs text-red-500">Your name is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={yourName} id="your-name" placeholder="Enter Your Full Name" aria-placeholder="Enter Your Full Name" />
                </div>

                <!-- Your Company Name -->
                <div class="grid gap-3">
                    <Label for="your-name">
                        Your Company&apos;s Name
                        {#if errors.yourCompany}
                            <span class="text-xs text-red-500">Company name is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={yourCompany} id="your-name" placeholder="e.g. Acme Inc." aria-placeholder="Enter Your Full Name" />
                </div>

                <!-- Your Address Line -->
                <div class="grid gap-3">
                    <Label for="your-address">
                        Your Address
                        {#if errors.yourStreetAddress}
                            <span class="text-xs text-red-500">&nbsp; Your address is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={yourStreetAddress} id="your-address" placeholder="e.g. 9999 Street name 1A" aria-placeholder="Enter your Address number, street name, and apartment number." />
                </div>

                <!-- Your City, Province, Postal Code Line -->
                <div class="grid gap-3">
                    <Label for="your-city-state-postalcode">
                        Your City, State, Postal Code
                        {#if errors.yourCityStatePostalCode}
                            <span class="text-xs text-red-500">&nbsp; Your city, state, and postal code are required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={yourCityStatePostalCode} id="your-city-state-postalcode" placeholder="e.g. New York City, NY 00000" aria-placeholder="Enter your City, State, and Postal Code." />
                </div>

                <!-- Your Country -->
                <div class="grid gap-3">
                    <Label for="your-country">
                        Your Country
                        {#if errors.yourCountry}
                            <span class="text-xs text-red-500">&nbsp; Your Country is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={yourCountry} id="your-country" placeholder="e.g. United States of America" aria-placeholder="Enter Your Country." />
                </div>
            </fieldset>

            <!-- Client Information -->
            <fieldset class="grid gap-6 rounded-lg border p-4 shadow-md bg-card/30">
                <legend class="-ml-1 px-1 text-sm font-medium mb-2"> Your Client&apos;s Information </legend>

                <!-- Client Name -->
                <div class="grid gap-3">
                    <Label for="client-name">
                        Client Name
                        {#if errors.clientName}
                            <span class="text-xs text-red-500">Client name is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={clientName} id="client-name" placeholder="Enter Client&apos;s Full Name" aria-placeholder="Enter Client&apos;s Full Name" />
                </div>

                <!-- Client Company Name -->
                <div class="grid gap-3">
                    <Label for="client-company">
                        Client Company
                        {#if errors.clientCompany}
                            <span class="text-xs text-red-500">Client company is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={clientCompany} id="client-company" placeholder="e.g. Acme Inc." aria-placeholder="Enter Client&apos;s Company Name" />
                </div>

                <!-- Client Street Address -->
                <div class="grid gap-3">
                    <Label for="client-address">
                        Client Address
                        {#if errors.clientStreetAddress}
                            <span class="text-xs text-red-500">&nbsp; Client address is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={clientStreetAddress} id="client-address" placeholder="e.g. 9999 Street name 1A" aria-placeholder="Enter Client Address number, street name, and apartment number." />
                </div>

                <!-- Client City, Province, Postal Code Line -->
                <div class="grid gap-3">
                    <Label for="client-city-state-postalcode">
                        Client City, State, Postal Code
                        {#if errors.clientCityStatePostalCode}
                            <span class="text-xs text-red-500">&nbsp; Client city, state, and postal code are required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={clientCityStatePostalCode} id="client-city-state-postalcode" placeholder="e.g. New York City, NY 00000" aria-placeholder="Enter Client's City, State, and Postal Code." />
                </div>

                <!-- Client Country -->
                <div class="grid gap-3">
                    <Label for="client-country">
                        Client Country
                        {#if errors.clientCountry}
                            <span class="text-xs text-red-500">&nbsp; Client Country is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={clientCountry} id="client-country" placeholder="e.g. United States of America" aria-placeholder="Enter Client Country." />
                </div>
            </fieldset>
        </div>

        <!-- Item Details -->
        <fieldset class="grid gap-6 rounded-lg border p-4 shadow-md bg-card/30">
            <legend class="-ml-1 px-1 text-sm font-medium"> Item Details </legend>

            <p class="inline-block sm:hidden text-xs text-muted-foreground">Swipe Left to Edit Item Details →</p>

            {#if (Object.values(errors).some(error => error))}
                <p class="text-xs text-red-500">Item details and numerical values are required.</p>
            {/if}

            <!-- Table with rows for Item Description, Quantity, Price, Select QST Option, and Select GST Option -->
            <Table.Root>
                <Table.Header>
                  <Table.Row>
                    <Table.Head class="min-w-[100px]">Description</Table.Head>
                    <Table.Head class="min-w-[100px] sm:w-[60px]">Quantity</Table.Head>
                    <Table.Head class="min-w-[110px] sm:w-[100px]">Price</Table.Head>
                    <Table.Head class="w-[100px]"><span class="hidden sm:inline-block">Apply&#160;</span>GST</Table.Head>
                    <Table.Head class="w-[100px]"><span class="hidden sm:inline-block">Apply&#160;</span>QST</Table.Head>
                  </Table.Row>
                </Table.Header>
                <Table.Body>
                    {#each invoiceItems as item, index}
                    <Table.Row>
                        <Table.Cell class="font-semibold">
                            <Input required aria-required type="text" bind:value={item.description} placeholder="Enter Item Description (max. 150 characters)" aria-placeholder="Enter Item Description" maxlength="150" />
                        </Table.Cell>
                        <Table.Cell>
                            <Label for="stock-{index}" class="sr-only">Stock</Label>
                            <Input required aria-required id="stock-{index}" type="number" bind:value={item.quantity} step="1" min="0" />
                        </Table.Cell>
                        <Table.Cell>
                            <Label for="price-{index}" class="sr-only">Price</Label>
                            <Input required aria-required id="price-{index}" type="number" bind:value={item.price} step="0.01" min="0" />
                        </Table.Cell>
                        <Table.Cell>
                            <div class="flex items-center justify-center">
                                <Switch bind:checked={item.applyGST} />
                            </div>
                        </Table.Cell>
                        <Table.Cell>
                            <div class="flex items-center justify-center">
                                <Switch bind:checked={item.applyQST} />
                            </div>
                        </Table.Cell>
                        <Table.Cell class="w-[50px]">
                            <Button size="sm" variant="ghost" class="gap-1" on:click={() => removeItem(index)}>
                                <CircleMinus class="h-3.5 w-3.5 text-red-500" />
                                <span class="text-red-500">Remove</span>
                            </Button>
                        </Table.Cell>
                    </Table.Row>
                    {/each}
                </Table.Body>
                
            </Table.Root>

            <Button size="sm" variant="ghost" class="gap-1" on:click={addItem}>
                <CirclePlus class="h-3.5 w-3.5" />
                Add an Item
            </Button>
        </fieldset>

        <!-- Additional Details -->
        <fieldset class="grid gap-6 rounded-lg border p-4 shadow-md bg-card/30">
            <legend class="-ml-1 px-1 text-sm font-medium"> Additional Details </legend>

            <!-- Notes Description -->
            <div class="grid w-full gap-1.5">
                <Label for="notes">Notes (Optional)</Label>
                <Textarea placeholder="Type your notes here." id="notes" bind:value={notesText} />
            </div>

            <!-- Terms and Conditions Description -->
            <div class="grid w-full gap-1.5">
                <Label for="terms-and-conditions">Terms & Conditions</Label>
                <Textarea placeholder="Type your terms and conditions here." id="terms-and-conditions" bind:value={termsAndConditionsText} />
                {#if errors.termsAndConditionsText}
                    <p class="text-xs text-red-500">Terms and conditionsa are required.</p>
                {/if}
            </div>
        </fieldset>
    </form>

    <div class="flex mx-auto items-center justify-center">
        <Button on:click={generatePDF}>
            <FileText class="mr-2 h-4 w-4" />
            <strong>Generate Custom Invoice Document</strong>
        </Button>
    </div>

    <Toaster  />

</div>
