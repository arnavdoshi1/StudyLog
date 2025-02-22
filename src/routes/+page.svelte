<script lang="ts">
    import { onMount } from "svelte";

    let subject: string = "";
    let duration: number = 0; // Set the initial value to 0
    let date: string = "";

    type StudySession = {
        id: number;
        subject: string;
        duration: number;
        date: string;
    };

    let studySessions: StudySession[] = [];
    let totalSessions: number = 0;
    let completedSessions: number = 0;

    // Fetch study sessions from backend
    async function fetchSessions() {
        try {
            const res = await fetch("https://studylog-backend-production.up.railway.app/api/study");
            studySessions = await res.json();
            completedSessions = studySessions.filter(session => session.duration > 0).length;
            totalSessions = studySessions.length;
        } catch (error) {
            console.error("Error fetching sessions:", error);
        }
    }

    // Add a study session
    async function addSession() {
        if (subject && duration && date) {
            const newSession = { subject, duration, date };

            try {
                const res = await fetch("https://studylog-backend-production.up.railway.app/api/study", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify(newSession),
                });

                if (res.ok) {
                    await fetchSessions(); // Refresh list
                }
            } catch (error) {
                console.error("Error adding session:", error);
            }

            subject = "";
            duration = 0; // Reset duration to 0
            date = "";
        }
    }

    // Delete a study session
    async function deleteSession(id: number) {
        try {
            const res = await fetch(`https://studylog-backend-production.up.railway.app/api/study/${id}`, {
                method: "DELETE",
            });

            if (res.ok) {
                await fetchSessions(); // Refresh list
            }
        } catch (error) {
            console.error("Error deleting session:", error);
        }
    }

    // Load study sessions on page load
    onMount(fetchSessions);
</script>

<main>
    <h1>StudyLog</h1>
    <p>Track your study sessions easily.</p>

    <div class="input-container">
        <input type="text" bind:value={subject} placeholder="Enter subject" />
        <input type="number" bind:value={duration} placeholder="Duration (minutes)" />
        <input type="date" bind:value={date} />
        <button on:click={addSession}>➕ Add Study Session</button>
    </div>

    <div class="progress">
        <h2>Progress</h2>
        <p>Completed Sessions: {completedSessions} / {totalSessions}</p>
        <div class="progress-bar">
            <div class="filled" style="width: {totalSessions ? (completedSessions / totalSessions) * 100 : 0}%"></div>
        </div>
    </div>

    <ul>
        {#each studySessions as session (session.id)}
            <li class="card">
                <div class="session-info">
                    <h3>{session.subject}</h3>
                    <p>{session.duration} mins on {session.date}</p>
                </div>
                <button class="delete-btn" on:click={() => deleteSession(session.id)}>❌ Delete</button>
            </li>
        {/each}
    </ul>
</main>

<style>
    main {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100vh;
        font-family: sans-serif;
        text-align: center;
        background-color: #f4f4f4;
    }

    h1 {
        font-size: 2rem;
        color: #0070f3;
        margin-bottom: 20px;
    }

    .input-container {
        display: flex;
        flex-direction: column;
        gap: 10px;
        margin-top: 20px;
        width: 300px;
    }

    input, button {
        padding: 10px;
        font-size: 1rem;
        border-radius: 5px;
        width: 100%;
    }

    button {
        background-color: #0070f3;
        color: white;
        border: none;
        cursor: pointer;
        transition: background 0.3s;
    }

    button:hover {
        background-color: #005bb5;
    }

    .progress {
        margin-top: 20px;
        width: 300px;
        text-align: left;
    }

    .progress-bar {
        width: 100%;
        height: 10px;
        background-color: #ccc;
        border-radius: 5px;
        margin-top: 10px;
    }

    .filled {
        height: 100%;
        background-color: #4caf50;
        border-radius: 5px;
    }

    ul {
        margin-top: 20px;
        list-style: none;
        padding: 0;
        width: 300px;
    }

    .card {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 10px;
        background: #fff;
        border-radius: 5px;
        margin-top: 10px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    .delete-btn {
        background-color: #ff4d4d;
        margin-left: 10px;
        cursor: pointer;
    }

    .delete-btn:hover {
        background-color: #cc0000;
    }
</style>
