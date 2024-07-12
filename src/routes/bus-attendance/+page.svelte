<script>
    import { onMount, onDestroy } from 'svelte';
    import Quagga from 'quagga';
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';
    import { supabase } from '$lib/supabaseClient';

    /** @type {string} */
    let scannedValue = '';
    /** @type {boolean} */
    let isQuaggaInitialized = false;
    /** @type {string} */
    let route = '';
    /** @type {string} */
    let busNo = '';
    /** @type {string} */
    let pickupDrop = '';
    /** @type {string} */
    let shift = '';
    /** @type {string} */
    let successMessage = '';

    /**
     * @typedef {Object} Student
     * @property {string} adm_no
     * @property {string} name
     * @property {string} class_sec
     * @property {string} route
     * @property {string} bus_no
     * @property {string} shift
     * @property {string} pickup_drop
     */

    /** @type {Student[]} */
    let busNew = [];

    onMount(() => {
        if (typeof window !== 'undefined') {
            startScanner();
            const searchParams = new URLSearchParams($page.url.searchParams);
            route = searchParams.get('route') || '';
            busNo = searchParams.get('busNo') || '';
            pickupDrop = searchParams.get('pickupDrop') || '';
            shift = searchParams.get('shift') || '';
            fetchScannedStudents();
        }
    });

    onDestroy(() => {
        if (isQuaggaInitialized) {
            Quagga.stop();
        }
    });

    function startScanner() {
        Quagga.init({
            inputStream: {
                type: 'LiveStream',
                target: document.querySelector('#scanner'),
                constraints: {
                    facingMode: 'environment'
                }
            },
            decoder: {
                readers: ['code_128_reader']
            }
        }, 
        /**
         * @param {Error|null} err
         */
        (err) => {
            if (err) {
                console.error(err);
                return;
            }
            Quagga.start();
            isQuaggaInitialized = true;
        });

        Quagga.onDetected(
            /**
             * @param {{ codeResult: { code: string } }} data
             */
            (data) => {
            scannedValue = data.codeResult.code;
            if (scannedValue.length === 8 && /^\d+$/.test(scannedValue)) {
                if (isQuaggaInitialized) {
                    Quagga.stop();
                    isQuaggaInitialized = false;
                }
                processScannedValue();
            }
        });
    }

    async function processScannedValue() {
        if (scannedValue.length === 8 && /^\d+$/.test(scannedValue)) {
            try {
                const { data: existingEntry, error: fetchError } = await supabase
                    .from('bus-exist')
                    .select('*')
                    .eq('adm_no', scannedValue)
                    .eq('route', route)
                    .eq('bus_no', busNo)
                    .eq('shift', shift)
                    .maybeSingle();

                if (fetchError) throw fetchError;

                if (!existingEntry) {
                    successMessage = 'No matching student found or route/bus number/shift mismatch.';
                    return;
                }

                const { adm_no, name, class_sec } = existingEntry;
                const { error: insertError } = await supabase
                    .from('bus-new')
                    .insert({
                        adm_no,
                        name,
                        class_sec,
                        route,
                        bus_no: busNo,
                        shift,
                        pickup_drop: pickupDrop
                    });

                if (insertError) throw insertError;

                successMessage = `Student ${name} (${adm_no}) successfully added to bus attendance.`;
                await fetchScannedStudents();
            } catch (error) {
                console.error('Error:', error);
                successMessage = 'An error occurred. Please try again.';
            }
        } else {
            successMessage = 'Please enter a valid 8-digit admission number.';
        }
        scannedValue = '';
        startScanner();
    }

    async function fetchScannedStudents() {
        try {
            const { data, error } = await supabase
                .from('bus-new')
                .select('*')
                .eq('route', route)
                .eq('bus_no', busNo)
                .eq('pickup_drop', pickupDrop)
                .eq('shift', shift);

            if (error) throw error;

            busNew = data || [];
        } catch (error) {
            console.error('Error fetching scanned students:', error);
            successMessage = 'Failed to fetch scanned students. Please try again.';
        }
    }

    function handleManualInput() {
        if (scannedValue.length === 8 && /^\d+$/.test(scannedValue)) {
            processScannedValue();
        }
    }

    function changeBus() {
        goto('/');
    }
</script>

<main>
    <h1>Bus Attendance</h1>
    <div id="scanner-container">
        <div id="scanner"></div>
    </div>
    <label>
        Scanned Value:
        <input 
            type="text" 
            bind:value={scannedValue} 
            on:input={handleManualInput}
            maxlength="8"
            pattern="\d*"
        />
    </label>
    {#if successMessage}
        <p class="success-message">{successMessage}</p>
    {/if}
    <button on:click={changeBus}>Change Bus</button>
    <div class="info">
        <p>Route: {route}</p>
        <p>Bus Number: {busNo}</p>
        <p>Pickup/Drop: {pickupDrop}</p>
        <p>Shift: {shift}</p>
    </div>

    <h2>Scanned Students</h2>
    <ul>
        {#each busNew as student}
            <li>{student.name} - {student.adm_no}</li>
        {/each}
    </ul>
</main>

<style>
    main {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: flex-start;
        min-height: 100vh;
        font-family: Arial, sans-serif;
        background-color: #f0f0f0;
        padding: 1rem;
    }

    h1, h2 {
        color: #333;
        margin-bottom: 1rem;
    }

    #scanner-container {
        width: 100%;
        max-width: 480px;
        height: auto;
        aspect-ratio: 4 / 3;
        position: relative;
        border: 1px solid #ccc;
        margin-bottom: 1rem;
        overflow: hidden;
        border-radius: 8px;
    }

    #scanner {
        width: 100%;
        height: 100%;
        position: absolute;
    }

    label {
        display: flex;
        flex-direction: column;
        margin-top: 1rem;
        width: 100%;
        max-width: 300px;
        gap: 0.5rem;
        font-weight: bold;
        color: #555;
    }

    input {
        padding: 0.5rem;
        width: 100%;
        border: 1px solid #ccc;
        border-radius: 4px;
        font-size: 1rem;
    }

    .success-message {
        margin-top: 1rem;
        padding: 0.5rem;
        background-color: #e6ffe6;
        border: 1px solid #b3ffb3;
        border-radius: 4px;
        color: #006600;
    }

    button {
        margin-top: 1rem;
        padding: 0.5rem 1rem;
        background-color: #0070f3;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 1rem;
    }

    button:hover {
        background-color: #0056b3;
    }

    .info {
        background-color: white;
        padding: 1rem;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        margin-top: 1rem;
        width: 100%;
        max-width: 300px;
    }

    .info p {
        margin: 0.5rem 0;
        color: #555;
    }

    ul {
        list-style-type: none;
        padding: 0;
        width: 100%;
        max-width: 300px;
        background-color: white;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        margin-top: 1rem;
    }

    li {
        padding: 0.75rem 1rem;
        border-bottom: 1px solid #eee;
    }

    li:last-child {
        border-bottom: none;
    }

    @media (max-width: 640px) {
        #scanner-container {
            max-width: 100%;
            aspect-ratio: 16 / 9;
        }
    }
</style>
