<script lang="ts">
  import ChatMessage from '$lib/components/Message.svelte';
  import type { ChatCompletionRequestMessage } from 'openai';
  import { SSE } from 'sse.js';

  let query: string = '';
  let answer: string = '';
  let loading: boolean = false;
  let chatMessages: ChatCompletionRequestMessage[] = [];
  let scrollToDiv: HTMLDivElement;

  function scrollToBottom() {
    setTimeout(function () {
      scrollToDiv.scrollIntoView({ behavior: 'smooth', block: 'end', inline: 'nearest' });
    }, 100);
  }

  const handleSubmit = async () => {
    loading = true;
    chatMessages = [...chatMessages, { role: 'user', content: query }];

    const eventSource = new SSE('/api/chad', {
      headers: {
        'Content-Type': 'application/json'
      },
      payload: JSON.stringify({ messages: chatMessages })
    });

    query = '';

    eventSource.addEventListener('error', handleError);

    eventSource.addEventListener('message', (e) => {
      scrollToBottom();
      try {
        loading = false;
        if (e.data === '[DONE]') {
          chatMessages = [...chatMessages, { role: 'assistant', content: answer }];
          answer = '';
          return;
        }

        const completionResponse = JSON.parse(e.data);
        const [{ delta }] = completionResponse.choices;

        if (delta.content) {
          answer = (answer ?? '') + delta.content;
        }
      } catch (err) {
        handleError(err);
      }
    });
    eventSource.stream();
    scrollToBottom();
  };

  function handleError<T>(err: T) {
    loading = false;
    query = '';
    answer = '';
    console.error(err);
  }
</script>

	<div>
		<h1 class="text-2xl font-bold w-full text-center">chadGPT</h1>
	</div>
	<div class="chatwrapper  bg-white rounded-t-lg p-3 overflow-y-auto flex flex-col gap-4">
		<div class="flex flex-col gap-2">
			<ChatMessage type="assistant" message="Sup dude?" />
			{#each chatMessages as message}
				<ChatMessage type={message.role} message={message.content} />
			{/each}
			{#if answer}
				<ChatMessage type="assistant" message={answer} />
			{/if}
			{#if loading}
				<ChatMessage type="assistant" message="Loading.." />
			{/if}
		</div>
		<div class="" bind:this={scrollToDiv} />
    
	</div>
	<form
		class="flex w-full rounded-b-lg gap-4 bg-white p-4 "
		on:submit|preventDefault={() => handleSubmit()}
	>
		<input type="text" class="input input-bordered w-full bg-slate-800" bind:value={query} />
		<button type="submit" class="btn bg-blue-700 hover:bg-slate-800 text-white"> Send </button>
	</form>

<style>
 .chatwrapper{
  height:40rem;
  width: 100%;


 }
 
</style>
