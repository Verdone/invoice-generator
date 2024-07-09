<script lang="ts">
    //@ts-nocheck
    import { onMount } from 'svelte';
    import { Input } from "$lib/components/ui/input";
    import { Label } from "$lib/components/ui/label/index.js";
    import toast, { Toaster } from 'svelte-french-toast';
    import * as ToggleGroup from "$lib/components/ui/toggle-group/index.js";
    import * as Table from "$lib/components/ui/table/index.js";
    import CirclePlus from "lucide-svelte/icons/circle-plus";
    import { Button } from "$lib/components/ui/button/index.js";
    import { Switch } from "$lib/components/ui/switch";
	import { FileText } from 'lucide-svelte';
    import CalendarIcon from "svelte-radix/Calendar.svelte";
    import {
        DateFormatter,
        type DateValue,
        getLocalTimeZone,
        today
    } from "@internationalized/date";
    import { cn } from "$lib/utils.js";
    import { Calendar } from "$lib/components/ui/calendar/index.js";
    import * as Popover from "$lib/components/ui/popover/index.js";
    import * as Select from "$lib/components/ui/select/index.js";
    
    const df = new DateFormatter("en-US", {
        dateStyle: "long"
    });

    const items = [
        { value: 0, label: "Today" },
        { value: 1, label: "Tomorrow" },
        { value: 3, label: "In 3 days" },
        { value: 7, label: "In a week" }
    ];

    // Invoice Information instance variables
    let invoiceNumber = '';
    let invoiceDateIssued: DateValue | undefined = undefined;
    let invoiceDueDate: DateValue | undefined = undefined;
    let invoiceStatus: '';

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
    
    // errors JSON variable for really, really rudimentary validation
    let errors = {
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
    };

    const addItem = () => {
        invoiceItems = [...invoiceItems, { description: '', quantity: 1, price: 0, applyQST: true, applyGST: true }];
    };

    // const removeItem = (index) => {
    //     invoiceItems.splice(index, 1);
    // };

    const validate = () => {
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

        // Validate inputs for notes and terms/conditions

    };

    // State variable to hold the selected value
    let selectedValue = "true";

    const handleValueChange = (value) => {
        selectedValue = value;
        console.log(selectedValue);
    };

    const generatePDF = async () => {
        // Call the validate() method to ensure inputs are acceptable
        validate();
        if (!errors.clientName && !errors.clientStreetAddress) {
            const pdfMake = (await import('pdfmake/build/pdfmake')).default;
            const pdfFonts = (await import('pdfmake/build/vfs_fonts')).default;
            pdfMake.vfs = pdfFonts.pdfMake.vfs;

            const dd = {
                content: [
                    {
                        columns: [
                            {
                                    image: 'data:image/png;base64,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',
                                    width: 150,
                            },
                            [
                                {
                                    text: 'Invoice',
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
                                                    text: '00001',
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
                                                    text: 'June 01, 2016',
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
                                                    text: 'June 01, 2016',
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
                                                    text: 'PAID', // var status
                                                    bold: true,
                                                    fontSize: 14,
                                                    alignment: 'right',
                                                    color: 'green', // var statusColor
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
                                        text: 'Price',
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
                                        text: 'Total',
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
                                        text: `$${item.price.toFixed(2)}`,
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
                    text: 'Some notes goes here \n Notes second line',
                    style: 'notesText',
                    },
                        {
                    text: 'Terms & Conditions ',
                    style: 'notesTitle',
                    },
                    {
                    text: 'Some notes goes here \n Notes second line',
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
		<h1 class="mb-2 text-sm sm:text-base font-semibold text-green-500">Invoice Generator</h1>
		<p class="mb-6 text-3xl sm:text-5xl font-bold">Create Invoices for Free</p>
		<p class="text-base sm:text-lg">
			Your invoice will be saved to your device as a PDF document.
		</p>
	</div>

    <!-- Form for Custom Invoice Parameters -->
    <form class="grid w-full items-start gap-6 overflow-auto p-4 pt-0">

        <!-- Document Information and Image Upload -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <!-- Document Information -->
            <fieldset class="grid gap-6 rounded-lg border p-4 shadow-md bg-card/30">
                <legend class="-ml-1 px-1 text-sm font-medium"> Basic Information </legend>

                <!-- Invoice Number -->
                <div class="grid gap-3">
                    <Label for="your-name">
                        Invoice No.
                        {#if errors.yourName}
                            <span class="text-xs text-red-500">Your name is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={yourName} id="your-name" placeholder="Enter Your Full Name" aria-placeholder="Enter Your Full Name" />
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                    <!-- Date Issued -->
                    <div class="grid gap-3">
                        <Label for="date-issued">
                            Date Issued
                        </Label>
                        
                        <Popover.Root openFocus>
                            <Popover.Trigger asChild let:builder>
                            <Button
                                variant="outline"
                                class={cn(
                                "w-[240px] justify-start text-left font-normal",
                                !invoiceDateIssued && "text-muted-foreground"
                                )}
                                builders={[builder]}
                            >
                                <CalendarIcon class="mr-2 h-4 w-4" />
                                {invoiceDateIssued ? df.format(invoiceDateIssued.toDate(getLocalTimeZone())) : "Pick a date"}
                            </Button>
                            </Popover.Trigger>
                            <Popover.Content class="flex w-auto flex-col space-y-2 p-2">
                            <Select.Root
                                {items}
                                onSelectedChange={(v) => {
                                if (!v) return;
                                invoiceDateIssued = today(getLocalTimeZone()).add({ days: v.value });
                                }}
                            >
                                <Select.Trigger>
                                <Select.Value placeholder="Select" />
                                </Select.Trigger>
                                <Select.Content>
                                {#each items as item}
                                    <Select.Item value={item.value}>{item.label}</Select.Item>
                                {/each}
                                </Select.Content>
                            </Select.Root>
                            <div class="rounded-md border">
                                <Calendar bind:invoiceDateIssued />
                            </div>
                            </Popover.Content>
                        </Popover.Root>
                    </div>

                    <!-- Due Date -->
                    <div class="grid gap-3">
                        <Label for="due-date">
                            Due Date
                        </Label>
                        
                        <Popover.Root openFocus>
                            <Popover.Trigger asChild let:builder>
                            <Button
                                variant="outline"
                                class={cn(
                                "w-[240px] justify-start text-left font-normal",
                                !invoiceDueDate && "text-muted-foreground"
                                )}
                                builders={[builder]}
                            >
                                <CalendarIcon class="mr-2 h-4 w-4" />
                                {invoiceDueDate ? df.format(invoiceDueDate.toDate(getLocalTimeZone())) : "Pick a date"}
                            </Button>
                            </Popover.Trigger>
                            <Popover.Content class="flex w-auto flex-col space-y-2 p-2">
                            <Select.Root
                                {items}
                                onSelectedChange={(v) => {
                                if (!v) return;
                                invoiceDueDate = today(getLocalTimeZone()).add({ days: v.value });
                                }}
                            >
                                <Select.Trigger>
                                <Select.Value placeholder="Select" />
                                </Select.Trigger>
                                <Select.Content>
                                {#each items as item}
                                    <Select.Item value={item.value}>{item.label}</Select.Item>
                                {/each}
                                </Select.Content>
                            </Select.Root>
                            <div class="rounded-md border">
                                <Calendar bind:invoiceDueDate />
                            </div>
                            </Popover.Content>
                        </Popover.Root>
                    </div>
                </div>

                <!-- Invoice Status -->
                <div class="grid gap-3">
                    <Label for="your-city-state-postalcode">
                        Invoice Status (Optional)
                        {#if errors.yourCityStatePostalCode}
                            <span class="text-xs text-red-500">&nbsp; Your city, state, and postal code are required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={yourCityStatePostalCode} id="your-city-state-postalcode" placeholder="e.g. Paid / Pending / Overdue" aria-placeholder="Enter your Invoice Status (Paid, Pending, Overdue)..." />
                </div>
            </fieldset>

            <!-- Company Image Upload -->
            <fieldset class="grid gap-6 rounded-lg border p-4 shadow-md bg-card/30">
                <legend class="-ml-1 px-1 text-sm font-medium"> Your Company&apos;s Logo </legend>

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
                        {#if errors.yourName}
                            <span class="text-xs text-red-500">Your name is required.</span>
                        {/if}
                    </Label>
                    <Input required aria-required type="text" bind:value={yourName} id="your-name" placeholder="e.g. Acme Inc." aria-placeholder="Enter Your Full Name" />
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
                        <Label for="stock-1" class="sr-only">Stock</Label>
                        <Input required aria-required id="stock-1" type="number" value="1" step="1" />
                        </Table.Cell>
                        <Table.Cell>
                        <Label for="price-1" class="sr-only">Price</Label>
                        <Input required aria-required id="price-1" type="number" value="0.00" step="0.01" />
                        </Table.Cell>
                        <Table.Cell>
                            <div class="flex items-center justify-center">
                                <Switch
                                    bind:checked={item.applyGST}
                                />
                            </div>
                        </Table.Cell>
                        <Table.Cell>
                            <div class="flex items-center justify-center">
                                <Switch
                                    bind:checked={item.applyQST}
                                 />
                            </div>
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

            <!-- Terms and Conditions Description -->

        </fieldset>
        
    </form>

    <Button on:click={generatePDF}>
        <FileText class="mr-2 h-4 w-4" />
        Generate Document
    </Button>

    <Toaster  />

</div>
