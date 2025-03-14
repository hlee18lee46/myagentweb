<script>
    import { onMount, afterUpdate } from "svelte";
    import { writable } from "svelte/store";

    let message = "";
    let messages = writable([]);
    let messagesEnd;

async function sendMessage() {
    if (message.trim() === "") return;

    // Add user's message to chat
    messages.update(m => [...m, { sender: "You", text: message }]);

    try {
        const response = await fetch("https://myagent-backend.onrender.com/api/chat", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ message: message.trim() }) // Ensure proper key
        });

        const data = await response.json();

        let replyText = "";

        if (data.results?.length) {
            // Use a Set to filter unique projects based on GitHub URL
            const uniqueProjects = new Map();

            data.results.forEach((project) => {
                if (!uniqueProjects.has(project.html_url)) {
                    uniqueProjects.set(project.html_url, project);
                }
            });

            // Format project data without duplicates
            replyText = Array.from(uniqueProjects.values())
                .map(r => 
                    `<div>
                        <h3>📌 ${r.name}</h3>
                        <p><strong>🔹 Description:</strong> ${r.description || "No description available"}</p>
                        
                        <p><strong>🛠 Tech Stack:</strong></p>
                        <ul>
                            ${r.tech_stack 
                                ? Object.entries(r.tech_stack)
                                    .map(([key, value]) => `<li><strong>${key}:</strong> ${value}</li>`)
                                    .join("")
                                : "<li>Not specified</li>"}
                        </ul>

                        <p><strong>✨ Features:</strong></p>
                        <ul>
                            ${r.features?.length 
                                ? r.features.map(feature => `<li>${feature}</li>`).join("")
                                : "<li>No features listed.</li>"}
                        </ul>

                        <p>
                            🔗 <strong>GitHub:</strong> <a href="${r.html_url}" target="_blank" rel="noopener noreferrer">${r.html_url}</a><br>
                            🌍 <strong>Demo:</strong> ${r.demo_link !== "No link available" 
                                ? `<a href="${r.demo_link}" target="_blank" rel="noopener noreferrer">${r.demo_link}</a>` 
                                : "No demo available"}<br>
                            🏆 <strong>Devpost:</strong> ${r.devpost_link !== "No link available" 
                                ? `<a href="${r.devpost_link}" target="_blank" rel="noopener noreferrer">${r.devpost_link}</a>` 
                                : "No Devpost available"}
                        </p>
                    </div>`
                )
                .join("<hr>");  // Add a horizontal line between projects for better readability
        } else if (data.reply) {
            // If no results but has a reply (e.g., greetings or fallback)
            replyText = `<p>${data.reply.replace(/\n/g, "<br>")}</p>`; // Ensure new lines are respected
        } else {
            replyText = "<p>No results found.</p>";
        }

        // Update the chat with the response
        messages.update(m => [...m, { sender: "Han", text: replyText }]);
    } catch (error) {
        console.error("Error:", error);
        messages.update(m => [...m, { sender: "Han", text: "<p>Oops! Something went wrong. Try again later.</p>" }]);
    }

    message = "";
}




    function handleKeyDown(event) {
        if (event.key === "Enter" && !event.shiftKey) {
            event.preventDefault();
            sendMessage();
        }
    }

    // Auto-scroll behavior when messages update
    afterUpdate(() => {
        if (messagesEnd) messagesEnd.scrollIntoView({ behavior: "smooth" });
    });
</script>

<section class="chat-container">
    <!-- Profile Section -->
    <aside class="profile-section">
		<h3>Chat with Me!</h3>
        <img src="/profile.jpg" alt="Profile Picture" class="profile-img" />
        <p class="profile-description">
            Hi, My name is <b>Han.</b> 👋. <br /><br />
            I graduated from U of Florida with Bachelor's in Computer Science, and I am pursuing a Master's in Computer Sceince at Georgia Institute of Technology. I interned as a software engineer intern at Boeing Intelligence & Analytics in 2024.<br /><br />
			I build iOS application, unity-based games, Machine Learning Models, full-stack web app and hardware using Arduinos! <br /><br />
			I like traveling, watching NBA and also love going to a bunch of hackathons!<br /><br />
			Chat with me about my projects!
        </p>
    </aside>

    <!-- Chat Section -->
    <div class="chat-area">
		<div class="messages">
			{#each $messages as msg}
				<div class="message {msg.sender === 'You' ? 'user-message' : 'agent-message'}">
					<strong>{msg.sender}:</strong> {@html msg.text} <!-- Enables HTML rendering -->
				</div>
			{/each}
			<div bind:this={messagesEnd}></div>  <!-- Invisible div for auto-scrolling -->
		</div>




        <div class="input-area">
            <textarea
                bind:value={message}
                placeholder="Type your message..."
                on:keydown={handleKeyDown}
                class="chat-input"
            ></textarea>
            <button on:click={sendMessage} class="send-button">Send</button>
        </div>
    </div>
</section>

<style>
    .chat-container {
        display: flex;
        width: 100%;
        height: 80vh; /* Enlarge height */
        max-width: 1200px;
        margin: auto;
        border-radius: 8px;
        box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
        overflow: hidden;
    }

    /* Profile Section */
    .profile-section {
        width: 20%;
        padding: 20px;
        background: #f4f4f4;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        text-align: center;
    }

    .profile-img {
        width: 180px;
        height: 180px;
        border-radius: 50%;
        margin-bottom: 10px;
    }

    .profile-description {
        font-size: 14px;
        color: #333;
    }

    /* Chat Section */
    .chat-area {
        width: 80%;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        background: white;
        padding: 20px;
    }

    .messages {
        flex-grow: 1;
        overflow-y: auto;
        padding: 10px;
        height: 100%;
        max-height: 65vh; /* Prevents excessive scrolling */
    }

    .message {
        margin-bottom: 10px;
        padding: 10px;
        border-radius: 5px;
        font-size: 14px;
    }

    .user-message {
        background-color: #d1e7ff;
        align-self: flex-end;
        text-align: right;
    }

    .agent-message {
        background-color: #e8e8e8;
        align-self: flex-start;
        text-align: left;
    }

    .input-area {
        display: flex;
        align-items: center;
        border-top: 1px solid #ddd;
        padding: 10px;
        background: #fff;
    }

    .chat-input {
        flex-grow: 1;
        height: 40px;
        border: 1px solid #ddd;
        border-radius: 5px;
        padding: 10px;
        font-size: 14px;
        resize: none;
        outline: none;
    }

    .send-button {
        margin-left: 10px;
        padding: 10px 15px;
        background-color: #007bff;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        font-size: 14px;
    }

    .send-button:hover {
        background-color: #0056b3;
    }

    /* Responsive Adjustments */
    @media (max-width: 768px) {
        .chat-container {
            flex-direction: column;
        }
        .profile-section {
            width: 100%;
            height: 20%;
        }
        .chat-area {
            width: 100%;
            height: 80%;
        }
    }
</style>