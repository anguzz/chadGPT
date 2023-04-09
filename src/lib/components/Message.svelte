<script lang="ts">
	import type { ChatCompletionRequestMessageRoleEnum } from 'openai'
	export let type: ChatCompletionRequestMessageRoleEnum
	export let message: string
</script>

<div class="chat {type === 'user' ? 'chat-end' : 'chat-start'} justify-end">
	<div class="chat-image avatar">
		<div class="w-10 rounded-full">
			<img
				src="/chad.jpg"
				alt="{type} avatar"
			/>
		</div>
	</div>
	<div class="chat-header">
		{type === 'user' ? 'Me' : 'Chad'}
	</div>
	<div class="-bubble bg-blue-700 p-2 rounded-2xl	">
		{message}
	</div>
</div>
