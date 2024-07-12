<script>
    import { onMount, onDestroy } from 'svelte';
    import Quagga from 'quagga';

    let scannedValue = '';
    let isQuaggaInitialized = false;

    /**
     * Starts the barcode scanner.
     */
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
         * @param {Error | null} err 
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
                if (isQuaggaInitialized) {
                    Quagga.stop();
                    isQuaggaInitialized = false;
                }
            });
    }

    onMount(() => {
        if (typeof window !== 'undefined') {
            startScanner();
        }
    });

    onDestroy(() => {
        if (isQuaggaInitialized) {
            Quagga.stop();
        }
    });
</script>

<main>
    <h1>Bus Attendance</h1>
    <div id="scanner-container">
        <div id="scanner"></div>
    </div>
    <label>
        Scanned Value:
        <input type="text" bind:value={scannedValue} readonly />
    </label>
</main>

<style>
    main {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100vh;
        font-family: Arial, sans-serif;
        background-color: #f0f0f0;
        padding: 1rem;
    }

    #scanner-container {
        width: 100%;
        max-width: 480px;
        height: auto;
        aspect-ratio: 4 / 3; /* Maintain 4:3 aspect ratio */
        position: relative;
        border: 1px solid #ccc;
        margin-bottom: 1rem;
        overflow: hidden; /* Ensures the content does not overflow */
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
    }

    input {
        padding: 0.5rem;
        width: 100%;
        border: 1px solid #ccc;
        border-radius: 4px;
    }

    @media (max-width: 640px) {
        #scanner-container {
            max-width: 100%;
            aspect-ratio: 16 / 9; /* Maintain 16:9 aspect ratio */
        }
    }
</style>
