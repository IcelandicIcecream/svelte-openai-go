<script lang="ts">
  import { SSE } from "sse.js"

  type OpenAIRequest = {
    role: string
    content: string
    messages?: OpenAIRequest[]
  }

  let input: string = ""
  let messages: OpenAIRequest[] = []

  function handleSubmit() {
    let userInput = {
      role: "user",
      content: input,
      messages,
    }

    // Assuming SSE setup is correct and can handle the payload as intended
    const eventSource = new SSE("http://localhost:8080/openai", {
      headers: {
        "Content-Type": "text/plain",
      },
      method: "POST",
      payload: JSON.stringify(userInput),
    })

    // Add the user's input to messages
    messages = [...messages, { role: "user", content: input }]

    // Reset input for the next message
    input = ""

    // Handle the message content event
    eventSource.addEventListener("message", (e) => {
      try {
        const messageData = e.data

        // Check if we should start a new message or update the existing one
        if (
          !messages.length ||
          messages[messages.length - 1].role !== "assistant"
        ) {
          // Start a new message
          messages.push({
            role: "assistant", // The role is known from the context
            content: messageData,
          })
        } else {
          // Update the last message's content
          messages[messages.length - 1].content += messageData
        }
      } catch (error) {
        console.log("ERROR handling message data:", error)
      }
    })

    // Handle errors
    addEventListener("error", (e) => {
      console.log("Request failed, HTTP status was: " + e.source.xhr.status)
    })

    // Begin listening to the stream
    eventSource.stream()
  }
</script>

<svelte:head>
  <title>OpenAI Chat</title>
  <meta name="description" content="OpenAI Chat" />
</svelte:head>

<section>
  <ul>
    {#each messages as message}
      <li>{message.role}: {message.content}</li>
    {/each}
  </ul>
  <form on:submit|preventDefault={handleSubmit}>
    <input
      name="message"
      class="input input-bordered w-full max-w-xs"
      bind:value={input}
    />
    <button type="submit">Send</button>
  </form>
</section>
