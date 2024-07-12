<script>
    import { goto } from '$app/navigation';

    let nurseryTvs = '';
    let forenoonAfternoon = '';
    let pickupDrop = '';
    let routeNumber = '';
    let busNumber = '';

    function continueToAttendance() {
        if (nurseryTvs && forenoonAfternoon && pickupDrop && routeNumber && busNumber) {
            goto('/bus-attendance');
        } else {
            alert('Please fill in all fields.');
        }
    }
</script>

<main>
    <h1>Welcome to Bus Attendance App</h1>
    <form on:submit|preventDefault={continueToAttendance}>
        <label>
            Nursery/TVS:
            <select bind:value={nurseryTvs} required>
                <option value="" disabled selected>Select an option</option>
                <option value="NURSERY">NURSERY</option>
                <option value="TVS">TVS</option>
            </select>
        </label>
        <label>
            Forenoon/Afternoon:
            <select bind:value={forenoonAfternoon} required>
                <option value="" disabled selected>Select an option</option>
                <option value="FORENOON">FORENOON</option>
                <option value="AFTERNOON">AFTERNOON</option>
            </select>
        </label>
        <label>
            Pickup/Drop:
            <select bind:value={pickupDrop} required>
                <option value="" disabled selected>Select an option</option>
                <option value="PICKUP">PICKUP</option>
                <option value="DROP">DROP</option>
            </select>
        </label>
        <label>
            Route Number:
            <input type="text" bind:value={routeNumber} required />
        </label>
        <label>
            Bus Number:
            <input type="text" bind:value={busNumber} required />
        </label>
        <button type="submit">Continue</button>
    </form>
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

    form {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        width: 300px;
    }

    label {
        display: flex;
        flex-direction: column;
    }

    button {
        padding: 0.5rem;
        background-color: #0070f3;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }

    button:hover {
        background-color: #005bb5;
    }
</style>
