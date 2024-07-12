<script>
    import { onMount, onDestroy } from 'svelte';
    import Quagga from 'quagga';

    let scannedValue = '';

    function startScanner() {
        Quagga.init({
            inputStream: {
                type: 'LiveStream',
                target: document.querySelector('#scanner'),
                constraints: {
                    width: 640,
                    height: 480,
                    facingMode: 'environment'
                }
            },
            decoder: {
                readers: ['code_128_reader']
            }
        }, /** @param {Error|null} err */ (err) => {
            if (err) {
                console.error(err);
                return;
            }
            Quagga.start();
        });

        Quagga.onDetected(/** @param {{ codeResult: { code: string } }} data */ (data) => {
            scannedValue = data.codeResult.code;
            Quagga.stop();
        });
    }

    onMount(() => {
        if (typeof window !== 'undefined') {
            startScanner();
        }
    });

    onDestroy(() => {
        // Quagga.stop(); // Temporarily comment out this line
    });
</script>

<main>
    <h1>Bus Attendance</h1>
    <div id="scanner" style="width: 100%; height: auto;"></div>
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
    }

    #scanner {
        width: 100%;
        max-width: 640px;
        height: 480px;
        border: 1px solid #ccc;
        margin-bottom: 1rem;
    }

    label {
        display: flex;
        flex-direction: column;
        margin-top: 1rem;
    }

    input {
        padding: 0.5rem;
        width: 100%;
        max-width: 300px;
        border: 1px solid #ccc;
        border-radius: 4px;
    }
</style>
