<script>
    import { supabase } from '$lib/supabaseClient';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    let email = '';
    let password = '';
    let error = '';

    async function signIn() {
        error = '';
        const { data, error: signInError } = await supabase.auth.signInWithPassword({ email, password });

        if (signInError) {
            error = signInError.message;
        } else {
            goto('/');
        }
    }

    onMount(async () => {
        const { data } = await supabase.auth.getUser();
        if (data.user) {
            goto('/');
        }
    });
</script>

<main>
    <h1>Sign In</h1>
    <form on:submit|preventDefault={signIn}>
        <label>
            Email:
            <input type="email" bind:value={email} required />
        </label>
        <label>
            Password:
            <input type="password" bind:value={password} required />
        </label>
        <button type="submit">Sign In</button>
    </form>
    {#if error}
        <p style="color: red;">{error}</p>
    {/if}
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
