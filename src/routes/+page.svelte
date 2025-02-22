<script lang="ts">
    import { onMount } from "svelte";
    import { fly } from 'svelte/transition';

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
    let completedSessions: StudySession[] = [];
    let pendingSessions: StudySession[] = [];

    // Fetch study sessions from backend
    async function fetchSessions() {
        try {
            const res = await fetch("https://studylog-backend-production.up.railway.app/api/study");
            const data = await res.json();
            
            // Log the fetched data for debugging
            console.log("Fetched sessions:", data);

            studySessions = data.map((session: StudySession) => {
                if (session.completed) {
                    completedSessions.push(session);
                } else {
                    pendingSessions.push(session);
                }
                return session;
            });
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

                if (res.ok) {
                    // Log the response to ensure the session was added
                    const addedSession = await res.json();
                    console.log("Added session:", addedSession);

                    // Fetch updated sessions after adding
                    await fetchSessions(); // Refresh list
                } else {
                    console.error("Error adding session:", res.status);
                }
            } catch (error) {
                console.error("Error adding session:", error);
            }

            // Reset form values
            subject = "";
            duration = 0;
            date = "";
        }
    }

    // Mark session as completed and move it to the completed list
    function completeSession(session: StudySession) {
        session.completed = true;
        completedSessions = [...completedSessions, session];
        pendingSessions = pendingSessions.filter((s) => s.id !== session.id);

        // Update the backend to mark it as completed
        fetch(`https://studylog-backend-production.up.railway.app/api/study/${session.id}`, {
            method: "PATCH",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ completed: true }),
        });
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

    <div class="sessions-container">
        <!-- Pending Sessions (To-Do) -->
        <div class="sessions-list">
            <h2>To-Do</h2>
            <ul>
                {#each pendingSessions as session (session.id)}
                    <li class="card" in:fly={{ x: -200 }} out:fly={{ x: 200 }}>
                        <div class="session-info">
                            <h3>{session.subject}</h3>
                            <p>{session.duration} mins on {session.date}</p>
                        </div>
                        <div>
                            <button class="complete-btn" on:click={() => completeSession(session)}>
                                ✅ Mark as Completed
                            </button>
                            <button class="delete-btn" on:click={() => deleteSession(session.id)}>❌ Delete</button>
                        </div>
                    </li>
                {/each}
            </ul>
        </div>

        <!-- Completed Sessions -->
        <div class="sessions-list">
            <h2>Completed</h2>
            <ul>
                {#each completedSessions as session (session.id)}
                    <li class="card" in:fly={{ x: 200 }} out:fly={{ x: -200 }}>
                        <div class="session-info">
                            <h3>{session.subject}</h3>
                            <p>{session.duration} mins on {session.date}</p>
                        </div>
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

    .sessions-container {
        display: flex;
        justify-content: space-between;
        width: 80%;
        margin-top: 40px;
    }

    .sessions-list {
        width: 45%;
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
        transition: transform 0.3s ease;
    }

    .complete-btn {
        background-color: #4caf50;
        margin-left: 10px;
        cursor: pointer;
    }

    .delete-btn {
        background-color: #ff4d4d;
        margin-left: 10px;
        cursor: pointer;
    }

    .complete-btn:hover {
        background-color: #388e3c;
    }

    .delete-btn:hover {
        background-color: #cc0000;
    }

    ul {
        margin-top: 20px;
        list-style: none;
        padding: 0;
    }

    .session-info h3 {
        margin: 0;
        color: #333;
    }
</style>
