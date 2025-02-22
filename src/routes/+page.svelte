<script lang="ts">
    let subject: string = "";
    let duration: string = "";
    let date: string = "";

    type StudySession = {
        id: number;
        subject: string;
        duration: string;
        date: string;
    };

    let studySessions: StudySession[] = [];

    // Ensure `localStorage` is only accessed in the browser
    if (typeof window !== "undefined") {
        studySessions = JSON.parse(localStorage.getItem("studySessions") || "[]");
    }

    function addSession() {
        if (subject && duration && date) {
            const newSession: StudySession = {
                id: Date.now(), // Unique ID for each session
                subject,
                duration,
                date
            };

            studySessions = [...studySessions, newSession];

            // Save to local storage (only in browser)
            if (typeof window !== "undefined") {
                localStorage.setItem("studySessions", JSON.stringify(studySessions));
            }

            subject = "";
            duration = "";
            date = "";
        }
    }

    function deleteSession(id: number) {
        studySessions = [...studySessions.filter(session => session.id !== id)];

        // Force reactivity by updating localStorage and reloading from it
        if (typeof window !== "undefined") {
            localStorage.setItem("studySessions", JSON.stringify(studySessions));
            studySessions = JSON.parse(localStorage.getItem("studySessions") || "[]");
        }
    }
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

    <ul>
        {#each studySessions as session (session.id)}
            <li>
                {session.subject} - {session.duration} mins on {session.date}
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
        width: 300px;
    }
</style>
