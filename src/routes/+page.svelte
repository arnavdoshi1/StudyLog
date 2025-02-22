<script lang="ts">
    import { onMount } from "svelte";

    let subject: string = "";
    let duration: number = 0;
    let date: string = "";

    type StudySession = {
        id: number;
        subject: string;
        duration: number;
        date: string;
        completed: boolean;
    };

    let studySessions: StudySession[] = [];

    // Fetch study sessions from backend
    async function fetchSessions() {
        try {
            const res = await fetch("https://studylog-backend-production.up.railway.app/api/study");
            studySessions = await res.json();
            console.log("Fetched sessions:", studySessions);
        } catch (error) {
            console.error("Error fetching sessions:", error);
        }
    }

    // Add a study session
    async function addSession() {
        if (subject && duration && date) {
            const newSession = { subject, duration, date, completed: false };

            try {
                const res = await fetch("https://studylog-backend-production.up.railway.app/api/study", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify(newSession),
                });

                const data = await res.json();
                console.log("Added session:", data);

                // Manually update studySessions with the new session immediately
                if (res.ok) {
                    studySessions = [data, ...studySessions]; // Add the new session to the start of the list
                }
            } catch (error) {
                console.error("Error adding session:", error);
            }

            // Clear form inputs
            subject = "";
            duration = 0; // Reset duration to 0
            date = "";
        }
    }

    // Complete a study session
    async function completeSession(id: number) {
        try {
            const sessionToComplete = studySessions.find(session => session.id === id);
            if (sessionToComplete) {
                sessionToComplete.completed = true;
                const res = await fetch(`https://studylog-backend-production.up.railway.app/api/study/${id}`, {
                    method: "PUT",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ completed: true }),
                });

                if (res.ok) {
                    await fetchSessions(); // Refresh list
                }
            }
        } catch (error) {
            console.error("Error completing session:", error);
        }
    }

    // Delete a study session
    async function deleteSession(id: number) {
        try {
            const res = await fetch(`https://studylog-backend-production.up.railway.app/api/study/${id}`, {
                method: "DELETE",
            });

            if (res.ok) {
                studySessions = studySessions.filter(session => session.id !== id); // Remove the session from the list
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

    <div class="study-sessions">
        <div class="sessions-left">
            <h2>To Do</h2>
            <ul>
                {#each studySessions.filter(session => !session.completed) as session (session.id)}
                    <li>
                        {session.subject} - {session.duration} mins on {session.date}
                        <button class="complete-btn" on:click={() => completeSession(session.id)}>✅ Complete</button>
                        <button class="delete-btn" on:click={() => deleteSession(session.id)}>❌ Delete</button>
                    </li>
                {/each}
            </ul>
        </div>
        <div class="sessions-right">
            <h2>Completed</h2>
            <ul>
                {#each studySessions.filter(session => session.completed) as session (session.id)}
                    <li>
                        {session.subject} - {session.duration} mins on {session.date}
                        <button class="delete-btn" on:click={() => deleteSession(session.id)}>❌ Delete</button>
                    </li>
                {/each}
            </ul>
        </div>
    </div>
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
    }

    h1 {
        font-size: 2rem;
        color: #0070f3;
    }

    .input-container {
        display: flex;
        flex-direction: column;
        gap: 10px;
        margin-top: 20px;
        width: 300px;
    }

    input {
        padding: 10px;
        font-size: 1rem;
        border: 1px solid #ccc;
        border-radius: 5px;
        width: 100%;
    }

    button {
        padding: 10px;
        font-size: 1rem;
        background-color: #0070f3;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        transition: background 0.3s;
    }

    button:hover {
        background-color: #005bb5;
    }

    .delete-btn {
        background-color: #ff4d4d;
        margin-left: 10px;
    }

    .delete-btn:hover {
        background-color: #cc0000;
    }

    .complete-btn {
        background-color: #4caf50;
        margin-left: 10px;
    }

    .complete-btn:hover {
        background-color: #2e7d32;
    }

    .study-sessions {
        display: flex;
        justify-content: space-between;
        margin-top: 20px;
        width: 80%;
    }

    .sessions-left, .sessions-right {
        width: 45%;
    }

    ul {
        margin-top: 20px;
        list-style: none;
        padding: 0;
    }

    li {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 10px;
        background: #f9f9f9;
        border-radius: 5px;
        margin-top: 10px;
        width: 100%;
    }
</style>
