<script>
    import { goto } from '$app/navigation';
    import { supabase } from '$lib/supabaseClient';

    let nurseryTvs = '';
    let forenoonAfternoon = '';
    let pickupDrop = '';
    let routeNumber = '';
    let busNumber = '';

    async function continueToAttendance() {
        if (nurseryTvs && forenoonAfternoon && pickupDrop && routeNumber && busNumber) {
            try {
                // Validate the route and bus number against your database
                const { data, error } = await supabase
                    .from('bus-exist')
                    .select('id')
                    .eq('route', routeNumber)
                    .eq('bus_no', busNumber)
                    .limit(1);

                if (error) throw error;

                if (!data || data.length === 0) {
                    alert('Invalid route or bus number.');
                    return;
                }

                goto(`/bus-attendance?route=${routeNumber}&busNo=${busNumber}&pickupDrop=${pickupDrop}&shift=${forenoonAfternoon}`);
            } catch (error) {
                console.error('Error:', error);
                alert('An error occurred. Please try again.');
            }
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

    h1 {
        color: #333;
        margin-bottom: 2rem;
    }

    form {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        width: 300px;
        background-color: white;
        padding: 2rem;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    label {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;
        font-weight: bold;
        color: #555;
    }

    select, input {
        padding: 0.5rem;
        border: 1px solid #ccc;
        border-radius: 4px;
        font-size: 1rem;
    }

    button {
        padding: 0.75rem;
        background-color: #0070f3;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 1rem;
        font-weight: bold;
        transition: background-color 0.3s ease;
    }

    button:hover {
        background-color: #005bb5;
    }
</style>
